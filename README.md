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

The results above show the top 10 industry groups with the highest carbon emissions. These industries should prioritize immediate actions to reduce their carbon footprint, with a strong focus on adopting green and sustainable production practices.
Especially, Industries like Electrical Equipment & Machinery and Automobiles & Components contribute the most to carbon emissions, making them key priorities for green innovation and emission reduction.

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
|id|company_id|country_id|industry_group_id|year|product_name|weight_kg|carbon_footprint_pcf|upstream_percent_total_pcf|operations_percent_total_pcf|downstream_percent_total_pcf|rn|
|--|----------|----------|-----------------|----|------------|---------|--------------------|--------------------------|----------------------------|----------------------------|--|
|10056-1-2014|82|28|2|2014|Frosted Flakes(R) Cereal|0.7485|2|57.50|30.00|12.50|1|
|10056-1-2015|82|28|15|2015|"Frosted Flakes, 23 oz, produced in Lancaster, PA (one carton)"|0.7485|2|57.50|30.00|12.50|1|
|10222-1-2013|83|28|8|2013|Office Chair|20.68|73|80.63|17.36|2.01|1|
|10261-1-2017|14|16|25|2017|Multifunction Printers|110.0|1488|30.65|5.51|63.84|1|
|10261-2-2017|14|16|25|2017|Multifunction Printers|110.0|1818|25.08|4.51|70.41|1|
|10261-3-2017|14|16|25|2017|Multifunction Printers|110.0|2274|20.05|3.61|76.34|1|
|10324-1-2016|15|16|19|2016|KURALON  fiber|1500.0|10000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10418-1-2013|84|9|19|2013|Portland Cement|1000.0|1102|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-1-2014|85|28|11|2014|501® Original Jeans – Dark Stonewash|0.997|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-1-2015|85|28|6|2015|501® Original Jeans – Dark Stonewash|0.997|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-1-2016|85|28|11|2016|501® Original Jeans – Dark Stonewash|0.997|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-10-2014|85|28|11|2014|Regular Straight 505® Jeans – Steel (Water<Less™)|0.7665|15|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-10-2015|85|28|6|2015|Regular Straight 505® Jeans – Steel (Water<Less™)|0.7665|15|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-10-2016|85|28|11|2016|Regular Straight 505® Jeans – Steel (Water<Less™)|0.7665|15|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-11-2014|85|28|11|2014|501® Original Jeans – Rinse Run|0.997|15|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-11-2015|85|28|6|2015|501® Original Jeans – Rinse Run|0.997|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-2-2014|85|28|11|2014|Slim Straight 514™ Jeans – Indigo Wash|0.68|9|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-2-2015|85|28|6|2015|Slim Straight 514™ Jeans – Indigo Wash|0.68|9|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-2-2016|85|28|11|2016|Slim Straight 514™ Jeans – Indigo Wash|0.88|9|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-3-2014|85|28|11|2014|Slim Straight 514™ Jeans – Rigid Tank|0.68|8|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-3-2015|85|28|6|2015|Slim Straight 514™ Jeans – Rigid Tank|0.68|8|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-3-2016|85|28|11|2016|Slim Straight 514™ Jeans – Rigid Tank|0.88|8|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-4-2014|85|28|11|2014|501® Original Jeans – Light Stonewash|0.997|15|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-4-2015|85|28|6|2015|501® Original Jeans – Light Stonewash|0.997|15|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-5-2014|85|28|11|2014|501® Original Jeans – Medium Stonewash|0.997|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-5-2015|85|28|6|2015|501® Original Jeans – Medium Stonewash|0.997|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-6-2014|85|28|11|2014|Slim Straight 514™ Jeans – Tumbled Rigid|0.68|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-6-2015|85|28|6|2015|Slim Straight 514™ Jeans – Tumbled Rigid|0.68|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-7-2014|85|28|11|2014|Regular Straight 505® Jeans – Range (Water<Less™)|0.7665|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-7-2015|85|28|6|2015|Regular Straight 505® Jeans – Range (Water<Less™)|0.7665|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-8-2014|85|28|11|2014|Regular Straight 505® Jeans – Range|0.7665|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-8-2015|85|28|6|2015|Regular Straight 505® Jeans – Range|0.7665|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-9-2014|85|28|11|2014|Regular Straight 505® Jeans – House Cat|0.7665|15|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10661-9-2015|85|28|6|2015|Regular Straight 505® Jeans – House Cat|0.7665|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-1-2014|16|28|25|2014|CX310DN - Color Laser Printer|31.365912|2358|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-10-2014|16|28|25|2014|MS315DN - Mono Laser Printer|16.03449|1616|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-11-2014|16|28|25|2014|MS410DN - Mono Laser Printer|15.898413|1523|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-12-2014|16|28|25|2014|MS415DN - Mono Laser Printer|16.03449|1525|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-13-2014|16|28|25|2014|MS510DN - Mono Laser Printer|16.66952|1618|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-14-2014|16|28|25|2014|MS610DE - Mono Laser Printer|18.37049|1978|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-15-2014|16|28|25|2014|MS610DN - Mono Laser Printer|17.576704|1958|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-16-2014|16|28|25|2014|MS810DE - Mono Laser Printer|27.4877|2107|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-17-2014|16|28|25|2014|MS810DN - Mono Laser Printer|27.44234|2005|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-18-2014|16|28|25|2014|MS811DN - Mono Laser Printer|27.44234|2388|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-19-2014|16|28|25|2014|MS812DE - Mono Laser Printer|27.75985|2744|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-2-2014|16|28|25|2014|CX410DE - Color Laser Printer|31.365912|3236|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-20-2014|16|28|25|2014|MS812DN - Mono Laser Printer|27.44234|2747|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-21-2014|16|28|25|2014|MX310DN - Mono Laser Printer|22.45282|1140|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-22-2014|16|28|25|2014|MX410DE - Mono Laser Printer|23.019813|1499|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-23-2014|16|28|25|2014|MX511DE - Mono Laser Printer|25.4012|1679|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-24-2014|16|28|25|2014|MX611DE - Mono Laser Printer|28.03201|2132|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-25-2014|16|28|25|2014|MX710DE - Mono Laser Printer|57.1526|2923|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-26-2014|16|28|25|2014|MX711DE - Mono Laser Printer|57.1526|3127|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-27-2014|16|28|25|2014|MX810DFE - Mono Laser Printer|118.07009|2536|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-28-2014|16|28|25|2014|MX811DFE - Mono Laser Printer|118.07009|2973|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-29-2014|16|28|25|2014|MX812DFE - Mono Laser Printer|118.07009|3336|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-3-2014|16|28|25|2014|CX510DE - Color Laser Printer|32.6587|3129|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-4-2014|16|28|25|2014|CS310DN - Color Laser Printer|20.32094|1969|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-5-2014|16|28|25|2014|CS410DN - Color Laser Printer|20.50238|3143|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-6-2014|16|28|25|2014|CS510DE - Color Laser Printer|20.91061|3025|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-7-2014|16|28|25|2014|M5163 - Mono Laser Printer|23.700201|2597|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-8-2014|16|28|25|2014|MS310DN - Mono Laser Printer|15.807694|1505|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10666-9-2014|16|28|25|2014|MS312DN - Mono Laser Printer|15.807694|1490|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-1-2014|86|22|19|2014|Mobile Batteries|1000.0|5846|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-1-2015|86|22|9|2015|Mobile Batteries|1000.0|10245|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-2-2014|86|22|19|2014|Automotive Batteries|1000.0|3113|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-2-2013|86|22|19|2013|"LG Chem’s automotive batteries are sustaining the undisputed No.1 position in the world based on more than 10 years of R&D capability. LG Chem is equipped with solutions for all types of electrical cars (Hybrid EV, Plugged-in EV, Battery EV) based on strong product development capability and manufacturing competitiveness via mass production experience. This enables LG Chem to supply battery cell, pack and BMS (Battery Management System) to global carmakers such as GM, Ford, Renault, Hyundai-Kia and Volvo. LG Chem expects the increased revenue from its automotive battery business by being awarded next-generation projects from existing customers while strengthening strategic partnerships, and securing new customers. Additionally, LG Chem plans to launch new products to meet customer needs in new applications such as SLI and garden tools."|1000.0|3197|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-2-2015|86|22|9|2015|Automotive Batteries|1000.0|6309|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-2-2016|86|22|19|2016|Automotive Battery|1000.0|4286|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-3-2015|86|22|9|2015|ABS (Acrylonitrile Butadiene Styrene)|1000.0|876|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-3-2016|86|22|19|2016|ABS(Acrylonitrile Butadiene Styrene)|1000.0|396|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-4-2016|86|22|19|2016|BPA(Bisphenol-A)|1000.0|1088|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10667-5-2016|86|22|19|2016|IT&New Application Battery|1000.0|10607|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|10670-1-2013|87|22|11|2013|LG-E970|0.1471|7|11.28|0.01|88.70|1|
|10834-1-2015|88|25|25|2015|"Mouse, M185"|0.136|4|4.63|60.42|34.95|1|
|1085-1-2013|35|27|2|2013|White crystalline sugar|1000.0|650|46.15|53.85|0.00|1|
|1085-1-2014|35|27|2|2014|Sugar|1000.0|650|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|1085-1-2016|35|27|2|2016|Sugar|1000.0|650|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|11152-1-2013|89|3|19|2013|grinding media cast.|1500.0|1750|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|11632-1-2016|92|26|22|2016|IC chips of Smartphone|0.1|2|72.39|27.61|0.00|1|
|1198-1-2013|1|28|24|2013|1 DVD Software Package|0.0907185|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|1198-1-2014|1|28|24|2014|DVD software|0.0907185|1|44.05|11.15|44.79|1|
|1198-1-2015|1|28|24|2015|DVD software (1 DVD)|0.0907185|1|44.05|11.15|44.80|1|
|1198-2-2013|1|28|24|2013|USB Software|0.0085|2|75.54|2.22|22.24|1|
|1198-2-2014|1|28|24|2014|USB software|0.0085|2|67.61|2.22|30.17|1|
|1198-2-2015|1|28|24|2015|USB software|0.0085|2|67.61|2.22|30.17|1|
|1198-4-2015|1|28|24|2015|DVD software (2 DVDs)|0.181437|2|43.72|17.13|39.15|1|
|12084-1-2016|95|23|19|2016|Filtro 26 WS|1.0|3|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12117-1-2013|96|27|10|2013|Dell Laptop|2.32|320|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12117-2-2013|96|27|10|2013|"Kimberley-Clark toilet tissue, 10,000 sheets"|4.2|20|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12134-1-2017|17|16|19|2017|Paraformaldehyde|500.0|200|56.50|29.93|13.57|1|
|12134-2-2017|17|16|19|2017|Methacrylic acid|20.0|71|64.44|34.13|1.43|1|
|12134-3-2017|17|16|19|2017|Hydrogen peroxide|20.0|4|65.22|34.54|0.24|1|
|12134-4-2017|17|16|19|2017|Amines|9.0|8|63.68|33.73|2.60|1|
|12134-5-2017|17|16|19|2017|Peroxides|25.0|42|63.82|33.80|2.38|1|
|12134-6-2017|17|16|19|2017|Super-pure hydrogen peroxide|17736.0|6469|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12134-7-2017|17|16|19|2017|ELM|160.0|139|7.13|3.77|89.10|1|
|12134-8-2017|17|16|19|2017|TCDE|12000.0|99075|64.95|34.40|0.66|1|
|12289-1-2017|18|16|8|2017|Zinc Oxide|20.0|6|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12348-1-2013|97|28|2|2013|500ml Coors Light brewed in the UK (steal can) CO2e expressed in CO2e/ hl|100.0|31|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12348-2-2013|97|28|2|2013|"Carling brewed in the UK, 24 440ml cans in 4 packs"|12.029|3|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12348-2-2014|97|28|2|2014|"Carling brewed in the UK, 24 440ml cans in 4 packs"|12.029|3|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12348-3-2013|97|28|2|2013|"Carling brewed in the UK, 24 568ml cans in 4 packs"|15.53|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12348-3-2014|97|28|2|2014|"Carling brewed in the UK, 24 568ml cans in 4 packs"|15.53|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12348-4-2014|97|28|2|2014|"Coors Light brewed in the UK, 24 500ml cans in 4 packs"|13.67|4|64.93|28.58|6.49|1|
|12903-1-2015|102|16|25|2015|Server|19.28|1600|5.40|2.06|92.54|1|
|12903-2-2015|102|16|25|2015|Broadband Routers|0.816|56|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12903-3-2015|102|16|25|2015|Mobile Network Equipment|19.28|10273|1.83|0.70|97.48|1|
|12942-1-2013|103|25|2|2013|"1 dl cup of spray dried soluble Nescafé coffee ready to be drunk, at the consumer’s home"|0.1|0|24.43|16.03|59.54|1|
|12942-1-2014|103|25|2|2014|Nescafé soluble coffee|0.1|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12942-1-2015|103|25|15|2015|Nescafé soluble coffee|0.1|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|12942-1-2016|103|25|2|2016|Nescafé soluble coffee|0.12|0|49.61|11.05|39.34|1|
|12942-2-2013|103|25|2|2013|"EcoShape  bottle of water (0.5 L): Serving of 500 ml of hydration to the consumer. The study examines the extraction and processing of all upstream raw materials to produce the products and packaging components; the manufacturing process; distribution logistics; product use and packaging disposal. The work has been conducted using the Ecoinvent database and SimaPro software.  Climate change impacts are measured in kg CO2 equivalents, following the IPCC’s most recent methodology).  Scenario evaluations have been used to examine the importance of several variables in the product comparison. These include a range of possible consumer behaviours regarding the use of the products, such as conditions of refrigeration, washing of glasses, and disposal routes for packaging, among others."|0.54167|0|45.88|4.92|49.20|1|
|12942-2-2014|103|25|2|2014|EcoShape  bottle of water (0.5 L)|0.54167|0|45.75|4.91|49.34|1|
|12942-2-2015|103|25|15|2015|EcoShape bottle of water (0.5 L)|0.54167|0|45.75|4.91|49.34|1|
|12942-2-2016|103|25|2|2016|EcoShape bottle of water (0.5 L)|0.54167|0|45.75|4.91|49.34|1|
|12942-3-2013|103|25|2|2013|"Nestlé Babyfood NaturNes packaging - plastic pot The good assessed:\" Packaging systems used to provide one baby food meal in France, Spain, and Germany\". The 200-g packaging size was selected as the basis for this study. (The total emissions in kg CO2-e per unit provided [in this report] is the figure for France as it is not possible to list all three data in one cell. Please find detail by country and life cycle stages in questiom SM3.2.b"|0.02|0|44.02|7.61|48.37|1|
|12942-4-2013|103|25|2|2013|"Nestlé Babyfood NaturNes packaging - plastic pot The good assessed:\" Packaging systems used to provide one baby food meal in France, Spain, and Germany\". The 200-g packaging size was selected as the basis for this study. (The total emissions in kg CO2-e per unit provided [in this report] is the figure for France as it is not possible to list all three data in one cell. Please find detail by country and life cycle stages in questiom SM3.2.b"|0.02|0|56.64|13.29|30.07|1|
|12942-5-2013|103|25|2|2013|"Nestlé Babyfood NaturNes packaging - plastic pot The good assessed:\" Packaging systems used to provide one baby food meal in France, Spain, and Germany\". The 200-g packaging size was selected as the basis for this study. (The total emissions in kg CO2-e per unit provided [in this report] is the figure for France as it is not possible to list all three data in one cell. Please find detail by country and life cycle stages in questiom SM3.2.b"|0.02|0|64.80|11.20|24.00|1|
|13360-1-2013|104|8|25|2013|Average mobile phone|0.218|12|54.17|4.17|41.67|1|
|13360-1-2014|104|8|25|2014|Mobile phone|0.218|12|54.17|4.17|41.67|1|
|13360-2-2013|104|8|25|2013|"Nokia Lumia 800. Nokia's high end smart phone, mass = 142 g including battery."|0.142|16|65.43|3.70|30.86|1|
|13360-3-2013|104|8|25|2013|"Nokia Asha 300. Nokia's low end device, mass = 85 g incluging battery."|0.085|8|37.83|7.09|55.08|1|
|13889-1-2014|105|16|25|2014|Blood Pressure Monitor HEM-7113|0.5448|28|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-10-2017|105|16|25|2017|ECU|1.06|8|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-11-2017|105|16|25|2017|Laser radar|1.34|86|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-12-2017|105|16|25|2017|Power window switch|0.45|9|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-13-2017|105|16|25|2017|Safety controller|0.42|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-14-2017|105|16|25|2017|Switch|0.25|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-2-2014|105|16|25|2014|Keyless Entry System for Automobiles(Transmitter + Receiver unit)|0.112|9|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-4-2014|105|16|25|2014|Signal Relay Type G6Zk-1F-TR|0.0028|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-5-2017|105|16|25|2017|Automotive Relay|0.034|3|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-6-2017|105|16|25|2017|Automotive Relay|0.034|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-7-2017|105|16|25|2017|Automotive Relay|0.034|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-8-2017|105|16|25|2017|Blood Pressure Monitor|0.25|28|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|13889-9-2017|105|16|25|2017|ECU|1.06|8|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14021-1-2013|19|16|30|2013|City gas|1.062|61|12.51|2.69|84.80|1|
|14021-1-2015|19|16|17|2015|City gas|1.062|61|12.51|2.69|84.80|1|
|14021-1-2016|19|16|30|2016|City gas|1.062|61|12.34|1.19|86.48|1|
|14605-1-2013|20|28|2|2013|Walkers Crisps 32.5 g sold in the United Kingdom|0.0325|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-1-2014|20|28|2|2014|Walkers Crisps 32.5 g sold in the United Kingdom|0.0325|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-1-2015|20|28|15|2015|Walkers Crisps 32.5 g sold in the United Kingdom|0.0325|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-2-2013|20|28|2|2013|Walkers Crisps 50g bag sold in the United Kingdom|0.05|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-2-2014|20|28|2|2014|Walkers Crisps 50g bag sold in the United Kingdom|0.05|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-2-2015|20|28|15|2015|Walkers Crisps 50g bag sold in the United Kingdom|0.05|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-3-2013|20|28|2|2013|Quaker Oats 1Kg box sold in the United Kingdom|0.04|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-3-2014|20|28|2|2014|Quaker Oats 1Kg box sold in the United Kingdom|0.04|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-3-2015|20|28|15|2015|Quaker Oats 1Kg box sold in the United Kingdom|0.04|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-4-2013|20|28|2|2013|Quaker Oats So Simple Original and Golden Syrup products|0.324|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-4-2014|20|28|2|2014|Quaker Oats So Simple Original and Golden Syrup products|0.324|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14605-4-2015|20|28|15|2015|Quaker Oats So Simple Original and Golden Syrup products|0.324|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|14697-1-2013|109|26|25|2013|HH DVD|0.0907185|26|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|149-1-2013|28|26|25|2013|power supply unit (MF6004-030L)|1.8|244|43.69|46.44|9.87|1|
|15391-1-2013|114|6|25|2013|CD-ROM|0.0907185|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15391-1-2014|114|6|25|2014|CD-ROM|0.0907185|12|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15398-1-2013|112|26|25|2013|"19” LCD Monitor  Weight: 2.4Kg  Diagonal size: 19\"  Power consumption: <16.25W (on mode), <0.72W (power saving mode), <0.53W (off mode) Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide"|2.4|70|98.40|1.29|0.31|1|
|15398-10-2014|112|26|25|2014|LED Lighting|6.3|33|98.38|1.38|0.24|1|
|15398-11-2014|112|26|25|2014|Projector|2.8|89|87.70|12.06|0.24|1|
|15398-12-2014|112|26|25|2014|Mobile Phone|0.31|19|88.18|11.71|0.10|1|
|15398-13-2015|112|26|25|2015|Monitor|7.4|66|97.21|2.24|0.56|1|
|15398-13-2016|112|26|25|2016|Monitor|5.8|75|97.06|1.56|1.38|1|
|15398-14-2015|112|26|25|2015|Projector|3.4|83|98.41|1.27|0.31|1|
|15398-14-2016|112|26|25|2016|Projector|4.4|100|97.26|2.41|0.33|1|
|15398-15-2015|112|26|25|2015|Mobile Phone|0.35|53|99.40|0.06|0.54|1|
|15398-15-2016|112|26|25|2016|Mobile Phone|0.35|16|99.38|0.43|0.19|1|
|15398-16-2015|112|26|25|2015|LED Lighting|8.2|69|82.96|16.88|0.16|1|
|15398-16-2016|112|26|25|2016|LED Lighting|5.8|24|90.10|9.57|0.33|1|
|15398-17-2015|112|26|25|2015|Industrial Solutions|14.2|146|96.88|2.70|0.42|1|
|15398-18-2016|112|26|25|2016|Industrial Solutions|7.0|66|89.48|10.02|0.50|1|
|15398-2-2013|112|26|25|2013|Projector  Weight: 7.5Kg Lamp: 24 pcs laser with phosphor Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide.|7.5|183|99.17|0.52|0.31|1|
|15398-3-2013|112|26|25|2013|Mobile phone Weight: 0.11kg Screen Diagonal: 3.7\ WVGA (800*480) 16M Color TFT-LCD Power Consumption: Standby time(1300m Ah): Up to 4 hrs depends on real-network Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide."|0.11|19|69.15|30.81|0.04|1|
|15398-4-2013|112|26|25|2013|Digital Photo Frame Weight: 0.8 kg Diagonal size: 7D Overall Dimension: 238*202*39.7mm Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide.|0.8|26|98.43|1.33|0.24|1|
|15398-5-2013|112|26|25|2013|Scanner  Weight: 1.4 kg Power supply: 12V/1.5A Excluded from the study of carbon footprint of product were CD、Manual and Quick Start Guide.|1.4|21|95.47|4.52|0.01|1|
|15398-6-2014|112|26|25|2014|Monitor|11.0|106|96.47|3.01|0.52|1|
|15398-7-2014|112|26|25|2014|42' Industrial Solutions|23.0|174|96.89|1.71|1.39|1|
|15398-8-2014|112|26|25|2014|Mobile Infotainment|0.7|26|97.20|2.60|0.19|1|
|15398-9-2014|112|26|25|2014|Surveillance Camera & Products|0.162|6|90.38|8.81|0.82|1|
|15673-1-2013|40|4|26|2013|BlackBerry Bold 9900|0.13|52|44.18|16.85|38.96|1|
|15673-2-2014|40|4|25|2014|BlackBerry Z10|0.1375|57|45.69|17.43|36.88|1|
|15673-3-2015|40|4|25|2015|BlackBerry Passport|0.205|65|43.73|16.68|39.59|1|
|15673-3-2016|40|4|25|2016|BlackBerry Passport|0.205|65|43.73|16.68|39.59|1|
|15673-4-2016|40|4|25|2016|BlackBerry Classic|0.178|63|44.03|16.79|39.17|1|
|15673-5-2016|40|4|25|2016|BlackBerry Priv|0.192|80|45.13|17.21|37.66|1|
|15763-1-2013|21|16|25|2013|"Ricoh Aficio MP 6002SP, a Mono-chrome MFP (Multi-function products) products that function as Copier, Network Printer, Scanner, (Facsimile)."|214.0|2400|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15763-10-2017|21|16|25|2017|Ricoh SP C352DN|33.0|1640|7.73|2.12|90.15|1|
|15763-2-2013|21|16|25|2013|"Ricoh Aficio MP C5502G, a Full-color MFP (Multi-function products) products that function as Copier, Network Printer, Scanner, (Facsimile)."|130.0|2100|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15763-3-2013|21|16|25|2013|"Ricoh Aficio SP 8300DN, a high-speed Mono-chrome Network Printer"|73.0|1297|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15763-4-2013|21|16|25|2013|"Ricoh Aficio SP C831DN, a high-speed Full-colour Network Printer"|97.0|3210|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15763-5-2013|21|16|25|2013|"Ricoh Pro C751EX, a Production Printer (Full Color) that is used primarily for high-speed and production type (large print volume) job"|550.0|7368|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15763-6-2014|21|16|25|2014|Ricoh MP C5503|99.9|1940|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15763-7-2014|21|16|25|2014|Ricoh MP C8002SP: The product is a high-speed (80 images/minute) color multi-functional product (MFP) and is a major representative of our image solution products business.  The lifecycle emissions data of this product is provided as a case example of our entire image solution business products line which covers 77% of Ricoh's total revenue.|241.0|6140|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15763-8-2015|21|16|25|2015|Ricoh MP 6054SPG|74.0|1330|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|15763-9-2015|21|16|25|2015|RICOH Pro C7110S QX100|600.0|8080|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|16290-1-2013|119|21|19|2013|SBS Board|1000.0|968|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|16290-1-2014|119|21|19|2014|AlgroDesign|1000.0|1032|30.17|69.83|0.00|1|
|16290-2-2013|119|21|19|2013|One side coated label paper|1000.0|949|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|16290-2-2014|119|21|19|2014|Parade Prima|1000.0|1043|30.92|69.08|0.00|1|
|16290-3-2013|119|21|19|2013|Uncoated flexible packaging paper|1000.0|1033|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|16290-3-2014|119|21|19|2014|"Leine Mühle, Leine Silk"|1000.0|1061|32.10|67.90|0.00|1|
|16290-4-2013|119|21|19|2013|One side coated flexible packaging paper|1000.0|973|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|16290-4-2014|119|21|19|2014|Algro Helix|1000.0|1083|33.46|66.54|0.00|1|
|16423-1-2013|120|9|8|2013|Acti9 – Residual current circuit breaker – iID K. The main function of the iID K product range is to ensure protection of people against electric shocks. PEP of this product is available here: http://planet.schneider-electric.com/C12577A200268439/all/14AA652B755F340C852577F3006F06D0/$File/envpep100201en.pdf|0.21|19|6.68|1.44|91.88|1|
|16423-1-2014|120|9|8|2014|ACTI9 IID K 2P 40A 30MA AC-TYPE RESIDUAL CURRENT CIRCUIT BREAKER|0.21|19|6.68|1.44|91.88|1|
|16423-1-2015|120|9|13|2015|ACTI9 IID K 2P 40A 30MA AC-TYPE RESIDUAL CURRENT CIRCUIT BREAKER|0.21|19|6.68|1.44|91.88|1|
|16423-2-2016|120|9|8|2016|ACTI9 IID K 2P 40A 30MA AC-TYPE RESIDUAL CURRENT CIRCUIT BREAKER|0.21|61|1.92|0.41|97.66|1|
|17280-1-2013|124|14|19|2013|Corrugated box|1000.0|746|87.01|12.99|0.00|1|
|17280-1-2014|124|14|19|2014|Corrugated Box|1000.0|746|87.01|12.99|0.00|1|
|17280-1-2015|124|14|12|2015|Corrugated box|1000.0|746|87.01|12.99|0.00|1|
|17280-2-2013|124|14|19|2013|Board packaging|1000.0|964|87.01|12.99|0.00|1|
|17280-2-2014|124|14|19|2014|Solid Board|1000.0|964|87.01|12.99|0.00|1|
|17280-2-2015|124|14|12|2015|Solid Board|1000.0|964|87.01|12.99|0.00|1|
|17296-1-2014|125|2|19|2014|Sodium Carbonate|1.0|1|1.52|98.48|0.00|1|
|17296-2-2014|125|2|19|2014|Sodium Bicarbonate|1.0|1|0.91|99.09|0.00|1|
|17296-2-2015|125|2|9|2015|Sodium Bicarbonate|1.0|1|9.90|90.10|0.00|1|
|17296-2-2016|125|2|19|2016|Sodium Bicarbonate|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17296-3-2015|125|2|9|2015|Dense Sodium Carbonate|1.0|1|10.85|89.15|0.00|1|
|17296-3-2016|125|2|19|2016|Dense Sodium Carbonate|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17296-4-2015|125|2|9|2015|Light Sodium Carbonate|1.0|1|10.92|89.08|0.00|1|
|17296-4-2016|125|2|19|2016|Light Sodium Carbonate|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17659-1-2014|24|28|21|2014|Staples Sustainable Earth 12A toner cartridge|0.8172|3|71.85|7.62|20.53|1|
|17659-1-2015|24|28|21|2015|Staples Sustainable Earth 12A toner cartridge|0.8172|3|71.85|7.62|20.53|1|
|17659-2-2014|24|28|21|2014|Staples Sustainable Earth 38A toner cartridge|0.8172|8|78.45|5.26|16.29|1|
|17659-2-2015|24|28|21|2015|Staples Sustainable Earth 38A toner cartridge|0.8172|8|78.45|5.26|16.29|1|
|17673-1-2015|23|28|13|2015|D21710 Type 5 Corded Power Drill|1.82|56|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17673-1-2016|23|28|8|2016|D21710 Type 5 Corded Power Drill|1.82|56|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17673-2-2015|23|28|13|2015|DC988 Type 11 Cordless Power Drill equipment with rechargeable NiCd batteries|2.7|88|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17673-2-2016|23|28|8|2016|Cordless Power Drill |2.7|88|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17673-3-2016|23|28|8|2016|33-725 Tape measure family|0.5|2|78.19|16.84|4.97|1|
|17788-1-2013|126|28|10|2013|Let’s B task chair|19.773|120|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-1-2014|126|28|10|2014|Let’s B task chair|19.773|120|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-10-2016|126|28|10|2016|B-Free small cube (European supply chain)|13.05|23|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-11-2016|126|28|10|2016|B-Free Table (European supply chain)|51.7|150|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-12-2016|126|28|10|2016|Eastside (European supply chain)|6.165|29|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-13-2016|126|28|10|2016|Flip Top Twin (European supply chain)|32.916|94|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-16-2016|126|28|10|2016|FrameOne Bench (European supply chain)|154.804|380|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-17-2016|126|28|10|2016|Fusion (European supply chain)|35.331|72|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-18-2016|126|28|10|2016|Gesture (European supply chain)|35.38|170|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-19-2016|126|28|10|2016|Gesture (North American supply chain)|35.38|180|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-2-2013|126|28|10|2013|"The Activa desks are highly ergonomic and offer different height adjustable versions in a single design. Its modular construction allows very easy assembly and reconfiguration.  The model chosen for analysis is the most frequently ordered one (reference W6412700) from the Activa Telescopic range. Standard features on this model include: - Top dimensions: 1600 mm x 800 mm - Melamine top, Snow WY - Steel leg and frame, Pearl Snow ZW - Telescopic height adjustment from 620 mm to 900 mm - Steel cable tray."|44.706|160|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-2-2014|126|28|10|2014|Activa desk|44.706|160|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-20-2016|126|28|10|2016|Implicit (European supply chain)|35.298|98|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-21-2016|126|28|10|2016|Kalidro (European supply chain)|37.0|52|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-22-2016|126|28|10|2016|Leap (European supply chain)|26.183|88|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-23-2016|126|28|10|2016|Let's B (European supply chain)|25.0|120|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-24-2016|126|28|10|2016|Movida (European supply chain)|38.337|40|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-25-2016|126|28|10|2016|New Think (European supply chain)|31.0|110|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-26-2016|126|28|10|2016|Ology (European supply chain)|40.9|95|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-27-2016|126|28|10|2016|Partito (European supply chain)|14.963|60|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-28-2016|126|28|10|2016|Please (European supply chain)|23.623|91|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-29-2016|126|28|10|2016|Qivi (European supply chain)|5.17|56|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-3-2013|126|28|10|2013|"Share It is a modular storage system. It offers personal storage, team storage, meeting point solutions and lockers.  Share It can be also used as space dividers, structuring workspaces. - It is modular and offers endless planning possibilities. - The range enhances collaboration providing communication platforms. - It helps people concentrate thanks to acoustics absorbing surfaces. - It offers wide range of finishes for different workplace ambiances."|52.221|111|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-3-2014|126|28|10|2014|Share It modular storage system|52.221|111|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-30-2016|126|28|10|2016|Reply (European supply chain)|17.7|101|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-32-2016|126|28|10|2016|Tenaro (European supply chain)|67.323|53|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-33-2016|126|28|10|2016|TNT* (European supply chain)|47.334|50|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-34-2016|126|28|10|2016|Universal Storage EU (European supply chain)|69.78|104|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-36-2016|126|28|10|2016|Westside (European supply chain)|5.057|19|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-4-2013|126|28|10|2013|The model chosen for analysis is the most popular model Think work chair.  It is a highly adjustable ergonomic chair equipped as follows: 1. Your PowerTM weight activated mechanism 2. Your ProfileTM seat and back flexors  3. Your PreferenceTM control  4. Adjustable seat depth 5. Adjustable seat height 6. Adjustable lumbar support 7. Adjustable armrests 8. Plastic base|17.252|86|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-4-2014|126|28|10|2014|Think work chair|17.252|86|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-5-2016|126|28|10|2016|32 Seconds (European supply chain)|21.77|100|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-6-2016|126|28|10|2016|Activa (European supply chain)|44.706|160|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-7-2016|126|28|10|2016|Amia (European supply chain)|13.01|111|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-8-2016|126|28|10|2016|B-Free big cube (European supply chain)|34.8|110|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|17788-9-2016|126|28|10|2016|B-Free Desk (European supply chain)|42.1|100|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18141-1-2015|129|25|9|2015|Fungicide|1.0|3|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18141-2-2015|129|25|9|2015|Herbicide|1.0|4|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18141-3-2015|129|25|9|2015|Insecticide|1.0|5|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18141-4-2015|129|25|9|2015|Seed treatment|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-1-2013|130|12|19|2013|Soda Ash for Unilever plc|1000.0|1282|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-1-2014|130|12|19|2014|Soda Ash|1000.0|1057|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-1-2015|130|12|9|2015|Soda Ash|1000.0|1006|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-1-2017|130|12|19|2017|Soda Ash|1000.0|870|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-2-2013|130|12|19|2013|"Soda Ash for Wal-Mart Stores, Inc."|1000.0|701|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-3-2014|130|12|19|2014|Soda Ash from Tata Chemicals Europe|1000.0|840|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-3-2015|130|12|9|2015|Soda Ash (Tata Chemicals Europe)|1000.0|764|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-4-2015|130|12|9|2015|Soda Ash (Tata Chemicals Magadi)|1000.0|504|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-5-2015|130|12|9|2015|Soda Ash (Tata Chemicals North America)|1000.0|701|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18288-6-2015|130|12|9|2015|"Soda Ash (Tata Chemicals Mithapur, India)"|1000.0|1083|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|18344-1-2014|131|12|19|2014|Hot Rolled Steel|1000.0|2425|25.35|70.60|4.05|1|
|18344-1-2015|131|12|4|2015|Hot Rolled Steel (HR)|1000.0|2615|23.36|65.05|11.59|1|
|18344-2-2014|131|12|19|2014|Cold Rolled Steel|1000.0|2549|25.35|70.60|4.05|1|
|18344-2-2015|131|12|4|2015|Cold Rolled Steel (CR)|1000.0|2746|23.35|65.04|11.61|1|
|18344-3-2014|131|12|19|2014|Galvanised Steel|1000.0|2611|25.35|70.59|4.06|1|
|18344-3-2015|131|12|4|2015|Galvanized Steel (Galv)|1000.0|2820|23.36|65.06|11.58|1|
|18585-1-2013|135|24|19|2013|Tetra Brik® Aseptic Slim 200ml|0.009|0|88.24|11.76|0.00|1|
|18585-1-2014|135|24|19|2014|Tetra Brik® Aseptic Slim 200ml|0.009|0|88.24|11.76|0.00|1|
|18585-1-2015|135|24|12|2015|Tetra Brik® Aseptic Slim 200ml|0.009|0|88.24|11.76|0.00|1|
|18585-1-2016|135|24|19|2016|Tetra Brik® Aseptic Slim 200ml|0.009|0|81.25|18.75|0.00|1|
|18585-2-2013|135|24|19|2013|Tetra Brik® Aseptic Slim 250ml|0.01|0|88.89|11.11|0.00|1|
|18585-2-2014|135|24|19|2014|Tetra Brik® Aseptic Slim 250ml|0.01|0|88.89|11.11|0.00|1|
|18585-2-2015|135|24|12|2015|Tetra Brik® Aseptic Slim 250ml|0.01|0|88.89|11.11|0.00|1|
|18585-2-2016|135|24|19|2016|Tetra Brik® Aseptic Slim 250ml|0.01|0|83.33|16.67|0.00|1|
|18585-3-2013|135|24|19|2013|Tetra Brik® Aseptic Base 250ml|0.01|0|88.89|11.11|0.00|1|
|18585-3-2014|135|24|19|2014|Tetra Brik® Aseptic Base 250ml|0.01|0|88.89|11.11|0.00|1|
|18585-3-2015|135|24|12|2015|Tetra Brik® Aseptic Base 250ml|0.01|0|88.89|11.11|0.00|1|
|18585-3-2016|135|24|19|2016|Tetra Brik® Aseptic Base 250ml|0.01|0|82.35|17.65|0.00|1|
|18585-4-2013|135|24|19|2013|Tetra Brik® Aseptic Base 1000ml|0.028|0|86.96|13.04|0.00|1|
|18585-4-2014|135|24|19|2014|Tetra Brik® Aseptic Base 1000ml|0.028|0|86.96|13.04|0.00|1|
|18585-4-2015|135|24|12|2015|Tetra Brik® Aseptic Base 1000ml|0.028|0|86.96|13.04|0.00|1|
|18585-4-2016|135|24|19|2016|Tetra Brik® Aseptic Base 1000ml|0.028|0|83.33|16.67|0.00|1|
|1884-1-2013|41|28|20|2013|"Bloomberg's standard-issue flat panel configuration (prior to 2010) was two 19\" panels mounted on a metal stand. In early 2010 Bloomberg engaged in the WRI Product Life Cycle Roadtest for this functional unit (cradle-to-grave). The functional unit has a lifespan of 5 years, so the emissions indicated [in this report] are the full emissions associated with that lifespan."|14.6|9438|64.62|0.57|34.81|1|
|1884-1-2014|41|28|20|2014|Bloomberg-Provided Flat Panel Unit|14.6|9438|64.62|0.57|34.81|1|
|1884-1-2015|41|28|20|2015|Bloomberg-Provided Flat Panel Unit|14.6|1766|19.53|1.02|79.46|1|
|1884-1-2016|41|28|20|2016|Bloomberg-Provided Flat Panel Unit|14.6|1766|19.53|1.02|79.46|1|
|1884-2-2013|41|28|20|2013|"Bloomberg's standard keyboard which is optional hardware available for lease to all Bloomberg Terminal customers.  The functional unit has a lifespan of 3 years, so the emissions indicated [in this report] are the full emissions associated with that lifespan."|1.08|94|33.96|11.12|54.92|1|
|1884-2-2014|41|28|20|2014|Bloomberg-Provided Keyboard|1.08|94|33.96|11.12|54.92|1|
|1884-2-2015|41|28|20|2015|Bloomberg-Provided Keyboard|1.08|40|56.43|4.31|39.26|1|
|1884-2-2016|41|28|20|2016|Bloomberg-Provided Keyboard|1.08|40|56.43|4.31|39.26|1|
|1884-3-2013|41|28|20|2013|"Bloomberg's biometric device (B-Unit) distributed to all Bloomberg Terminal customers.  The functional unit has a lifespan of 3 years, so the emissions indicated [in this report] are the full emissions associated with that lifespan."|0.029|2|84.33|6.91|8.76|1|
|1884-3-2014|41|28|20|2014|Bloomberg B-Unit|0.029|2|84.33|6.91|8.76|1|
|1884-3-2015|41|28|20|2015|Bloomberg B-Unit|0.029|2|78.39|15.58|6.03|1|
|1884-3-2016|41|28|20|2016|Bloomberg B-Unit|0.029|2|78.39|15.58|6.03|1|
|19100-1-2013|133|9|20|2013|"TG582n Home Gateway to provide multiple play telecom data services (data, voice, TV) in the home."|0.599|111|4.19|1.60|94.21|1|
|19100-1-2014|133|9|20|2014|Home Gateway TG582n|0.599|111|4.19|1.60|94.21|1|
|19100-1-2015|133|9|20|2015|Home Gateway TG582n|0.599|111|4.19|1.60|94.21|1|
|19238-3-2017|25|16|10|2017|GL film packages|13.44|378|70.15|27.65|2.20|1|
|19238-4-2017|25|16|10|2017|MR-PET film package|13.44|363|68.94|31.06|0.00|1|
|19429-1-2016|137|24|8|2016|"Agricultural tires, BlueTire Yechnology"|370.69|1409|63.92|32.99|3.09|1|
|20033-1-2013|115|27|8|2013|Polyethylene pipe|1.0|3|91.39|7.94|0.67|1|
|20309-1-2013|139|10|7|2013|VW Polo V 1.6 TDI BlueMotion Technology|993.0|21725|18.63|5.75|75.62|1|
|20309-1-2014|139|10|7|2014|VW Polo|993.0|21725|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-1-2015|139|10|7|2015|Volkswagen Polo|993.0|21725|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-1-2016|139|10|7|2016|Volkswagen Polo|993.0|21725|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-2-2013|139|10|7|2013|VW Golf VII 1.6 TDI BlueMotion Technology|1358.056|22218|19.52|6.02|74.46|1|
|20309-2-2014|139|10|7|2014|VW Golf|1394.343|22218|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-2-2015|139|10|7|2015|Volkswagen Golf|1394.343|22218|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-2-2016|139|10|7|2016|Volkswagen Golf|1358.056|22218|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-3-2013|139|10|7|2013|Passat B7 2.0 TDI BlueMotion Technology|1539.039|28011|20.54|6.33|73.12|1|
|20309-3-2014|139|10|7|2014|VW Passat|1539.039|28011|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-3-2015|139|10|7|2015|Volkswagen Passat|1539.039|28011|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-3-2016|139|10|7|2016|Volkswagen Passat|1539.039|28011|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-4-2013|139|10|7|2013|Audi A3 2.0 TDI|1460.114|30647|15.68|4.84|79.48|1|
|20309-5-2014|139|10|7|2014|Mean value for a vehicle within the Volkswagen Group according to CDP Report 2013 (global scope)|1539.039|34694|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-5-2015|139|10|7|2015|Mean value for a vehicle within the Volkswagen Group according to CDP Report 2014 (global scope)|1539.039|33683|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-6-2014|139|10|7|2014|VW up!|929.8644|21488|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-6-2015|139|10|7|2015|Volkswagen up!|929.8644|21488|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-6-2016|139|10|7|2016|Volkswagen up!|929.8644|21488|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-7-2014|139|10|7|2014|Audi A6|1895.109|37094|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-7-2015|139|10|7|2015|Audi A6|1895.109|37094|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-7-2016|139|10|7|2016|Audi A6|1895.109|37094|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-8-2014|139|10|7|2014|VW Caddy|1128.0|28608|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-8-2015|139|10|7|2015|Volkswagen Caddy|1128.0|28608|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-8-2016|139|10|7|2016|Volkswagen Caddy|1128.0|28608|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20309-9-2016|139|10|7|2016|Volkswagen e-up|1185.0|7550|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20523-1-2013|140|28|5|2013|Alliance HPLC (High Peformance Liquid Chromatography)  The Alliance is an HPLC that is unique in that it has a single set of electronic boards that control the functions for both the solvent delivery system and the autosampler in the liquid chromatograph.|59.0|15197|2.95|0.64|96.41|1|
|20523-1-2014|140|28|5|2014|Alliance (HPLC)|59.0|40215|1.41|0.30|98.28|1|
|20523-2-2013|140|28|5|2013|"ACQUITY® UPLC, ACQUITY® I-Class, ACQUITY® H-Class"|25.9|17074|4.07|0.88|95.05|1|
|20527-1-2013|141|3|8|2013|Electric Motor|90.0|53058|0.01|0.00|99.99|1|
|20527-1-2014|141|3|8|2014|Electric Motor|90.0|87589|0.02|0.00|99.98|1|
|20527-1-2015|141|3|13|2015|Electric Motor|90.0|20008|0.10|0.02|99.88|1|
|20822-1-2013|144|28|25|2013|Phaser 6600|25.5|584|3.60|67.29|29.11|1|
|20822-1-2014|144|28|25|2014|Phaser 6600|25.5|584|3.60|67.29|29.11|1|
|20822-1-2015|144|28|24|2015|Phaser 6600|25.5|584|3.60|67.29|29.11|1|
|20822-1-2016|144|28|24|2016|Phaser 6600|25.5|584|3.60|67.29|29.11|1|
|20822-2-2013|144|28|25|2013|ColorQube 8900|46.308|2445|1.60|27.72|70.69|1|
|20822-2-2014|144|28|25|2014|ColorQube 8900|46.308|2445|1.60|27.72|70.69|1|
|20822-2-2015|144|28|24|2015|ColorQube 8900|46.308|2445|1.60|27.72|70.69|1|
|20822-2-2016|144|28|24|2016|ColorQube 8900|46.308|2445|1.60|27.72|70.69|1|
|20822-3-2013|144|28|25|2013|ColorQube 9200|368.8|7090|1.26|32.44|66.29|1|
|20822-3-2014|144|28|25|2014|ColorQube 9200|368.8|7090|1.27|32.57|66.16|1|
|20822-3-2015|144|28|24|2015|ColorQube 9200|368.8|7090|1.27|32.57|66.16|1|
|20822-3-2016|144|28|24|2016|ColorQube 9200|368.8|7090|1.27|32.57|66.16|1|
|20822-4-2013|144|28|25|2013|ColorQube 8870|34.0194|2110|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20822-4-2014|144|28|25|2014|ColorQube 8870|34.0194|2011|3.88|9.10|87.02|1|
|20822-4-2015|144|28|24|2015|ColorQube 8870|34.0194|2011|3.88|9.10|87.02|1|
|20822-4-2016|144|28|24|2016|ColorQube 8870|34.0194|2011|3.88|9.10|87.02|1|
|20822-5-2014|144|28|25|2014|Phaser 7100|44.0|2000|13.31|2.12|84.57|1|
|20822-5-2015|144|28|24|2015|Phaser 7100|44.0|2000|13.31|2.12|84.57|1|
|20822-5-2016|144|28|24|2016|Phaser 7100|44.0|2000|13.31|2.12|84.57|1|
|20822-6-2015|144|28|24|2015|WorkCentre 5945/5955|158.5|4216|64.27|0.00|35.73|1|
|20822-6-2016|144|28|24|2016|WorkCentre 5945/5955|158.5|4216|64.29|0.00|35.71|1|
|20822-7-2015|144|28|24|2015|WorkCentre 5325|137.2|1300|47.58|2.61|49.81|1|
|20822-7-2016|144|28|24|2016|WorkCentre 5325|137.2|1300|47.58|2.61|49.81|1|
|20822-8-2015|144|28|24|2015|WorkCentre 5330|137.2|1500|41.53|2.28|56.20|1|
|20822-8-2016|144|28|24|2016|WorkCentre 5330|137.2|1500|41.53|2.28|56.20|1|
|20822-9-2015|144|28|24|2015|WorkCentre 5335|137.2|1700|37.51|2.06|60.44|1|
|20822-9-2016|144|28|24|2016|WorkCentre 5335|137.2|1700|37.51|2.06|60.44|1|
|20871-1-2015|37|16|25|2015|Pressure transmitter (圧力発信器) GTX|3.6|42|54.23|20.36|25.41|1|
|20871-2-2015|37|16|25|2015|Photoelectric sensor (光電センサー)|0.11|14|1.70|0.13|98.17|1|
|20871-5-2015|37|16|25|2015|Neosensor indoor temperature sensor (ネオセンサ 室内用温度センサ)|0.11|3|96.00|2.80|1.20|1|
|20871-6-2015|37|16|25|2015|Temperature sensor for duct (ダクト用温度センサ)|0.24|8|99.81|0.00|0.19|1|
|20947-1-2014|26|16|7|2014|1 fuel-efficient tire for passenger cars (乗用車用低燃費タイヤ)|8.7|257|9.36|2.73|87.91|1|
|20947-1-2015|26|16|27|2015|1 fuel-efficient tire for passenger cars (乗用車用低燃費タイヤ)|8.7|288|13.77|4.00|82.24|1|
|20947-2-2014|26|16|7|2014|1 fuel-efficient truck and buss tireトラック・バス用低燃費タイヤ１本当たり|160.0|1881|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|20947-2-2015|26|16|27|2015|1 fuel-efficient truck and buss tireトラック・バス用低燃費タイヤ１本当たり|160.0|1734|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2116-1-2015|43|1|29|2015|CHEP wooden exchange (pooled) pallet in USA 48x40|31.8|41|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2116-2-2015|43|1|29|2015|CHEP wooden exchange (pooled) pallet in Canada 48x40|31.8|35|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2116-3-2015|43|1|29|2015|Foldable reusable plastic crates (RPCs) in Australia|2.5|20|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2116-5-2015|43|1|29|2015|CHEP wooden exchange (pooled) pallet in Canada 48x20|15.9|19|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2116-6-2015|43|1|29|2015|IFCO reusable plastic crates (RPCs) in North America|2.4062|90|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2116-8-2015|43|1|29|2015|Rigid side reusable plastic crates (RPCs) in Australia|2.4062|34|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2135-1-2013|44|3|19|2013|"The Polyethylene (PE), molecular formula (CH2)n, is currently the world's main plastic material. Its properties depend on the shape of the polyethylene — given that the ones of low density are more flexible while the high density polyethylenes are more rigid. This material is produced from the polymerization of the monomer ethylene. Despite the differences between the types of polyethylene produced by Braskem, this assessment was targeted at finding an average value of emissions for all of the resins produced. The figure provide in column \"Total emissions in kg CO2e per unit\" has the dimension of tCO2e/ ton product."|1.0|1|82.40|17.60|0.00|1|
|2135-1-2014|44|3|19|2014|Polyethylene|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2135-1-2015|44|3|9|2015|Polyethylene|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2135-1-2016|44|3|19|2016|Polyethylene|1000.0|1320|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2135-2-2013|44|3|19|2013|"Polypropylene (PP) or Polypropene, which has the molecular formula of (CH (CH3) - CH2)n is a resin produced from the polymerization of monomer Propene, its main feedstock. It may also be produced from a mixture of two types of monomers (ethylene and propylene), producing the PP Copolymer. This is the carbon footprint for the Polypropylene produced in Brazil. The figure provide in column \"Total emissions in kg CO2e per unit\" has the dimension of tCO2e/ ton product."|1.0|1|81.69|18.31|0.00|1|
|2135-2-2014|44|3|19|2014|Polypropylene|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2135-2-2015|44|3|9|2015|Polypropylene|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2135-2-2016|44|3|19|2016|Polypropylene|1000.0|1330|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2135-3-2013|44|3|19|2013|"The Polyvinyl Chloride (PVC; -(CH2-CHCl)n-) is the oldest plastic material. This material is produced from the polymerization of the monomer MVC. Braskem has a chlorine production unit, base material, which shall be used for the production of DCE (1,2-dichloroethane) from the mixture of chloride and ethylene. Afterwards, the DCE reacts to form MVC, the monomer for the polymerization process. The figure provide in column \"Total emissions in kg CO2e per unit\" has the dimension of tCO2e/ ton product."|1.0|2|44.07|13.56|42.37|1|
|2135-3-2014|44|3|19|2014|Polyvinyl Chloride|1.0|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2135-3-2015|44|3|9|2015|Polyvinyl Chloride|1.0|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2135-3-2016|44|3|19|2016|Polyvinyl Chloride|1000.0|1770|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21394-1-2013|142|26|25|2013|Acer 15.6\ Notebook"|2.35868|9|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-1-2015|113|26|25|2015|Note book|1000.0|1330|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-10-2015|113|26|25|2015|LCD PC|1000.0|1600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-11-2015|113|26|25|2015|PND|1000.0|10202|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-12-2015|113|26|25|2015|Auto|1000.0|10202|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-13-2015|113|26|25|2015|Watch|1000.0|10202|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-14-2015|113|26|25|2015|Camera|1000.0|10202|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-14-2016|113|26|25|2016|Camera|0.25|5|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-15-2016|113|26|25|2016|AIO|8.05|5|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-16-2016|113|26|25|2016|AIO|8.05|5|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-17-2016|113|26|25|2016|Notebook|2.95|5|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-2-2015|113|26|25|2015|Servers|1000.0|3248|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-20-2016|113|26|25|2016|Server|19.97|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-21-2016|113|26|25|2016|Tablet|0.35|5|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-22-2016|113|26|25|2016|VR|0.35|4|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-3-2015|113|26|25|2015|DT|1000.0|1600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-4-2015|113|26|25|2015|Phone|1000.0|1600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-5-2015|113|26|25|2015|Pad|1000.0|1600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-6-2015|113|26|25|2015|CHROMEBOX|1000.0|1600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-7-2015|113|26|25|2015|Lamp|1000.0|1600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-8-2015|113|26|25|2015|POS|1000.0|1600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21477-9-2015|113|26|25|2015|Speaker|1000.0|1600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2156-1-2016|45|16|7|2016|Tire including low fuel consumption tire; e.g. ECOPIA series|9.07185|118|9.19|3.88|86.93|1|
|21673-1-2014|59|22|11|2014|Air Purifier|7.2|126|26.58|0.79|72.63|1|
|21889-1-2013|68|5|19|2013|Folding boxboard produced at the Maule and Valdivia mills.|1000.0|549|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|21889-1-2014|68|5|19|2014|Folding boxboard|1000.0|555|26.42|73.58|0.00|1|
|21889-1-2015|68|5|3|2015|Folding boxboard|1000.0|451|26.42|73.58|0.00|1|
|22295-1-2013|93|8|19|2013|"Simcote is a fully coated folding boxboard with a washed groundwood middle layer, based on 100 % fresh forest fibres. Typical end-uses are food and general packaging."|1000.0|597|21.27|71.19|7.54|1|
|22295-1-2015|93|8|3|2015|Simcote|1000.0|782|15.73|78.01|6.27|1|
|22295-10-2015|93|8|3|2015|Tako CX Lite|1000.0|918|11.00|83.55|5.45|1|
|22295-11-2015|93|8|3|2015|Tako CX Lite S|1000.0|918|11.00|83.55|5.45|1|
|22295-12-2015|93|8|3|2015|Tako CX Lite S2|1000.0|918|11.00|83.55|5.45|1|
|22295-13-2015|93|8|3|2015|Tako CX Lite OBA|1000.0|918|11.00|83.55|5.45|1|
|22295-14-2015|93|8|3|2015|Tako CX White S|1000.0|900|11.22|83.22|5.56|1|
|22295-15-2016|93|8|19|2016|Metsä Board Classic FBB (Simcote)|1000.0|807|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|22295-16-2016|93|8|19|2016|Metsä Board Classic FBB CX (Tako Lite CX))|1000.0|767|25.03|74.97|0.00|1|
|22295-17-2016|93|8|19|2016|Metsä Board Natural FSB Cup - new product|1000.0|369|48.51|51.49|0.00|1|
|22295-18-2016|93|8|19|2016|Metsä Board Prime FBB (Carta Elega)|1000.0|804|22.87|77.13|0.00|1|
|22295-19-2016|93|8|19|2016|Metsä Board Prime FBB Bright (Carta Integra)|1000.0|534|36.70|63.30|0.00|1|
|22295-2-2013|93|8|19|2013|"Carta Elega is a fully coated folding boxboard, GC1, based on 100 % fresh forest fibres. Typical end uses are cosmetics and other luxury packagings and graphical applications."|1000.0|482|26.35|62.45|11.20|1|
|22295-2-2015|93|8|3|2015|Carta Elega|1000.0|804|15.55|77.11|7.34|1|
|22295-20-2016|93|8|19|2016|Metsä Board Prime FBB CX (Tako CX White S)|1000.0|767|28.07|71.93|0.00|1|
|22295-21-2016|93|8|19|2016|Metsä Board Prime FBB CXB (Carta Allura)|1000.0|766|27.32|72.68|0.00|1|
|22295-22-2016|93|8|19|2016|Metsä Board Prime FBB CXL (Tako HiD L JTI)|1000.0|767|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|22295-23-2016|93|8|19|2016|Metsä Board Prime FBB CXX (Tako HiD X JTI)|1000.0|767|28.07|71.93|0.00|1|
|22295-24-2016|93|8|19|2016|Metsä Board Pro FBB (Avanta Prima)|1000.0|811|21.21|78.79|0.00|1|
|22295-25-2016|93|8|19|2016|Metsä Board Pro FBB Bright (Carta Solida)|1000.0|534|36.70|63.30|0.00|1|
|22295-26-2016|93|8|19|2016|MetsäBoard Pro FBB CX (Tako CX Lite S)|1000.0|776|25.90|74.10|0.00|1|
|22295-3-2013|93|8|19|2013|"Avanta Prima is a fully coated folding boxboard, GC2, based on 100 % fresh forest fibress. Typical end uses are high-quality pharmaceuticals, daily cosmetics and food packagings."|1000.0|478|24.43|64.30|11.27|1|
|22295-3-2015|93|8|3|2015|Avanta Prima|1000.0|811|14.06|78.42|7.52|1|
|22295-4-2013|93|8|19|2013|"Carta Solida is a fully coated BCTMP board with a white back, based on 100 % fresh forest fibres. Carta Solida is suitable for packaging and graphical end uses and it is available in reels and sheets."|1000.0|359|42.34|44.01|13.65|1|
|22295-4-2015|93|8|3|2015|Carta Solida|1000.0|534|27.53|63.30|9.18|1|
|22295-5-2013|93|8|19|2013|"Carta Integra is a fully coated BCTMP board with a coated back,  based on 100 % fresh forest fibres. Carta Integra is suitable for graphical and packaging end uses and it is available in reels and sheets."|1000.0|359|42.34|44.01|13.65|1|
|22295-5-2015|93|8|3|2015|Carta Integra|1000.0|534|27.53|63.30|9.18|1|
|22295-6-2013|93|8|19|2013|"Kemiart Brite is a tradional white top kraftliner for retail packages with large colour-printed areas and line pictures with excellent contrast, based on 100 % fresh forest fibres. It is used for corrugated packaging."|1000.0|398|37.69|47.74|14.57|1|
|22295-7-2013|93|8|19|2013|"Batabak 515 is a fully coated bleached paperboard with uncoated back, based on 100 % fresh forest fibres. Typical end use is cigarette packaging."|1000.0|781|17.16|76.31|6.53|1|
|22295-8-2013|93|8|19|2013|"Batabak 580 is a fully coated bleached paperboard with  a coated back, based on 100 % fresh forest fibres. Typical end use is cigarette packaging."|1000.0|792|18.71|74.84|6.45|1|
|22295-9-2013|93|8|19|2013|"Batabak 510 is a fully coated bleached paperboard with uncoated back, based on 100 % fresh forest fibres. Typical end use is cigarette packaging."|1000.0|781|17.16|76.31|6.53|1|
|22322-1-2014|106|28|19|2014|Glass container|0.416|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|22322-2-2014|106|28|19|2014|Glass container|0.416|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|22322-3-2014|106|28|19|2014|Glass container|0.416|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|22322-4-2014|106|28|19|2014|Glass container|0.416|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|22379-1-2014|39|24|19|2014|BillerudKorsnäs Liquid LC|1000.0|407|69.40|9.70|20.90|1|
|22379-1-2015|39|24|12|2015|BillerudKorsnäs Liquid LC|1000.0|407|69.40|9.70|20.90|1|
|22710-1-2014|56|25|2|2014|Coca-Cola (all packaging and sizes)|1.093|0|84.28|11.12|4.60|1|
|22710-1-2015|56|25|15|2015|Coca-Cola (all packaging and sizes)|1.093|0|38.37|12.71|48.93|1|
|22710-1-2016|56|25|2|2016|Coca-Cola (all packaging and sizes)|1.093|0|42.17|10.98|46.85|1|
|22917-2-2015|10|23|13|2015|Wind Turbine G90 2 Megawats|361000.0|1251625|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|22917-3-2015|10|23|13|2015|Wind Turbine G114 2 Megawats|400000.0|1532608|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|22645-1-2013|52|26|25|2013|"TFT-LCD products is the use of the optical properties of liquid crystals to achieve the image display, liquid crystal main structure is poured into the vacuum between the two pieces of glass, glass, applied voltage and control the proper spacing of the deflection characteristics of the incident light can be changed to to image display. For the automobile products's TFT-LCD Module, the product is defined as B to B (Business to Business) products, ranging from the cradle to the door inventory, according to standard PCR system boundaries should be compulsory included in carbon footprint assessment phase are as follows:  . Mining and raw materials made  . Manufacture of major components  . Transport components  . Product assembly stage. The models 069LA of automobile panel was complete the dual certification of carbon footprint and water footprint  by SGS in 2011."|0.28|12|57.00|32.00|11.00|1|
|22917-4-2015|10|23|13|2015|Wind Turbine G128 5 Megawats|600000.0|3718044|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|22917-5-2015|10|23|13|2015|Wind Turbine G132 5 Megawats|600000.0|3276187|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23195-1-2013|74|28|25|2013|HP LaserJet Enterprise M4555fskm MFP (note that this product is representative of 61 similar products for which full LCAs were performed)|52.5|16100|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23195-10-2013|74|28|25|2013|HP EliteDisplay E201 20-inch LED Backlit Monitor|7.1|300|0.83|46.90|52.28|1|
|23195-11-2013|74|28|25|2013|HP Compaq LA2206xc 21.5-inch Webcam Monitor|8.3|380|0.86|43.06|56.08|1|
|23195-12-2013|74|28|25|2013|HP Compaq LA2405x 24-inch LED Backlit Monitor|10.1|460|1.04|38.32|60.63|1|
|23195-2-2013|74|28|25|2013|HP Envy 120 InkJet Printer|7.2|231|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23195-3-2013|74|28|25|2013|HP ScanJet 7500 (note that this product is representative of 5 similar products for which full LCAs were performed)|10.61|262|34.50|13.16|52.35|1|
|23195-4-2013|74|28|25|2013|HP Compaq Elite 8300 USDT PC|3.1|420|0.52|36.38|63.10|1|
|23195-5-2013|74|28|25|2013|HP Compaq 8200 Elite SFF Business PC|7.6|500|0.77|33.23|66.00|1|
|23195-6-2013|74|28|25|2013|HP Compaq Elite 8300 MT PC|11.2|580|0.75|26.92|72.33|1|
|23195-7-2013|74|28|25|2013|HP EliteBook Folio 9470m|1.61|230|0.47|76.66|22.87|1|
|23195-8-2013|74|28|25|2013|HP EliteBook 8460p Notebook PC|2.25|290|0.59|62.52|36.89|1|
|23195-9-2013|74|28|25|2013|HP EliteBook 8570w Mobile Workstation|3.0|370|0.24|69.08|30.68|1|
|23634-1-2015|73|28|1|2015|Embody Chair|23.59|98|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-1-2016|73|28|11|2016|Embody Chair|23.59|98|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-10-2016|73|28|11|2016|Canvas Wood Lateral|70.2|54|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-11-2016|73|28|11|2016|Canvas Wood Pedestal 2016|88.7|77|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-12-2016|73|28|11|2016|Canvas Wood Tower|107.0|43|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-13-2016|73|28|11|2016|New Aeron Chair|19.323|87|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-2-2015|73|28|1|2015|Caper Stacking Chair|9.53|26|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-2-2016|73|28|11|2016|Caper Stacking Chair|9.53|26|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-3-2015|73|28|1|2015|Aeron Chair|24.95|91|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-3-2016|73|28|11|2016|Aeron Chair|24.95|98|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-4-2015|73|28|1|2015|Mirra 2 Chair|19.05|105|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-4-2016|73|28|11|2016|Mirra 2 Chair|19.05|105|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-5-2015|73|28|1|2015|"Sayl Work Chair, Suspension Back"|17.24|82|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-5-2016|73|28|11|2016|"Sayl Work Chair, Suspension Back"|17.24|82|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-6-2015|73|28|1|2015|Setu Multipurpose Chair with Lyris 2|7.71|48|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-6-2016|73|28|11|2016|Setu Multipurpose chair with Lyris 2|7.71|48|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-7-2015|73|28|1|2015|Celle Chair|24.95|72|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-7-2016|73|28|11|2016|Celle Chair|24.95|72|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-8-2016|73|28|11|2016|Aeron Work Stool|34.93|102|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|23634-9-2016|73|28|11|2016|Canvas Wood Credenza 2016|99.5|76|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2377-1-2014|46|27|26|2014|Graphite 2500 DECT|1.4|5|19.23|2.99|77.78|1|
|2377-1-2015|46|27|26|2015|Graphite 2500 DECT|1.4|5|19.23|2.99|77.78|1|
|2377-2-2014|46|27|26|2014|Home Hub 2.0|1.1|57|12.01|0.37|87.62|1|
|2377-2-2015|46|27|26|2015|Home Hub 2.0|1.1|57|11.86|0.37|87.78|1|
|2377-3-2014|46|27|26|2014|Home Hub 3.0|1.2|33|6.88|1.07|92.05|1|
|2377-3-2015|46|27|26|2015|Home Hub 3.0|1.2|33|6.88|1.07|92.05|1|
|2377-4-2014|46|27|26|2014|Vision+ box|1.7|88|11.23|0.05|88.72|1|
|2377-4-2015|46|27|26|2015|Vision+ box|1.7|88|11.23|0.05|88.72|1|
|2555-1-2014|48|28|19|2014|Carbon Black|1000.0|2280|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2688-1-2014|49|16|25|2014|Multi Function Printer|71.0|39|55.85|21.30|22.86|1|
|2688-2-2014|49|16|25|2014|Multi Function Printer with a built-in facsimile|30.98|57|56.52|21.55|21.93|1|
|2688-3-2014|49|16|25|2014|Digital camera|0.553|13|63.81|24.34|11.85|1|
|2688-4-2014|49|16|25|2014|Single Lens Reflex camera system|1.53|19|57.39|21.89|20.73|1|
|2802-1-2013|50|28|2|2013|Ingeo--biopolymer (polylactid) made from corn starch|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2802-1-2014|50|28|2|2014|Ingeo|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|28843-1-2013|36|27|8|2013|"Series 29 Hydrant - Squat Hydrant suitable for use with water and neutral liquids, to a maximum temperature of 70 degrees centigrade. Complies with requirements of BS750:2006 and BSEN1074-2:2004 and EN14339:2005, underground hydrants. Also to BS EN 1074-6 for potable (drinking) water. This product has kitemark and WRAS approval. The 2938832112 replaces the 2928832112."|13.0|31|39.38|56.85|3.77|1|
|28843-1-2014|36|27|8|2014|Series 29/388 Squat Hydrant|13.0|30|97.64|0.13|2.22|1|
|28843-2-2013|36|27|8|2013|"The series 21/75 is a resilient seat, wedge gate valve for isolation purposes, suitable for use with water and neutral liquids (sewage), to a maximum temperature of +70 degrees centegrade. The series 21/75 is a new alternative to a series 21/50 in some applications. Therefore where a 21/75 is sold we can calculate the emissions saved in contrast to the sale of a 21/50."|41.0|94|45.25|46.95|7.80|1|
|28843-3-2013|36|27|8|2013|"Gas Product - Small 555 series cast iron softseal valve, PN16. This series 555 softseal valve is a double-faced, resilient seat wedge gate valve. It is designed primarily for the isolation of natural gas and towns gas. Kitemark approved for GIS/V7 part 1."|23.0|44|72.07|17.08|10.84|1|
|28843-3-2014|36|27|8|2014|Series 555/300 Gate Valve|23.0|33|99.48|0.09|0.43|1|
|28843-4-2014|36|27|8|2014|Series 21/50 Gate Valve|41.0|63|97.85|0.05|2.11|1|
|28843-5-2014|36|27|8|2014|Series 202/31 Repair Clamp|7.0|57|98.76|0.02|1.23|1|
|28921-1-2016|107|28|2|2016|See attached sheet|907.19|187|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|28933-1-2013|22|28|11|2013|Ecoworx Carpet Tile with Shaw's Eco Solution Q Nylon 6 face fiber|0.2679066667|15|82.31|9.52|8.16|1|
|28991-1-2016|117|17|19|2016|PET preforms for bottles|0.03|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|2908-1-2013|2|16|11|2013|Page Printer SPEEDIA N3600|45.0|2344|11.52|0.55|87.92|1|
|2908-1-2014|2|16|11|2014|SPEEDIA N3600|45.0|2344|11.52|0.55|87.92|1|
|3022-1-2016|6|23|14|2016|Lineal Alkyl Bencene (LAB)|1000.0|6109|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3022-2-2016|6|23|14|2016|Lineal Alkyl Bencene (LAB)|1000.0|890|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|30495-1-2016|128|28|8|2016|E - Series Solar Panels (SPR-327NE-WHT-D AR modules)|18.6|281|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|31487-1-2013|98|28|11|2013|Droid Razr|0.127|30|0.07|92.35|7.59|1|
|3193-1-2013|79|26|25|2013|"50” TFT-LCD TV module which consisted of the following main components: PCBA, converter, TFT-LCD panel, LED backlight"|14.96|145|60.00|40.00|0.00|1|
|32738-1-2013|118|16|8|2013|Can/Bottle vending machine  -30Selections  -Cooling / warming  - R744 Heat Pump Refrigeration System  - LED Lighting|342.462|3140|28.42|4.25|67.33|1|
|32738-1-2014|118|16|8|2014|Vending Machine|342.462|2403|35.98|5.77|58.25|1|
|32738-1-2015|118|16|13|2015|Vending Machine|342.462|2268|42.63|6.67|50.70|1|
|32838-1-2015|60|23|9|2015|Sodium sulphate|1000.0|180|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|32838-1-2016|60|23|19|2016|sodium sulphate|1000.0|140|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3329-2-2013|3|28|25|2013|IP Phone|3.5|278|19.39|7.40|73.21|1|
|3329-3-2013|3|28|25|2013|Small Rack Mount Switch|2.0|1473|5.50|2.10|92.40|1|
|33496-2-2014|108|28|16|2014|Walmart Brand Products|454.0|164|52.74|27.75|19.51|1|
|33496-2-2015|108|28|16|2015|Walmart Brand Products|454.0|179|48.22|25.37|26.40|1|
|33496-2-2016|108|28|2|2016|Walmart Brand Products|0.454|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3435-1-2016|53|22|2|2016|GMP (5'-Guanylate)|1000.0|24839|34.13|65.43|0.43|1|
|3435-2-2016|53|22|2|2016|I&G (5'-Ribonucleotide)|1000.0|21126|23.59|75.93|0.49|1|
|3435-3-2016|53|22|2|2016|IMP (5'-Inosinate)|1000.0|18042|16.72|83.07|0.21|1|
|3435-4-2016|53|22|2|2016|L-Arginine|1000.0|26836|29.89|69.84|0.26|1|
|3435-5-2016|53|22|2|2016|MSG (Monosodium Glutamate)|1000.0|3859|22.73|76.08|1.19|1|
|3435-6-2016|53|22|2|2016|SUGAR|1000.0|115|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|34386-1-2016|58|10|2|2016|Nicotine|1.0|17|23.57|12.40|64.03|1|
|3448-1-2013|54|25|19|2013|"Organic pigments. PCF value for organic pigments, cradle to gate"|1.0|26|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3448-2-2013|54|25|19|2013|"PCF value for metal and effect pigments, cradle to gate"|1.0|26|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3448-3-2014|54|25|19|2014|Organic Pigments|1.0|26|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3448-3-2016|54|25|19|2016|Organic Pigments|1.0|26|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3448-4-2016|54|25|19|2016|De-icing fluids|1.0|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3448-5-2016|54|25|19|2016|Exolit ammonium polyphosphate\t|1.0|5|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3448-6-2016|54|25|19|2016|Exolit OP phosphinates\t|1.0|9|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3448-7-2016|54|25|19|2016|Sunliquid\t|0.0333|0|22.36|11.85|65.79|1|
|35290-1-2013|121|9|18|2013|"Verre Infini, perfumery glass made only with domestic glass from segregation. Middle bottle weighing 100g"|0.1|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|35290-2-2013|121|9|18|2013|Perfumery specific glass. Middle bottle weighing 100g|0.1|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|35311-1-2013|13|28|11|2013|Interface United States carpet tile (The average square meter of carpet tile sold as Cool Carpet by our Interface business in the United States)|4.164|13|54.89|13.82|31.29|1|
|35311-1-2014|13|28|11|2014|Interface US carpet|4.164|13|54.63|13.76|31.61|1|
|35311-10-2016|13|28|11|2016|Interface Brazil Carpet|4.164|10|59.46|14.97|25.56|1|
|35311-11-2016|13|28|11|2016|Interface Canada Carpet|4.164|11|52.51|13.22|34.27|1|
|35311-12-2016|13|28|11|2016|Interface Global LVT|4.164|17|56.95|14.34|28.71|1|
|35311-2-2013|13|28|11|2013|Interface EMEAI carpet tile (The average square meter of carpet tile sold as Cool Carpet by our Interface business in EMEAI)|4.164|10|52.60|13.24|34.16|1|
|35311-2-2015|13|28|6|2015|Interface EMEA Carpet|4.164|11|66.48|2.00|31.51|1|
|35311-3-2013|13|28|11|2013|Interface Thailand carpet tile (The average square meter of carpet tile sold as Cool Carpet by our Interface business in Southeast Asia)|4.164|15|60.06|15.12|24.82|1|
|35311-3-2014|13|28|11|2014|Interface Asia carpet|4.164|14|57.94|14.59|27.47|1|
|35311-3-2016|13|28|11|2016|Interface Thailand Carpet|4.164|11|57.33|14.44|28.23|1|
|35311-4-2013|13|28|11|2013|Interface Australia carpet tile (The average square meter of carpet tile sold as Cool Carpet by our Interface business in Australia)|4.164|17|56.13|14.13|29.74|1|
|35311-4-2015|13|28|6|2015|Interface Australia Carpet|4.164|16|53.67|15.38|30.95|1|
|35311-4-2016|13|28|11|2016|Interface Australia Carpet|4.164|13|50.02|12.60|37.38|1|
|35311-5-2014|13|28|11|2014|Interface Europe carpet|4.164|10|52.36|13.18|34.46|1|
|35311-5-2016|13|28|11|2016|Interface Europe Carpet|4.164|10|48.42|12.19|39.39|1|
|35311-6-2015|13|28|6|2015|Interface Asia Carpet|4.164|14|65.08|6.97|27.95|1|
|35311-7-2015|13|28|6|2015|Interface Americas Carpet|4.164|13|64.57|4.65|30.78|1|
|35311-7-2016|13|28|11|2016|Interface Americas Carpet|4.164|11|53.14|13.38|33.48|1|
|35311-8-2015|13|28|6|2015|Interface China Carpet|4.164|15|60.91|7.77|31.33|1|
|35311-8-2016|13|28|11|2016|Interface China Carpet|4.164|15|43.49|10.95|45.55|1|
|35374-1-2013|116|3|14|2013|"Ethanol. In addition to its use as fuel (in vehicles), ethanol is an input to the food, chemical and cosmetic industries."|789.0|750|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3565-1-2013|5|28|2|2013|Coca-Cola 300ml can|0.3|0|69.76|7.00|23.24|1|
|3565-10-2013|5|28|2|2013|Coke Zero 330 ml glass bottle|0.541|0|73.95|3.42|22.63|1|
|3565-11-2013|5|28|2|2013|Coke Zero 500ml PET|0.541|0|52.09|12.32|35.59|1|
|3565-12-2013|5|28|2|2013|Coke Zero- 2 litre plastic bottle|2.0|0|50.20|21.60|28.20|1|
|3565-13-2013|5|28|2|2013|Oasis Summer Fruits  - 375ml glass|0.614|0|72.51|4.29|23.21|1|
|3565-14-2013|5|28|2|2013|Oasis Summer Fruits - 500ml plastic bottle|0.541|0|56.84|11.43|31.73|1|
|3565-2-2013|5|28|2|2013|Coca-Cola 300ml glass bottle|0.492|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3565-3-2013|5|28|2|2013|Coca-Cola 500ml plastic bottle|0.541|0|57.18|11.02|31.80|1|
|3565-4-2013|5|28|2|2013|Coca-Cola 2 litre plastic bottle|2.0|1|60.96|17.12|21.92|1|
|3565-5-2013|5|28|2|2013|Diet Coke 300ml can|0.3|0|66.44|7.66|25.90|1|
|3565-6-2013|5|28|2|2013|Diet Coke 330ml glass bottle|0.541|0|73.38|3.50|23.12|1|
|3565-7-2013|5|28|2|2013|Diet Coke 500 ml plastic bottle|0.541|0|52.32|12.22|35.46|1|
|3565-8-2013|5|28|2|2013|Diet Coke - 2 Litre Plastic Bottle|2.0|0|46.40|23.28|30.32|1|
|3565-9-2013|5|28|2|2013|Coke Zero 300ml can|0.3|0|52.07|12.33|35.60|1|
|3652-1-2013|57|26|25|2013|Netbook (SKU: High efficient CPU.10.1-inch LED backlight)|2.0|157|60.59|0.73|38.68|1|
|3652-2-2013|57|26|25|2013|Notebook|2.58|132|30.00|2.00|68.00|1|
|3768-1-2016|55|28|14|2016|Natural Gas|0.0012701804|0|5.69|3.02|91.29|1|
|37921-1-2015|138|28|9|2015|Plastics|1.0|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|37921-1-2016|138|28|19|2016|Plastics|1000.0|120|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|37921-2-2015|138|28|9|2015|Latex|1.0|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|37921-3-2015|138|28|9|2015|Rubber|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|37921-3-2016|138|28|19|2016|Rubber|1000.0|1153|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|37921-4-2016|138|28|19|2016|Binders|1000.0|210|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|37977-1-2015|134|28|13|2015|T300|181.0|655|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|37977-1-2016|134|28|8|2016|T300|181.0|4469|17.33|3.73|78.94|1|
|38359-1-2014|38|15|16|2014|Pasta produced in United States|1.0|1|54.65|37.53|7.82|1|
|38359-1-2015|38|15|16|2015|Pasta produced in United States|1.0|1|56.90|35.51|7.59|1|
|38359-10-2016|38|15|16|2016|Pesto alla Genovese sauce|0.19|1|84.49|14.23|1.28|1|
|38359-2-2014|38|15|16|2014|Pasta produced in Italy (local consumption)|1.0|1|69.38|26.40|4.22|1|
|38359-2-2015|38|15|16|2015|Pasta produced in Italy (local consumption)|1.0|1|67.06|28.57|4.37|1|
|38359-3-2014|38|15|16|2014|Pasta produced in Italy (for export)|1.0|1|63.97|24.34|11.68|1|
|38359-3-2015|38|15|16|2015|Pasta produced in Italy (for export)|1.0|1|64.47|27.47|8.06|1|
|38359-4-2014|38|15|16|2014|Pasta produced in Greece|1.0|2|60.84|25.15|14.01|1|
|38359-4-2015|38|15|16|2015|Pasta produced in Greece|1.0|2|59.59|26.65|13.76|1|
|38359-5-2014|38|15|16|2014|Pasta produced in Turkey|1.0|1|68.03|23.54|8.42|1|
|38359-5-2015|38|15|16|2015|Pasta produced in Turkey|1.0|1|74.97|16.68|8.34|1|
|38359-6-2014|38|15|16|2014|Crisp’n light 7 grains|0.14|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|38359-6-2015|38|15|16|2015|Crisp’n light 7 grains|0.14|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|38359-7-2014|38|15|16|2014|Light rye|0.27|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|38359-7-2015|38|15|16|2015|Light rye|0.27|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|38359-7-2016|38|15|16|2016|Light Rye|0.27|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|38359-8-2014|38|15|16|2014|Multi grain|0.275|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|38359-8-2015|38|15|16|2015|Multi grain|0.275|1|65.61|16.10|18.28|1|
|38359-8-2016|38|15|16|2016|Multi Grain|0.275|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|38359-9-2016|38|15|16|2016|Durum wheat semolina pasta in paperboard box (produced in United States)|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|38459-1-2013|47|28|2|2013|5 oz can whitemeat tuna|0.179|1|80.88|14.71|4.41|1|
|38459-2-2013|47|28|2|2013|5 oz can lightmeat tuna|0.179|0|84.62|5.13|10.26|1|
|39118-1-2013|127|28|2|2013|This is the average footprint of the 31 products we provided Walmart in 2012. The unit is in kg CO2e/tonne of product.|1000.0|1361|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|39118-1-2014|127|28|2|2014|Average SKU|1000.0|1360|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3928-1-2014|132|27|19|2014|Finished cold rolled coil|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3928-2-2014|132|27|19|2014|Hot Dip Galvanised Coil|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|3928-3-2014|132|27|19|2014|Hot rolled coil|1.0|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-1-2015|31|16|15|2015|L-Lysine Monohydrochloride(For Feed)|25.0|130|33.41|63.64|2.94|1|
|394-1-2017|31|16|2|2017|L-Lysine Monohydrochloride(For Feed)|25.0|130|33.41|63.64|2.94|1|
|394-10-2017|31|16|2|2017|HON-DASHI(R)|0.12|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-11-2017|31|16|2|2017|Knorr(R) Cup Soup Tsubu Tappuri Corn Cream|0.0465|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-12-2017|31|16|2|2017|L-Arginine|20.0|301|39.91|59.54|0.55|1|
|394-13-2017|31|16|2|2017|Lemon and Basil Fried Chicken|0.126|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-14-2017|31|16|2|2017|L-Glutamine|25.0|319|36.95|60.44|2.61|1|
|394-15-2017|31|16|2|2017|L-Isoleucine|20.0|433|27.91|70.59|1.49|1|
|394-16-2017|31|16|2|2017|L-leucine|20.0|585|18.45|80.45|1.11|1|
|394-17-2017|31|16|2|2017|L-Valine|20.0|393|20.19|78.16|1.65|1|
|394-18-2017|31|16|2|2017|Masako Ayam|0.011|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-19-2017|31|16|2|2017|Mentsuyu|0.4|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-2-2017|31|16|2|2017|AGF Blendy Stick Cafe au Lait|0.12|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-20-2017|31|16|2|2017|Monosodium L-Glutamate|20.0|58|43.90|53.48|2.61|1|
|394-21-2017|31|16|2|2017|Nabe Cube(R) Toridashi Umashio|0.058|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-22-2017|31|16|2|2017|Rosdee Pork|0.075|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-3-2017|31|16|2|2017|Aji-ngon Pork|0.4|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-4-2017|31|16|2|2017|Ajinomoto KK Consomme(Granules)|0.085|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-5-2017|31|16|2|2017|Ajinomoto KK Shirogayu 250g|0.25|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-6-2017|31|16|2|2017|Aspartame|25.0|780|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-7-2017|31|16|2|2017|Cook Do(R) Hoikoro|0.09|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-8-2017|31|16|2|2017|Cook Do(R) kyo-no Ozara Butabara Daikon|0.1|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|394-9-2017|31|16|2|2017|Di-sodium 5'-Inosinate|12.0|155|42.81|56.61|0.58|1|
|40638-1-2016|29|23|2|2016|Diced tomato|220.0|156|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|40638-2-2016|29|23|2|2016|Tomato fibre|25.0|13|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|40638-3-2016|29|23|2|2016|Tomato paste|238.0|156|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|40638-4-2016|29|23|2|2016|Tomato powder|25.0|16|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|40661-1-2014|90|10|2|2014|Peppermint tea infusion|0.002|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|40661-1-2015|90|10|2|2015|Peppermint tea infusion|0.002|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|40920-1-2015|99|21|12|2015|PET preforms|1000.0|871|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|41515-1-2013|136|28|8|2013|Cable of LMR-195-LLPL|0.03|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|41515-1-2014|136|28|8|2014|Cable|0.03|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|41515-1-2015|136|28|13|2015|Cable|0.03|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|41697-1-2016|76|28|10|2016|Float Table Top & Base|59.02|40|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|41697-2-2016|76|28|10|2016|Smart Chair|16.78|34|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4226-1-2013|7|16|8|2013|Residential Air Conditioner|17.46331|3653|3.78|0.63|95.59|1|
|4226-1-2014|7|16|8|2014|Residential Air Conditioner|17.46331|3505|3.94|0.66|95.41|1|
|4226-1-2015|7|16|8|2015|Residential Air Conditioner|17.46331|3505|3.94|0.66|95.41|1|
|4226-1-2017|7|16|8|2017|Residential Air Conditioner|17.46331|4218|3.96|0.55|95.50|1|
|4226-2-2017|7|16|8|2017|Commercial Air Conditioner|149.685|51066|0.98|0.13|98.88|1|
|4226-3-2017|7|16|8|2017|Light commercial Air Conditioner|128.367|39653|0.70|0.10|99.21|1|
|4235-1-2015|61|10|7|2015|Mercedes-Benz A-Class|1425.0|30500|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-10-2015|61|10|7|2015|Mercedes-Benz CLS-Class|1790.0|57100|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-11-2015|61|10|7|2015|Mercedes-Benz E-Class BlueTEC Hybrid|1735.0|40600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-12-2015|61|10|7|2015|Mercedes-Benz GLA-Class|1535.0|31500|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-13-2015|61|10|7|2015|Mercedes-Benz GLK-Class|1850.0|48800|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-14-2015|61|10|7|2015|Mercedes-Benz SL-Class|1730.0|69000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-15-2016|61|10|7|2016|Mercedes-Benz A-Class (A 180 BlueEFFICIENCY)|1200.0|32000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-16-2016|61|10|7|2016|Mercedes-Benz B-Class (B 180 BlueEFFICIENCY)|1200.0|34000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-17-2016|61|10|7|2016|Mercedes-Benz B-Class Electric Drive (B 250 e)|1650.0|12000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-18-2016|61|10|7|2016|Mercedes-Benz C-Class (C 180)|1400.0|36000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-19-2016|61|10|7|2016|Mercedes-Benz C-Class (C 250)|1800.0|39000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-2-2015|61|10|7|2015|Mercedes-Benz B-Class|1425.0|32500|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-20-2016|61|10|7|2016|Mercedes-Benz C-Class Plug-In Hybrid (C 350 e)|1925.0|24000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-21-2016|61|10|7|2016|Mercedes-Benz CLA (CLA 180)|1200.0|31000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-22-2016|61|10|7|2016|Mercedes-Benz CLS (350 BlueEFFICIENCY)|2220.0|60000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-23-2016|61|10|7|2016|Mercedes-Benz E-Class (E 200)|1500.0|50000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-24-2016|61|10|7|2016|Mercedes-Benz E-Class (E 220 d (Estate))|2320.0|41000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-25-2016|61|10|7|2016|Mercedes-Benz E-Class (E 220 d)|2100.0|38000|18.46|5.69|75.85|1|
|4235-26-2016|61|10|7|2016|Mercedes-Benz E-Class Plug-In Hybrid (E 350 e)|1925.0|26000|31.44|9.70|58.87|1|
|4235-27-2016|61|10|7|2016|Mercedes-Benz GLA (GLA 200)|1400.0|33000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-28-2016|61|10|7|2016|Mercedes-Benz GLC (220 d 4MATIC)|2500.0|39000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-29-2016|61|10|7|2016|Mercedes-Benz GLC Plug-In Hybrid (GLC 350 e 4MATIC)|2000.0|27000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-3-2015|61|10|7|2015|Mercedes-Benz C-Class|1505.0|50500|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-30-2016|61|10|7|2016|Mercedes-Benz GLE (GLE 500 4MATIC)|3500.0|91000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-31-2016|61|10|7|2016|Mercedes-Benz GLE Plug-In Hybrid - GLE 500 e 4MATIC|3500.0|39000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-32-2016|61|10|7|2016|Mercedes-Benz S-Class (S 500)|2050.0|85000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-33-2016|61|10|7|2016|Mercedes-Benz S-Class Hybrid (S 300 BlueTEC HYBRID)|2715.0|51000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-34-2016|61|10|7|2016|Mercedes-Benz S-Class Hybrid (S 400 h)|2595.0|66000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-35-2016|61|10|7|2016|Mercedes-Benz S-Class Plug-In Hybrid (S 500 e)|2585.0|38000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-36-2016|61|10|7|2016|Mercedes-Benz SL (SL 350)|2065.0|72000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-37-2016|61|10|7|2016|Mercedes-Benz SLC (SLK 200 BlueEFFICIENCY)|2015.0|45000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-4-2015|61|10|7|2015|Mercedes-Benz C-Class e|1580.0|27400|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-5-2015|61|10|7|2015|Mercedes-Benz E-Class Saloon|1680.0|47200|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-6-2015|61|10|7|2015|Mercedes-Benz M-Class - Passenger Car|2110.0|43600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-7-2015|61|10|7|2015|Mercedes-Benz S-Class|2015.0|54000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-8-2015|61|10|7|2015|Mercedes-Benz B-Class electric drive|1725.0|22600|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4235-9-2015|61|10|7|2015|Mercedes-Benz CLA-Class|1490.0|30000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4311-1-2013|62|9|2|2013|Set yoghourt|1.0|2|65.19|11.30|23.51|1|
|4311-2-2013|62|9|2|2013|Drink yoghourt|1.0|1|63.45|9.10|27.45|1|
|4311-3-2013|62|9|2|2013|Plain Water (SKU: Bonafont 1L film bottle)|1.0|0|35.63|17.24|47.13|1|
|4311-4-2015|62|9|15|2015|BONAFONT NATURAL 6000 ML X2 FILM 84 (6L RPET Bottle)_54742|1.0|0|36.24|19.78|43.98|1|
|4311-5-2015|62|9|15|2015|DANONE MPCK STRAWBERRY 1960G (8X245G)_87121|1.0|2|78.26|5.48|16.26|1|
|4311-6-2015|62|9|15|2015|SILHOUETTE ST/PE/RA/BL 100GX16_14967|1.0|1|78.78|4.91|16.31|1|
|4311-7-2015|62|9|15|2015|Activia STR/PCH/BLU 24X4OZ CLUB_67740|1.0|2|74.78|7.31|17.91|1|
|432-1-2016|30|9|19|2016|Argon|1.785714286|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|432-3-2016|30|9|19|2016|Nitrogen|1.25|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4332-1-2014|63|26|25|2014|Keyboards|0.445|10|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4332-1-2015|63|26|25|2015|Keyboard|0.158|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4416-1-2016|65|26|25|2016|DC fan|0.26|2|84.09|15.91|0.00|1|
|4416-2-2016|65|26|25|2016|External power supply/ portable adaptor|0.363|8|96.84|3.16|0.00|1|
|4416-3-2016|65|26|25|2016|PV Inverter|81.918782|686|95.14|4.86|0.00|1|
|4416-4-2016|65|26|25|2016|TPS (Telecom Power System)|4.1|69|5.49|94.51|0.00|1|
|4433-1-2013|64|28|25|2013|"The Latitude E6400 is a typical high-volume, mainstream business laptop that is representative of a range of similar laptop  products. It is Energy Star® 5.0 qualified and EPEAT Gold registered."|2.52|350|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4433-10-2015|64|28|25|2015|Latitude E6440|2.12|348|48.89|18.64|32.47|1|
|4433-11-2015|64|28|25|2015|Latitude E6540|2.56|408|45.95|17.53|36.52|1|
|4433-12-2015|64|28|25|2015|Latitude E7240|1.36|310|58.66|22.37|18.97|1|
|4433-13-2015|64|28|25|2015|Latitude E7440|1.63|276|56.39|21.51|22.10|1|
|4433-14-2015|64|28|25|2015|OptiPlex 3030 all-in-one|7.03|480|36.35|13.86|49.79|1|
|4433-2-2013|64|28|25|2013|"The OptiPlex 780 Mini Tower is a typical high-volume, mainstream business desktop that is representative of a range of similar desktop products. It is Energy Star® 5.0 compliant and EPEAT Gold registered."|11.7|800|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4433-3-2013|64|28|25|2013|"PowerEdge R710 is a typical high-volume, next-generation Intel Xeon processor-based 2U Rack Server that is representative of a range of similar server products."|26.1|6360|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|4433-3-2015|64|28|25|2015|PowerEdge R710|26.1|6360|5.72|2.18|92.10|1|
|4433-4-2015|64|28|25|2015|OptiPlex 3010 Minitower (MT)|5.7|624|32.60|12.43|54.97|1|
|4433-5-2015|64|28|25|2015|OptiPlex 9010 Small Form Factor (SFF)|6.0|667|28.39|10.83|60.78|1|
|4433-6-2015|64|28|25|2015|Latitude E3440|2.0|314|50.19|19.14|30.67|1|
|4433-7-2015|64|28|25|2015|Latitude E3540|2.22|309|53.71|20.48|25.81|1|
|4433-8-2015|64|28|25|2015|Latitude E5440|2.0|292|55.78|21.27|22.95|1|
|4433-9-2015|64|28|25|2015|Latitude E5540|2.3|311|53.80|20.52|25.68|1|
|462-1-2013|32|20|19|2013|Beverage can inside spray lacquer steel - Aqualure 905|1000.0|2000|95.96|4.04|0.00|1|
|462-1-2014|32|20|19|2014|Beverage can inside spray lacquer steel|1000.0|2000|95.96|4.04|0.00|1|
|462-10-2013|32|20|19|2013|Average of all products sold to Braskem|1.0|6|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-10-2014|32|20|19|2014|Average of all products sold to Braskem|1000.0|5500|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-10-2015|32|20|9|2015|Average of all products sold to Braskem|1000.0|6500|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-10-2016|32|20|19|2016|Average of all products sold to Braskem|1000.0|6240|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-11-2014|32|20|19|2014|Aggregated Global Sales for KAO|1000.0|3100|83.55|16.45|0.00|1|
|462-12-2014|32|20|19|2014|Average of all products sold to KAO|1000.0|4900|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-13-2015|32|20|9|2015|Aggregated Global Sales for Customer CNH Industrial NV|1000.0|4700|91.30|8.70|0.00|1|
|462-14-2016|32|20|19|2016|Aggregated global sales for Dell Inc|1000.0|1320|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-15-2016|32|20|19|2016|Aggregated sales to Symrise|1000.0|141|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-16-2016|32|20|19|2016|Caustic products delivered to ICL|1000.0|440|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-17-2016|32|20|19|2016|Chelates and Micronutrients sold to ICL|1.0|3|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-18-2016|32|20|19|2016|Chlormethane products delivered to ICL|1000.0|1800|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-2-2013|32|20|19|2013|"Beverage can inside spray lacquer aluminium - Aqualure 900, Aqualure 2000 etc."|1000.0|1600|94.94|5.06|0.00|1|
|462-2-2014|32|20|19|2014|Beverage can inside spray lacquer aluminium|1000.0|1600|94.94|5.06|0.00|1|
|462-2-2015|32|20|9|2015|Beverage can inside spray lacquer aluminium|1000.0|1600|94.94|5.06|0.00|1|
|462-3-2013|32|20|19|2013|Overprint varnishes  (also used to approximate rim varnishes)|1000.0|2200|96.33|3.67|0.00|1|
|462-3-2014|32|20|19|2014|Overprint varnishes|1000.0|2200|96.33|3.67|0.00|1|
|462-3-2015|32|20|9|2015|Overprint varnishes|1000.0|2200|96.33|3.67|0.00|1|
|462-4-2013|32|20|19|2013|Aggregated Global Sales for Unilever - Polyacrylates and other products|1.0|3|70.20|29.80|0.00|1|
|462-4-2014|32|20|19|2014|Aggregated Global Sales for Unilever|1000.0|3420|71.64|28.36|0.00|1|
|462-4-2015|32|20|9|2015|Aggregated Global Sales for Unilever|1000.0|2580|74.81|25.19|0.00|1|
|462-4-2016|32|20|19|2016|Aggregated Global Sales for Unilever|1.0|3|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-5-2013|32|20|19|2013|Aggregated Global Sales for L'Oreal - Polyacrylates and other products|1.0|2|50.18|49.82|0.00|1|
|462-5-2014|32|20|19|2014|Aggregated Global Sales for L'Oreal|1000.0|3010|65.45|34.55|0.00|1|
|462-5-2015|32|20|9|2015|Aggregated Global Sales for L'Oreal|1000.0|2760|64.74|35.26|0.00|1|
|462-5-2016|32|20|19|2016|Aggregated Global Sales for L'Oreal|1.0|3|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-6-2013|32|20|19|2013|Aggregated Global Sales for Johnson & Johnson - Polyacrylates and other products|1.0|2|63.73|36.27|0.00|1|
|462-6-2014|32|20|19|2014|Aggregated Global Sales for Johnson & Johnson|1000.0|3400|75.88|24.12|0.00|1|
|462-6-2015|32|20|9|2015|Aggregated Global Sales for Johnson & Johnson|1000.0|2910|74.23|25.77|0.00|1|
|462-6-2016|32|20|19|2016|Aggregated Global Sales for Johnson & Johnson|1.0|4|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|462-7-2013|32|20|19|2013|Sodium chlorate produced in North America.|1000.0|1200|15.38|84.62|0.00|1|
|462-8-2013|32|20|19|2013|"Sodium chlorate as solid phase, produced in South America."|1000.0|220|59.09|40.91|0.00|1|
|462-9-2013|32|20|19|2013|Chlorine dioxide water solution produced in South America.|1000.0|850|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|46693-1-2013|101|21|2|2013|Extra Neutral Potable Ethanol 96.4%|1041.666667|2019|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|46693-1-2016|101|21|2|2016|Extra Neutral Potable Ethanol 96.4%|1041.666667|1280|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|46702-1-2014|100|27|16|2014|Malt non HDP|1000.0|603|49.92|50.08|0.00|1|
|46702-2-2015|100|27|16|2015|Malted barley|1000.0|520|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|46702-2-2016|100|27|2|2016|Malted Barley|1000.0|561|56.15|29.54|14.31|1|
|46702-3-2016|100|27|2|2016|Malt extract|1000.0|947|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|46702-4-2016|100|27|2|2016|Malt flour in sack|1000.0|599|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|46702-5-2016|100|27|2|2016|Unmalted barley at harvest|1000.0|240|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|46807-1-2016|42|15|19|2016|H10311 - La vie est belle|0.354|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|46807-2-2016|42|15|19|2016|H4953 - Armani IO 100 ml|0.195|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|47200-1-2015|77|22|25|2015|H000 (Set-top Box)|1.98804|79|11.75|1.61|86.60|1|
|47410-1-2014|91|26|7|2014|Tire|12.7|106|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|47498-1-2013|51|15|19|2013|Corrugated box average Corrugated box Sab miller|1.0|1|88.79|11.21|0.00|1|
|47502-1-2013|69|7|2|2013|metal crown caps|0.00235|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|47502-1-2014|69|7|2|2014|Crown caps|0.00235|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|48227-1-2013|122|6|19|2013|Sodium sulbactam 3|1.0|70|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|48227-2-2013|122|6|19|2013|cyanodiester|1.0|6|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|48354-1-2014|27|28|19|2014|480ml Bottle|0.048|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|48354-2-2014|27|28|19|2014|18mm Cap|0.0434|0|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|48440-1-2016|110|13|19|2016|Fajar Board|1000.0|721|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|48775-1-2013|111|19|2|2013|Stevia Sweeteners|1.0|66|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|48775-2-2013|111|19|2|2013|Stevia Sweetener|1.0|48|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|48775-3-2013|111|19|2|2013|Natural Flavors|1.0|46|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|48775-4-2013|111|19|2|2013|Stevia Sweeteners|1.0|75|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|50932-1-2015|94|11|28|2015|Tobacco|1.0|1|57.14|42.86|0.00|1|
|511-2-2014|33|28|19|2014|Semi-finished aluminum sheet (hot rolled products)|1000.0|3940|26.42|73.58|0.00|1|
|511-2-2016|33|28|19|2016|Semi-finished aluminum sheet (hot rolled products)|1000.0|410|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|511-3-2014|33|28|19|2014|Semi-finished aluminum sheet (cold rolled products)|1000.0|5340|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|511-3-2016|33|28|19|2016|Semi-finished aluminum sheet (cold rolled products)|1000.0|410|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|511-4-2016|33|28|19|2016|Alumina|1000.0|2219|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|511-5-2016|33|28|19|2016|Primary aluminum ingot|1000.0|6512|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|511-7-2016|33|28|19|2016|Cast aluminum products|1000.0|720|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|51489-1-2014|66|6|11|2014|kettle|1.3|50|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|52397-1-2016|145|6|8|2016|aluminium wheel|1.0|3|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|5344-1-2013|67|24|11|2013|Vaccum Cleaners (Canisters & Uprights)|6.4|409|7.03|1.77|91.20|1|
|5344-1-2014|67|24|11|2014|Vaccum Cleaners|6.4|409|7.03|1.77|91.20|1|
|5344-1-2015|67|24|1|2015|Vaccum Cleaners|6.4|409|7.03|1.77|91.20|1|
|5414-1-2013|8|26|25|2013|"Notebook (Classmate PC) E11IS is manufactured & designed by ECS's factory located in eastern China, its GHG emission inventoried in 2012 by methodology of PAS 2050:2011 and verified by SGS ready."|1.3|213|97.98|2.02|0.00|1|
|5414-3-2013|8|26|25|2013|"30L desktop model vBeijing2_I for design stage plus manufacturing stage, made by ECS's factory loacted in southern China (GHG emission inventoried in 2012 by methodology of ISO 14064-1 and allocation basis by weight)"|9.26|2|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|61026-1-2016|143|10|7|2016|Beedwire 0.96|450.0|502|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|61026-2-2016|143|10|7|2016|Beedwire 1.27|450.0|446|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|6860-1-2013|9|16|25|2013|C1760nw Perinter|13.0|491|14.26|3.87|81.87|1|
|6860-10-2017|9|16|25|2017|S3840cdn|26.0|2300|8.32|0.21|91.47|1|
|6860-11-2017|9|16|25|2017|S3845cdn|33.0|2400|10.92|0.21|88.87|1|
|6860-2-2013|9|16|25|2013|2150cdn  printer|24.0|573|12.57|3.32|84.12|1|
|6860-3-2014|9|16|25|2014|C2660dn|25.600753|1400|11.83|2.07|86.10|1|
|6860-3-2015|9|16|25|2015|C2660dn|25.600753|1400|11.83|2.07|86.10|1|
|6860-3-2017|9|16|25|2017|C2660dn|25.600753|1400|11.83|2.07|86.10|1|
|6860-4-2014|9|16|25|2014|C2665dnf|115.0|1500|15.58|1.51|82.92|1|
|6860-4-2015|9|16|25|2015|C2665dnf|115.0|1500|14.00|1.53|84.47|1|
|6860-5-2017|9|16|25|2017|H625cdw|31.7|1200|15.22|2.11|82.67|1|
|6860-6-2017|9|16|25|2017|H815dw|19.0|2300|5.64|0.61|93.75|1|
|6860-7-2017|9|16|25|2017|H825cdw|31.7|1300|14.14|1.96|83.90|1|
|6860-8-2017|9|16|25|2017|S2815dn|19.0|2300|5.64|0.61|93.75|1|
|6860-9-2017|9|16|25|2017|S2825cdn|31.7|1300|14.14|1.96|83.90|1|
|6874-1-2013|70|16|25|2013|Desk Top PC  ESPRIMO Ｄ582/Ｅ|5.89|140|0.93|0.35|98.72|1|
|6874-1-2014|70|16|24|2014|Desk top personal computer|5.89|140|53.78|1.28|44.94|1|
|6874-3-2017|70|16|24|2017|Server|7.0|690|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|7164-1-2013|71|28|7|2013|Average of all GM vehicles produced and used in the 10 year life-cycle.|1820.086|27588|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|7164-1-2014|71|28|7|2014|Average of all GM vehicles produced and used in the 10 year life-cycle.|1820.086|33933|1.90|0.59|97.51|1|
|7164-1-2015|71|28|7|2015|Average of all GM vehicles produced and used in the 10 year life-cycle.|1820.086|39100|1.61|0.50|97.89|1|
|7164-1-2016|71|28|7|2016|Average GM Vehicle|1820.086|36386|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|7283-1-2013|72|25|8|2013|Cast iron pipes and fittings|1.0|2|82.64|16.53|0.83|1|
|73475-1-2016|4|28|14|2016|Coal|1000.0|3025|18.87|9.99|71.14|1|
|8338-1-2014|12|16|25|2014|Elevator|2270.0|9328|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|8362-1-2016|11|16|7|2016|Land Cruiser Prado. FJ Cruiser. Dyna trucks. Toyoace.IMV def unit.|2272.33|191687|2.90|0.25|96.85|1|
|8517-1-2014|75|24|19|2014|Invercote|1000.0|370|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|8517-1-2015|75|24|3|2015|Invercote|1000.0|211|19.29|53.72|26.99|1|
|8517-2-2014|75|24|19|2014|Incada|1000.0|292|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|8517-2-2015|75|24|3|2015|Incada|1000.0|210|22.65|63.07|14.29|1|
|8663-1-2015|123|22|23|2015|4Gb LPDDR2 SDRAM|0.0084|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|8663-2-2015|123|22|23|2015|64Gb NAND Flash MLC|0.0084|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|8663-3-2015|123|22|23|2015|4Gb LPDDR3 SDRAM|0.0084|1|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|904-1-2013|34|18|19|2013|Organic coated coil for the Nature product range (which includes the products Granite® and Estetic®) having a mean surface mass of 4.7kg. The Nature range of innovative chromium-free organic coated steels for the construction industry is the result of 15 years of research in partnership with paint suppliers to develop safer and greener corrosion-inhibiting pigments.|4.7|7|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|904-2-2013|34|18|19|2013|Retaining wall structure with a main wall (sheet pile): 136 tonnes of steel sheet piles and 4 tonnes of tierods per 100 meter wall|140000.0|167000|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|9101-1-2015|78|9|25|2015|Ingenico terminals|0.6|50|58.66|2.79|38.54|1|
|9298-1-2013|80|28|25|2013|Server CPU|0.4082331|29|24.14|68.97|6.90|1|
|9298-1-2014|80|28|22|2014|Server CPU|0.4082331|29|32.07|54.14|13.79|1|
|9298-2-2013|80|28|25|2013|Desktop CPU|0.4082331|14|21.43|71.43|7.14|1|
|9298-2-2014|80|28|22|2014|Desktop CPU|0.4082331|14|32.14|54.29|13.57|1|
|9298-3-2013|80|28|25|2013|Mobile CPU|0.4082331|7|28.57|71.43|0.00|1|
|9298-3-2014|80|28|22|2014|Mobile CPU|0.4082331|7|31.43|54.29|14.29|1|
|9792-1-2017|81|27|19|2017|Complete catalyst system for diesel-powered passenger car exhaust|2.0|188|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|
|9792-2-2017|81|27|19|2017|Three-way Catalyst for gasoline-powered passenger car exhaust|2.0|63|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|N/a (product with insufficient stage-level data)|1|

![image](https://github.com/dinhhluu/carbon_emission_analysis/blob/main/Trend%20of%20PCF%20over%20the%20years.png)

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

