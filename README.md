
# Capstone-Project-of-Summer-Analytics-2025

**Project Title:** Dynamic Pricing for Urban Parking Lots  
**Submitted for:** Consulting and Analytics Club, IIT Guwahati    
**Submitted by:**  Billa Saikrishna

-----

# Project Summary

This repository presents the implementation of a real-time dynamic pricing system for urban parking lots, developed as the final submission for the Summer Analytics 2025 program hosted by the Consulting & Analytics Club, IIT Guwahati. The system aims to adjust parking prices dynamically based on real-world demand signals, spatial competition, and user behavior to improve lot utilization, reduce traffic congestion, and increase transparency in parking management.

The project includes three progressively advanced pricing models implemented in Python, supported by Pathway for real-time stream processing and Bokeh for interactive visualizations.

-----

# Table of Contents

Problem Statement

Technologies Used

Model Overview

Model 1: Baseline Linear Model

Model 2: Demand-Based Price Function

Model 3: Competitive Pricing Model

Project Files

How to Run

Results & Outputs

Author

-----

# Problem Statement

Urban parking systems typically suffer from poor demand allocation due to static pricing mechanisms. Drivers often spend excessive time searching for affordable or available parking, increasing congestion and emissions. Parking lot operators also fail to optimize revenue because pricing doesn't adapt to demand fluctuations.

This project addresses the challenge by developing a dynamic pricing system using streaming data. The models incorporate occupancy rates, queue lengths, traffic conditions, special day indicators, vehicle types, and spatial competition to compute optimal parking prices in real-time.

-----

# Technologies Used

Python 3.8+

Pandas / NumPy for data manipulation

Pathway for real-time streaming pipeline

Bokeh for interactive visualizations

Seaborn / Matplotlib for analytical plots

Geopy for distance-based competitor pricing

Google Colab for experimentation and notebook execution

-----


# Model Overview

**Model** 1: Baseline Linear Model

A simple time-dependent model that increases price with rising occupancy.

**Formula:**  P_(t+1) = P_(t) + α * (Occupancy / Capacity)

Constant α: 2.0 (determined through testing)

Price Constraints: Clipped between $5 and $20

Purpose: Establish a baseline response to internal demand

---

# Model 2: Demand-Based Price Function

An enhanced model incorporating multiple real-time signals to calculate a demand score.

Features Used:

Occupancy rate

Queue length

Traffic conditions

Special day indicator

Vehicle type

Time of day

EMA of occupancy

Nonlinear interaction terms

Demand Score: Weighted sum of normalized features

Price Formula:FinalPrice = ρ * PreviousPrice + (1 - ρ) * BasePrice * (1 + λ * NormalizedDemand)

Weights: Tuned through correlation heatmaps and scatter plot validation (see docs/model2_notes.md)

---

# Model 3: Competitive Pricing Model

A market-aware pricing model that adjusts prices based on spatial competition.

Geospatial Logic: Uses Haversine distance to find competitors within 0.5 km

Adjustments:

If competitors are cheaper → price decreases

If competitors are expensive and demand is high → price increases

If demand is equal → stabilize based on average

Combination: Merges demand-based pricing from Model 2 with competitor price influence

-----

# Project files

--

Capstone-Project-of-Summer-Analytics-2025/
├── notebooks/
│   ├── model1_bokeh_demand.ipynb
│   ├── model2_enhanced_demand.ipynb
│   └── model3_proximity_competition.ipynb
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

--

-----


# How to Run

Clone the repository

git clone https://github.com/<saikrishna-billa>/Capstone-Project-of-Summer-Analytics-2025.git
cd Capstone-Project-of-Summer-Analytics-2025

# Install dependencies

!pip install pandas numpy pathway bokeh geopy seaborn matplotlib


# Open and execute the notebooks

Navigate to notebooks/

Run each notebook (Model_1_Baseline_linear.ipynb,  Model_2_Demand_Based_pricing.ipynb,  Model_3_Competitive.ipynb) in sequence

Visualize results

Each model generates visualizations using Bokeh

Output pricing is stored in data/*.jsonlines


-----

# Results & Outputs

All models perform well under their respective logic, with Model 3 showing strong pricing competitiveness in high-density areas.

Output format for real-time pipelines is standardized using Pathway’s streaming interface.

All models are validated visually and analytically using Bokeh, heatmaps, and scatter plots.

-----


# Visual Summary:

Model 1: Price vs. Time (Bokeh)

Model 2: Price vs. Demand Factors

Model 3: Price vs. Competitor Price and Distance


-----

# Author

This project is solely developed and maintained by Billa Saikrishna as part of the final project submission for Summer Analytics 2025, hosted by the Consulting and Analytics Club, IIT Guwahati.

# For queries, collaboration, or feedback, feel free to connect via GitHub.