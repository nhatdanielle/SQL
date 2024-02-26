## 2022 Census of Employment and Wages

## Table of Content

* Introduction
* Data Source and Processing Tool
* EDA
* Conclusion

***

## Introduction
In this project, we will analyze the comprehensive 2022 employment data and wage statistics across all industries in the United States. Our primary objective is to identify the state and industry with the highest earning wages, allowing us to ascertain the average wage per capita in that particular region. Our focus will be on leveraging the data from the first quarter of 2022 to estimate annual wages for the entire year. MySQL will be the main tool to help explore and analyze these key indicators, we aim to gain valuable insights into the economic landscape, and prepare a understanding of the workforce dynamics in the United States.

## Data Source and Processing Tool
Data is collected from U.S. Bureau of Statistic
* 2022 Quarter Data
* 2022 Data by Industry
* Data is imported to Microsoft SQL Server Management Studio 

## EDA

#### 1. Quarter 1 2022, The US wages composition

````sql

WITH wage_cte AS (
    SELECT 
		[Ownership],
		ROUND(SUM([Total Quarterly Wages])/1000000,2) AS [Total Wages (Mil USD)],
		ROUND((SUM([Total Quarterly Wages]) / SUM(SUM([Total Quarterly Wages])) OVER()) * 100, 2) AS [Percentage Composition]

    FROM [df].[dbo].[2022_q1]
	WHERE 
		 [St] = 'US' and
		 [NAICS] = '10' and
		 [Own] <> '0'
    GROUP BY [Ownership]
)

SELECT TOP 10
	[Ownership],
    [Total Wages (Mil USD)],
	[Percentage Composition]
FROM
    wage_cte
ORDER BY
    [Total Wages (Mil USD)] DESC;

````
![image](https://github.com/nhatdanielle/SQL/assets/161078943/90f1a322-3f3f-4e91-8621-f2fa63566421)


#### Step
* Use **Group By** to summary total wages and percentage composition of ownership types in the United States during the first quarter of 2022.

#### Comment

* Private ownership takes the largest portion of the US employment at 86.93% wages paid in the first quarter of 2022

#### 2. Highest earning wage industry in the first quarter of 2022

````sql
SELECT TOP 10
    [St Name],
    [Industry],
	ROUND([Private] / 1000000, 2) AS [Q1 Wages (Mil USD)]

FROM (
    SELECT
        [St Name],
        [Ownership],
        [Industry],
        [Total Quarterly Wages]
    FROM [df].[dbo].[2022_q1]
    WHERE [NAICS] <> '10' AND
        [Area Type] = 'State' 
) AS SourceTable
PIVOT (
    SUM([Total Quarterly Wages]) FOR [Ownership] IN ([Private])
) AS PivotTable
ORDER BY [Private] DESC;

````
![image](https://github.com/nhatdanielle/SQL/assets/161078943/3bba414b-3032-4239-91df-3fee7786d04f)

#### Step
* Pivot to summary wages by state and industry in the first quarter of 2022.
* Only show the 10 highest earning states

#### Comment

* During the first quarter of 2022, the Service industry emerges as a dominant force, securing 8 out of the top 10 positions in the list of highest-earning industries in the United States. This underscores the pivotal role played by the Service sector as the primary economic driver for the country.

* California's Leadership: Among these high-performing states, California stands out as a leader, securing the No. 1 position in the Service industry category. The state's remarkable contribution amounts to $2,666 billion, emphasizing California's significant role in driving the country's economic prosperity. Last but not least, California also secures the No. 5 position in the Professional and Business Services category. With a substantial contribution of $82.906 billion, California demonstrates its influence across multiple economic sectors.

#### 3. Employment and quarterly wages in first quartef of 2022

````sql

WITH total_employment_cte AS (
    SELECT
        [Area_Code],
        [St Name],
        ROUND(AVG([January Employment] + [February Employment] + [March Employment]) / 3 / 1000, 2) AS [Average Employment (Thousands)],
        ROUND(AVG([Total Quarterly Wages]) / 1000000, 2) AS [Total Quarterly Wages (Mil USD)]
    FROM
        [df].[dbo].[2022_q1]
    WHERE
        [Ownership] = 'Private' AND [Industry] = '102 Service-providing' AND [Area Type] = 'State'
    GROUP BY
        [Area_Code],
        [St Name]
)

SELECT
    [Area_Code],
    [St Name],
    [Average Employment (Thousands)],
    [Total Quarterly Wages (Mil USD)],
    ROUND([Total Quarterly Wages (Mil USD)] / [Average Employment (Thousands)] * 1000, 2) AS [Average Quarter Wages (USD)],
    ROUND([Total Quarterly Wages (Mil USD)] / [Average Employment (Thousands)] * 1000 * 4, 2) AS [Estimated Annual Wages (USD)]
INTO 2022_q1_summary
FROM 
    total_employment_cte
ORDER BY 
    [Estimated Annual Wages (USD)] DESC;

SELECT *
FROM [df].[dbo].[2022_q1_summary]

````
#### Result

![image](https://github.com/nhatdanielle/SQL/assets/161078943/2f426b58-eef5-4e07-a5bc-f10860a5b352)

#### Step
* Calculates and summarizes average employment and total quarterly wages for private ownership in the Service Providing industry for each state during the first quarter of 2022.
* Estimate the annual wage of Service Providing industry per capita in 2022 based on the first quarter of 2022.
* The results are stored in a new table (2022_q1_summary) and ordered based on estimated annual wages in descending order

#### Comment

* Despite having a relatively low employment figure of 486,550 in the Service Providing industry, Washington DC stands out with the highest estimated annual wage per capita at an impressive $117,185.
* This finding suggests that, on average, workers in the Service Providing industry in Washington DC may earn more compared to their counterparts in other states, including California.
* California emerges as the leader in terms of employment, boasting the highest number of workers in the Service Providing industry among all states at 12,537,000.
* Despite its dominance in employment, California ranks 7th nationally in terms of average annual earnings. This indicates that while the state offers abundant opportunities in the Service Providing industry, workers might not necessarily have the highest average wages.

#### 4. 2022 Service providing's wages by state

````sql

WITH annual_wage_cte AS (
    SELECT 
		[Area_Code], [St Name], [Total Quarterly Wages]
	FROM 
		[df].[dbo].[2022_q1]
    WHERE 
	[Ownership] = 'Private' and [Area Type] = 'State' and [Industry] = '102 Service-providing'
	UNION ALL
    SELECT 
		[Area_Code], [St Name], [Total Quarterly Wages] 
	FROM 
		[df].[dbo].[2022_q2] 
    WHERE 
		[Ownership] = 'Private' and [Area Type] = 'State' and [Industry] = '102 Service-providing'
	UNION ALL
    SELECT 
		[Area_Code], [St Name], [Total Quarterly Wages] 
	FROM 
		[df].[dbo].[2022_q3]
    WHERE 
		[Ownership] = 'Private' and [Area Type] = 'State' and [Industry] = '102 Service-providing'
	UNION ALL
    SELECT 
		[Area_Code], [St Name], [Total Quarterly Wages] 
	FROM 
		[df].[dbo].[2022_q4] 
	WHERE 
		[Ownership] = 'Private'  and [Area Type] = 'State' and [Industry] = '102 Service-providing'
)
SELECT 
	TOP 10 [Area_Code], 
		[St Name], 
		ROUND(SUM([Total Quarterly Wages])/1000000,3) as [2022 Wages (Mil USD)],
		ROUND((SUM([Total Quarterly Wages]) / SUM(SUM([Total Quarterly Wages])) OVER()) * 100, 2) AS [Percentage Composition]

FROM 
	annual_wage_cte
GROUP BY 
	[St Name],
	[Area_Code]
Order BY 
	[2022 Wages (Mil USD)] DESC;

````
#### Step
* Use **Union All** to merge data from 4 tables: 2022 Quarter 1, 2022 Quarter 2, 2022 Quarter 3, 2022 Quarter 4.
* Use **Sum** to calculate 2022 Wages for each state in private industry sector.
* Percentage Compposition = Each State's Wages / 2022 National Wages
* Only show the 10 highest earning states

#### Result

![image](https://github.com/nhatdanielle/SQL/assets/161078943/c479d5c5-c18a-4db9-afee-652d461cdb19)

#### Comment
* As expecyed in 2022, California emerges as the leading state, contributing the largest portion of the total wages in the United States. California's share amounts to an impressive 14.3% of the industry wages, equivalent to a substantial $1,051 billion USD.

#### 5. Compare estimated annual wage vs actual wage in 2022

````sql
SELECT 
    [Area_Code],
    [St Name],
    [Average Employment (Thousands)],
    [Total Quarterly Wages (Mil USD)],
    [Average Quarter Wages (USD)],
    [Estimated Annual Wages (USD)],
	[avg_annual_pay] As [Actual Annual Wages (USD)],
	ROUND(([avg_annual_pay] - [Estimated Annual Wages (USD)]),2) AS [Difference],
	ROUND(([avg_annual_pay] - [Estimated Annual Wages (USD)]) / [avg_annual_pay],2) AS [Percentage]
FROM 
    [df].[dbo].[2022_q1_summary]
INNER JOIN 
    [df].[dbo].[2022_Service_providing] ON [2022_q1_summary].[Area_Code] = [2022_Service_providing].[area_fips]
WHERE
    [2022_Service_providing].[own_title] = 'Private'
ORDER BY 
    [Estimated Annual Wages (USD)] DESC;
````

#### Step 

* Use **JOIN** to merge Actual Annual Wage from table 2022_service_providing to table 2022_q1_summary

#### Result

![image](https://github.com/nhatdanielle/SQL/assets/161078943/c117454c-7c6d-40fa-aacb-79a9a852cd9b)

#### Comment

* The estimated annual wages per capita is very close to the actual wages in 2022 for most of the states. The percentage of diffences are approximately low under 5%.
* New York and Connecticut have unusual variance between estimated wages and actual wages. Both states have higher estimated wages than actual wages. This could be lower number of employment in Service providing industry during the year due to the shift in employment opportunity or the unsual raise of unemployment rate. This could be analyzed further by look deeper into other industries or unemployment rate throughout the year.
