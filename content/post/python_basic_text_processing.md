+++
Tags = ["machine learning", "pandas", "text processing", "python"]
# Description = ""
date = "2018-06-10T20:17:56-04:00"
title = "Python basic text processing"
# menu = "main"
# Categories = ["Development","GoLang"]
+++

Let's demonstrate some basic text processing techniques available with Pandas. 
We will discuss some basic feature extraction methods like number of words, number of characters, 
basic preprocessing like lower casing, stopwords removal, punctuation removal, as well as some more 
advanced methods like extracting N-grams, TF/IDF calculation, sentiment analysis and word embedding. 

<!--more-->

## Dataset 

We take Twitter sentiment analysis dataset from here: [Twitter sentiment analysis problem](https://datahack.analyticsvidhya.com/contest/practice-problem-twitter-sentiment-analysis).
You can find it locally: here.

First, let's explore this dataset and find out what data it has and in what form:
```python
import pandas as pd
df = pd.read_csv('train_E6oV3lV.csv')
df.columns
df['label'].value_counts()
```

So, we can see that dataset contains 3 columns: `id`, `label` and `tweet`. Column `label` has 29270 zeroes, and 2242 ones. 

Right now we are not much interested in `id` and `label`, we just look into text (i.e. data in `tweet` column) 
and demonstrate several techniques and analysis methods that can be applied to the text.  

## Basic feature extraction 

We extract following features from the text: 

* Number of words
* Number of characters
* Average word length
* Number of stopwords
* Number of special characters
* Number of numerics
* Number of uppercase words

### Number of words

### Number of characters

### Average word length

### Number of stopwords

### Number of special characters

### Number of numerics

### Number of uppercase words

## Basic text preprocessing

## More advanced text processing

## Links

[Ultimate guide to deal with Text Data (using Python) – for Data Scientists & Engineers](https://www.analyticsvidhya.com/blog/2018/02/the-different-methods-deal-text-data-predictive-python)