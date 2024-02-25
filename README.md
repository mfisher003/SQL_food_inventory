## Analyzing multiple tables to review food and drink inventories with SQL. 

### Project Goal
The goal is to begin to become familiar with an in-depth analysis of several data sets using SQL. The dataset was created as part of the Analytics mentor SQL certificate. 

### SQL Commands Explored
- `count`
- `where`
- `case`
- `update`
- `CTE`
- `join`
- `union`
- `left join`

### First, I started by uploading the `.CSV`files into PostgreSQL and then running the SQL analysis in pgAdmin4.

 - I took an initial look at the data by running a count function the number of food records.
```SQL
select
	Count (*) as food_items
From foods f
```

- Then I pulled the entire table with all column names

```SQL
select 
	  food_id
	, item_name
	, storage_type
	, package_size
	, package_size_uom
	, brand_name
	, package_price
	, price_last_updated_ts
From 
	foods f
```
 - Here is the result of foods table. I took note of the information included, the data types, and structure.
 
 ![enter image description here](https://i.imgur.com/IIfRmtI.png)
 - I wanted to take a closer look at the price_last_updated_ts field so I editted the table to move the column to the front. 
```SQL
select 
	 f.price_last_updated_ts
	, f.food_id
	, f.item_name
	, f.storage_type
	, f. package_size
	, f.package_size_uom
	, f.brand_name
	, f.package_price
	, f.price_last_updated_ts
From 
	foods f
```
```SQL

select 
	  f.price_last_updated_ts
	, f.food_id
	, f.item_name
	, f.storage_type
	, f. package_size
	, f.package_size_uom as package_size_unit_of_measurement
	, f.brand_name
	, f.package_price
	, f.price_last_updated_ts
From 
	foods f
where
	brand_name ilike 'H-E-B (private label)'
````	

 - Updated table to include a new column with canned_yn
```SQL
select 
	  f.price_last_updated_ts
	, f.food_id
	, f.item_name
	, f.storage_type
	, f. package_size
	, f.package_size_uom as package_size_unit_of_measurement
	, f.brand_name
	, f.package_price
	, f.price_last_updated_ts
	, case when item_name ilike '%canned%' then 'Y' else 'N' end as is_canned_yn
From 
	foods f
````

 - view all null values from brand_name column

```SQL
select 
	  f.price_last_updated_ts
	, f.food_id
	, f.item_name
	, f.storage_type
	, f. package_size
	, f.package_size_uom as package_size_unit_of_measurement
	, f.brand_name
	, f.package_price
	, f.price_last_updated_ts
From 
	foods f
where 
	brand_name is null

update foods
	set storage_type = 'unknown'
	where storage_type is null 
````
 - Pull specific integer values from the food_id category and a look for a specific string value using the `where`command. 
```SQL	
select 
	 f.food_id
	, f.item_name
	, f.storage_type
	, f. package_size
	, f.package_size_uom
	, f.brand_name
	, f.package_price
	, f.price_last_updated_ts
from 
	foods f
where
	(
	food_id = 13
	or food_id = 15
	or food_id = 17)
 and brand_name ilike 'h-e-b (private label)'
 ````
 ```SQL
select 
 	  f.item_name
	, f.brand_name
	,count(*) as record_count
From 
	foods f
group by 
	  f.item_name
	, f.brand_name
````	

 - Use the `cast` command to calculate the percentage of heb private label over the total records. 

```SQL
select 
	cast(n.heb_records as decimal(10,2)) / cast(d.total_records as decimal (10,2))
from
	(
	select 
		count (*) as heb_records
	from 
		foods f
	where 
		brand_name ilike 'h-e-b (private label)'
	)n
	cross join (
					select 
						count(*) as total_records
					from 
						foods f
					)d
```
 - Return the distinct records for price last updated

```SQL
select 
	distinct 
		f.price_last_updated_ts
from
	foods f
```

	
- Analyze the days since last updated compared to todays date using the timestamp command in SQL.
```SQL
select 
	 f.food_id
	, f.item_name
	, f.storage_type
	, f. package_size
	, f.package_size_uom
	, f.brand_name
	, f.package_price
	, f.price_last_updated_ts at time zone 'America/Denver' as price_last_updated_ts
	, current_timestamp
	,( 
		current_timestamp
		- (f.price_last_updated_ts at time zone 'America/Denver')
	 ) as days_since_price_last_updated
from 
	foods f
```	

 - Compare the days since last updated compared to todays date using timestamp and `cast` commands to create a calculation within SQL.
```SQL
select 
	*
From (
		select 
			 f.food_id
			, f.item_name 
			, f.storage_type
			, f. package_size
			, f.package_size_uom
			, f.brand_name
			, f.package_price
			, f.price_last_updated_ts at time zone 'America/Denver' as price_last_updated_ts
			, current_timestamp
			,( 
				current_timestamp
				- (f.price_last_updated_ts at time zone 'America/Denver')
			 ) as days_since_price_last_updated
			 , current_date - cast(
					(f.price_last_updated_ts at time zone 'America/Denver') as date
					 ) as days_since_last_updated_days
		from 
			foods f
	)f
where 
	f.days_since_last_updated_days > 30
```
![enter image description here](https://i.imgur.com/IBjjbTP.png)

- Review and list all the columns and all rows for foods and drinks datasets.
```SQL
select 
	  f.food_id
	, null as drink_id
	, f.item_name
	, f.storage_type
	, f. package_size
	, f.package_size_uom
	, f.brand_name
	, f.package_price
	, f.price_last_updated_ts
	, 'foods_data' as source_table
From 
	foods f

union all 

select 
	   null as food_id
	  ,d.drink_id
	, d.item_name
	, d.storage_type
	, d.package_size
	, d.package_size_uom
	, d.brand_name
	, d.package_price
	, d.price_last_updated_ts
	, 'drinks_data' as source_table
From 
	drinks d
```
- Review and list specific columns and rows for the inventory and foods datasets. 
```SQL
select 
	i.food_inventory_id
	,i.food_item_id
	,i.quantity
	,i.inventory_dt
From 
	inventories i

select 
	  f.food_id
	, f.item_name
	, f.storage_type
	, f. package_size
	, f.package_size_uom
	, f.brand_name
	, f.package_price
	, f.price_last_updated_ts
from 
	foods f
```
```SQL
select 
	max(i.inventory_dt) as max_inventory_dt
From 
	inventories i
	
select 
	i.food_inventory_id
	,i.food_item_id
	,i.quantity
	,i.inventory_dt
from 
	inventories i
where
	i.inventory_dt = (
						select
							max(i.inventory_dt) as max_inventory_dt
						from 
							inventories i
					 )
```
- Lastly, using a `left join` command to join the food and inventory datasets
```SQL			 
select 
	  f.food_id
	, f.item_name
	, f.storage_type
	, f. package_size
	, f.package_size_uom
	, f.brand_name
	, f.package_price
	, f.price_last_updated_ts
	, i.quantity as inventory_quantity
from 
	foods f
		 left join( 
					select 
						i.food_inventory_id
						,i.food_item_id
						,i.quantity
						,i.inventory_dt
					from 
						inventories i
					where
						i.inventory_dt = (
											select
												max(i.inventory_dt) as max_inventory_dt
											from 
												inventories i
										)
			) i
			on f.food_id = i.food_item_id
```
![enter image description here](https://i.imgur.com/LhhvZYT.png)

