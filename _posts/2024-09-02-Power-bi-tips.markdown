---
layout: post
title:  "Taming requirements in Power BI"
date:   2024-09-02
categories: PowerBI
---

**Introduction** <br><br>
Working with complex customer requirements can not only be a technical DAX challenge, but also a large effort related to planning of the development of the Power BI data model / data set. <br><br>

In your Power BI journey you will encounter various approaches to problem modelling, and many calculations with logic you might consider odd. For example, if you had previous experience in SQL, you might be tempted to write calculated columns, or use DAX in a way that is not optimal for performance.If, on the other hand, you're a python programmer, you might be consider writing a function that does the same thing as built-in DAX function. <br><br>

Some developers do not write any DAX and just use built-in aggregations provided by Power BI Desktop. While that is a good approach for beginners, it is not considered a best practice in the industry. Some will create multiple calculated columns, and use those in measures. That is what I found myself doing in the beginning quite a lot, and there are quite many scenarios, where a calculaed column is indeed necessary, but this won't be covered in this post. <br><br>

And finally you have people who would write measures organized in a ‘Measures’ folder/table, and use variables to simplify the DAX (by far the most sensible approach, endorsed by many professionals). <br><br>

While those approaches may appear very similar, they differ substantially in terms of how Power BI does these calculations and comes up with results. DAX is a language that relies  on ‘row’ or ‘filter’ context, that means you can either navigate in your expressions like, for instance, in excel (‘row’ context) or work on multiple related tables at once, like you would in relational database (‘filter’ context). <br><br>

**Consequences of speeding things up** <br><br>
In the introduction mentioned creating calculated columns and using those in measures or other calculations. While that may not be the worst approach, performance-wise, it makes your data model gain weight (Megabytes of weight, in fact). Depending on the volume of data you’re dealing with, it might be worth avoiding calculated columns. <br>
Writing DAX measures exclusively is a perfect world scenario, which we don’t live in. In All branches of industry doing analytics and reporting, the data is so disparate, that one, by-the-book modelling approach does not apply to all real-life scenarios. On top of that, we often have to deal with technical debt, which is a result of not having enough time to implement the best practices. <br><br>

Some calculations your end users might want to see, like running totals, ‘lag’ calculations, or partitioning are easier to achieve in your Data Warehouse (DWH). The Power BI team at Microsoft is working on delivering more DAX functions to Power BI Desktop, however it may take a while for them to finally appear. <br><br>
In some scenarios, it may be better to prepare a view, with your desired calculations as per the customer’s requirements. A view is essentially a table that only exists as a script, saved in your DWH or data mart. When the script is invoked, the table is served to the user or person calling the script, thus only putting load on the DB engine when in use. When choosing this approach, it is important to keep in mind, that for Power BI reporting you must consider the data types of key attributes like dates, or other columns that will serve as the foundations of DAX expressions.

**Simplify DAX** <br><br>
(using variables & return, using lesser-known functions, splitting calculations into steps with examples)
Many will agree, that the first, unspoken rule of DAX inside Power BI is to ‘Keep it simple’ .
But, in order to ‘Keep it simple’, one must first understand the axioms and inner workings of Power BI to a certain extent.
If your team has a data engineer, ask him/her to perform some of those transformations, and serve the data as a view. Some are difficult to achieve in DAX or very verbose, leading to more lines of code, which we ultimately don’t want, as that will be harder to maintain. <br>
Furthermore, it is not a bad idea to introduce variables into your DAX measure. Variables are created in the following way:

```DAX
VAR myVariable = CALCULATE(SUM('Table'[Column]), 'Table'[Column] = "Value")
```

Doing this greatly improves readability of DAX code. Note, that in order to use variables, you must then use a ‘RETURN’ statement at the end of your measure (The keyboard shortcut to break lines in DAX is shift+enter / alt+enter). Like so:

```DAX
VAR myVariable = CALCULATE(SUM('Table'[Column]), 'Table'[Column] = "Value")
RETURN DIVIDE( myVariable, 10, 0)
```

Putting down your thought process on paper might help you understand which parts of your thought process can be treated as variables. Proper formatting – using indentation on nested functions – is always welcome. There is even an online tool you might use for that [DAX Formatter](https://www.daxformatter.com/).

**Handling relationships, cardinality & cross-filter direction** <br><br>
In PBI, users need to be aware that relationships are an inseparable part of the data model. Consciously creating those relationships with the right parameters can be a challenge even for experienced developers. Under no conditions should we try to evade relationships, as they are the backbone of the data model. <br><br>
Cardinality in Power BI refers to the relationship between tables in a data model. It determines how rows in one table are related to rows in another table. 
There are three types of cardinality in Power BI:
 - **One-to-One**: Each row in one table is related to exactly one row in another table. <br><br>
 - **One-to-Many**: Each row in one table can be related to multiple rows in another table, but each row in the other table is related to only one row in the first table. <br>
 **Example**: A customer can have multiple orders, but each order is associated with only one customer. <br><br>
 - **Many-to-Many**: Each row in one table can be related to multiple rows in another table, and each row in the other table can be related to multiple rows in the first table. <br>
 **Example**: A student can enroll in multiple courses, and a course can have multiple students. <br><br>
 Establishing the correct cardinality is crucial for creating accurate relationships and performing effective data analysis in Power BI.

<!-- 
Title: Power BI Tips
Filepath: /d:/jekyll-projects/DrBo/_posts/2024-09-02-Power-bi-tips.markdown
Description: This article discusses the concept of bidirectional filtering in Power BI models and provides tips on when to consider using it.
-->

<!-- 
For more information on bidirectional filtering in Power BI, you can refer to the official Microsoft documentation: 

-->
As we develop power BI models, we sometimes forget to consider the need of [Bidirectional filtering](https://docs.microsoft.com/en-us/power-bi/transform-model/desktop-bidirectional-filtering). 
This feature allows both tables on either side of the relationship to filter one another. However, this reduces overall report performance. This is not always necessary in reports, therefore it is worth setting the cross-filter direction to ‘single’, where requirements allow. 


**Other aspects of performance**<br>
While the points above are more or less ordered by importance, there is one more aspect that should not be overlooked, and that is performance. In a whirlwind of DAX writing and visual creation we often forget about the performance of our report. <br>

Guidelines for keeping performance at a acceptable level include: <br>
 - reducing or controlling the amount of visuals on your page,
 - don’t reinvent the wheel: use built-in DAX functions,
 - occasionally, fire up the performance analyzer, to optimize your report.

**Optimizing visuals**<br>
Ask yourself the following questions: <br><br>
*What is this page trying to tell me?* <br>
*Are the visuals on this page related and interactive?* <br>
*‘Can this page be read from left to right?* <br>

and so on.

Be aware that DAX offers a breadth of functions, which are often overlooked. For reference you may find more information on [DAX Guide](https://dax.guide/), which is written and updated by SQLBI, a company specializing in Business Intelligence and SQL, holding many developer events anually. <br>
Here are some examples of functions that are very useful: <br><br>
`CALENDAR()`, `SAMEPERIODLASTYEAR()`, `PARALLELPERIOD()`, `DATESYTD()`, `DIVIDE()`, `CONCATENATEX()`, `DATESBETWEEN()`, `DATESINPERIOD()`, `EOMONTH()`, `FIRSTNONBLANK()`, `HASONEVALUE()`, `SELECTEDVALUE()`, `ISCROSSFILTERED()`

and many more… <br><br>
**Organize your measures** <br>
When your data model grows quickly, and new measures are added, that’s where things start to get difficult to manage. You may have measures referencing other measures, which reference other measures. In order to make life a bit easier for the next person, organize your DAX calculations into a table called ‘Measures’. Such a table can be created with the following code, when creating a new table:
```DAX
_Measures = {1,1} //creates a table with only one value, meant to store measures
```

Now, you can put your measures into this table by right-clicking on its name in the table list, and selecting ‘new measure’. This, single-handedly makes picking up your report much faster, thus reducing amount of time spent on implementing changes. Power BI is developing a tool called ‘Model Explorer’, which will prove handy for this particular purpose in the future. <br><br>
**Naming conventions** <br>
Finally, on top of the aforementioned suggestions, you may want to implement a naming convention into your measures. If you have measures referencing other measures, consider giving them a number, based on the hierarchy of reference.Or you could address the matter differently, by naming your measures according to what they do. Give names, that describe what the measure does, trying, as a rule of thumb, not to exceed 20 characters.

Use abbreviations like ‘avg’, ‘cnt’, ‘agg’, ‘fst’ (first), ‘lst’ (last), ‘CY’ (calendar year), ‘PY’ ( Previous year), ‘SPLY’ (`SAMEPERIODLASTYEAR()`() ), ‘SV’ (`Selectedvalue()` ), ‘HOV’ (`Hasonevalue()` ) etc…
