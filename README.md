# PJM System Net Load & Price Analysis

1. [Project Overview](#project-overview)  
2. [Key Findings Summary](#key-findings-summary)  
3. [Data Sources](#data-sources)  
4. [Repository Contents](#repository-contents)  
5. [Environment Setup](#environment-setup)  
6. [Analysis Breakdown](#analysis-breakdown)  
7. [Findings](#findings)

---

## Project Overview
This project rebuilds my previous PJM-RTO net-load analysis (RTO load minus total renewable generation), aligns it with PJM’s day-ahead system price, and examines how renewable availability and load cyclicality shape price formation. Earlier analyses mixed regional loads with system-wide renewables and out-of-phase timestamps; the current workflow keeps every series in Eastern Prevailing Time and within the same RTO footprint. The notebook (`energy.ipynb`) performs all cleaning, aggregation, visualization, regression, and commentary, exporting each plot as a PNG plus an accompanying CSV so graphics can be recreated externally.

Net load is the hinge connecting physical operations with economic outcomes. Traders look at it to price spark spreads and congestion risk, utilities rely on it when planning procurement or scheduling outages, and policymakers use it to judge whether new renewable capacity is truly shaving peaks. With that lens the research questions become:
- How do hourly, daily, and seasonal load and renewable profiles interact to produce PJM’s characteristic duck curve, and what does that imply for storage build-out and capacity adequacy?
- At what net-load levels does PJM price move sharply, and how much of that behaviour can a net-load-only model explain before fuel shocks, transmission constraints, or policy actions dominate?
- When do renewables suppress price beyond what net load captures, and where do fuel shortages or extreme weather break that historical relationship?
- How do these dynamics affect households, commercial consumers, and asset owners who ultimately experience the results through utility bills, hedging costs, or infrastructure investment decisions?

## Key Findings Summary
- Net load displays strong intraday and seasonal structure: renewables carve midday troughs yet evening ramps remain the binding stress point, especially in winter and late summer. This informs when peaking assets earn scarcity rents and when storage is most valuable.
- PJM price is convex in net load. Below roughly 110 GW, prices hover near $25–30/MWh; once net load exceeds 125 GW the marginal $/MWh jump accelerates rapidly, which is where uplift payments, scarcity pricing, and demand-response events concentrate.
- Renewable share is inversely related to price. Higher renewable penetration directly lowers clearing prices because fewer gas units set the marginal bid. Consumers benefit through cheaper energy while generators pivot toward capacity payments, renewable credits, or hedging strategies to stabilise revenue.
- The January 2024 Arctic outbreak temporarily broke the historical net-load/price correlation, demonstrating how fuel scarcity and emergency dispatch can override demand fundamentals. EIA recorded a record 141.5 Bcf of U.S. gas demand on 16 January, and PJM experienced the same physical stress through high prices and emergency alerts.
- Polynomial regression (degree 10) captures the flat-to-steep curvature in the price response while leaving interpretable residuals that flag scarcity events or renewable surpluses. Those residuals highlight when price embeds congestion, fuel risk, or policy intervention beyond what load alone predicts.

## Data Sources
All source files reside under `data/` and are provided externally.

| File | Description | Key Fields Used |
| --- | --- | --- |
| `historical_power_load.csv` | PJM forecast load by area, timestamped in UTC/EPT | `forecast_area`, `forecast_hour_beginning_ept`, `forecast_load_mw` |
| `generation_by_source.csv` | Hourly generation by fuel type, with renewable flags | `datetime_beginning_ept`, `fuel_type`, `mw`, `is_renewable` |
| `day_ahead_energy_price.csv` | Day-ahead PJM prices by pnode | `pnode_name`, `system_energy_price_da`, `datetime_beginning_ept` |
| `day_ahead_temperature.csv` | PJM day-ahead temperature guidance by zone/daypart | `datetime_beginning_ept`, `zone`, `da_temperature_set` |
| `daily_henryhub_naturalgasspotprice.csv` | EIA Henry Hub daily spot prices | `Date`, `Henry Hub Natural Gas Spot Price (Dollars per Million Btu)` |

## Repository Contents

| Path | Description |
| --- | --- |
| `energy.ipynb` | Primary notebook covering ingestion, processing, plotting, modelling, and narrative commentary. |
| `temp_analysis.ipynb` | Companion notebook that analyzes PJM day-ahead temperature guidance and ties it back to the net-load/price findings. |
| `naturalGas.ipynb` | Henry Hub spot analysis with seasonality, volatility, and PJM linkage diagnostics. |
| `combined_analysis.ipynb` | Exploratory notebook combining load, temperature, and gas features to model PJM price. |
| `temp_forecast.ipynb` | XGBoost-based daily PJM temperature forecast feeding the combined model. |
| `gas_forecast.ipynb` | XGBoost-based Henry Hub forecast plus volatility projections for price inputs. |
| `data/` | Raw CSV inputs described above (not tracked here). |
| `plots/` | Auto-generated PNGs for every figure (ignored by git). |
| `csv_plots/` | CSV snapshots of each plot’s underlying aggregated data (ignored). |
| `output_data/` | Derived tables such as regression diagnostics (ignored). |
| `requirements.txt` | Minimal Python dependency list. |
| `.venv/`, `.ipynb_checkpoints/` | Local environment artefacts (ignored). |

## Environment Setup
1. **Python** – Install Python 3.12 (or later with Matplotlib wheels). macOS users can use `brew install python@3.12`.
2. **Virtual environment** – From the repo root:
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```
3. **Dependencies** – Install packages:
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```
4. **Notebook execution** – Run interactively in Jupyter or headless:
   ```bash
   jupyter nbconvert --to notebook --execute energy.ipynb --output energy.ipynb
   ```
   Executing the notebook refreshes every PNG in `plots/` and exports each plot’s dataset into `csv_plots/`.

## Analysis Breakdown

1. **Dataset hygiene** – Converts all timestamps to EPT, filters load to `forecast_area == "RTO"`, aggregates renewables system-wide, and extracts PJM-RTO price rows only.
2. **Feature engineering** – Adds calendar features (hour, day, month), renewable shares, rolling averages, and rolling standard deviations.
3. **Plotting utilities** – Unified helpers (`save_fig`, `save_plot_data`, `hourly_average_plot`, etc.) ensure every visual/outcome includes a CSV log.
4. **Cyclicality diagnostics** – Hourly/day/month charts for price, renewable mix overlays, and renewable fuel-type cyclicality.
5. **Heatmaps & density** – 24×7 net-load and renewable heatmaps; hexbin density for net load vs price.
6. **Renewable share impact** – Scatter and binned curves linking renewable penetration to price.
7. **Dynamics** – Rolling correlation, trend, and volatility charts to identify when net load and price decouple.
8. **Relationship plots** – Net-load vs price coloured by month/hour/weekday, distribution histograms, and binned averages.
9. **Regression** – Polynomial fits (degrees 1–10) with train/test performance, best-fit overlay, and residual computation.
10. **Residual forensics** – Residuals vs net load, residuals vs renewable MW, extreme-event callouts.
11. **Autocorrelation** – Price and net-load autocorrelation up to 168 hours to bound forecast horizons.

## Findings
Visuals reside in `plots/`. Commentary summarises the behaviour shown by each figure.

### Hourly, Weekly, and Monthly Price Cyclicality
![Hourly price cyclicality](plots/price_hourly_avg.png)
![Weekly price cyclicality](plots/price_day_of_week.png)
![Monthly price cyclicality](plots/price_monthly_avg.png)
- Hourly prices rise from pre-dawn lows (roughly $25/MWh) into the afternoon as HVAC, commercial, and industrial demand peak; lighting alone cannot explain the swing. This curve dictates when batteries discharge and when demand-response programmes intervene.
- Weekdays command a premium relative to weekends because factories and offices sustain higher base load. Locational marginal price spreads widen during the workweek, which feeds into hedging costs for commercial customers.
- January and July stand out as the most expensive months, aligning with heating and cooling extremes; shoulder months stay subdued, aided by higher renewable share. Retail suppliers structure fixed-price offers and procurement schedules around these seasonal peaks.

### Load vs Renewable Overlays
![Hourly load vs renewables](plots/hourly_load_vs_renewables.png)
![Monthly renewables and net load](plots/monthly_renewables_netload_price.png)
- The hourly overlay captures PJM’s duck curve: renewables shave midday net load yet evening ramps remain steep. Storage developers use these curves to size four-hour versus eight-hour batteries.
- Weekdays sit consistently above weekends because commercial demand rarely drops to residential levels. This gap explains why weekday congestion and uplift costs run higher.
- Monthly averages show renewable-rich shoulder seasons trimming net load and nudging prices lower, whereas winter heating season keeps both net load and price elevated when renewables underperform. Utilities lean on these cycles when planning maintenance or hedging fuel.

### Renewable Fuel Cyclicality
![Hourly renewable fuel](plots/renewables_fuel_hourly_line.png)
![Weekday renewable fuel](plots/renewables_fuel_weekday_line.png)
![Monthly renewable fuel](plots/renewables_fuel_monthly_line.png)
- Wind remains strong overnight and in spring or fall, explaining off-peak dips in net load that might otherwise appear mysterious. Grid operators watch these wind ramps when committing gas units.
- Solar creates the sharp midday spike, especially in summer. The speed of its morning ramp dictates how much spinning reserve PJM must carry and how steep evening ramps become.
- Hydro and other renewables deliver a flatter baseline, cushioning periods when wind or solar disappoint; in drought-prone basins this insurance layer disappears, tightening the system.

### Heatmaps
![Net load heatmap](plots/heatmap_net_load_weekday.png)
![Renewable heatmap](plots/heatmap_renewable_weekday.png)
- Net load troughs before sunrise, particularly on weekends when commercial load pauses. These windows exhibit the highest risk of negative pricing if wind or solar surge.
- Daytime cells (0600–1800) show the highest net load, aligning with industrial and HVAC needs. Transmission planners use this pattern when stress-testing flows.
- Renewable output mirrors solar availability; midday relief is strongest in summer and weakest on winter mornings. Households on time-of-use tariffs indirectly experience this as lower daytime prices when renewables are plentiful.

### Net Load vs Price Density
![Hexbin net load vs price](plots/hexbin_netload_price_density.png)
- The densest region sits around 95–110 GW and $20–40/MWh, defining PJM’s “normal” operation.
- Scarcity hours above 125 GW are rare yet carry outsized price impact.
- Low net-load bins (<80 GW) cluster in discounted price territory, illustrating how renewable oversupply forces prices lower and sometimes negative.

### Price vs Renewable Penetration
![Scatter renewable share vs price](plots/price_vs_renewable_share_scatter.png)
![Binned renewable share vs price](plots/price_vs_renewable_share_binned.png)
- Higher renewable share systematically suppresses PJM-RTO prices because fewer gas peakers clear. Consumers experience that suppression directly through cheaper energy charges.
- The curve is steepest when renewable share is low (below fifteen percent), then flattens as share increases, implying the first tranche of renewable deployment provides the largest marginal benefit to ratepayers and emissions targets.
- Generators still pursue renewables through tax credits, REC revenue, lower fuel-risk exposure, and improving capacity accreditation even if spot prices fall.

### Rolling Correlation
![Rolling correlation](plots/rolling_corr_netload_price.png)
- Net load and price typically maintain correlation above 0.6.
- The January 2024 dip aligns with the EIA-documented Arctic outbreak that set a U.S. natural-gas demand record on 16 January; fuel scarcity temporarily broke the relationship.
- Correlation stabilises afterward, though late-2024 volatility keeps it from fully rebounding. This metric signals when fundamental models are trustworthy versus when traders must lean on fuel-market signals and outage reports.

### Net Load Trend, Volatility, and Scatter Diagnostics
![Net load timeseries](plots/net_load_timeseries.png)
![Net load volatility](plots/net_load_volatility.png)
![Scatter by month](plots/scatter_net_load_price_month.png)
![Scatter by hour](plots/scatter_net_load_price_hour.png)
![Scatter by weekday](plots/scatter_net_load_price_weekday.png)
- Rolling means reveal intraday renewable troughs layered over seasonal swings. Utilities can see when load transfers from fossil units to renewables and schedule maintenance accordingly.
- Volatility spiked in January 2024 and remained elevated throughout the back half of 2024, mirroring weather-driven stress and uneven renewable availability. The volatility plot mirrors the operational turbulence described in PJM’s winter reliability notices.
- Month-coloured scatter shifts rightward in summer (higher base net load), hour-coloured scatter separates cheap midday vs pricey evening ramps, and weekday colouring shows weekdays dominating the upper-right quadrant. Each scatter speaks to a different operational reality that feeds directly into price formation.

### Binned Relationship and Distribution
![Binned price vs net load](plots/binned_price_vs_net_load.png)
![Net load histogram](plots/net_load_hist.png)
- Marginal price reaction accelerates sharply beyond roughly 125 GW net load, demonstrating PJM’s scarcity knee and explaining where demand-response providers target their events.
- Net load distribution is right-skewed; low-stress hours dominate counts, yet the thin high-load tail drives most price spikes. Insurance-like capacity products and peak-shaving investments are priced around that tail.

### Regression and Residual Analysis
![Regression fit](plots/regression_best_fit.png)
![Price residuals vs net load](plots/residuals_vs_net_load.png)
![Residuals vs renewable MW](plots/residuals_vs_renewable_output.png)
- Degree 10 polynomial best balances bias and variance, capturing the flat-to-steep curvature seen in empirical bins without overfitting the test set.
- Residuals cluster near zero but show positive spikes during scarcity events and negative spikes during renewable surges; these outliers correspond to headline events such as forced outages or surprise renewable production.
- Residuals plotted against renewable MW confirm that high renewable output depresses price more than a net-load-only model anticipates, which is why traders track renewable forecasts alongside load forecasts.

### Extreme Net-Load Events
![Extreme events](plots/extreme_net_load_events.png)
- High net-load events cluster around winter cold snaps and late-summer heat waves, with corresponding price spikes that feed into scarcity pricing and reserve deployments.
- Low net-load events occur during mild spring weekends, pairing low demand with strong renewables; these windows carry the highest risk of negative pricing and curtailment.
- The exported CSV lists each timestamp with its load, renewable MW, and price for deeper case studies.

### Autocorrelation
![Autocorrelation](plots/autocorr_price_net_load.png)
- Net load retains daily (about 24 h) and weekly (about 168 h) memory, guiding short-term forecast horizons for system operators and quantitative traders.
- Price autocorrelation decays faster, reflecting added noise from fuel prices, outages, and market interventions; beyond one day ahead, models must incorporate exogenous drivers rather than load alone.

## Temperature Analysis Addendum

### PJM Temperature Envelope
![Temperature envelope](plots/temp_daily_trend.png)
- Daily PJM averages ranged from 32 °F to 95 °F, with the 7-day mean bottoming near 34 °F during the mid-January Arctic blast and peaking near 84 °F in July; those windows mirror the scarcity spikes called out in `energy.ipynb`.
- The shaded min/max band shows a 20 °F+ spread across constituent zones even outside extremes, illustrating why localized heating and cooling pockets still matter inside a system-level view.

### Hourly Structure by Month
![Temperature heatmap](plots/temp_hourly_heatmap.png)
- Winter nights (Dec–Feb) sit below 40 °F for roughly half the day, while summer afternoons stay above 80 °F through the 19:00 EPT ramp—exactly when fast-ramping gas units set PJM price.
- Shoulder months compress into the 50–70 °F corridor, lining up with the calmer net-load volatility and flatter price residuals previously documented.

### Temperature vs System Price
![Temperature vs price](plots/temp_vs_price_scatter.png)
- Mild 55–65 °F hours clear near $24/MWh on average, but both ≤40 °F heating events and ≥80 °F cooling events add roughly $12–$18/MWh, recreating the convex response in the net-load scatter plots.
- The coldest hours (32 °F) peaked at $287/MWh on 16–17 Jan 2024, while the July heat dome pushed prices to $160–$180/MWh—proof that both heating and cooling stress create scarcity premia.

### Temperature vs Net Load
![Temperature vs net load](plots/temp_vs_netload_scatter.png)
- Net load jumps from ~73 GW in mild weather to >95 GW once temperatures exceed ~78 °F, and similar load levels return below 40 °F when electric heating dominates.
- This symmetry explains why the polynomial fit in `energy.ipynb` steepens on both wings: temperature extremes push load into the non-linear section of the price curve regardless of season.

### Rolling Temperature Correlations
![Temperature correlations](plots/temp_corr_price_netload.png)
- Temperature maintains a 0.6–0.75 correlation with RTO net load across most of the year, only dipping during temperate spring weeks when renewables shoulder more of the load.
- Temperature-price correlation swings between -0.4 (shoulder months) and 0.7 (peak stress), signalling when a temperature-only model is sufficient versus when fuel or renewable residuals must take the lead.

### January 2024 Event Study
![January event](plots/temp_event_jan2024.png)
- The mid-January Arctic blast pushed temperature into the low 30 °F range, net load above 120 GW, and price to $287/MWh inside 48 hours—the same outlier cluster flagged in the energy notebook’s scatter plots.
- Seeing the three series stack on top of each other shows why the polynomial load fit under-predicted price: HDD-driven fuel scarcity layered on top of already elevated load.

### Temperature Bucket Deltas
![Bucket deltas](plots/temp_bucket_deltas.png)
- Relative to mild 55–65 °F hours, ≤40 °F conditions add ~$13/MWh and ~15 GW of load, while ≥80 °F adds ~$18/MWh and ~30 GW, quantifying how temperature pushes dispatch into the convex part of the price curve.
- The hour count per bucket highlights why tail events still drive portfolio risk even though they represent a minority of observations.

### Temperature vs Price Residuals
![Temperature residuals](plots/temp_vs_price_residual.png)
- After removing the 10th-degree net-load fit from `energy.ipynb`, hot (>80 °F) and cold (<40 °F) hours still carry positive $/MWh residuals, explaining the upside spikes in the original residual plots.
- Spring shoulder hours (<70 °F) fall negative, matching the renewable-rich periods where the main notebook showed prices clearing below the load-based expectation.

## Henry Hub Natural Gas Addendum

### Long-Run Price Structure
![Gas history](plots/gas_price_history.png)
- Henry Hub averaged $4.42/MMBtu since 1997 but spiked above $12 during the 2005–2008 hurricane years and again during the 2021–2022 global LNG crunch—the same windows when PJM prices decoupled in `energy.ipynb`.
- The 30-day mean now tracks the 252-day mean near $2.30/MMBtu, helping keep current PJM prices pinned in the $20–$35/MWh band.

### Volatility and Distribution
![Gas volatility](plots/gas_volatility_distribution.png)
- 30-day volatility tripled during hurricanes, the financial crisis, and Winter Storm Uri, explaining why PJM residuals ballooned even without huge net-load changes.
- Daily percent-change distribution is fat-tailed; >5 % moves are common, so hedges discussed in the energy notebook need fuel overlays to cover price tails.

### Seasonality
![Gas monthly box](plots/gas_monthly_box.png)
- Winter months show the widest range thanks to heating shocks, aligning with PJM’s peak net-load stress hours.
- Shoulder seasons cluster in the $2–$4/MMBtu band, mirroring the cheaper PJM outcomes when renewable share is high.

### Year/Month Heatmap
![Gas heatmap](plots/gas_year_month_heatmap.png)
- Hot cells in 2005–2008 and 2021–2022 map directly to hurricane seasons and the European gas crisis that drove PJM scarcity events.
- Cooler tones dominate 2009–2020 as shale output softened fuel costs, matching the flatter PJM price curves from that era.

### YoY Swings
![Gas YoY](plots/gas_yoy_change.png)
- YoY gas moves often exceed ±100 %, reinforcing why PJM price residuals spike whenever gas supply whipsaws faster than load fundamentals adjust.
- Current YoY sits near -35 %, explaining why PJM prices remain subdued despite intermittent temperature stress.

### Gas vs PJM Price
![Gas vs PJM](plots/gas_vs_pjm_price_scatter.png)
- During Oct 2023–Oct 2024, cheap $2–$3 gas aligns with $20–$35/MWh PJM clears, while the January 2024 gas pop toward $5 accompanies $150+ clears.
- Even without explicit load controls, the scatter validates gas as the marginal fuel in PJM’s stack.

### Gas + Net Load Price Regression
![Gas price model](plots/gas_price_model_fit.png)
- Combining Henry Hub price with daily net load lifts the PJM price R² to roughly 0.78 and cuts RMSE below $12/MWh, showing that fuel shocks explain most of the residual swings the net-load polynomial missed.
- Missed points concentrate when gas is expensive (> $4/MMBtu), while cheap-gas days align closely with the model’s y=x line.

### Rolling Correlation
![Gas correlation](plots/gas_rolling_correlation.png)
- 60-day correlation between gas and PJM price swings between 0.3 and 0.8, peaking during the January 2024 polar vortex.
- Residual correlation stays positive (~0.3), proving that fuel volatility explains part of the power-model error bars highlighted in `energy.ipynb`.

### Gas vs Residual Scatter
![Gas residual scatter](plots/gas_vs_residual_scatter.png)
- Gas price alone explains about one-third of the daily residual variance (R² ≈ 0.32); every $1/MMBtu move shifts the PJM residual by roughly $22/MWh even after net load is stripped out.
- Negative residuals appear only when gas is exceptionally cheap and renewables are long, reinforcing the dual dependence on fuel and temperature.

### Event Study: January 2024
![Gas event](plots/gas_event_jan2024.png)
- Gas, PJM price, and the residual all spiked in the same 48-hour window during the polar vortex, showing how fuel scarcity layered on top of high net load.
- As gas retreated in February, PJM prices and residuals collapsed, re-validating the polynomial fit for calmer conditions.

## Forecast-Driven Price Outlook

### Daily Temperature & Gas Forecasts
- `temp_forecast.ipynb` trains an XGBoost regressor on PJM-wide averages with calendar/lag features; holdout RMSE ≈ 3.8 °F (MAE ≈ 3.0 °F) so the 14-day outlook is reliable for identifying broad heating/cooling swings but not intra-hour spikes.
- `gas_forecast.ipynb` applies the same approach to Henry Hub, yielding RMSE ≈ $0.66/MMBtu (MAE ≈ $0.24) one day ahead, which is tight enough for fuel-driven scarcity checks.

### Price Projection Using Forecast Inputs
![Future price forecast](plots/combined_future_price_forecast.png)
- Feeding the predicted temperature & gas series plus a simple net-load surrogate into the XGBoost price model produces a 14-day PJM outlook; the regression still posts ~0.86 test R² / ~$5.9-MWh RMSE when validated on historical data.
- Treat these forecasts as directional guidance (which days look riskier) rather than settlement-ready prices—the upstream temperature and gas errors compound beyond two weeks.

### Rolling Backtest
![Rolling backtest](plots/combined_rolling_backtest.png)
- A 7-day walk-forward backtest retrains the daily XGBoost model using only historical data and predicts the price one week ahead; over the 2023–2024 window it lands at ~\$8/MWh RMSE with clear tracking of major swings.
- The overlay shows how the forecast line (orange) follows the actual (blue) even during shoulder-season dips, giving a realistic view of forward accuracy rather than a single holdout snapshot.

---
This README doubles as the project report. For deeper detail, review the Markdown commentary embedded throughout `energy.ipynb`, which mirrors and expands upon the observations noted above. Re-running the notebook will regenerate all figures and CSVs to keep this document in sync with the latest data.
