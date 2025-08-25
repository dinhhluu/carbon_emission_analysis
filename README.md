# carbon_emission_analysis
This project analyzes carbon emissions across industries, countries, and years to identify sectors with the highest footprints and uncover trends. Carbon emissions account for over 75% of global emissions, driving climate change and environmental challenges. 
The goal is to provide insights into the environmental impact of industries and support sustainable development decisions.
## Data Overview
### Where does our data come from?
Our dataset is compiled from publicly available data from nature.com and encompasses the product carbon footprints (PCF) for various companies. PCFs represent the greenhouse gas emissions associated with specific products, quantified in CO2 (carbon dioxide equivalent).
### Data Structure
The dataset consists of 4 tables containing information regarding carbon emissions generated during the production of goods.



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
|Wind Turbine G128 5 Megawats|3718044.0000|
|Wind Turbine G132 5 Megawats|3276187.0000|
|Wind Turbine G114 2 Megawats|1532608.0000|
|Wind Turbine G90 2 Megawats|1251625.0000|
|Land Cruiser Prado. FJ Cruiser. Dyna trucks. Toyoace.IMV def unit.|191687.0000|
|Retaining wall structure with a main wall (sheet pile): 136 tonnes of steel sheet piles and 4 tonnes of tierods per 100 meter wall|167000.0000|
|TCDE|99075.0000|
|Mercedes-Benz GLE (GLE 500 4MATIC)|91000.0000|
|Mercedes-Benz S-Class (S 500)|85000.0000|
|Mercedes-Benz SL (SL 350)|72000.0000|

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
|product_name|pcf|
|------------|---|
|Wind Turbine G128 5 Megawats|3718044.0000|
|Wind Turbine G132 5 Megawats|3276187.0000|
|Wind Turbine G114 2 Megawats|1532608.0000|
|Wind Turbine G90 2 Megawats|1251625.0000|
|Land Cruiser Prado. FJ Cruiser. Dyna trucks. Toyoace.IMV def unit.|191687.0000|
|Retaining wall structure with a main wall (sheet pile): 136 tonnes of steel sheet piles and 4 tonnes of tierods per 100 meter wall|167000.0000|
|TCDE|99075.0000|
|Mercedes-Benz GLE (GLE 500 4MATIC)|91000.0000|
|Mercedes-Benz S-Class (S 500)|85000.0000|
|Mercedes-Benz SL (SL 350)|72000.0000|


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
|product_name|pcf|
|------------|---|
|Wind Turbine G128 5 Megawats|3718044.0000|
|Wind Turbine G132 5 Megawats|3276187.0000|
|Wind Turbine G114 2 Megawats|1532608.0000|
|Wind Turbine G90 2 Megawats|1251625.0000|
|Land Cruiser Prado. FJ Cruiser. Dyna trucks. Toyoace.IMV def unit.|191687.0000|
|Retaining wall structure with a main wall (sheet pile): 136 tonnes of steel sheet piles and 4 tonnes of tierods per 100 meter wall|167000.0000|
|TCDE|99075.0000|
|Mercedes-Benz GLE (GLE 500 4MATIC)|91000.0000|
|Mercedes-Benz S-Class (S 500)|85000.0000|
|Mercedes-Benz SL (SL 350)|72000.0000|

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

### We identify the countries with the highest carbon emissions by aggregating data at the country level.



