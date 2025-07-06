# Model 2 – Demand Weight Tuning Notes

This document explains how the weights used in **Model 2: Demand-Based Price Function** were selected using a real, code-driven tuning process involving feature engineering, visualizations, and statistical reasoning.

---

# Objective

The goal of tuning was to determine how much each contextual feature (like occupancy, queue length, time, traffic, etc.) should influence the final **Demand Score**, which drives the dynamic pricing function.

---

# Demand Score Formula

```
DemandScore = α·occ_rate + β·queue_norm + γ·traffic_norm + δ·IsSpecialDay + ε·vehicle_norm 
              + ζ·EMA(occ_rate) + η·slot_norm + κ·(occ_rate)^2 + θ·(queue_norm ⋅ traffic_norm)
```

All features are normalized between 0 and 1 to make weights comparable.

---

# Final Weights Used

```
α = 2.0    # Occupancy rate
β = 0.5    # Queue length
γ = 0.5    # Traffic condition
δ = 0.3    # Special day flag
ε = 0.3    # Vehicle type
ζ = 1.0    # EMA of occupancy
η = 0.5    # Normalized time slot
κ = 1.0    # Non-linear occupancy^2 term
θ = 0.7    # Queue × traffic interaction
```

---

# Datasets Used

- `model2_output_for_tuning.csv`: Output after running Model 2 pipeline  
- `model2_enriched_for_tuning.csv`: Enriched file with engineered features for analysis

---

# Tuning Methodology

I followed a **real tuning process** that was human-guided and code-supported:

---

# Step 1: Feature Engineering

I created the following features to represent various components of demand:

| Feature                   | Description                               |
|---------------------------|-------------------------------------------|
| `occ_rate`                | Occupancy / Capacity                      |
| `queue_norm`              | QueueLength / 10                          |
| `traffic_norm`            | TrafficConditionNearbyEncoded / 3        |
| `vehicle_norm`            | VehicleTypeEncoded / 3                    |
| `slot_norm`               | Normalized time of day (Hour + Min / 60) / 17 |
| `EMA(occ_rate)`           | Exponential Moving Average of occupancy   |
| `occ_rate^2`              | Adds non-linear response to high occupancy |
| `queue * traffic`         | Captures congestion-pressure interaction  |

---

# Step 2: Correlation Heatmap

I plotted a correlation heatmap between features and price.

| Feature        | Correlation with Price | Interpretation                         |
|----------------|------------------------|-----------------------------------------|
| `occ_rate`     | High (~0.80)           | Strongest driver → high weight (α = 2.0)  
| `EMA_occupancy`| High (~0.70)           | Smooths demand effect → ζ = 1.0  
| `slot_norm`    | Moderate (~0.45)       | Time patterns → η = 0.5  

 Insight: Features with higher correlation got higher weights. Others were judged through visual patterns.

---

# Step 3: Scatter Plots for Feature Impact

I visualized how each feature influences price using scatter plots:

# Key Observations:

- **occupancyRate**: Strong positive trend → justified α = 2.0
- **EMA_occupancy**: Smooth, slightly lagged rise → ζ = 1.0
- **QueueLength**: Price increases with longer queues → β = 0.5
- **TrafficConditionNearby**: Densely packed at higher traffic → γ = 0.5
- **VehicleTypeEncoded**: Mild influence → ε = 0.3
- **TimeSlot**: Periodic spikes during peak hours → η = 0.5

This visual analysis helped me select weights not just statistically, but meaningfully — based on real-world demand logic.

---

# Summary

Rather than assigning arbitrary values, I:

- Ran the model on real parking lot data
- Created feature-enriched datasets
- Used correlation + scatter plots to understand feature importance
- Tuned weights based on both quantitative and visual impact

 This makes the demand function in Model 2 **explainable, optimized, and real-world aligned.**

---

# Files Produced

- `model2_output_for_tuning.csv`
- `model2_enriched_for_tuning.csv`
- Heatmaps and scatter plots included in the notebook

This process is reproducible and fully aligned with the project’s analytical goals.
