+++
date = "2019-04-12T20:00:00Z"
title = "XGBoost applied to Fashion MNIST"
Tags = ["machine learning", 'xgboost', 'fashion mnist']
+++

Now let's consider applying XGBoost to Fashion MNIST dataset. As well as in 2 previous posts about XGBoost data are eready to use, 
and do not require any additional preprocessing in order to get accuracy near 90%. This makes this case similar to previous two (Iris and MNIST). 

<!--more-->

## Data

Fashion MNIST dataset can be downloaded from Kaggle: [Kaggle - Fashion MNIST. An MNIST-like dataset of 70,000 28x28 labeled fashion images](https://www.kaggle.com/zalando-research/fashionmnist). Dataset comes as two files: 

* `fashion-mnist_train.csv` - train data, 60 000 data rows
* `fashion-mnist_test.csv` - test data, 10 000 data rows

## Code

Now let's jump into the code. 

Import necessary libs:

```
import numpy as np
import pandas as pd
import xgboost as xgb
```

Load train data, split it to one-column data frame with label only and data frame with actual image data. Then convert it to XGBoost `DMatrix` form. 

```
train = pd.read_csv('./fashion-mnist_train.csv', header=0)
train_label = train['label']
train_data = train.drop(['label'],axis=1)
dtrain = xgb.DMatrix(train_data, label=train_label)
```

Define parameters for XGBoost.

```
param = {
    'max_depth': 5,                 # the maximum depth of each tree
    'eta': 0.3,                     # the training step for each iteration
    'verbosity': 1,                 # logging mode - warn
    'objective': 'multi:softmax',   # multiclass classification using the softmax objective
    'num_class': 10                 # the number of classes that exist in this datset
}  
num_round = 50 # the number of training iterations
```

Train our model:

```
import time
start = time. time()
bstmodel = xgb.train(param, dtrain, num_round, evals=[(dtrain, 'label')], verbose_eval=10)
end = time.time()
exec_time = end - start
print('Execution time, secs: %d' % exec_time)
```

Training on my laptop takes `350 secs`.

Then let's prepare test data in the same fashion as test data:

```
test = pd.read_csv('./fashion-mnist_test.csv', header=0)

test_label = test['label']
test_data = test.drop(['label'],axis=1)
dtest = xgb.DMatrix(test_data, label=test_label)
```

Predict: 

```
preds = bstmodel.predict(dtest)
```

Then assess results: 

```
from sklearn import metrics
acc = metrics.accuracy_score(test_label, preds)
print('Accuracy: %f' % acc)
```

Accuracy is `0.895300`.
