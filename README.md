# World Bank Real-Time Food Prices: Bronze / Silver Pipeline

Databricks (PySpark + Delta Lake) pipeline that loads the World Bank Real-Time Food Prices dataset into a **Bronze** table (raw strings) and a **Silver** table (typed, validated, de-duplicated).
Catalog `workspace`; schemas `bronze`, `silver`, `rejects`, `audit`.

| Notebook | Purpose |
|---|---|
| `01_world_bank_bronze` | Downloads the API to CSV and MERGEs it into `bronze.world_bank_food_prices`. |
| `02_world_bank_validation` | Optional checks on Bronze (keys, coordinates, dates). |
| `03_silver` | Casts, validates, de-duplicates and MERGEs into `silver.world_bank_food_prices`. |

Bronze -> Silver; bad rows go to `rejects.*`, run logs to `audit.*`.

# Data models

**Primary key (Bronze and Silver): `ISO3` + `geo_id` + `price_date`.** Delta does not enforce keys; uniqueness is enforced by the MERGE condition and checked after each load (no duplicates, no null keys).

## `bronze.world_bank_food_prices` (835 columns)

Raw layer: every source column is `string`. Rows are keyed by `ISO3 + geo_id + price_date`.

**Descriptive columns (19)**

| Column | Data type | Primary key |
|---|---|---|
| `ISO3` | string | PK |
| `country` | string |  |
| `adm1_name` | string |  |
| `adm2_name` | string |  |
| `mkt_name` | string |  |
| `lat` | string |  |
| `lon` | string |  |
| `geo_id` | string | PK |
| `price_date` | string | PK |
| `year` | string |  |
| `month` | string |  |
| `currency` | string |  |
| `components` | string |  |
| `start_dense_data` | string |  |
| `last_survey_point` | string |  |
| `data_coverage` | string |  |
| `data_coverage_recent` | string |  |
| `index_confidence_score` | string |  |
| `spatially_interpolated` | string |  |

**Price columns (811): all `string`.** Full list under *Price columns* below.

**Metadata columns (5)**

| Column | Data type | Primary key |
|---|---|---|
| `_run_id` | string |  |
| `_source_system` | string |  |
| `_source_file` | string |  |
| `_ingestion_timestamp` | timestamp |  |
| `load_timestamp` | timestamp |  |

## `silver.world_bank_food_prices` (837 columns)

Typed layer. Same columns as Bronze with real types, plus two Silver audit columns.

**Descriptive columns (19)**

| Column | Data type | Primary key |
|---|---|---|
| `ISO3` | string | PK |
| `country` | string |  |
| `adm1_name` | string |  |
| `adm2_name` | string |  |
| `mkt_name` | string |  |
| `lat` | double |  |
| `lon` | double |  |
| `geo_id` | string | PK |
| `price_date` | date | PK |
| `year` | int |  |
| `month` | int |  |
| `currency` | string |  |
| `components` | string |  |
| `start_dense_data` | string |  |
| `last_survey_point` | string |  |
| `data_coverage` | double |  |
| `data_coverage_recent` | double |  |
| `index_confidence_score` | double |  |
| `spatially_interpolated` | boolean |  |

**Price columns (811): all `double`.** Full list under *Price columns* below.

**Metadata columns (7)**

| Column | Data type | Primary key |
|---|---|---|
| `_run_id` | string |  |
| `_source_system` | string |  |
| `_source_file` | string |  |
| `_ingestion_timestamp` | timestamp |  |
| `load_timestamp` | timestamp |  |
| `_silver_run_id` | string |  |
| `_silver_processed_at` | timestamp |  |

## Price columns (811)

The same 811 column names exist in Bronze (`string`) and Silver (`double`), in this order within each group. None is part of the primary key. Nulls are expected (a market reports only a few commodities per month).

**Commodity price columns (115)**

```
apples, bananas, beans, beans_fao, bread, bread_fao, butter, cabbage, carrots, cassava, cassava_fao,
cassava_flour, cassava_flour_fao, cassava_leaves, cassava_meal, cheese, chickpeas, chili, cocoa, cocoyam_fao,
coffee_instant, couscous, couscous_fao, cowpeas, cucumbers, dates, eggplants, eggs, fish, fish_barbel_sole,
fish_catfish, fish_goldstripe_sardinella, fish_mackerel, fish_mullet_catfish, fish_salted,
fish_sardine_canned, fish_smoked, fish_tilapia, fish_tuna_canned, fonio, gari_fao, garlic, groundnuts,
groundnuts_paste, honey, lemons, lentils, lettuce, livestock_camel_fao, livestock_goat, livestock_goat_fao,
livestock_goat_s_fao, livestock_sheep, livestockgoat_castrated_male, livestockgoat_male,
livestocksheep_castrated_male, livestocksheep_male, maize, maize_fao, maize_flour, maize_meal, mangoes,
meat_beef, meat_beef_canned, meat_beef_fao, meat_buffalo, meat_camel_fao, meat_chicken, meat_chicken_broiler,
meat_goat, meat_lamb, meat_pork, meat_sheep, milk, milk_fao, millet, millet_fao, millet_flour, oil, oil_fao,
okra, onions, onions_fao, oranges, pasta, peppers, plantains, plantains_fao, potatoes, potatoes_fao, pulses,
pumpkin, rice, rice_fao, rice_various, salt, sesame, shrimps, sorghum, sorghum_fao, sorghum_food_aid, sugar,
tea, teff_fao, tomatoes, tomatoes_fao, tomatoes_paste, watermelons, wheat, wheat_fao, wheat_flour,
wheat_flour_fao, wheat_flour_low_price_fao, yam, yogurt
```

**`o_*` columns**

```
o_apples, o_bananas, o_beans, o_beans_fao, o_bread, o_bread_fao, o_butter, o_cabbage, o_carrots, o_cassava,
o_cassava_fao, o_cassava_flour, o_cassava_flour_fao, o_cassava_leaves, o_cassava_meal, o_cheese, o_chickpeas,
o_chili, o_cocoa, o_cocoyam_fao, o_coffee_instant, o_couscous, o_couscous_fao, o_cowpeas, o_cucumbers,
o_dates, o_eggplants, o_eggs, o_fish, o_fish_barbel_sole, o_fish_catfish, o_fish_goldstripe_sardinella,
o_fish_mackerel, o_fish_mullet_catfish, o_fish_salted, o_fish_sardine_canned, o_fish_smoked, o_fish_tilapia,
o_fish_tuna_canned, o_fonio, o_gari_fao, o_garlic, o_groundnuts, o_groundnuts_paste, o_honey, o_lemons,
o_lentils, o_lettuce, o_livestock_camel_fao, o_livestock_goat, o_livestock_goat_fao, o_livestock_goat_s_fao,
o_livestock_sheep, o_livestockgoat_castrated_male, o_livestockgoat_male, o_livestocksheep_castrated_male,
o_livestocksheep_male, o_maize, o_maize_fao, o_maize_flour, o_maize_meal, o_mangoes, o_meat_beef,
o_meat_beef_canned, o_meat_beef_fao, o_meat_buffalo, o_meat_camel_fao, o_meat_chicken, o_meat_chicken_broiler,
o_meat_goat, o_meat_lamb, o_meat_pork, o_meat_sheep, o_milk, o_milk_fao, o_millet, o_millet_fao,
o_millet_flour, o_oil, o_oil_fao, o_okra, o_onions, o_onions_fao, o_oranges, o_pasta, o_peppers, o_plantains,
o_plantains_fao, o_potatoes, o_potatoes_fao, o_pulses, o_pumpkin, o_rice, o_rice_fao, o_rice_various, o_salt,
o_sesame, o_shrimps, o_sorghum, o_sorghum_fao, o_sorghum_food_aid, o_sugar, o_tea, o_teff_fao, o_tomatoes,
o_tomatoes_fao, o_tomatoes_paste, o_watermelons, o_wheat, o_wheat_fao, o_wheat_flour, o_wheat_flour_fao,
o_wheat_flour_low_price_fao, o_yam, o_yogurt, o_food_price_index
```

**`h_*` columns**

```
h_apples, h_bananas, h_beans, h_beans_fao, h_bread, h_bread_fao, h_butter, h_cabbage, h_carrots, h_cassava,
h_cassava_fao, h_cassava_flour, h_cassava_flour_fao, h_cassava_leaves, h_cassava_meal, h_cheese, h_chickpeas,
h_chili, h_cocoa, h_cocoyam_fao, h_coffee_instant, h_couscous, h_couscous_fao, h_cowpeas, h_cucumbers,
h_dates, h_eggplants, h_eggs, h_fish, h_fish_barbel_sole, h_fish_catfish, h_fish_goldstripe_sardinella,
h_fish_mackerel, h_fish_mullet_catfish, h_fish_salted, h_fish_sardine_canned, h_fish_smoked, h_fish_tilapia,
h_fish_tuna_canned, h_fonio, h_gari_fao, h_garlic, h_groundnuts, h_groundnuts_paste, h_honey, h_lemons,
h_lentils, h_lettuce, h_livestock_camel_fao, h_livestock_goat, h_livestock_goat_fao, h_livestock_goat_s_fao,
h_livestock_sheep, h_livestockgoat_castrated_male, h_livestockgoat_male, h_livestocksheep_castrated_male,
h_livestocksheep_male, h_maize, h_maize_fao, h_maize_flour, h_maize_meal, h_mangoes, h_meat_beef,
h_meat_beef_canned, h_meat_beef_fao, h_meat_buffalo, h_meat_camel_fao, h_meat_chicken, h_meat_chicken_broiler,
h_meat_goat, h_meat_lamb, h_meat_pork, h_meat_sheep, h_milk, h_milk_fao, h_millet, h_millet_fao,
h_millet_flour, h_oil, h_oil_fao, h_okra, h_onions, h_onions_fao, h_oranges, h_pasta, h_peppers, h_plantains,
h_plantains_fao, h_potatoes, h_potatoes_fao, h_pulses, h_pumpkin, h_rice, h_rice_fao, h_rice_various, h_salt,
h_sesame, h_shrimps, h_sorghum, h_sorghum_fao, h_sorghum_food_aid, h_sugar, h_tea, h_teff_fao, h_tomatoes,
h_tomatoes_fao, h_tomatoes_paste, h_watermelons, h_wheat, h_wheat_fao, h_wheat_flour, h_wheat_flour_fao,
h_wheat_flour_low_price_fao, h_yam, h_yogurt, h_food_price_index
```

**`l_*` columns**

```
l_apples, l_bananas, l_beans, l_beans_fao, l_bread, l_bread_fao, l_butter, l_cabbage, l_carrots, l_cassava,
l_cassava_fao, l_cassava_flour, l_cassava_flour_fao, l_cassava_leaves, l_cassava_meal, l_cheese, l_chickpeas,
l_chili, l_cocoa, l_cocoyam_fao, l_coffee_instant, l_couscous, l_couscous_fao, l_cowpeas, l_cucumbers,
l_dates, l_eggplants, l_eggs, l_fish, l_fish_barbel_sole, l_fish_catfish, l_fish_goldstripe_sardinella,
l_fish_mackerel, l_fish_mullet_catfish, l_fish_salted, l_fish_sardine_canned, l_fish_smoked, l_fish_tilapia,
l_fish_tuna_canned, l_fonio, l_gari_fao, l_garlic, l_groundnuts, l_groundnuts_paste, l_honey, l_lemons,
l_lentils, l_lettuce, l_livestock_camel_fao, l_livestock_goat, l_livestock_goat_fao, l_livestock_goat_s_fao,
l_livestock_sheep, l_livestockgoat_castrated_male, l_livestockgoat_male, l_livestocksheep_castrated_male,
l_livestocksheep_male, l_maize, l_maize_fao, l_maize_flour, l_maize_meal, l_mangoes, l_meat_beef,
l_meat_beef_canned, l_meat_beef_fao, l_meat_buffalo, l_meat_camel_fao, l_meat_chicken, l_meat_chicken_broiler,
l_meat_goat, l_meat_lamb, l_meat_pork, l_meat_sheep, l_milk, l_milk_fao, l_millet, l_millet_fao,
l_millet_flour, l_oil, l_oil_fao, l_okra, l_onions, l_onions_fao, l_oranges, l_pasta, l_peppers, l_plantains,
l_plantains_fao, l_potatoes, l_potatoes_fao, l_pulses, l_pumpkin, l_rice, l_rice_fao, l_rice_various, l_salt,
l_sesame, l_shrimps, l_sorghum, l_sorghum_fao, l_sorghum_food_aid, l_sugar, l_tea, l_teff_fao, l_tomatoes,
l_tomatoes_fao, l_tomatoes_paste, l_watermelons, l_wheat, l_wheat_fao, l_wheat_flour, l_wheat_flour_fao,
l_wheat_flour_low_price_fao, l_yam, l_yogurt, l_food_price_index
```

**`c_*` columns**

```
c_apples, c_bananas, c_beans, c_beans_fao, c_bread, c_bread_fao, c_butter, c_cabbage, c_carrots, c_cassava,
c_cassava_fao, c_cassava_flour, c_cassava_flour_fao, c_cassava_leaves, c_cassava_meal, c_cheese, c_chickpeas,
c_chili, c_cocoa, c_cocoyam_fao, c_coffee_instant, c_couscous, c_couscous_fao, c_cowpeas, c_cucumbers,
c_dates, c_eggplants, c_eggs, c_fish, c_fish_barbel_sole, c_fish_catfish, c_fish_goldstripe_sardinella,
c_fish_mackerel, c_fish_mullet_catfish, c_fish_salted, c_fish_sardine_canned, c_fish_smoked, c_fish_tilapia,
c_fish_tuna_canned, c_fonio, c_gari_fao, c_garlic, c_groundnuts, c_groundnuts_paste, c_honey, c_lemons,
c_lentils, c_lettuce, c_livestock_camel_fao, c_livestock_goat, c_livestock_goat_fao, c_livestock_goat_s_fao,
c_livestock_sheep, c_livestockgoat_castrated_male, c_livestockgoat_male, c_livestocksheep_castrated_male,
c_livestocksheep_male, c_maize, c_maize_fao, c_maize_flour, c_maize_meal, c_mangoes, c_meat_beef,
c_meat_beef_canned, c_meat_beef_fao, c_meat_buffalo, c_meat_camel_fao, c_meat_chicken, c_meat_chicken_broiler,
c_meat_goat, c_meat_lamb, c_meat_pork, c_meat_sheep, c_milk, c_milk_fao, c_millet, c_millet_fao,
c_millet_flour, c_oil, c_oil_fao, c_okra, c_onions, c_onions_fao, c_oranges, c_pasta, c_peppers, c_plantains,
c_plantains_fao, c_potatoes, c_potatoes_fao, c_pulses, c_pumpkin, c_rice, c_rice_fao, c_rice_various, c_salt,
c_sesame, c_shrimps, c_sorghum, c_sorghum_fao, c_sorghum_food_aid, c_sugar, c_tea, c_teff_fao, c_tomatoes,
c_tomatoes_fao, c_tomatoes_paste, c_watermelons, c_wheat, c_wheat_fao, c_wheat_flour, c_wheat_flour_fao,
c_wheat_flour_low_price_fao, c_yam, c_yogurt, c_food_price_index
```

**`inflation_*` columns**

```
inflation_apples, inflation_bananas, inflation_beans, inflation_beans_fao, inflation_bread,
inflation_bread_fao, inflation_butter, inflation_cabbage, inflation_carrots, inflation_cassava,
inflation_cassava_fao, inflation_cassava_flour, inflation_cassava_flour_fao, inflation_cassava_leaves,
inflation_cassava_meal, inflation_cheese, inflation_chickpeas, inflation_chili, inflation_cocoa,
inflation_cocoyam_fao, inflation_coffee_instant, inflation_couscous, inflation_couscous_fao,
inflation_cowpeas, inflation_cucumbers, inflation_dates, inflation_eggplants, inflation_eggs, inflation_fish,
inflation_fish_barbel_sole, inflation_fish_catfish, inflation_fish_goldstripe_sardinella,
inflation_fish_mackerel, inflation_fish_mullet_catfish, inflation_fish_salted, inflation_fish_sardine_canned,
inflation_fish_smoked, inflation_fish_tilapia, inflation_fish_tuna_canned, inflation_fonio,
inflation_gari_fao, inflation_garlic, inflation_groundnuts, inflation_groundnuts_paste, inflation_honey,
inflation_lemons, inflation_lentils, inflation_lettuce, inflation_livestock_camel_fao,
inflation_livestock_goat, inflation_livestock_goat_fao, inflation_livestock_goat_s_fao,
inflation_livestock_sheep, inflation_livestockgoat_castrated_male, inflation_livestockgoat_male,
inflation_livestocksheep_castrated_male, inflation_livestocksheep_male, inflation_maize, inflation_maize_fao,
inflation_maize_flour, inflation_maize_meal, inflation_mangoes, inflation_meat_beef,
inflation_meat_beef_canned, inflation_meat_beef_fao, inflation_meat_buffalo, inflation_meat_camel_fao,
inflation_meat_chicken, inflation_meat_chicken_broiler, inflation_meat_goat, inflation_meat_lamb,
inflation_meat_pork, inflation_meat_sheep, inflation_milk, inflation_milk_fao, inflation_millet,
inflation_millet_fao, inflation_millet_flour, inflation_oil, inflation_oil_fao, inflation_okra,
inflation_onions, inflation_onions_fao, inflation_oranges, inflation_pasta, inflation_peppers,
inflation_plantains, inflation_plantains_fao, inflation_potatoes, inflation_potatoes_fao, inflation_pulses,
inflation_pumpkin, inflation_rice, inflation_rice_fao, inflation_rice_various, inflation_salt,
inflation_sesame, inflation_shrimps, inflation_sorghum, inflation_sorghum_fao, inflation_sorghum_food_aid,
inflation_sugar, inflation_tea, inflation_teff_fao, inflation_tomatoes, inflation_tomatoes_fao,
inflation_tomatoes_paste, inflation_watermelons, inflation_wheat, inflation_wheat_fao, inflation_wheat_flour,
inflation_wheat_flour_fao, inflation_wheat_flour_low_price_fao, inflation_yam, inflation_yogurt,
inflation_food_price_index
```

**`trust_*` columns**

```
trust_apples, trust_bananas, trust_beans, trust_beans_fao, trust_bread, trust_bread_fao, trust_butter,
trust_cabbage, trust_carrots, trust_cassava, trust_cassava_fao, trust_cassava_flour, trust_cassava_flour_fao,
trust_cassava_leaves, trust_cassava_meal, trust_cheese, trust_chickpeas, trust_chili, trust_cocoa,
trust_cocoyam_fao, trust_coffee_instant, trust_couscous, trust_couscous_fao, trust_cowpeas, trust_cucumbers,
trust_dates, trust_eggplants, trust_eggs, trust_fish, trust_fish_barbel_sole, trust_fish_catfish,
trust_fish_goldstripe_sardinella, trust_fish_mackerel, trust_fish_mullet_catfish, trust_fish_salted,
trust_fish_sardine_canned, trust_fish_smoked, trust_fish_tilapia, trust_fish_tuna_canned, trust_fonio,
trust_gari_fao, trust_garlic, trust_groundnuts, trust_groundnuts_paste, trust_honey, trust_lemons,
trust_lentils, trust_lettuce, trust_livestock_camel_fao, trust_livestock_goat, trust_livestock_goat_fao,
trust_livestock_goat_s_fao, trust_livestock_sheep, trust_livestockgoat_castrated_male,
trust_livestockgoat_male, trust_livestocksheep_castrated_male, trust_livestocksheep_male, trust_maize,
trust_maize_fao, trust_maize_flour, trust_maize_meal, trust_mangoes, trust_meat_beef, trust_meat_beef_canned,
trust_meat_beef_fao, trust_meat_buffalo, trust_meat_camel_fao, trust_meat_chicken, trust_meat_chicken_broiler,
trust_meat_goat, trust_meat_lamb, trust_meat_pork, trust_meat_sheep, trust_milk, trust_milk_fao, trust_millet,
trust_millet_fao, trust_millet_flour, trust_oil, trust_oil_fao, trust_okra, trust_onions, trust_onions_fao,
trust_oranges, trust_pasta, trust_peppers, trust_plantains, trust_plantains_fao, trust_potatoes,
trust_potatoes_fao, trust_pulses, trust_pumpkin, trust_rice, trust_rice_fao, trust_rice_various, trust_salt,
trust_sesame, trust_shrimps, trust_sorghum, trust_sorghum_fao, trust_sorghum_food_aid, trust_sugar, trust_tea,
trust_teff_fao, trust_tomatoes, trust_tomatoes_fao, trust_tomatoes_paste, trust_watermelons, trust_wheat,
trust_wheat_fao, trust_wheat_flour, trust_wheat_flour_fao, trust_wheat_flour_low_price_fao, trust_yam,
trust_yogurt, trust_food_price_index
```

## Supporting tables

| Table | Primary key | Columns (type) |
|---|---|---|
| `audit.pipeline_runs` | `run_id`, `layer` | run_id, source_name, layer, load_type, parameter_processed (string); start_time, end_time (timestamp); records_read, records_inserted, records_updated, records_rejected (bigint); status, error_message (string) |
| `audit.pipeline_execution_logs` | `run_id` | run_id, layer, load_type, parameter_processed, source_file, status, error_message (string); start_time, end_time (timestamp); rows_inserted, rows_updated, rows_processed (bigint) |
| `rejects.silver_food_price_rejects_v2` | `ISO3`, `geo_id`, `price_date`, `reject_run_id` | ISO3, country, geo_id, reject_reason, reject_run_id (string); price_date (date); lat, lon (double); year, month (int); rejected_at, load_timestamp (timestamp) |
| `rejects.bronze_schema_rejects` | `run_id` | run_id, source_file, reject_reason, source_columns, target_columns (string); detected_at (timestamp) |
| `rejects.silver_schema_drift_logs` | `run_id`, `column_name`, `drift_type` | run_id, column_name, drift_type, source_type, target_type (string); detected_at (timestamp) |
| `rejects.silver_type_violations` | `run_id`, `column_name` | run_id, column_name (string); bad_value_count (bigint); load_timestamp (timestamp) |

Silver rejects a row if `ISO3` is not 3 characters, or `country`, `geo_id` or `price_date` is missing, or `lat`/`lon` is missing or out of range, or `month` is outside 1..12. Null price values are kept.

# Run instructions

Run order: **01 (Bronze) -> 03 (Silver)**; 02 is optional. Set the widgets at the top of each notebook:

| Widget | Notebooks | Values |
|---|---|---|
| `source_path` | 01 | CSV path, default `/Volumes/workspace/bronze/raw_files/world_bank_food_prices_2011_2026.csv` |
| `load_type` | 01, 03 | `full`, `incremental`, `backfill` |
| `start_date`, `end_date` | 01, 03 | `YYYY-MM-DD`; filter `price_date` for incremental and backfill (ignored for `full`) |
| `batch_id` | 01, 03 | optional |

Run the widgets cell first; the later cells depend on it.

## Full load
Set `load_type = full` in 01 and 03, then run 01 and 03 top to bottom.

## Incremental load (recent window)
1. In 01 set `load_type = incremental`, `start_date`/`end_date` to the new months (e.g. `2026-08-01` to `2026-09-30`) and run it. New keys are inserted; existing keys update only if values changed.
2. In 03 set the same `load_type`, `start_date` and `end_date` and run it. Silver MERGEs that window on the primary key.
3. Do not run the optional `#cleanup` cell in 03 (it drops Silver, reject and audit tables).

## Backfill (historical range)
1. In 01 set `load_type = backfill`, `start_date`/`end_date` to the range (e.g. `2015-01-01` to `2015-12-31`) and run it.
2. Run 03 with the same values.
3. The MERGE is on the primary key, so a backfill never creates duplicates and is safe to rerun.

## Check the load
```sql
SELECT ISO3, geo_id, price_date, COUNT(*) FROM silver.world_bank_food_prices
GROUP BY ISO3, geo_id, price_date HAVING COUNT(*) > 1;   -- expect 0 rows
SELECT * FROM audit.pipeline_runs ORDER BY start_time DESC LIMIT 5;
```
