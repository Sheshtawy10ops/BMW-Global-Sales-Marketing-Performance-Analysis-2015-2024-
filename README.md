# 🚗 BMW Global Sales & Marketing Performance Analysis (2015–2024)

## 📌 1. Project Overview
This project delivers an end-to-end Business Intelligence solution in **Microsoft Power BI** analyzing **10 years (2015–2024)** of BMW global sales, powertrain adoption, retail dealership reach, and marketing efficiency across **10 international markets** and **10 vehicle models**. 

Using a dataset of **1,000 regional-monthly sales records** (`bmw_global_sales_dataset.csv`), the report transforms raw transactional and marketing metrics into an interactive, three-page executive dashboard (`BMW SALES.pbix`) designed to evaluate revenue trajectories, Electric Vehicle (EV) transition momentum, pricing positioning, and Marketing Efficiency Ratio (MER).

* **Time Horizon:** January 2015 – December 2024 (10 Years)
* **Markets Covered (10):** China, UK, France, India, Brazil, Australia, Germany, USA, Japan, Canada
* **Models Analyzed (10):** X1, X3, X5, X7, 3 Series, 5 Series, 7 Series, i4, i7, iX
* **Vehicle Segments (3):** Electric, Sedan, SUV
* **Powertrains / Engine Types (4):** Electric, Hybrid, Diesel, Petrol
* **Headline Performance:** **$47.97B** Total Revenue | **633,192** Units Sold | **$279.49M** Marketing Spend | **171.63x** Marketing Efficiency Ratio (MER)

---

## 🎯 2. Business Problems Identified

### A. Strategic Business Problems
1. **Revenue Volatility & Macro Cyclicality:** Global revenue experienced sharp cyclical swings between 2015 ($5.79B) and 2023 ($4.41B) before rebounding in 2024 ($5.16B). Leadership needed visibility into which regions and models stabilize top-line performance during market contractions.
2. **Powertrain Transition & EV Adoption Tracking:** As automotive regulations tighten globally, management must track the multi-year shift from internal combustion engines (Diesel/Petrol) to electrified powertrains (Electric/Hybrid) to guide R&D and production capacity.
3. **Price vs. Volume Positioning across the Portfolio:** Understanding whether high-ticket models (`X1`, `X7`, `i4`) sacrifice volume for margin, or if core mid-tier models (`X3`, `3 Series`) can simultaneously drive unit volume and total revenue yield.
4. **Marketing Budget Misallocation & Diminishing Returns:** Marketing investments ($279.49M total) were distributed across regions and models without clear Return on Ad Spend (ROAS / MER) benchmarking—resulting in certain models (`3 Series`) and markets (`Canada`) absorbing high marketing spend with below-average revenue efficiency.
5. **Dealership Network Distribution:** Assessing whether dealership network reach aligns with the highest-converting vehicle models across global territories.

### B. Dataset Quality & Structural Challenges Discovered
1. **Fragmented Temporal Fields (`year` & `month`):** The raw CSV stored `year` (integer) and `month` (integer) in separate columns without a unified `Date` data type, preventing native Power BI Time Intelligence (`SAMEPERIODLASTYEAR`, continuous date hierarchies, and drill-downs).
2. **Missing Row-Level Revenue Field:** The raw dataset contained `price_usd` and `units_sold` but lacked a pre-calculated `Revenue` column, requiring either a Power Query custom column or a DAX iterator (`SUMX`) to avoid aggregation errors (since `SUM(price_usd) * SUM(units_sold)` produces an invalid cross-product).
3. **Zero-Variance Macroeconomic Columns:** Three columns in the raw dataset—`fuel_price_usd` (constant at **0.97**), `gdp_growth_percent` (constant at **6.29%**), and `interest_rate_percent` (constant at **1.20%**)—exhibit zero variance across all 1,000 rows and provide no analytical explanatory power.
4. **Synthetic Domain Noise in `segment` & `engine_type`:** Because the dataset is synthetically generated, models appear across multiple segments and engine types (e.g., `i4` and `i7` rows labeled with `Diesel`/`Petrol` powertrains, or `X5` rows labeled as `Sedan`). *(Note: The current dashboard visualizes the dataset as-is; for production deployment, a deterministic mapping rule in Power Query is documented below.)*

---

## 🛠️ 3. Data Cleaning & Transformation in Power BI (Power Query)

The following ETL steps were executed in **Power Query Editor** to prepare the data model for reporting:

1. **Data Ingestion & Schema Enforcement:**
   * Imported `bmw_global_sales_dataset.csv` (1,000 rows × 14 columns).
   * Promoted headers and enforced strict data types:
     * **Text:** `country`, `model`, `segment`, `engine_type`
     * **Whole Number:** `year`, `month`, `dealership_count`, `competition_index`, `units_sold`
     * **Currency / Decimal Number:** `price_usd`, `marketing_spend_usd`, `fuel_price_usd`, `gdp_growth_percent`, `interest_rate_percent`
2. **Date Consolidation for Time Intelligence (`Date` Column):**
   * Merged `year` and `month` into a standardized **Start-of-Month `Date` column** (`#date([year], [month], 1)`) so Power BI could generate the `Date Hierarchy` (`Year` → `Quarter` → `Month` → `Day`) used across all trend charts and slicers.
3. **Null & Duplicate Verification:**
   * Verified 0% null values and 0 duplicate records across all 1,000 rows.
4. **Feature Selection & Model Optimization:**
   * Excluded the zero-variance macro columns (`fuel_price_usd`, `gdp_growth_percent`, `interest_rate_percent`) and `competition_index` from visual layers to keep the executive view focused on actionable levers (Price, Volume, Dealerships, and Marketing Spend).
   * *(Optional Production Enhancement in Power Query M)*: Standardize `segment` by model family (`X1/X3/X5/X7` → `SUV`, `3/5/7 Series` → `Sedan`, `i4/i7/iX` → `Electric`) to eliminate synthetic cross-labeling.

---

## 🔄 4. End-to-End Project Workflow (Steps Done)

1. **Requirement Gathering & KPI Definition:** Defined core executive KPIs across three analytical pillars: *Executive Overview*, *Product & Powertrain Positioning*, and *Marketing ROI*.
2. **Data Profiling & ETL in Power Query:** Audited raw CSV distributions, verified data types, constructed the unified `Date` dimension hierarchy, and validated row-level granularity.
3. **DAX Measure Engineering:** Authored 11 explicit DAX measures covering revenue iterators, Year-over-Year (YoY) unit growth, market share percentages, unit economics, and marketing efficiency ratios.
4. **Visual Filtering & Ranking Logic:** Applied **Top N (Top 5)** visual-level filters on regional and model rankings (`Global Revenue Distribution`, `Top 5 Revenue-Generating Models`, `Retail Network Reach per Model`, `Budget Allocation by Product Line`, and `Regional Marketing Investment`) to highlight primary performance drivers without visual clutter.
5. **UI/UX Dashboard Design:** Built a 3-page interactive Power BI interface featuring custom BMW branding (`bmw-logo`), a cohesive navigation bar (`pageNavigator`), interactive hierarchical date slicers (`Year`, `Quarter`, `Month`), and KPI summary cards.

---

## 📐 5. DAX Measures Library

Below are the exact DAX measures powering the three dashboard pages:

### A. Core Revenue & Volume Measures (Page 1: `OVERVIEW`)
```dax
// 1. Total Revenue (Row-level price multiplied by units sold)
Total Revenue = 
SUMX(
    'bmw_global_sales_dataset',
    'bmw_global_sales_dataset'[price_usd] * 'bmw_global_sales_dataset'[units_sold]
)

// 2. Total Units / Total Units Sold
Total Units = SUM('bmw_global_sales_dataset'[units_sold])

Total Units Sold = SUM('bmw_global_sales_dataset'[units_sold])

// 3. Total Models
Total Models = DISTINCTCOUNT('bmw_global_sales_dataset'[model])

// 4. Unit Growth (YoY)
Unit Growth (YOY) = 
VAR CurrentYearUnits = [Total Units]
VAR PrevYearUnits = 
    CALCULATE(
        [Total Units],
        SAMEPERIODLASTYEAR('bmw_global_sales_dataset'[Date].[Date])
    )
RETURN
    DIVIDE(CurrentYearUnits - PrevYearUnits, PrevYearUnits, 0)
    // 5. Top Model (Units)
Top Model (Units) = 
CALCULATE(
    SELECTEDVALUE('bmw_global_sales_dataset'[model]),
    TOPN(
        1,
        ALL('bmw_global_sales_dataset'[model]),
        [Total Units Sold],
        DESC
    )
)

// 6. % Unit Share
% Unit Share = 
DIVIDE(
    [Total Units Sold],
    CALCULATE([Total Units Sold], ALL('bmw_global_sales_dataset')),
    0
)

// 7. Revenue per Unit (Weighted Realized Average Selling Price)
Revenue per Unit = 
DIVIDE([Total Revenue], [Total Units Sold], 0)

// 8. Average Price (Unweighted Catalog Price Average)
Average Price = 
AVERAGE('bmw_global_sales_dataset'[price_usd])
// 9. Total Marketing Spend
Total Marketing Spend = 
SUM('bmw_global_sales_dataset'[marketing_spend_usd])

// 10. Marketing Efficiency Ratio (MER / ROAS)
MARKETING EFFECIENCY RATIO = 
DIVIDE([Total Revenue], [Total Marketing Spend], 0)

// 11. Marketing Cost per Car (Customer Acquisition Cost per Unit)
Marketing Cost per Car = 
DIVIDE([Total Marketing Spend], [Total Units Sold], 0)
```
## 📊 6. Dashboard Breakdown & Key Insights
## Page 1: OVERVIEW — Executive Revenue & Regional Performance
Visuals Included: KPI Cards (Total Revenue, Total Units, Total Models, Unit Growth (YOY)), Top 5 Bar Chart (Global Revenue Distribution), Area Chart (Annual Revenue Performance Trend), Donut Chart (Revenue Share by Vehicle Category), Top 5 Clustered Column Chart (Top 5 Revenue-Generating Models), and Date Hierarchy Slicer.

Key Insights:

Portfolio Scale: Over the 10-year period, BMW generated $47.97B in total revenue across 633,192 units sold spanning 10 models.

Top 5 Revenue-Generating Countries: China leads globally with $6.07B in revenue (12.66% global share; 77,488 units), followed by the UK ($5.49B; 11.44%), France ($5.03B; 10.48%), India ($4.97B; 10.37%), and Brazil ($4.96B; 10.34%). Together, these top 5 markets account for 55.3% ($26.52B) of global revenue.

Top 5 Revenue-Generating Models: The X3 is BMW's #1 revenue generator at $5.62B (11.72% share), followed by the i4 ($5.34B; 11.14%), 3 Series ($5.28B; 11.01%), X5 ($5.03B; 10.48%), and 7 Series ($4.87B; 10.15%).

Balanced Category Revenue Split: Revenue is evenly diversified across segments, with Electric leading slightly at $16.61B (34.62%), followed by Sedan at $15.77B (32.88%) and SUV at $15.59B (32.50%).

Annual Trajectory & 2024 Rebound: After peaking in 2015 ($5.79B) and experiencing cyclical dips in 2016 ($3.98B) and 2023 ($4.41B), annual revenue surged +16.84% YoY in 2024 to $5.16B, driven by a +19.99% YoY jump in units sold (from 57,036 in 2023 to 68,440 in 2024).

## Page 2: Product & Engine Analysis — Powertrain Shift & Unit Economics
Visuals Included: KPI Cards (Top Model (Units), % Unit Share, Revenue per Unit, Average Price), Scatter Plot (Price vs. Volume Positioning Matrix), Funnel Chart (Sales Volume by Engine Type), Top 5 Clustered Column Chart (Retail Network Reach per Model), and 100% Stacked Area Chart (Powertrain Adoption Roadmap).

Key Insights:

Unit Volume Leader: The X3 ranks #1 in physical volume with 73,358 units sold (11.59% of total volume), closely followed by the 3 Series (70,886 units; 11.20%) and i4 (69,528 units; 10.98%).

Pricing vs. Realized Revenue per Unit: Across the portfolio, the unweighted Average Price is $76,535.75, while the volume-weighted Revenue per Unit is $75,757.69. This -$778.06 spread indicates that slightly lower-priced configurations convert higher unit volumes. The X7 commands the highest realized revenue per unit ($79,395.40), whereas the 5 Series has the lowest ($72,298.55).

Powertrain Volume Ranking (Funnel): Electric powertrains lead overall cumulative volume with 170,694 units (26.96%), outperforming Hybrid (159,624 units; 25.21%), Diesel (157,598 units; 24.89%), and Petrol (145,276 units; 22.94%).

Decade-Long EV Adoption Surge (2015 vs. 2024): The Powertrain Adoption Roadmap reveals a structural transformation:

In 2015, Diesel dominated with 29.27% of unit sales while Electric accounted for just 18.89%.

By 2024, Electric unit share nearly doubled to 34.68% (more than 1 in 3 vehicles sold), while Diesel contracted sharply to 17.47%. Combined electrified powertrains (Electric + Hybrid) reached 59.02% of total unit volume in 2024.

Retail Network Reach: The 3 Series (115 active market-month allocations), 7 Series (107), i4 (107), X3 (105), and X5 / i7 (102) enjoy the broadest retail distribution frequency across global dealerships.

## Page 3: Marketing & Market Strategy — Budget Allocation & ROI
Visuals Included: KPI Cards (Total Revenue, Total Marketing Spend, MARKETING EFFECIENCY RATIO, Marketing Cost per Car), Scatter Chart (Marketing Efficiency vs. Revenue Yield), Top 5 Column Chart (Budget Allocation by Product Line), Top 5 Bar Chart (Regional Marketing Investment), and Marketing Spend Trend Area Chart.

Key Insights:

Headline Marketing Efficiency: BMW invested $279.49M in marketing over the decade, achieving an average Marketing Efficiency Ratio (MER) of 171.63x ($171.63 in revenue per $1 of marketing spend, or a 0.58% marketing-to-sales ratio) and a Marketing Cost per Car of $441.40.

Model-Level ROI Gap (X3 & iX vs. 3 Series):

Budget Allocation Top 5: The 3 Series received the highest marketing budget at $34.36M, followed by 7 Series ($30.41M), i4 ($30.40M), X5 ($30.15M), and X3 ($29.00M).

Efficiency Winners & Laggards: Despite ranking 5th in marketing spend ($29.00M), the X3 delivered #1 in revenue ($5.62B) and the highest MER of any model at 193.88x ($395.39 marketing cost per car). Similarly, the iX achieved the 2nd-highest MER (186.21x) on the smallest marketing budget ($20.67M). Conversely, the heavily funded 3 Series posted the lowest MER across all models at 153.72x ($484.73 marketing cost per car).

Regional ROI Disparity (China & UK vs. Canada & Brazil):

Top 5 Regional Marketing Spend: China ($32.28M), UK ($29.63M), Brazil ($29.33M), India ($28.20M), and France ($28.16M).

Regional Efficiency: China (188.12x MER) and the UK (185.21x MER) pair high investment with top-tier efficiency. In contrast, Canada absorbed $27.83M in marketing spend (6th highest) but generated the lowest revenue ($4.10B) and the lowest MER globally at 147.36x ($497.35 marketing cost per car).

## 💡 7. Strategic Business Recommendations
PriorityAreaData-Backed FindingActionable RecommendationExpected Business Impact1. Reallocate Model Marketing Budgets3 Series consumes the #1 budget ($34.36M) at the lowest MER (153.72x), while X3 (193.88x) and iX (186.21x) are under-indexed in ad spend relative to yield.Shift 15%–20% ($5M–$6.8M) of the 3 Series and 7 Series marketing budgets toward high-MER growth models (X3, iX, and i4).Lifts portfolio-wide MER from 171.6x toward ~180x+ without increasing total ad spend.2. Optimize Regional Ad Spend in North AmericaCanada has the lowest MER globally (147.36x, $27.83M spend for $4.10B revenue), while the USA receives the lowest marketing spend ($24.68M) despite higher MER (168.62x) and higher average selling prices.Audit Canadian campaign channels to reduce customer acquisition cost from $497/car toward the global benchmark ($441/car), and redeploy surplus budget into high-margin USA, France, and UK campaigns.Improves North American regional ROAS and captures untapped volume in the US market.3. Accelerate EV & Hybrid Inventory in Top MarketsElectric share surged from 18.89% (2015) to 34.68% (2024), while Diesel fell to 17.47%; i4 is already the #2 revenue model globally ($5.34B).Phase down Diesel dealership allocations by 25%+ in 2025–2026 and expand i4, i7, and iX showroom inventory across top-performing markets (China, UK, France).Aligns supply chain and dealership stock with the 59% electrified (EV + Hybrid) demand trend.4. Expand Dealership Footprint for High-Margin ModelsX7 generates the highest realized revenue per unit ($79,395) and iX delivers the 2nd highest MER (186.21x), but both have the lowest dealership distribution frequency (90 and 81 records vs. 115 for 3 Series).Increase flagship dealership allocations for the X7 and iX in high-net-worth territories (China, UK, USA, Germany).Captures high-margin luxury SUV/EV demand and raises overall portfolio Revenue per Unit.5. Data Governance & Pipeline StandardizationRaw dataset contains zero-variance macro columns (fuel_price_usd, gdp_growth_percent, interest_rate_percent) and cross-labeled segment/engine_type records.Implement an automated Power Query validation table (DimModel & DimMacro) to enforce strict 1-to-1 model-to-segment mapping and ingest live macroeconomic APIs by country-year.Eliminates synthetic data noise and enables true econometric elasticity modeling in future releases.
