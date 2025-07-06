
# Report – Summer Analytics 2025 Capstone Project

**Project Title:** Dynamic Pricing for Urban Parking Lots  
**Submitted for:** Consulting and Analytics Club, IIT Guwahati  
**Tools Used:** Python, Pandas, NumPy, Pathway, Bokeh  
**Submitted by:**  Billa Saikrishna


-----


# Problem Statement & Importance

Urban areas often face severe parking shortages and congestion, especially during peak hours or special events. Static pricing models (fixed rates regardless of demand) result in inefficient space utilization — with some lots overcrowded while others remain underused. 

To address this, the project aims to implement **dynamic pricing** for parking lots. This means adjusting the parking price based on real-time demand, traffic conditions, special days, time of day, and competition from nearby lots. The ultimate goal is to:

- **Optimize revenue** for lot operators
- **Balance load** across multiple lots
- **Guide drivers** to available, cost-effective parking spots
- **Reduce traffic congestion** caused by searching for parking

The problem involves designing data-driven models that can predict and set fair prices in a dynamic urban environment while ensuring the experience remains transparent and competitive for users. Each model progressively integrates more complexity — from occupancy tracking to full competitive awareness.


-----


# Preprocessing

Before implementing any pricing model, it was essential to clean and preprocess the raw dataset to ensure consistency and usability.

1. The original data contained separate `LastUpdatedDate` and `LastUpdatedTime` fields. These were combined to create a unified `Timestamp` column, which was then converted into Python datetime format for accurate time-based computations.

2. Categorical variables were encoded numerically for model compatibility:
   - `VehicleType` was encoded as: bike → 1, car → 2, truck → 3
   - `TrafficConditionNearby` was encoded as: low → 1, medium → 2, high → 3

3. Missing or invalid values were handled gracefully to ensure smooth pipeline execution. Rows with empty or non-parsable values were either imputed or excluded as appropriate.

4. The dataset was sorted chronologically based on `Timestamp` to maintain temporal consistency for time series analysis and exponential moving average calculations.

5. The final cleaned dataset was saved as `preprocessed_dataset.csv` and used as the input source across all three models.

This preprocessing step was crucial for maintaining data integrity, enabling reliable feature engineering, and ensuring accurate real-time pricing predictions.


---


# Model 1 – Baseline Linear Model

Model 1 serves as a simple foundational approach to dynamic pricing. It uses only the **current occupancy level** of a parking lot to decide how the price should increase or decrease over time. This baseline model helps establish a reference for evaluating more complex demand-aware strategies in later models.

# Logic

At each time step, the price for a parking lot is updated using a basic linear formula:

Pₜ₊₁ = Pₜ + α · (Occupancy / Capacity)

Where:
- `Pₜ` = Current price at time step *t*
- `Pₜ₊₁` = Updated price at the next time step
- `Occupancy / Capacity` = Current occupancy ratio
- `α` = A fixed sensitivity constant (we used α = 2.0)

This model increases the price proportionally with how full the parking lot is. If the occupancy is low, the price remains closer to the base. If the occupancy is high, the price increases more aggressively.

To maintain realistic pricing, the output price is **clipped between $5 and $20**.

# Why α = 2 ?

Through experimentation, I found that setting α = 2.0 allowed for noticeable but not overly aggressive price changes. It balances **responsiveness** with **stability**, ensuring that the price adjusts meaningfully as demand fluctuates without creating sharp spikes or dips.

# Implementation Details

- A base price of $10 is assigned initially to all parking lots.
- A Python dictionary is used to track each lot’s previous price.
- For every new row (time step), the model computes a new price using the occupancy ratio.
- All logic was implemented in a Pathway UDF and run in static mode for evaluation.

# Visualization

An interactive Bokeh line chart was created to visualize how prices evolve over time for selected parking lots. Each lot's price is plotted on the same graph, with a dynamic legend allowing users to isolate individual trends.

The visual clearly shows how prices rise and fall in sync with occupancy, validating the expected behavior of the linear pricing rule.

 Notebook: `notebooks/model1_bokeh_demand.ipynb`


 -----


 # Model 2: Demand-Based Price Function

Model 2 builds on the baseline by incorporating real-world features that drive parking demand. The goal is to compute a **demand score** using multiple inputs and use that to adjust the parking price dynamically.

# Demand Score Formula

```
DemandScore = α · occ_rate + β · queue_norm + γ · traffic_norm + δ · IsSpecialDay + ε · vehicle_norm
              + ζ · EMA(occ_rate) + η · slot_norm + κ · (occ_rate)^2 + θ · (queue_norm · traffic_norm)
```

**Where:**
- `occ_rate` = Occupancy / Capacity
- `queue_norm` = QueueLength / 10
- `traffic_norm` = TrafficConditionEncoded / 3
- `vehicle_norm` = VehicleTypeEncoded / 3
- `slot_norm` = Normalized time of day = (Hour + Minute/60) / 17
- `EMA(occ_rate)` = Exponential moving average of occupancy rate

All features are **normalized** between 0 and 1 for consistency.

---

# Price Calculation

```
NormalizedDemand = 2 / (1 + exp(-DemandScore)) − 1
Pricet = BasePrice · (1 + λ · NormalizedDemand)
FinalPrice = ρ · PreviousPrice + (1 − ρ) · Pricet
```

**Constants used:**
- `λ` = 0.3 (demand sensitivity)
- `ρ` = 0.7 (smoothing factor)
- `BasePrice` = $10  
- Prices are clipped between **$5 and $20**

---

# Tuning the Weights

I tuned the weights using a combination of data-driven analysis and domain reasoning. The process involved:

# Step 1: Export Output

- After running the model with initial weights, I saved the output in:
  - `model2_output_for_tuning.csv`
  - `model2_enriched_for_tuning.csv` (with derived features like EMA and slot time)

# Step 2: Correlation Heatmap

- I plotted a **correlation heatmap** to understand how features like `occ_rate`, `queue_norm`, and `EMA(occ_rate)` relate to the predicted price.
- Features like `occ_rate`, `EMA`, and `slot_norm` showed the strongest positive correlations.

# Step 3: Visual Analysis

I plotted:
- **Price vs occ_rate**: clear upward trend.
- **Price vs EMA(occ_rate)**: consistent trend over time.
- **Price vs queue_norm**: impact was moderate, but interaction with traffic was useful.

These plots helped us estimate relative importance.

# Step 4: Iterative Adjustment

- I gradually adjusted weights and observed their effect on **price smoothness**, **variation across lots**, and **responsiveness to change**.
- Interaction terms like `(queue_norm · traffic_norm)` were introduced to capture compound effects.

---

# Final Weights (After Tuning)

```
α = 2.0  → Occupancy
β = 0.5  → Queue Length
γ = 0.5  → Traffic
δ = 0.3  → Special Day
ε = 0.3  → Vehicle Type
ζ = 1.0  → EMA of occupancy
η = 0.5  → Time of Day
κ = 1.0  → Quadratic occupancy
θ = 0.7  → Interaction of queue and traffic
```

These values were selected to balance **sensitivity** with **stability**.

---

# Output and Visualization

- Prices evolve smoothly across time.
- Output stored in: `data/output_model2.jsonlines`
- Bokeh visualizations show trends across different lots.
- All tuning steps and visual validations are detailed in `docs/model2_notes.md`

 Notebook: `notebooks/model2_enhanced_demand.ipynb`  
 Notes: `docs/model2_notes.md`
 Output: `data/output_model2.jsonlines`


-----


# Model 3: Competitive Pricing Model

This model introduces **spatial competition**, which is a key driver in real-world pricing behavior. Unlike previous models that used only internal signals (like occupancy or traffic), this model takes into account **prices from nearby competing lots**.

The idea is simple: in a real market, a parking lot must remain attractive compared to its neighbors. If two lots serve the same region, pricing should adjust based on their relative demand and pricing.

---

# Business Intuition

- If demand at a lot is **high** and **competitor prices are also high**, our lot can **raise** prices confidently.
- If demand is **low** and **competitor prices are lower**, we must **reduce** our price to stay competitive.
- If our lot is full and competitors are empty, we **increase** price due to scarcity.
- If our lot is empty and nearby lots are full, we may **keep the price stable** or slightly reduce it to capture demand overflow.

This is classic **competitive pricing strategy** in business analytics.

---

# Technical Implementation

I build upon the demand score from **Model 2**, and adjust prices further based on competitor information.

**Step 1: Define Nearby Competitors**

- I used `geopy.distance` to calculate the **Haversine distance** between parking lots using latitude and longitude.
- Lots within **0.5 km** are considered **competitors**.
- The competitor list is refreshed periodically based on current location data.

**Step 2: Gather Competitor Prices**

- For each timestamp, I gathered prices of nearby lots from **Model 2’s price output**.
- I calculated the **average price** among all competitors at the same timestamp.

**Step 3: Apply Competitive Pricing Logic**

```
If demand_score is high and competitor price is low:
    price += (lambda1 * demand_score) - (lambda2 * price_diff)

Else if demand_score is low and competitor price is high:
    price -= (lambda3 * price_diff)
    
Else:
    price = demand_based_price (from Model 2)
```

**Where:**
- `price_diff = our price - competitor average price`
- `λ` values control the adjustment strength

Final price is clipped between **$5 and $20** and smoothed similarly to previous models.

---

# Pathway + Streaming

- Used **Pathway** to ingest the enriched dataset and apply real-time transformation.
- Competitor prices are computed on the fly using `pw.iter_windows` and spatial filters.
- Ensured that each lot **reacts to only nearby competitors** without introducing unnecessary global coupling.

---

# Visual and Analytical Output

- Created visualizations to show:
  - **Price over time per lot**
  - **Competitor price vs our price**
  - **Dynamic changes in reaction to competitors**

These helped validate that the model behaves logically under competitive pressure.

---

# Files

- Notebook: `notebooks/model3_proximity_competition.ipynb`
- Output file: `data/output_model3.jsonlines`


-----


# Visualizations

Created interactive Bokeh plots for:

Model 1: Price vs Time per lot

Model 2: Feature scatter plots + price evolution

Model 3: Price comparison across nearby competitors.


-----


# Evaluation

Since this is a project, the objective is interpretability and explainability:

Code is modular and well-commented

Pricing decisions are traceable

Results show clear progression across models

Visualizations reinforce the logic used.


-----


# Repository Structure

----


 Capstone-Project-of-Summer-Analytics-2025
├── notebooks/
│   ├── Model_1_Baseline_linear.ipynb
│   ├── model_2_Demand_Based_Pricing.ipynb
│   └── model_3_Competitive_Pricing.ipynb
├── data/
│   ├── dataset.csv
│   ├── preprocessed_dataset.csv
│   ├── output_model1.jsonlines
│   ├── output_model2.jsonlines
│   ├── output_model3.jsonlines
│   ├── model2_output_for_tuning.csv
│   └── model2_enriched_for_tuning.csv
├── docs/
│   ├── report.md
│   └── model2_notes.md
└── problem_statement.pdf


----

# Author Note

This entire project — from code to logic to documentation — was implemented by Billa SaiKrishna as part of the Summer Analytics 2025 capstone.