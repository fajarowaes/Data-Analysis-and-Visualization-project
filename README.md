1. Domain & Source Identification
1.1 Project Concept
The chosen domain is food-price inflation, food affordability, and nutrition affordability. Food inflation affects household budgets more directly than many other forms of inflation, especially in lower- and middle-income countries where food may consume a large share of household expenditure. A single national inflation percentage does not reveal whether food is rising faster than other consumer goods, whether healthy diets are becoming less affordable, or whether countries face simultaneous global food-commodity shocks.
The project will answer the central question: “Which countries are under the greatest food-price pressure, where is food inflation outpacing general inflation, and how does this relate to nutrition affordability and global food-price shocks?”
The proposed solution is named the Food Price Pressure Index. It will produce transparent country-month indicators, a watchlist, and dashboard visuals, rather than claiming to directly measure individual hunger or household purchasing behaviour.
1.2 Exact Data Sources
Source	Purpose in Project	Exact Reference / Extraction Method
FAOSTAT – Food and Agriculture Organization of the United Nations	Primary country-level monthly CPI source. The pipeline will use Food CPI and General/All-items CPI to calculate monthly and year-over-year food-price pressure.	FAOSTAT portal: https://www.fao.org/faostat/en/#home. Bulk data catalogue/API access will be used to download the official Consumer Price Indices dataset.
World Bank DataBank – Food Prices for Nutrition	Nutrition-affordability context. Offers food-price and diet-affordability/nutrition-related series that can enrich the country-level pressure analysis.	https://databank.worldbank.org/source/food-prices-for-nutrition. Data will be selected/exported as CSV/Excel from DataBank for the chosen countries/series/time period.
FAO Food Price Index (FFPI)	Global market-shock context. Provides overall and commodity-group international food-price indices such as cereals, vegetable oils, dairy, meat, and sugar.	https://www.fao.org/worldfoodsituation/foodpricesindex/. Monthly downloadable Excel/CSV data will be used.

FAOSTAT provides free food and agriculture data for more than 245 countries and territories and offers bulk downloads and an API developer portal. The World Bank Food Prices for Nutrition database provides selectable country, series, and time dimensions and supports download in formats such as CSV and Excel.
1.3 Ingestion Pattern
Full Load – Historical Baseline:
●	FAOSTAT: download the complete available Consumer Price Indices historical extract; retain the raw records for Food CPI and General/All-items CPI.
●	World Bank Food Prices for Nutrition: export the selected historical country/series/time data as a CSV or Excel extract.
●	FAO Food Price Index: download the complete historical monthly index file, including the overall and commodity-group series.
●	Load the historical extracts into Bronze with source URL, source filename, extraction timestamp, load type, and batch ID metadata.
Incremental Load – New and Updated Records:
The datasets update at different frequencies, so the pipeline supports source-specific incremental ingestion:
Source	Expected Update Pattern	Incremental Strategy
FAOSTAT CPI	Monthly / periodic release and revision	Download the newest CPI extract. Filter to newly available months plus a rolling two-to-three-month revision window. Merge by Area Code + Item Code + Year + Months Code.
World Bank Food Prices for Nutrition	Periodic database update;	Re-export or retrieve the latest available observations and retain the latest period plus a revision window. Merge by country, indicator/series, and period.
FAO Food Price Index	Monthly	Download the newest file, extract new month(s) plus a revision window, and merge by month and index series.

For Phase 1, the latest official source extract will be chronologically partitioned: records through December 2025 will represent the historical full-load baseline, and the latest available 2026 months will represent the subsequent incremental payload. In later phases, incremental batches will come from newly published source releases.
2. Data Samples & Volume
2.1 Sample Files
Sample File	Source	Contents
full_load_sample.csv	FAOSTAT CPI	Historical Food CPI and General CPI raw observations for selected countries, e.g. January 2020–December 2025. Original source columns, values, flags, and notes remain unchanged.
incremental_load_sample.csv	FAOSTAT CPI	Later raw observations for the same countries/items, e.g. January–March 2026. Schema identical to the full-load sample.
fao_ffpi_full_sample.csv	FAO Food Price Index	Optional historical monthly overall and commodity-group index sample.
worldbank_fpn_sample.csv	World Bank Food Prices for Nutrition	selected country/series/time export for nutrition-affordability context.
2.2 Volume and Frequency Estimation
Main data source is FAOSTAT CPI so its full load data is around 3000kb and incremental load is around 700kb.
The project will use monthly data only, which is sufficient for food-price analysis and avoids unnecessary storage/compute use in free-tier environments.
3. Security & Compliance
3.1 PII and Sensitive Data Assessment
The planned raw sources contain aggregated country-level or global statistical data: country names/codes, dates, CPI values, food-price indices, and nutrition-affordability indicators. They do not contain personal names, email addresses, phone numbers, precise user locations, IP addresses, payment records, bank details, individual household records, or transaction-level information.
Therefore, the project does not expect to ingest Personally Identifiable Information (PII). Country names and national statistics are not PII because they do not identify an individual.
3.2 Handling Strategy
●	No hashing or masking is required for the planned sources because they contain aggregate public statistics rather than personal data.
●	Raw source fields such as FAOSTAT flags and notes will be preserved for quality and provenance, not removed.
●	If a later enrichment dataset unexpectedly contains individual- or market-vendor-level identifiers, those fields will be dropped before entering Silver unless there is a documented lawful need and explicit instructor approval.
●	Source URLs, download timestamps, file names, batch IDs, and quality flags will be retained to make data lineage auditable.
4. High-Level Medallion Data Modeling
4.1 Bronze Layer: Raw and Immutable
The Bronze layer will preserve source data with minimal transformation. Every ingestion will include metadata fields: source_name, source_url, source_file_name, ingestion_timestamp, load_type, and batch_id.
Bronze Table	Raw Contents
bronze_faostat_cpi_raw	Original FAOSTAT CPI records: Area Code, Area Code (M49), Area, Item Code, Item, Element Code, Element, Months Code, Months, Year Code, Year, Unit, Value, Flag, Note.
bronze_fao_ffpi_raw	Original FAO Food Price Index monthly file with overall and commodity-group indices.
bronze_worldbank_fpn_raw	Original World Bank Food Prices for Nutrition DataBank CSV/Excel export.
4.2 Silver Layer: Cleansed and Conformed
The Silver layer will standardize schemas, data types, dates, codes, and quality handling across all sources. Main transformations from Bronze to Silver:
●	Cast raw string fields into appropriate numeric, date, and decimal types.
●	Build an observation_date from FAOSTAT Year and Months Code.
●	Filter FAOSTAT to the official Food CPI and General/All-items CPI item codes only.
●	Standardize country names and map them to ISO-3 country codes where possible.
●	Retain raw Flag and Note columns and map only verified flag meanings into quality-status fields.
●	Deduplicate FAOSTAT records using Area Code + Item Code + Year + Months Code.
●	Calculate monthly and year-over-year CPI changes using the same calendar month in the preceding year.
●	Standardize World Bank series, time period, country code, unit, and value columns.
●	Standardize FAO Food Price Index dates and numeric commodity-group fields.
Silver Table	Key Fields / Role
silver_country_monthly_cpi	country_code, country_name, observation_date, item_code, item_name, food_cpi, general_cpi, cpi_value, cpi_yoy_pct, source_flag, source_note.
silver_global_food_price_index	observation_date, ffpi_overall, ffpi_cereals, ffpi_oils, ffpi_dairy, ffpi_meat, ffpi_sugar.
silver_food_prices_nutrition	country_code, country_name, observation_period, indicator_code, indicator_name, indicator_value, unit, source metadata.
4.3 Gold Layer: Business and Policy Analytics
Gold tables will be designed for analysts and dashboard users. They combine cleaned national CPI, global shock context, and nutrition-affordability indicators.
Gold Table	Business Purpose / Key Measures
gold_country_food_pressure_monthly	One row per country-month. Includes Food CPI, General CPI, food inflation YoY, general inflation YoY, food-vs-general inflation gap, latest global FFPI, and a transparent pressure category.
gold_food_pressure_watchlist	Ranks countries by an explainable Food Pressure Index based on normalized food inflation, the food-vs-general gap, global food-shock context, and available nutrition-affordability/vulnerability indicators.
gold_nutrition_affordability_comparison	Compares food-price pressure with selected World Bank Food Prices for Nutrition indicators, helping identify where a healthy diet may be relatively less affordable.
gold_global_food_shock_timeline	Monthly global FFPI trend, long-term deviation, commodity-group drivers, and shock category.
gold_regional_income_group_summary	Aggregates average food pressure and affordability measures by region/income group, subject to available country classifications.

The Food Pressure Index will be clearly labeled as a student-designed, transparent analytical metric — not an official FAO or World Bank indicator. Its weights and normalization approach will be documented in the GitHub README and dashboard methodology page.
5. Business Intelligence & Dashboards
5.1 Intended Dashboard Purpose
The final dashboard (built in Power BI or Tableau) will convert complex CPI and food-price series into understandable evidence. It is designed for policy analysts, NGOs, researchers, journalists, and members of the public who need to identify where food-price pressure is unusually high and where affordability/nutrition context may make that pressure more serious.
5.2 Business Questions
1.	Which countries currently have the highest year-over-year food inflation?
2.	Where is food inflation rising faster than general consumer inflation?
3.	Which countries show persistent food-price pressure over several consecutive months?
4.	How do national food-price trends compare with global FAO food commodity shocks?
5.	How does food-price pressure relate to selected nutrition-affordability indicators from the World Bank Food Prices for Nutrition database?
6.	Which countries should appear on a monitoring watchlist based on pressure, affordability, and available vulnerability context?
5.3 Planned Visuals and Metrics
Visual / Metric	Description	Question Answered
World map: Latest Food Pressure Index	Country map coloured by Low, Medium, High, and Very High pressure. Tooltip shows Food CPI YoY, General CPI YoY, gap, and selected nutrition-affordability context.	Where is food-price pressure highest?
Ranked bar chart: High-pressure watchlist	Top 10–15 countries ranked by Food Pressure Index or food-vs-general inflation gap.	Which countries require closer monitoring?
Multi-line trend: Food vs General Inflation	Selected country comparison of food inflation and general inflation over time.	Is food becoming expensive faster than the overall basket?
6. Engineering Setup & FinOps
6.1 Version Control
The project repository has been created at:
https://github.com/fajarowaes/Data-Analysis-and-Visualization-project
6.2 Infrastructure and FinOps
The team plans to use Databricks Community/Free Edition or Azure for Students, Apache Spark, Delta/Parquet where available, GitHub, and Power BI/Tableau for the dashboard. The following FinOps controls will be applied:
●	Use monthly data rather than unnecessarily large daily or hourly data.
●	Begin development with small raw sample files before processing the full historical extracts.
●	Limit early development to selected countries and essential indicators; scale only after transformations are validated.
●	Preserve raw Bronze files once and avoid repeated unnecessary downloads or recomputation.
●	Partition cleaned data by year/month where appropriate and use Delta/Parquet for efficient downstream reads where supported.
●	Use incremental merges with a limited revision window instead of repeatedly reprocessing the complete history.
●	Shut down compute resources when not in active use and avoid retaining temporary test tables/files.
