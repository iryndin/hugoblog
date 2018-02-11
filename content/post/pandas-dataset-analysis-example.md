+++
date = "2018-02-11T16:04:46-05:00"
draft = false
title = "Pandas dataset analysis example"
tags = ["pandas", "machine learning", "data science"]
+++

Let's show how can we use [Pandas](https://pandas.pydata.org/) for data analysis. 
Let's take a [cardio dataset](https://github.com/iryndin/Machine-Learning/blob/master/data/01-cardio/cardio.csv) 
from here: https://github.com/iryndin/Machine-Learning/tree/master/data/01-cardio 

<!--more-->

## Task 0. Import dataset

First, import necessary libs and read the data: 

```
import pandas as pd
import numpy as np

df = pd.read_csv('cardio.csv', sep=';', index_col='id')

```

Now we have DataFrame (`df`), which is one of main entities Pandas work with. 

This dataset contains following fields:

| Field         | Description  |
| ------------- | ------------- | 
| id      | Patient ID |
| age      | Age in days |   
| gender | Male or female  |
| age      | Age in days |  
| height      | Height in cm |  
| weight      | Weight in kg |  
| ap_hi      | Arterial Pressure (high) |  
| ap_lo      | Arterial Pressure (low) |  
| cholesterol  | ? |  
| gluc      | ? |  
| smoke      | Smoking or not |  
| alco      | Drinks alco or not |  
| active      | ? |  
| cardio      | Has cardio probem or not |  

## Task 1. Count males and females

Let's count total number of males and females. OK, we have a column `gender` which contains values `1` and `2`.
But which of them is male, and which is female? It is not clear. 

We can determine this using reasonable assumption that males are in average higher than females. So, we need to filter those rows
which have `gender=1` and calculate mean height for them. Then we should calculate mean height value for rows with `gender=2`. 
Then we should check which of values if bigger, and hence assume that those are for males. Let's do this: 

```
>>> df[df['gender'] == 1]['height'].mean()
161.35561168460356
>>> df[df['gender'] == 2]['height'].mean()
169.94789538210054
```

Here we see that for rows with `gender=2` mean height is greater that for rows with `gender=1`. 
So we can conclude that `gender=2` is for males, while `gender=1` is for females. Let's create separate dataframes
bot for males and females:

```
males = df[df['gender'] == 2]
females = df[df['gender'] == 1]
```

Now let's answer this task's question about number of males and females:

```
>>> males['age'].count()
24470
>>> females['age'].count()
45530
```

So the answer to Task 1 is: 

* 24470 males
* 45530 females



## Task 2. Calculate who drinks alcohol less (in average): males or females

Alcohol usage column is `alco` and it contains values `1` (in use) and `0` (not in use). 
So we simply need to calculate a portion of alco drinkes for males and females and compare these values. 

We can do it in two ways: 

* filter by `alco` column value and calculate proportion
* simply sum `alco` column values and divide by total number of males/females respectively. 

Let's do both. First one: 

```
>>> males[males['alco']==1]['age'].count()/float(males['age'].count())
0.10637515324887617
>>> females[females['alco']==1]['age'].count()/float(females['age'].count())
0.025499670546892159
```

Second approach:

```
>>> males['alco'].sum()/float(males['age'].count())
0.10637515324887617
>>> females['alco'].sum()/float(females['age'].count())
0.025499670546892159
```

We see that portion of males drinking alco is 10.6%, while for females it is 2.5%. 

So the answer to Task 2 is: females drink alco less.


## Task 3. Calculate how many times greater percentage of smoking males is compared to that of females?

Let's calculate this:

```
>>> smoking_males_percentage = 100*males[males['smoke']==1]['age'].count()/float(males['age'].count())
>>> smoking_females_percentage = 100*females[females['smoke']==1]['age'].count()/float(females['age'].count())
>>> round(smoking_males_percentage/smoking_females_percentage)
12.0
```

Answer to Task 3 is: 12

## Task 4. In what units age is measured here? What is difference (in months) between median age of smokers and non-smokers?

OK, let's look at age stats:

```
>>> df['age'].describe()
count    70000.000000
mean     19468.865814
std       2467.251667
min      10798.000000
25%      17664.000000
50%      19703.000000
75%      21327.000000
max      23713.000000
Name: age, dtype: float64
```

Ok, this is not similar to years, and even to months. May be in days? Let's check: 

```
>>> df['age'].mean()/365
53.339358395303321
```

This is better. So age here is measured in days. Now let's calculate difference (in months) between median age of smokers and non-smokers:

```
>>> round((df[df['smoke']==0]['age'].median() - df[df['smoke']==1]['age'].median())/30)
20.0
```

The answer for Task 4 is: 

* age here is in days
* difference (in months) between median age of smokers and non-smokers: 20


## Task 5. Create 2 special datasets from initial one and calculate in how many times portion of sick people in one dataset is greater than that of another dataset?

Look at Wikipedia article about [HeartScore](https://ru.wikipedia.org/wiki/%D0%A1%D0%B5%D1%80%D0%B4%D0%B5%D1%87%D0%BD%D0%BE-%D1%81%D0%BE%D1%81%D1%83%D0%B4%D0%B8%D1%81%D1%82%D1%8B%D0%B9_%D1%80%D0%B8%D1%81%D0%BA), 
it gives clarification to following actions. 

Do following: 

* Create new column - `age_years`, round its values to integers. Then for this task take only smoking males aged 60 to 64 (inclusive).
* In the dataset cholesterol level 1 is equal to 4 mmol/liter, 2 -> 5-7 mmol/liter, 3 maps to 8 mmol/liter.
* Create 2 datasets out of smoking males 60-64: `ap_hi < 120` and `cholesterol=1` , another:  `160 <= ap_hi < 180` and `cholesterol=3`
* For given 2 datasets calculate in how many times portion of cardio-sick people in one dataset is greater than that of another dataset

Do this: 

```
>>> males['age_years'] = males.apply(lambda row: int(round(row.age/365.0)), axis=1)
>>> smoking_males = males[(males['smoke']==1) & (males['age_years']>=60) & (males['age_years']<65)]
>>> smoking_males_1 = smoking_males[(smoking_males['cholesterol']==1) & (smoking_males['ap_hi']<120)] 
>>> smoking_males_2 = smoking_males[(smoking_males['cholesterol']==3) & (smoking_males['ap_hi']<180) & (smoking_males['ap_hi']>=160)] 
>>> cardio_percentage_1 = 100*smoking_males_1[smoking_males_1['cardio']==1]['age'].count()/float(smoking_males_1['age'].count())
>>> cardio_percentage_2 = 100*smoking_males_2[smoking_males_2['cardio']==1]['age'].count()/float(smoking_males_2['age'].count())
>>> round(cardio_percentage_2/cardio_percentage_1)
3
```

Answer to Task 5: 3


## Task 6. Assess some BMI statements

BMI (Body Mass index) is calculated as weight in kg divided by square of height in meters. Normal BMI values are between 18.5 and 25. 

Assess following statements (is it true of false): 

* Median BMI is greater than its normal values
* Females BMI is in average less than males BMI
* Cardio-healthy persons have in average greater BMI than cardio-sick ones
* For cardio-healthy non-alco males BMI values in average are closer to norm than for cardio-healthy non-alco females

Solution: 

```
>>> df['bmi'] = df.apply(lambda row: row.weight/((row.height**2)/10000.0), axis=1)
>>> df['bmi'].median()
26.374068120774975
>>> df[df['gender']==1]['bmi'].mean()
27.98758344183285
>>> df[df['gender']==2]['bmi'].mean()
26.754442357289349
>>> df[df['cardio']==0]['bmi'].mean()
26.548175206794504
>>> df[df['cardio']==1]['bmi'].mean()
28.56606062701535
>>> df[(df['cardio']==0) & (df['alco']==0) & (df['gender']==2)]['bmi'].mean()
25.872638075460163
>>> df[(df['cardio']==0) & (df['alco']==0) & (df['gender']==1)]['bmi'].mean()
26.845406594131507
```

Answer to Task 6:

* Median BMI is greater than its normal values. TRUE. (Because `26.374068120774975 > 25`)
* Females BMI is in average less than males BMI. FALSE (Because `27.98758344183285 > 26.754442357289349`)
* Cardio-healthy persons have in average greater BMI than cardio-sick ones. FALSE. (Because `26.548175206794504 < 28.56606062701535`)
* For cardio-healthy non-alco males BMI values in average are closer to norm than for cardio-healthy non-alco females. TRUE. (Because `25.872638075460163` is closer to `25` than `26.845406594131507`)

## Task 7. Clean up data. 

Filter data according to the following (throw out rows matching following criteria):

* `ap_lo > ap_hi`
* height is less than 2.5 percentile or greater than 97.5 percentile
* weight is less than 2.5 percentile or greater than 97.5 percentile

How many percents of data was thrown out?

```
>>> dfclean = df[df['ap_lo'] <= df['ap_hi']]
>>> h025 = df['height'].quantile(0.025)
>>> h975 = df['height'].quantile(0.975)
>>> dfclean = dfclean[(dfclean['height'] >= h025) & (dfclean['height'] <= h975)]
>>> w025 = df['weight'].quantile(0.025)
>>> w975 = df['weight'].quantile(0.975)
>>> dfclean = dfclean[(dfclean['weight'] >= w025) & (dfclean['weight'] <= w975)]
>>> size = df['age'].count()
>>> sizeclean = dfclean['age'].count()
>>> round(100.0*(1 - sizeclean/float(size)))
10
```

Answer to Task 7: 10


## Links

* [ML Course Open](https://github.com/Yorko/mlcourse_open) 
* [ML Course Open. Lecture 1](https://habrahabr.ru/company/ods/blog/322626/)
* [ML Course Open. Hometask 1.](http://nbviewer.jupyter.org/github/Yorko/mlcourse_open/blob/master/jupyter_russian/homeworks/hw1_session3_data_analysis_pandas.ipynb?flush_cache=true)
* [Pandas Cheat Sheet](https://www.dataquest.io/blog/pandas-cheat-sheet/)




