---
layout: post
title: Cumulative totals three different ways in data science
---


![image](http://nancyfriedman.typepad.com/.a/6a00d8341c4f9453ef01a73dafb230970d-pi =400x300) 

The data analysis field has moved away from querying tools like Crystal Reports (**shudder**), OLAP cubes, and Excel, to programming languages closer to the raw data. 

I primarily work with IPython notebooks, R, and SQL these days, and I thought it would be interesting to look at the differences between these three tools and three ways of thinking through a single task. 

Something people often ask of data scientists (or analysts or engineers or what have you) is to just find out how many total customers the company has from the very beginning on a cumulative basis. Often product heads like to see this to figure out if their sexy new product is showing that legendary [hockey stick growth](http://lunarmobiscuit.com/the-hockey-stick/). 

Often times, this data is only available at a granular level (i.e. you have a base number and the adds every month.) For example, let's say we want to find out how many total employees each of these Silicon Valley Companies have in any given month. Are they growing as quickly as Hooli? 


Here's the CSV file you're given to work with: (also available in [the associated GitHub repo](https://github.com/veekaybee/cumtotal) for all the code in this post. ) 

|Company	| Month	| Employees|
| ------------- |:-------------:| -----:|
|Hooli	| Jan-2014	|123,456|
|Hooli	| Feb-2014	|1,434|
|Hooli	| Mar-2014|	2,455|
|Pied Piper	| Jan-2014|	1|
|Pied Piper	| Feb-2014|	2|
|Pied Piper|	 Mar-2014|	2|
|Raviga|	Jan-14|	50|
|Raviga	| Feb-2014|	-2|
|Raviga	| Mar-2014|	17|



But what we really want is this: 

|Company	| Month	| Employees|
| ------------- |:-------------:| -----:|
|Hooli	| Jan-2014	|123,456|
|Hooli	| Feb-2014	|124,890|
|Hooli	| Mar-2014|	127,345 |
|Pied Piper	| Jan-2014|	1|
|Pied Piper	| Feb-2014|	3|
|Pied Piper|	 Mar-2014|	5|
|Raviga|	Jan-14|	50|
|Raviga	| Feb-2014|	48|
|Raviga	| Mar-2014|	65|



What I often want to do is create a cumulative total, by month and by company. In Excel, this is super-easy. Just add a new column and add the previous row's values to it. I've shown the formula view: 

![Total](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/exceltot.png)

But, it's extremely annoying when there's any more than 100 rows of data, because then I'm resorting to cut and paste and manually tweaking. As soon as I start to do things manually, I make mistakes, and it's [not very good data practice](https://github.com/jtleek/datasharing) to have code you can't reproduce. 

Doing this kind of thing in more data science-y environments is great because then you have reproducible code you can run over and over again. However, it does take a bit more legwork and requires thinking about the problem in a little bit of a different way than just summing cells. 

So here are three typical ways to do cumulative totals in three pretty typical data science environments. 
  

###Cumulative totals in Python

Python, like most programming languages, performs operations over rows of data sequentially, stopping when it hits a new column. (for example, it doesn't see data as a matrix, but more as individual values. ) 

This is a "problem" for data people with all languages: Software developers often see data as means to an end (compiling the program). This means data doesn't need to be organized exactly, just well enough to move through different pipes.  Whereas for data people, data is the end goal. (If you're interested in more about this, I gave a presentation a year or so ago with graphics illustrating the topic [here](http://www.slideshare.net/vickiboykis/analyst-30599322).)


This "problem" (which is really just a feature of programming) is easy to solve with Pandas, which transforms data once again into matrices that need to be operated on in their entirerty. These matrices are called data frames. But it's a little unwieldy when performing operations on operations on matrices, so  [you have to do](http://stackoverflow.com/questions/22650833/pandas-groupby-cumulative-sum) a cumulative sum of a cumulative sum: 


###Import Pandas for working with data


    import pandas as pd
    from pandas import DataFrame
    import dateutil.parser as parser


###Read in CSV file 


```df=pd.read_csv('sv.csv')```

```print df```

          Company   Month  New Employess
    0       Hooli  14-Jan         123456
    1       Hooli  14-Feb           1434
    2       Hooli  14-Mar           2455
    3  Pied Piper  14-Jan              1
    4  Pied Piper  14-Feb              2
    5  Pied Piper  14-Mar              2
    6      Raviga  14-Jan             50
    7      Raviga  14-Feb             -2
    8      Raviga  14-Mar             17


###Make sure date and number values are rendered correctly from CSV file (the dateutil library)


```df['Month'] = pd.to_datetime(df['Month'])
    df.convert_objects(convert_numeric=True)```




<div style="max-height:1000px;max-width:1500px;overflow:auto;">
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Company</th>
      <th>Month</th>
      <th>New Employess</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>      Hooli</td>
      <td>2015-01-14</td>
      <td> 123456</td>
    </tr>
    <tr>
      <th>1</th>
      <td>      Hooli</td>
      <td>2015-02-14</td>
      <td>   1434</td>
    </tr>
    <tr>
      <th>2</th>
      <td>      Hooli</td>
      <td>2015-03-14</td>
      <td>   2455</td>
    </tr>
    <tr>
      <th>3</th>
      <td> Pied Piper</td>
      <td>2015-01-14</td>
      <td>      1</td>
    </tr>
    <tr>
      <th>4</th>
      <td> Pied Piper</td>
      <td>2015-02-14</td>
      <td>      2</td>
    </tr>
    <tr>
      <th>5</th>
      <td> Pied Piper</td>
      <td>2015-03-14</td>
      <td>      2</td>
    </tr>
    <tr>
      <th>6</th>
      <td>     Raviga</td>
      <td>2015-01-14</td>
      <td>     50</td>
    </tr>
    <tr>
      <th>7</th>
      <td>     Raviga</td>
      <td>2015-02-14</td>
      <td>     -2</td>
    </tr>
    <tr>
      <th>8</th>
      <td>     Raviga</td>
      <td>2015-03-14</td>
      <td>     17</td>
    </tr>
  </tbody>
</table>
</div>



###check each column's datatype


    
 ```print df.dtypes```

    Company                  object
    Month            datetime64[ns]
    New Employess           float64
    dtype: object



  ```print df.groupby(by=['Company','Month']).sum().groupby(level=[0]).cumsum()```

                           New Employess
    Company    Month                    
    Hooli      2015-01-14         123456
               2015-02-14         124890
               2015-03-14         127345
    Pied Piper 2015-01-14              1
               2015-02-14              3
               2015-03-14              5
    Raviga     2015-01-14             50
               2015-02-14             48
               2015-03-14            65 
               
               




##Cumulative Totals in R

R, in theory, operates on matrices, but it sees matrices in a pretty rigid, mathematical way, instead of the looser way they're implemented in databases. 

In order to do cumulative operations,   you can do this by converting your dataset to a more table-like format. (Check out [this link](http://stackoverflow.com/questions/22824662/calculate-cumulative-sum-of-one-column-based-on-another-columns-rank) for more detail on how to. )

```
install.packages("data.table", lib="/Library/Frameworks/R.framework/Versions/3.1/Resources/library") #get the data table package
sv <- read.csv("~/Desktop/ipythondata/sv.csv") #read in data
require(data.table) #package for transforming to data table
View(sv)
setDT (sv) #set the table as your dataset

setkey(sv, Company,Month) #sort in chronological order and groups
sv[,csum := cumsum(New.Employees),by=Company] #cumulative sum
View(sv) #view your results
```
![RTotal](https://raw.githubusercontent.com/veekaybee/veekaybee.github.io/master/images/rtotal.png)



##Cumulative Totals in SQL

This one is a little trickier because you have to start a database instance...somewhere. You can set up MySQL or Postgres locally.  

I have a Digital Ocean droplet that has Postgres installed. [There is a bunch of admin work](https://wiki.postgresql.org/wiki/First_steps) that will have to be done before you can create tables in Postgres, but then you're on your way on the command line: 


```
postgres@data:~$ psql
postgres=# CREATE SCHEMA employees; #creates the database where you'll be doing stuff
CREATE SCHEMA
postgres=#  CREATE TABLE cumtot(company CHAR(50) NOT NULL, month DATE NOT NULL,nemp NUMERIC NOT NULL); #create the table and specify the column types
CREATE TABLE
postgres=# \d #take a look a the column types
```
         List of relations
 Schema |  Name  | Type  |  Owner   
--------+--------+-------+----------
 public | cumtot | table | postgres
 ```
 
 Then we copy the data to the table from our file:
 ```
 postgres=# copy cumtot FROM '/data/sv.csv' DELIMITER ',' CSV HEADER;
COPY 9
```
In the file, we first have to change the date format because Postgres only takes certain formats http://www.postgresql.org/docs/9.1/static/datatype-datetime.html

So what we're importing is: 

```
Company,Month,NewEmployees
Hooli,2014-Jan-01,123456
Hooli,2014-Feb-01,1434
Hooli,2014-Mar-01,2455
Pied Piper,2014-Jan-01,1
Pied Piper,2014-Feb-01,2
Pied Piper,2014-Mar-01,2
Raviga,2014-Jan-01,50
Raviga,2014-Feb-01,-2
Raviga,2014-Mar-01,17
```
```
\d cumtot
        Table "public.cumtot"
 Column  |     Type      | Modifiers 
---------+---------------+-----------
 company | character(50) | not null
 month   | date          | not null
 nemp    | numeric       | not null
```

postgres=# select * from cumtot; #view the whole table

```
company         |   month    |  nemp  
--------------------------------------
 Hooli          | 2014-01-01 | 123456
 Hooli          | 2014-02-01 |   1434
 Hooli          | 2014-03-01 |   2455
 Pied Piper     | 2014-01-01 |      1
 Pied Piper     | 2014-02-01 |      2
 Pied Piper     | 2014-03-01 |      2
 Raviga         | 2014-01-01 |     50
 Raviga         | 2014-02-01 |     48
 Raviga         | 2014-03-01 |     65
```

That was just the gruntwork. Now we get to actually do the cumulative total, which requires a window function. [Window functions](http://sqlschool.modeanalytics.com/advanced/window-functions.html) in SQL seem complicated but they're pretty easy once you get the hang of them. They say, "don't look at this entire table, look at a portion of the table in a specific order." 

```
postgres=# select company, month, nemp, sum(nemp) OVER (PARTITION BY company ORDER BY month) as cum_tot from cumtot ORDER BY company, month;
```

```
  company                 |   month    |  nemp  | cum_tot 
----------------------------------------------------+------------
 Hooli                    | 2014-01-01 | 123456 |  123456
 Hooli                    | 2014-02-01 |   1434 |  124890
 Hooli                    | 2014-03-01 |   2455 |  127345
 Pied Piper               | 2014-01-01 |      1 |       1
 Pied Piper               | 2014-02-01 |      2 |       3
 Pied Piper               | 2014-03-01 |      2 |       5
 Raviga                   | 2014-01-01 |     50 |      50
 Raviga                   | 2014-02-01 |     -2 |      48
 Raviga                   | 2014-03-01 |     17 |      65 
 ```
 
 
So that's pretty much it. Three different approaches to cumulative totals, that will each give you the right answer. Some are better for specific use cases than others.  For small data sets, R and IPython should be just fine.  Once you get larger, Python and SQL will start to work better than R. If you already have this data in a SQL database, then there's no point in exporting it, and so on. 