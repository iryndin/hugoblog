+++
date = "2019-04-10T13:29:00Z"
title = "XGBoost applied to Iris dataset"
Tags = ["machine learning", 'xgboost']
+++

Let's show here a very simple example of applying XGBoost to Iris dataset. Initial Iris dataset is at [UCI data repository](http://archive.ics.uci.edu/ml/datasets/iris). But we will use ready-to-use [Iris dataset contained in sklearn](https://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_iris.html).

<!--more-->

## Run Jupyter with Docker and install XGBoost

Let's run our data science Docker, as described here: [Run Jupyter notebooks with Docker](http://iryndin.net/post/run-jupyter-with-docker/).
Run command: 

```
docker run --rm -p 8888:8888 --name myds1 jupyter/scipy-notebook:latest
```

But this Docker image does not contain XGBoost, so let's manually install it. 
After container is started, let's exec following command to install XGBoost: 

```
docker exec -it myds1 pip install xgboost
```

## Load Iris dataset, split it to train and test

Load necessary imports, load data and split dataset:

```
import xgboost
from sklearn import datasets
from sklearn.cross_validation import train_test_split

iris = datasets.load_iris()
X = iris.data
y = iris.target

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

## Train XGBoost, predict data and compare

XGBoost needs that Numpy arrays be loaded in special DMatrix format: 

```
dtrain = xgboost.DMatrix(X_train, label=y_train)
dtest = xgboost.DMatrix(X_test, label=y_test)
```

Then let's set up XGBoost params:

```
param = {
    'max_depth': 3,                 # the maximum depth of each tree
    'eta': 0.3,                     # the training step for each iteration
    'silent': 1,                    # logging mode - quiet
    'objective': 'multi:softmax',   # multiclass classification using the softmax objective
    'num_class': 3                  # the number of classes that exist in this datset
}  
num_round = 20  # the number of training iterations
```

More about XGBoost params can be found here: [XGBoost Parameters](https://xgboost.readthedocs.io/en/latest/parameter.html).

Now train our model. And, if you want to look at how XGBModel looks like, dump it at text file and then simply take a look at it. 

```
bstmodel = xgboost.train(param, dtrain, num_round)
bstmodel.dump_model('dump.bstmodel.txt')
```

Then predict on test data and calculate accuracy score: 
```
preds = bst.predict(dtest)

from sklearn import metrics
acc = metrics.accuracy_score(y_test, preds2)
acc
```

That is it! 

## Links

* [Ieva Zarina - Iris dataset and xgboost simple tutorial](http://ieva.rocks/2016/08/25/iris_dataset_and_xgboost_simple_tutorial/)
* [Analytics Vidhya - Complete Guide to Parameter Tuning in XGBoost (with codes in Python)](https://www.analyticsvidhya.com/blog/2016/03/complete-guide-parameter-tuning-xgboost-with-codes-python/)
* [Towards Data Science - Fine-tuning XGBoost in Python like a boss](https://towardsdatascience.com/fine-tuning-xgboost-in-python-like-a-boss-b4543ed8b1e)
* [Analytics Vidhya - An End-to-End Guide to Understand the Math behind XGBoost](https://www.analyticsvidhya.com/blog/2018/09/an-end-to-end-guide-to-understand-the-math-behind-xgboost/)
* [Kaggle - Kernels using xgboost](https://www.kaggle.com/kernels?search=tag:%27xgboost%27)
* [XGBoost on MNIST and Fashion MNIST](https://github.com/anktplwl91/fashion_mnist.git)
* [Kaggle - XGBoost on titanic](https://www.kaggle.com/cbrogan/xgboost-example-python)
* [Kaggle - XGBoost on mnist](https://www.kaggle.com/anktplwl91/mnist-xgboost)
* [IBM - XGBoost and breast cancer](https://dataplatform.cloud.ibm.com/analytics/notebooks/v2/de824290-6811-455b-b2a4-678fd6ae06ee/view?access_token=cb51caafa25bb0e596867d19fd0e96c77baeffca4b561091e38f0cd4476dc682)
* [XGBoost demos from XGBoost Git repo](https://github.com/dmlc/xgboost/tree/master/demo)
* [Analytics Vidhya - Kaggle Solution: What’s Cooking ? (Text Mining Competition)](https://www.analyticsvidhya.com/blog/2015/12/kaggle-solution-cooking-text-mining-competition/)
* [MachineLearningMastery - How to Develop Your First XGBoost Model in Python with scikit-learn](https://machinelearningmastery.com/develop-first-xgboost-model-python-scikit-learn/)
* [XGBoost docs](ttps://xgboost.readthedocs.io/en/latest/)
* [jessesw.com - Good data scientist blog](https://jessesw.com)
