+++
date = "2018-11-05T16:49:39Z"
title = "Kaggle What's cooking competition"
Tags = ["machine learning", 'kaggle', 'kaggle whats cooking']
+++

Solving Kaggle's amazing [What's cooking](https://www.kaggle.com/c/whats-cooking) competition using simple Bag of Words model and coding it by hands, 
without usage of any machine learning library.

<!--more-->

Kaggle's [What's cooking](https://www.kaggle.com/c/whats-cooking) competition is about guessing cousine by provided ingredients of the recipe. Train and test data come in JSON format and are pretty clear. Here are two first records from train data:

```
[
  {
    "id": 10259,
    "cuisine": "greek",
    "ingredients": [
      "romaine lettuce",
      "black olives",
      "grape tomatoes",
      "garlic",
      "pepper",
      "purple onion",
      "seasoning",
      "garbanzo beans",
      "feta cheese crumbles"
    ]
  },
  {
    "id": 25693,
    "cuisine": "southern_us",
    "ingredients": [
      "plain flour",
      "ground pepper",
      "salt",
      "tomatoes",
      "ground black pepper",
      "thyme",
      "eggs",
      "green tomatoes",
      "yellow corn meal",
      "milk",
      "vegetable oil"
    ]
  },
```

Target variable is `cousine`, and items in `ingredients` array are features using which we should classify (it is clearly classifiation task) to what cousine this recipe belongs. OK, let's explore our train data.

## Data exploration

### Available cousines

Let's find out how many cousines are available in train data and what is their recipe distribution. 
Below there is a small script that shows available cousines and a number of recipes for each, as well as draws bar chart of recipes distribution. 

```python
import json

with open('train.json') as data_file:    
  train = json.load(data_file)

print('Total number of recipes: %d' % len(train))

cousine_map = {}
for recipe in train:
  cousine_name = recipe['cuisine']
  recipe_count = cousine_map.get(cousine_name, 0)
  cousine_map[cousine_name] = recipe_count+1

# get a sorted (by number of recipes) list of cousines 
cousines = sorted(list(cousine_map.items()), key=lambda tup: -tup[1])       

def draw_cousines_barchart(cousines):
  import matplotlib.pyplot as plt
  import numpy as np

  names = [c[0] for c in cousines]
  values = [c[1] for c in cousines]

  fig_size = plt.rcParams["figure.figsize"]
  fig_size[0] = 12
  fig_size[1] = 8
  plt.rcParams["figure.figsize"] = fig_size

  index = np.arange(len(names))
  bars = plt.bar(index, values, align='center', alpha=0.5, 
               color=['darksalmon', 'sienna', 'gold', 'olivedrab', 'darkgreen',
                     'lightseagreen', 'darkturquoise', 'slategray', 'navy', 'darkorchid',
                     'plum', 'lightcoral', 'darkorange', 'lawngreen', 'g',
                     'c', 'violet', 'crimson', 'peru', 'aqua'])

  plt.xticks(index, names, fontsize=10, rotation=60)
  plt.ylabel('Nr of recipes')
  plt.title('Cousine')

  plt.show()

draw_cousines_barchart(cousines)  
```

And we get following barchart:

<img src="/img/kaggle_whats_cooking_cousine_barchart.png" style="width:80%" />

Facts: 

* Total number of recipes in train data: 39774 
* We see that three most used cousines are: `italian` (7838), `mexican` (6438) and `southern_us` (4320). In sum they have almost half (to be exact, 18596 recipes) of all the data.

Here is full distribution of recipes by cousine:

```
[
  ('italian', 7838), 
  ('mexican', 6438), 
  ('southern_us', 4320), 
  ('indian', 3003), 
  ('chinese', 2673), 
  ('french', 2646), 
  ('cajun_creole', 1546), 
  ('thai', 1539), 
  ('japanese', 1423), 
  ('greek', 1175), 
  ('spanish', 989), 
  ('korean', 830), 
  ('vietnamese', 825), 
  ('moroccan', 821), 
  ('british', 804), 
  ('filipino', 755), 
  ('irish', 667), 
  ('jamaican', 526), 
  ('russian', 489), 
  ('brazilian', 467)
]
```

### Ingredients analysis

Now let's find out how much unique ingredients are in the train dataset, what are 10 most frequently used ingredients overall.

```python
all_ingredients = {}
for recipe in train:
    ingredients = recipe['ingredients']
    for ing_name in ingredients:
        ing_count = all_ingredients.get(ing_name.lower(), 0)
        all_ingredients[ing_name.lower()] = ing_count+1

print("Total ingredients: %d" % len(all_ingredients))
```

Total ingredients: 6703

Now let's see 10 most used ingredients and its number of occurrences:

```python
# get a sorted (by number of occurences) list of ingredients 
sorted_ingredients = sorted(list(all_ingredients.items()), key=lambda tup: -tup[1])  
print(sorted_ingredients[:10])
```

Here are 10 most used ingredients accross all cousines:

```
[
   ('salt', 18049), 
   ('onions', 7972), 
   ('olive oil', 7972), 
   ('water', 7457), 
   ('garlic', 7380), 
   ('sugar', 6434), 
   ('garlic cloves', 6237), 
   ('butter', 4848), 
   ('ground black pepper', 4785), 
   ('all-purpose flour', 4632)
]
```

## Model

Model that is employed here is [Bag of words](https://en.wikipedia.org/wiki/Bag-of-words_model). The idea of _bag of words_ is to take into account each word 
multiplicity in the text disregarding grammar and word order at the same time. Thus, each word can be considered as a component of a vector of all the words (dictionary).

So let's consider each recipe as a document. Thus each ingredient is a word in the document. Bag of words algorithm will be following:

* For train data: for each cousine calculate all ingredients used in the recipes belonging to the cousine. 
For each ingredient in the cousine calculate number of occurrences. 
Also calculate total sum of all ingredient occurrences in the cousine - it will be used to normalize data. 
* For test data: for each recipe for all ingredients in this recipe calculate total number of occurrences in the each cousine. 
If ingredient is not used in the cousine - then add nothing to score. 
Then total score divide by total sum of all ingredient occurrences in the cousine.
Then simply select cousine with the largest score. 

For each cousine calculate number of rences for each ingredient and total sum of all occurrences:

```python
# this map contains ingredient occurrences for each cousine.
# keys are cousine names
# values are maps, for which keys are ingredient names, 
# and values are occurrences of this particular ingredient for all recipes 
# belonging to this cousine
cousine_ingredients = {}

# this map contains total sum of all occurrences of all ingredients for the cousine
# keys are cousine names
# values are total sum of all ingredient occurrences
cousine_totals = {}
for recipe in train:
    ingredients = recipe['ingredients']
    cousine_name = recipe['cuisine']
    cousine_map = cousine_ingredients.get(cousine_name, {})
    cousine_total_icount = cousine_totals.get(cousine_name, 0)
    for iname in ingredients:
        iname = iname.lower()
        icount = cousine_map.get(iname, 0)
        cousine_map[iname] = icount+1
        cousine_total_icount = cousine_total_icount + 1
    cousine_ingredients[cousine_name] = cousine_map
    cousine_totals[cousine_name] = cousine_total_icount
```

Now let's calculate cousine scores for each recipe in test data:
```python
with open('test.json') as test_data_file:    
    test_data = json.load(test_data_file)

recipe_map = {}
for recipe in test_data:
    recipe_id = recipe['id']
    recipe_ingredients = recipe['ingredients']
    recipe_cousine_map = {}
    for cousine_name in cousine_ingredients:
        cousine_map = cousine_ingredients[cousine_name]
        cousine_score = 0
        for iname in recipe_ingredients:
            ingredient_score = cousine_map.get(iname,0)
            cousine_score = cousine_score + ingredient_score            
        cousine_score_normalized = cousine_score/cousine_totals[cousine_name]
        recipe_cousine_map[cousine_name] = cousine_score_normalized
    recipe_map[recipe_id] = recipe_cousine_map   
```

In `recipe_map` keys are recipe IDs. Values are maps, for which keys are cousine names, and values are cousine scores calculated for this recipes.
Now for each recipe let's select cousine with the largest score:

```python
recipe_results = {}
for recipe_id in recipe_map:
    recipe_cousine_map = recipe_map[recipe_id]
    # get cousine with max score from recipe_cousine_map
    cousine_name = max(recipe_cousine_map, key=recipe_cousine_map.get)
    recipe_results[recipe_id]=cousine_name
```

And let's write it to file:

```python
with open('result.csv', 'w') as rst_file:
    rst_file.write('id,cuisine'+'\n')
    for recipe_id in recipe_results:
        cousine_name = recipe_results[recipe_id]
        rst_file.write(str(recipe_id)+','+cousine_name+'\n')
```

Score that this model gives: **0.38505**

## Summary

Here we implemented very simple _Bag of words_ approach by hands without using any library. 
Score that it earns is not that high, only **0.38505**. 

Another good exercise would be to implement the same approach using some library, e.g. sklearn. 
But that would be topic for another post. 

## Links

I found some publicly available posts devoted to analysis of this task. Here are some of them:

* [CS570 Final Project - Kaggle: What’s cooking?](http://www.mathcs.emory.edu/~lxiong/cs570/share/project/qiyang_whats_cooking.pdf)
* [Jeff Wen - What's cooking?](http://jeffwen.com/2015/12/19/whats_cooking)
* [Frolian's blog - The Kaggle What's Cooking challenge](https://flothesof.github.io/kaggle-whats-cooking-machine-learning.html)
* [Analytics Vidhya - Kaggle Solution: What’s Cooking ? (Text Mining Competition)](https://www.analyticsvidhya.com/blog/2015/12/kaggle-solution-cooking-text-mining-competition/)
* [Félix Luginbühl - Using Recipe Ingredients to Categorize the Cuisine](http://felixluginbuhl.com/cooking/)
* [Oguzhan Gencoglu - Github repo with solution for What's cooking](https://github.com/ogencoglu/WhatsCooking)
* [Beautiful visualisations using word embedding](http://www.cs.toronto.edu/~manwar/projects/whatscooking/whatscooking/)
* [What’s Cooking? Predicting Cuisines from Recipe Ingredients](http://kevinkdo.com/sta561/final_project.pdf)

There is a number of other good stuff which can be googled by query `what's cooking kaggle`.

I found also some really good stuff: [KAGGLE ENSEMBLING GUIDE](https://mlwave.com/kaggle-ensembling-guide/), which explains how ensembling should be done for kaggle competitions. A ton of useful stuff. 

Also here is a [list of available colors in matplotlib](https://matplotlib.org/examples/color/named_colors.html) - nice stuff when you need to draw a number of entities (and not only bars) in different colors.



