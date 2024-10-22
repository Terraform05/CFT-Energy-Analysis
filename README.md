
#CFT Energy Analysis

This repository contains the analysis and results of a cyclicality study of energy prices and net load in relation to fuel types, using historical data and various visualization techniques. The primary focus is on understanding the cyclic nature of energy prices and how different fuel types affect these prices on an hourly, daily, and monthly basis.

## Table of Contents
1. [Project Overview](#project-overview)
2. [Key Features](#key-features)
3. [Data Sources](#data-sources)
4. [Setup Instructions](#setup-instructions)
5. [Analysis Breakdown](#analysis-breakdown)
6. [Results](#results)

## Project Overview
The `CFT NME Energy Analysis` project aims to analyze the cyclicality of energy prices and their relationship with net load (load minus renewable generation), broken down into hourly, weekly, and monthly patterns. It uses data preparation, cyclicality analysis, and polynomial regression to identify patterns between energy prices and net load.

### Key Highlights:
- **Hourly, Daily, and Monthly Cyclicality:** Energy prices fluctuate based on demand patterns, with gas showing the highest cyclicality.
- **Net Load vs Energy Prices:** A regression analysis was performed to identify the relationship between net load and energy prices.
  
## Key Features
- **Cyclicality Analysis:** 
  - Analyzes energy price cyclicality across different timeframes (hourly, weekly, and monthly).
  - Highlights fuel type behaviors (gas, coal, nuclear) in terms of cyclicality.
  
- **Net Load & Energy Price Relationship:**
  - Uses polynomial regression to model the relationship between net load and energy prices.
  - Displays various polynomial degrees for detailed insights.

- **Visualization:** Clear plots of net load cyclicality and energy price trends are generated for better data interpretation.

## Data Sources
- **Energy Prices**: Hourly system energy prices.
- **Fuel Types**: Data regarding energy generation from different fuel types such as gas, nuclear, and coal.
- **Net Load**: Calculated as the difference between total load and renewable generation (wind, solar, hydro).

## Setup Instructions
1. Clone this repository to your local machine.
   ```bash
   git clone https://github.com/your-username/cft-nme-energy-analysis.git
   ```
2. Install the required dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Run the Jupyter notebook to reproduce the results:
   ```bash
   jupyter notebook energy_analysis.ipynb
   ```

## Analysis Breakdown
1. **Hourly Cyclicality Analysis**: 
   - The notebook performs a breakdown of energy prices on an hourly basis to detect daily trends, showing that prices tend to be lowest early in the morning and highest in the late afternoon.

2. **Weekly Cyclicality**:
   - Analysis of weekly energy prices shows that prices peak at the beginning of the week and drop as the weekend approaches.

3. **Monthly Cyclicality**:
   - Monthly analysis reveals higher prices in extreme temperature months like January (winter) and July (summer).

4. **Net Load vs Energy Price Analysis**:
   - A regression analysis between net load and energy price is carried out. Several polynomial equations are used to best fit the data, demonstrating the complexity of the relationship between net load and energy price.

## Results
- **Fuel Type Cyclicality**: 
  - Gas is the most cyclic fuel, showing significant fluctuations throughout the day.
  - Nuclear remains stable with the least cyclic behavior.

- **Energy Price Cyclicality**: 
  - Prices are influenced by the time of day and the season, with peak demand driving up costs.
  
- **Net Load vs Energy Price Regression**: 
  - Different polynomial degrees (up to degree 5) were tested to find the best fit between net load and energy price data.
