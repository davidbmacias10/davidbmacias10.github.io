---
layout: page
title: Home Valuation Model - Cost and Market Approaches
description: Developing a Machine Learning Assisted Appraisal Model
img: assets/img/10.jpg
importance: 1
category: work
related_publications: false
---

## Project Summary

This project focuses on developing a valuation model for single-family residential properties, applying both the cost approach and market-based methods. Drawing from principles in the book *Real Estate Valuation Theory* by Manya M. Mooya, I am constructing a hybrid appraisal framework using structured data sourced via the Attom Data API. The goal is to create an automated valuation model (AVM) that blends theoretical rigor with real-world property data to estimate property values more accurately and transparently.

The project is currently a work in progress. I anticipate producing a fully operational and tested model by mid-summer 2025, complete with validation results and use-case demonstrations.

---

## Cost Approach Formula

In real estate appraisal, the **Cost Approach** estimates property value based on the cost to reproduce or replace the structure, minus depreciation, plus the land value. The general formula is:

> **Property Value = (Replacement/Reproduction Cost - Depreciation) + Land Value**

Where:
- **Replacement/Reproduction Cost**: The cost to build a similar property today.
- **Depreciation**: Loss in value from physical deterioration, functional obsolescence, or external obsolescence.
- **Land Value**: Estimated separately based on comparable land sales.

## Python Code Implementation

```python
def estimate_initial_cost_per_sqft(base_cost, location_multiplier=1.0, quality_multiplier=1.0, size_multiplier=1.0):
    """
    Estimate initial construction cost per square foot.

    Args:
        base_cost (float): Base national or regional average cost per square foot.
        location_multiplier (float): Regional adjustment factor (default 1.0 = no adjustment).
        quality_multiplier (float): Quality adjustment factor (default 1.0 = average quality).
        size_multiplier (float): Size efficiency adjustment factor (default 1.0 = standard size).

    Returns:
        float: Adjusted estimated cost per square foot.
    """
    return base_cost * location_multiplier * quality_multiplier * size_multiplier

def calculate_replacement_cost(cost_per_sqft, living_area_sqft):
    """
    Calculate replacement cost.

    Args:
        cost_per_sqft (float): Cost to rebuild per square foot.
        living_area_sqft (float): Living area size in square feet.

    Returns:
        float: Replacement cost estimate.
    """
    return cost_per_sqft * living_area_sqft

def calculate_depreciation(replacement_cost, age, useful_life=50):
    """
    Calculate straight-line depreciation amount.

    Args:
        replacement_cost (float): Replacement cost.
        age (int): Age of structure in years.
        useful_life (int): Expected useful life (default 50 years).

    Returns:
        float: Depreciation amount.
    """
    depreciation_percentage = min(age / useful_life, 1.0)  # Cap at 100%
    return replacement_cost * depreciation_percentage

def estimate_property_value(
    base_cost,
    living_area_sqft,
    land_value,
    age,
    location_multiplier=1.0,
    quality_multiplier=1.0,
    size_multiplier=1.0,
    useful_life=50
):
    """
    Full Home Value Estimation based on cost approach.

    Args:
        base_cost (float): Base average cost per square foot.
        living_area_sqft (float): Living area of the home.
        land_value (float): Land value.
        age (int): Age of the structure in years.
        location_multiplier (float): Adjustment for regional construction costs.
        quality_multiplier (float): Adjustment for construction quality.
        size_multiplier (float): Adjustment for size efficiency.
        useful_life (int): Expected useful life (default 50 years).

    Returns:
        dict: Full breakdown including cost_per_sqft, replacement_cost, depreciation, final_property_value.
    """
    cost_per_sqft = estimate_initial_cost_per_sqft(
        base_cost,
        location_multiplier,
        quality_multiplier,
        size_multiplier
    )
    replacement_cost = calculate_replacement_cost(cost_per_sqft, living_area_sqft)
    depreciation_amount = calculate_depreciation(replacement_cost, age, useful_life)
    final_property_value = (replacement_cost - depreciation_amount) + land_value

    return {
        "Cost per Square Foot": cost_per_sqft,
        "Replacement Cost": replacement_cost,
        "Depreciation Amount": depreciation_amount,
        "Final Estimated Property Value": final_property_value
    }

# --------------------------
# Example Usage
# --------------------------

# Assume:
base_cost = 120               # Base cost per sqft (national/regional average)
living_area_sqft = 2000        # Living area size
land_value = 50000             # Land value in dollars
age_of_home = 20               # Age in years
location_multiplier = 1.10     # Slightly higher regional construction costs (+10%)
quality_multiplier = 1.15      # Higher-quality construction (+15%)
size_multiplier = 0.95         # Larger home (slight discount per sqft)

# Calculate
results = estimate_property_value(
    base_cost,
    living_area_sqft,
    land_value,
    age_of_home,
    location_multiplier,
    quality_multiplier,
    size_multiplier
)

# Print Results
for key, value in results.items():
    print(f"{key}: ${value:,.2f}")
```