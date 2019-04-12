+++
date = "2019-04-12T13:29:00Z"
title = "XGBoost applied to MNIST"
Tags = ["machine learning", 'xgboost', 'mnist']
+++

Continue playing with XGBoost. Today let's apply it to MNIST dataset. In fact, there is very small difference between 
[applying XGBoost to Iris](http://iryndin.net/post/xgboost_for_iris_dataset/) or to MNIST.
I will not comment much here, and just give here a python script with the solution. 
If something is unclear, go read my previous post about [XGboost and Iris](http://iryndin.net/post/xgboost_for_iris_dataset/). 

<!--more-->

## Code

```
import xgboost
from sklearn.datasets import load_digits
from sklearn.cross_validation import train_test_split

digits = load_digits()
X = digits.data
y = digits.target

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

dtrain = xgboost.DMatrix(X_train, label=y_train)
dtest = xgboost.DMatrix(X_test, label=y_test)

param = {
    'max_depth': 5,                 # the maximum depth of each tree
    'eta': 0.3,                     # the training step for each iteration
    'silent': 1,                    # logging mode - quiet
    'objective': 'multi:softmax',   # multiclass classification using the softmax objective
    'num_class': 10                 # the number of classes that exist in this datset
}  
num_round = 50  # the number of training iterations

iris = datasets.load_iris()
X = iris.data
y = iris.target

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

bstmodel = xgboost.train(param, dtrain, num_round)

preds = bstmodel.predict(dtest)

from sklearn import metrics
acc = metrics.accuracy_score(y_test, preds)

print('Accuracy: %f' % acc)
```

Accuracy here is `0.966667`.

## Links

Another couple of links about XGBoost. 

* [Github - Repositories which are labeled by topic 'xgboost'](https://github.com/topics/xgboost)
* [ArXiv - Original XGBoost article. XGBoost: A Scalable Tree Boosting System](https://arxiv.org/abs/1603.02754)
* [Github - wepe/tgboost. Tiny XGBoost. A simple Java implementation of XGboost with some variations from original](https://github.com/wepe/tgboost)

