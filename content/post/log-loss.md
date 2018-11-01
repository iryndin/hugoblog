+++
date = "2018-11-01T14:20:39Z"
title = "Log loss metric explained"
Tags = ["machine learning"]
+++

_LogLoss_ is a classification metric based on probabilities. 
It measures the performance of a classification model where the prediction input is a probability value between 0 and 1.
For any given problem, a smaller _LogLoss_ value means better predictions.

<!--more-->

## Formula 

In order to calculate _LogLoss_ the classifier must assign a probability to each class rather than simply yielding the most likely class (class with the largest probability).  _LogLoss_ formula is as follows:

<img src="/img/logloss.gif" style="width:50%" />

Let's find out what every symbol in this formula means. 

* N - is a number of samples (or instances)
* M - is a number of possible labels
* y_ij - takes value **1** if label **j** is the correct classification for instance **i**, and **0** otherwise
* p_ij - is the model probability of assigning label **j** to instance **i**

## Example calculation

Let's calculate _LogLoss_ in 2 ways: 

* employ excellent [sklearn](http://scikit-learn.org/stable) library
* calculate with pure Python without any libraries

### Data

Let's assume that we predict car maker. We have only three possible labels: `audi`, `bmw`, `tesla`. And we have 8 rows of data with known labels: 

```{python}
data = ['audi', 'tesla', 'tesla', 'bmw', 'audi', 'bmw', 'audi', 'tesla']
```

And we have a classifier that for given data gives following probabilities: 

```{python}
probs = [
  [0.6, 0.3, 0.1], [0.45, 0.45, 0.1], [0.50, 0.00, 0.50], [1.00, 0.00, 0.00],
  [0.2, 0.6, 0.2], [0.10, 0.10, 0.8], [0.33, 0.33, 0.34], [0.30, 0.40, 0.30]
]
```

Please notice that lengh of `probs` array is equal to length of `data` array, and each array in `probs` array has size 3 (that is equal to the number of all possible labels). Internal arrays in `probs` array are probabilities of labels: 1st item is for `audi`, 2nd item is for `bmw`, 3rd item is for `tesla`. 
E.g. consider first array: `[0.6, 0.3, 0.1]`. Here `0.6` is probability of `audi`, `0.3` - for `bmw` and `0.1` for `tesla`. Also notice, that labels are sorted alphabetically, and so are probabilities for them (I will rely on this fact when implementing our own function that calculates _LogLoss_).

Now let's calculate _LogLoss_ for this classifier.

### LogLoss in sklearn

Here is how to use [sklearn.metrics.log_loss](http://scikit-learn.org/stable/modules/generated/sklearn.metrics.log_loss.html#sklearn.metrics.log_loss) for this:

```{python}
from sklearn.metrics import log_loss
data = ['audi', 'tesla', 'tesla', 'bmw', 'audi', 'bmw', 'audi', 'tesla']
probs = [
  [0.6, 0.3, 0.1], [0.45, 0.45, 0.1], [0.50, 0.00, 0.50], [1.00, 0.00, 0.00],
  [0.2, 0.6, 0.2], [0.10, 0.10, 0.8], [0.33, 0.33, 0.34], [0.30, 0.40, 0.30]
]
ll = log_loss(data, probs)
print(ll)
```

The result is `5.53374909081`.

### Handy LogLoss calculation

Now let's calculate _LogLoss_ with pure Python, not using any fancy library. Here is calculation for each data instance, summing all it up finally:  

```
from math import log
ll = [None]*8
EPS = 1e-15
ll[0] = 1*log(0.6)+0*log(0.3)+0*log(0.1)
ll[1] = 0*log(0.45)+0*log(0.45)+1*log(0.1)
ll[2] = 0*log(0.5)+0*log(EPS)+1*log(0.5)
ll[3] = 0*log(1.0)+1*log(EPS)+0*log(EPS)
ll[4] = 1*log(0.2)+0*log(0.6)+0*log(0.2)
ll[5] = 0*log(0.1)+1*log(0.1)+0*log(0.8)
ll[6] = 1*log(0.33)+0*log(0.33)+0*log(0.34)
ll[7] = 0*log(0.3)+0*log(0.4)+1*log(0.3)
logloss = -(1/8)*sum(ll)
print(logloss)
```

The result will we the same as with `sklearn`: `5.53374909081`. 

Let's consider this calculation. You see that instead of zero I am using here a small value: `EPS`, equal to `1e-15`. This is because logarythm of `0.0` is minus infinity. To workaround this I am using reasonably small value: `1e-15`.

Now let's write our own function to calculate _LogLoss_. Let's split it to 2 fundtions: 

* one is validating input data: `logloss_validate`
* another one is doing actual calculations, only if validation was successful: `logloss`


```{python}
def logloss_validate(a_true, a_probs, eps):
  # 1. Validate input variables
  if len(a_true) != len(a_probs):
    raise ValueError('Length of a_true does not match length of a_probs')
  unique_labels = set(a_true)
  # 2. Validate number of unique labels
  if len(unique_labels) < 2:
    raise ValueError('Should have at least 2 unique labels')
  # 3. Validate eps value  
  if eps <= 0.0:
    raise ValueError('Eps should be very small (near zero) positive value')
  # 4. Validate probabilities  
  for item in a_probs:
    # 4.1. Check that item in a_probs is iterable
    try:
      itemiter = iter(item)
    except TypeError as te:
      raise ValueError('Item in a_probs should be an iterable')
    # 4.2. Check that length of item in a_probs is equal to the number of unique labels  
    if len(item) != len(unique_labels):
      raise ValueError('Size of item in a_probs does not match number of unique labels')
    for i in item:
      if i < 0.0:
        raise ValueError('Some items of a_probs have negative values')
    
   
def logloss(a_true, a_probs, eps=1e-15):
  logloss_validate(a_true, a_probs, eps)
  from math import log

  result = 0.0
  # uniqalize labels and sort them alphabetically
  unique_sorted_labels = sorted(set(a_true))
  label_map = {}
  label_idx = 0
  # put index of each unique sorted label into a map
  # I will use this map to get this label probability by label value itself
  for label in unique_sorted_labels:
    label_map[label] = label_idx
    label_idx = label_idx + 1
  for true_label, probs in zip(a_true, a_probs):
    true_label_idx = label_map[true_label]
    # Here I rely on the fact that label probabilities are sorted alphabetically
    prob = probs[true_label_idx]
    if (prob < eps):
      prob = eps
    result = result + log(prob)
  return -1/len(a_true)*result;
```

Then, if we call `logloss(data, probs)` we will get the same result for _LogLoss_ metric: `5.53374909081`.

## Links

* [Datawookie - Making Sense of Logarithmic Loss](https://datawookie.netlify.com/blog/2015/12/making-sense-of-logarithmic-loss/)
* [Kaggle - What is Log Loss](https://www.kaggle.com/dansbecker/what-is-log-loss)
* [Mark Needham - scikit-learn: First steps with log_loss](https://markhneedham.com/blog/2016/09/14/scikit-learn-first-steps-with-log_loss/)
* [FastAI Wiki - Log Loss](http://wiki.fast.ai/index.php/Log_Loss)
* [Online LaTeX Equation Editor - create, integrate and download](https://www.codecogs.com/latex/eqneditor.php) - I used it for creation of _LogLoss_ formula image
