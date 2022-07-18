# Exploring LEGO Dataset with SQL | Data-Analysis Project

In this project, i went through the data model of the Rebrickable LEGO Dataset and then create a database for analytics. i went through real scenarios of answering questions about data with SQL. i went from basic SQL queries to exploring complex problems using JOINS, CTEs, and WINDOW functions.

I work on the following skills:

- Using SQL Server Integration Services (SSIS) to import data from CSV files into SQL Server Database.
- Creating a Database in SQL Server
- Two different methods of Importing Files into a Database
- SQL Joins
- SQL Window Functions
- Common Table Expressions (CTEs)


## LEGO Data Analysis - Rebrickable
The dataset is from  [LEGO Dataset](https://rebrickable.com/downloads/)

From the Rebrickable website - "The LEGO Parts/Sets/Colors and Inventories of every official LEGO set in the Rebrickable database is available for download as csv files here. These files are automatically updated daily. If you need more details, you can use the API which provides real-time data, but has rate limits that prevent bulk downloading of data."

Rebrickable provides this database to be used for any purpose.

## LEGO Data Model



![](LEGO_DataModeling.png)


## CREATE VIEW

```
Create view [dbo].[analytics_main] as

select s.set_num, s.name as set_name, s.year, s.theme_id, cast(s.num_parts as numeric) num_parts, t.name as theme_name, t.parent_id, p.name as parent_theme_name,
case 
	when s.year between 1901 and 2000 then '20th_Century'
	when s.year between 2001 and 2100 then '21st_Century'
end
as Century
from dbo.sets s
left join [dbo].[themes] t
	on s.theme_id = t.id
left join [dbo].[themes] p
	on t.parent_id = p.id
GO
```



## What is the total number of parts per theme
```
--select * from dbo.analytics_main
select theme_name, sum(num_parts) as total_num_parts 
from dbo.analytics_main
--where parent_theme_name is not null
group by theme_name
order by 2 desc
```


## What is the total number of parts per year

```
select year, sum(num_parts) as total_num_parts 
from dbo.analytics_main
where parent_theme_name is not null
group by year
order by 2 desc
```

## How many sets where created in each Century in the dataset

```
select Century, count(set_num) as total_set_num
from dbo.analytics_main
---where parent_theme_name is not null
group by Century
```

## What percentage of sets ever released in the 21st Century were Trains Themed 

```
;with cte as 
(
	select Century, theme_name, count(set_num) total_set_num
	from analytics_main
	where Century = '21st_Century'
	group by Century, theme_name
)
select sum(total_set_num), sum(percentage)
from(
	select Century, theme_name, total_set_num, sum(total_set_num) OVER() as total,  cast(1.00 * total_set_num / sum(total_set_num) OVER() as decimal(5,4))*100 Percentage
	from cte	
	--order by 3 desc
	)m
where theme_name like '%Star wars%'
```


## What was the popular theme by year in terms of sets released in the 21st Century

```
select year, theme_name, total_set_num
from (
	select year, theme_name, count(set_num) total_set_num, ROW_NUMBER() OVER (partition by year order by count(set_num) desc) rn
	from analytics_main
	where Century = '21st_Century'
		--and parent_theme_name is not null
	group by year, theme_name
)m
where rn = 1	
order by year desc
```


## What is the most produced color of lego ever in terms of quantity of parts?

```
select color_name, sum(quantity) as quantity_of_parts
from 
	(
		select
			inv.color_id, inv.inventory_id, inv.part_num, cast(inv.quantity as numeric) quantity, inv.is_spare, c.name as color_name, c.rgb, p.name as part_name, p.part_material, pc.name as category_name
		from inventory_parts inv
		inner join colors c
			on inv.color_id = c.id
		inner join parts p
			on inv.part_num = p.part_num
		inner join part_categories pc
			on part_cat_id = pc.id
	)main

group by color_name
order by 2 desc
```
