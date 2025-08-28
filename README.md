# carbon_emission_analysis
This project analyzes carbon emissions across industries, countries, and years to identify sectors with the highest footprints and uncover trends. Carbon emissions account for over 75% of global emissions, driving climate change and environmental challenges. 
The goal is to provide insights into the environmental impact of industries and support sustainable development decisions.
## Data Overview
### Where does our data come from?
Our dataset is compiled from publicly available data from nature.com and encompasses the product carbon footprints (PCF) for various companies. PCFs represent the greenhouse gas emissions associated with specific products, quantified in CO2 (carbon dioxide equivalent).
### Data Structure

The dataset consists of 4 tables containing information regarding carbon emissions generated during the production of goods.

![image](https://github.com/dinhhluu/carbon_emission_analysis/blob/main/Database%20diagram.png)

### Table Previews
Let's have a look the original table 'product_emissions':

```
`select *
from product_emissions 
limit 5`
```
|id|company_id|country_id|industry_group_id|year|product_name|weight_kg|carbon_footprint_pcf|upstream_percent_total_pcf|operations_percent_total_pcf|downstream_percent_total_pcf|
|--|----------|----------|-----------------|----|------------|---------|--------------------|--------------------------|----------------------------|----------------------------|
|10056-1-2014|82|28|2|2014|Frosted Flakes(R) Cereal|0.7485|2|57.50|30.00|12.50|
|10056-1-2015|82|28|15|2015|"Frosted Flakes, 23 oz, produced in Lancaster, PA (one carton)"|0.7485|2|57.50|30.00|12.50|
|10222-1-2013|83|28|8|2013|Office Chair|20.68|73|80.63|17.36|2.01|
|10261-1-2017|14|16|25|2017|Multifunction Printers|110.0|1488|30.65|5.51|63.84|
|10261-2-2017|14|16|25|2017|Multifunction Printers|110.0|1818|25.08|4.51|70.41|


## Pre-processing
We performed a duplicate check to ensure data quality. Duplicates were identified using SQL groupings and removed to avoid double counting in emissions analysis.
```
SELECT id, company_id, country_id, industry_group_id, year 
from product_emissions 
Group by id, company_id, country_id, industry_group_id, year 
Having count(*) > 1
```
```
WITH handle_duplicates AS (
    SELECT *
    FROM (
        SELECT 
            pe.*,
            ROW_NUMBER() OVER (
                PARTITION BY id 
                ORDER BY id
            ) AS rn
        FROM product_emissions AS pe
    ) AS raw
    WHERE raw.rn = 1
)
SELECT *
FROM handle_duplicates;
```
## Analyse
### We analyze which products contribute the most to carbon emissions in order to identify major sources of environmental impact.
```
SELECT 
    product_name,
    Round(AVG(carbon_footprint_pcf), 2) AS pcf
FROM product_emissions
GROUP BY product_name
ORDER BY pcf DESC
LIMIT 10;
```
|product_name|pcf|
|------------|---|
|Wind Turbine G128 5 Megawats|3718044.00|
|Wind Turbine G132 5 Megawats|3276187.00|
|Wind Turbine G114 2 Megawats|1532608.00|
|Wind Turbine G90 2 Megawats|1251625.00|
|Land Cruiser Prado. FJ Cruiser. Dyna trucks. Toyoace.IMV def unit.|191687.00|
|Retaining wall structure with a main wall (sheet pile): 136 tonnes of steel sheet piles and 4 tonnes of tierods per 100 meter wall|167000.00|
|TCDE|99075.00|
|Mercedes-Benz GLE (GLE 500 4MATIC)|91000.00|
|Mercedes-Benz S-Class (S 500)|85000.00|
|Mercedes-Benz SL (SL 350)|72000.00|

Wind energy equipment (e.g., large wind turbines) shows the highest carbon footprints during production, followed by the automobile sector (SUVs and luxury cars such as Land Cruiser, Mercedes-Benz). This indicates that heavy manufacturing and the automotive industry are the top contributors to carbon emissions, making them priority areas for emission reduction strategies and sustainable innovation.

### We examine which industry groups these products fall under to link product-level emissions with sector-level impacts.
```
WITH top_products AS (
    SELECT 
        p.product_name,
        p.industry_group_id,
        p.carbon_footprint_pcf
    FROM product_emissions AS p
    ORDER BY p.carbon_footprint_pcf DESC
    LIMIT 10
)
SELECT 
    tp.product_name,
    i.industry_group,
    tp.carbon_footprint_pcf
FROM top_products tp
JOIN industry_groups i
    ON tp.industry_group_id = i.id;
```
|product_name|industry_group|carbon_footprint_pcf|
|------------|--------------|--------------------|
|Wind Turbine G128 5 Megawats|Electrical Equipment and Machinery|3718044|
|Wind Turbine G132 5 Megawats|Electrical Equipment and Machinery|3276187|
|Wind Turbine G114 2 Megawats|Electrical Equipment and Machinery|1532608|
|Wind Turbine G90 2 Megawats|Electrical Equipment and Machinery|1251625|
|Land Cruiser Prado. FJ Cruiser. Dyna trucks. Toyoace.IMV def unit.|Automobiles & Components|191687|
|Retaining wall structure with a main wall (sheet pile): 136 tonnes of steel sheet piles and 4 tonnes of tierods per 100 meter wall|Materials|167000|
|TCDE|Materials|99075|
|TCDE|Materials|99075|
|Mercedes-Benz GLE (GLE 500 4MATIC)|Automobiles & Components|91000|
|Electric Motor|Capital Goods|87589|

The top 10 products by carbon footprint are dominated by Electrical Equipment & Machinery, with wind turbines occupying the top 4 spots (up to ~3.7M PCF). Only two other sectors appear — Automobiles & Components and Materials — with much lower emissions (~0.17–0.19M PCF). This highlights a paradox: while wind turbines enable renewable energy, their production generates very high emissions, revealing a large gap across industries.

### We identify the industries with the highest total carbon emissions by aggregating product footprints at the industry level
```
SELECT 
    i.industry_group,
    Round(SUM(p.carbon_footprint_pcf), 2) AS total_emissions
FROM product_emissions AS p
JOIN industry_groups AS i
    ON p.industry_group_id = i.id
GROUP BY i.industry_group
ORDER BY total_emissions DESC
LIMIT 10;

```
|industry_group|total_emissions|
|--------------|---------------|
|Electrical Equipment and Machinery|9801558.00|
|Automobiles & Components|2582264.00|
|Materials|577595.00|
|Technology Hardware & Equipment|363776.00|
|Capital Goods|258712.00|
|"Food, Beverage & Tobacco"|111131.00|
|"Pharmaceuticals, Biotechnology & Life Sciences"|72486.00|
|Chemicals|62369.00|
|Software & Services|46544.00|
|Media|23017.00|

The results above show the top 10 industry groups with the highest carbon emissions. These industries should prioritize immediate actions to reduce their carbon footprint, with a strong focus on adopting green and sustainable production practices.
Especially, Industries like Electrical Equipment & Machinery and Automobiles & Components contribute the most to carbon emissions, making them key priorities for green innovation and emission reduction.

### We identify the companies with the highest contribution to carbon emissions by aggregating product footprints at the company level.
```
select
	c.company_name,
	Round(SUM(p.carbon_footprint_pcf), 2) AS total_emissions
from product_emissions AS p
Join companies as c
	on p.company_id = c.id
group by c.company_name
Order by total_emissions DESC 
limit 10; 
```
|company_name|total_emissions|
|------------|---------------|
|"Gamesa Corporación Tecnológica, S.A."|9778464.00|
|Daimler AG|1594300.00|
|Volkswagen AG|655960.00|
|"Mitsubishi Gas Chemical Company, Inc."|212016.00|
|"Hino Motors, Ltd."|191687.00|
|Arcelor Mittal|167007.00|
|Weg S/A|160655.00|
|General Motors Company|137007.00|
|"Lexmark International, Inc."|132012.00|
|"Daikin Industries, Ltd."|105600.00|

Companies at the top of the list, such as Gamesa Corporación Tecnológica, Daimler AG, and Volkswagen AG, account for the highest carbon emissions. These leading emitters should implement immediate and effective strategies to reduce their carbon footprint and move towards greener, more sustainable operations

### We identify the countries with the highest carbon emissions by aggregating data at the country level.

```
select
	co.country_name,
	Round(SUM(p.carbon_footprint_pcf), 2) AS total_emissions
from product_emissions AS p
Join countries as co
	on p.company_id = co.id
group by co.country_name
Order by total_emissions DESC 
limit 10; 
```
|country_name|total_emissions|
|------------|---------------|
|Germany|9778464.00|
|Lithuania|212016.00|
|Greece|191687.00|
|Japan|132012.00|
|Colombia|105600.00|
|South Africa|35505.00|
|France|21364.00|
|Italy|20000.00|
|Ireland|11160.00|
|India|9328.00|

![image](https://github.com/dinhhluu/carbon_emission_analysis/blob/main/total%20emission%20by%20country.png)

Germany dominates emissions (~9.8M PCF), far surpassing all other countries (<0.25M PCF), mainly due to heavy industries such as automobile manufacturing

### We analyze the trend of product carbon footprints (PCFs) over the years by aggregating and comparing yearly averages.
```
SELECT 
    r.year,
    ROUND(AVG(r.carbon_footprint_pcf), 2) AS avg_pcf,
    ROUND(SUM(r.carbon_footprint_pcf), 2) AS total_pcf
FROM handle_duplicates AS r
GROUP BY r.year
ORDER BY r.year;
```
|year|avg_pcf|total_pcf|
|----|-------|---------|
|2013|2771.37|496076.00|
|2014|2885.42|548229.00|
|2015|49817.54|10810407.00|
|2016|7397.98|1612760.00|
|2017|3685.98|228531.00|

![image](https://github.com/dinhhluu/carbon_emission_analysis/blob/main/Trend%20of%20PCF%20over%20the%20years.png)

2013–2014: Emissions were low and steady (~2.8K).
2015: Huge jump (avg ~49.8K, total ~10.8M) — likely caused by very high-emission products added that year.
2016–2017: Emissions went down but stayed above early levels.
Overall: A big spike in 2015, then a drop, but not back to the original low values.

### We identify which industry groups show the largest reduction in product carbon footprints (PCFs) over time by grouping data by industry and year, then comparing the year-over-year averages.

```
SELECT 
    r.year,                         
    i.industry_group,              
    ROUND(SUM(r.carbon_footprint_pcf), 2) AS sum_pcf   
FROM handle_duplicates AS r
JOIN industry_groups i 
    ON r.industry_group_id = i.id  
GROUP BY 
    r.year, 
    i.industry_group               
ORDER BY 
    i.industry_group, 
    r.year,
    sum_pcf;
```
|year|industry_group|sum_pcf|
|----|--------------|-------|
|2015|"Consumer Durables, Household and Personal Products"|931.00|
|2013|"Food, Beverage & Tobacco"|4308.00|
|2014|"Food, Beverage & Tobacco"|2023.00|
|2015|"Food, Beverage & Tobacco"|0.00|
|2016|"Food, Beverage & Tobacco"|99639.00|
|2017|"Food, Beverage & Tobacco"|3162.00|
|2015|"Forest and Paper Products - Forestry, Timber, Pulp and Paper, Rubber"|8909.00|
|2015|"Mining - Iron, Aluminum, Other Metals"|8181.00|
|2013|"Pharmaceuticals, Biotechnology & Life Sciences"|32271.00|
|2014|"Pharmaceuticals, Biotechnology & Life Sciences"|40215.00|
|2015|"Textiles, Apparel, Footwear and Luxury Goods"|228.00|
|2013|Automobiles & Components|130189.00|
|2014|Automobiles & Components|230015.00|
|2015|Automobiles & Components|817227.00|
|2016|Automobiles & Components|1404833.00|
|2013|Capital Goods|60117.00|
|2014|Capital Goods|93699.00|
|2015|Capital Goods|3505.00|
|2016|Capital Goods|6369.00|
|2017|Capital Goods|94943.00|
|2015|Chemicals|44939.00|
|2013|Commercial & Professional Services|817.00|
|2014|Commercial & Professional Services|477.00|
|2016|Commercial & Professional Services|2890.00|
|2017|Commercial & Professional Services|741.00|
|2013|Consumer Durables & Apparel|2860.00|
|2014|Consumer Durables & Apparel|3123.00|
|2016|Consumer Durables & Apparel|1114.00|
|2015|Containers & Packaging|2988.00|
|2015|Electrical Equipment and Machinery|9801558.00|
|2013|Energy|750.00|
|2016|Energy|10024.00|
|2015|Food & Beverage Processing|138.00|
|2014|Food & Staples Retailing|773.00|
|2015|Food & Staples Retailing|706.00|
|2016|Food & Staples Retailing|2.00|
|2015|Gas Utilities|61.00|
|2013|Household & Personal Products|0.00|
|2013|Materials|194464.00|
|2014|Materials|66719.00|
|2016|Materials|61887.00|
|2017|Materials|107129.00|
|2013|Media|9645.00|
|2014|Media|9645.00|
|2015|Media|1919.00|
|2016|Media|1808.00|
|2014|Retailing|11.00|
|2015|Retailing|11.00|
|2014|Semiconductors & Semiconductor Equipment|50.00|
|2016|Semiconductors & Semiconductor Equipment|2.00|
|2015|Semiconductors & Semiconductors Equipment|3.00|
|2013|Software & Services|3.00|
|2014|Software & Services|143.00|
|2015|Software & Services|22851.00|
|2016|Software & Services|22846.00|
|2017|Software & Services|690.00|
|2013|Technology Hardware & Equipment|60539.00|
|2014|Technology Hardware & Equipment|101153.00|
|2015|Technology Hardware & Equipment|93807.00|
|2016|Technology Hardware & Equipment|1285.00|
|2017|Technology Hardware & Equipment|21866.00|
|2013|Telecommunication Services|52.00|
|2014|Telecommunication Services|183.00|
|2015|Telecommunication Services|183.00|
|2015|Tires|2022.00|
|2015|Tobacco|1.00|
|2015|Trading Companies & Distributors and Commercial Services & Supplies|239.00|
|2013|Utilities|61.00|
|2016|Utilities|61.00|

Food, Beverage & Tobacco shows the most dramatic reduction: from ~4,300 (2013) → ~2,000 (2014) → 0 in 2015, then slightly back (~99K in 2016, ~3K in 2017).
Pharmaceuticals, Biotechnology & Life Sciences declined after peaking in 2014 (~40K → 32K in 2013).
Other industries (e.g., Automobiles & Components, Mining, Forest Products) have smaller or fluctuating changes, but no clear long-term downward trend.
