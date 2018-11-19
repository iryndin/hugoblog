+++
date = "2018-11-18T21:00:00Z"
title = "MNIST digit recognition with CNN and Keras"
Tags = ["machine learning", "image classigication"]
+++

In this post I will briefly go through application of CNN (Convolutional Neural Networks) to well known MNIST dataset. I will use Keras for this. There is a well-known example at Keras repo: [mnist_cnn.py](https://github.com/keras-team/keras/blob/master/examples/mnist_cnn.py), and I will use its code for this blog post. So there is nothing new in this blog post. Rather it is a try to put some basics into my head for further use. 

<!--more-->

## Imports

```python
from __future__ import print_function
import keras
from keras.datasets import mnist
from keras.models import Sequential
from keras.layers import Dense, Dropout, Flatten
from keras.layers import Conv2D, MaxPooling2D
from keras import backend as K
```

## Define variables and load MNIST dataset

```python
batch_size = 128
num_classes = 10
epochs = 12

# input image dimensions
img_rows, img_cols = 28, 28

# the data, split between train and test sets
(x_train, y_train), (x_test, y_test) = mnist.load_data()
```

After this we have train data (`x_train`, `y_train`) and test data (`x_test`, `y_test`). 

## Data preparation and reshaping

```python
if K.image_data_format() == 'channels_first':
    x_train = x_train.reshape(x_train.shape[0], 1, img_rows, img_cols)
    x_test = x_test.reshape(x_test.shape[0], 1, img_rows, img_cols)
    input_shape = (1, img_rows, img_cols)
else:
    x_train = x_train.reshape(x_train.shape[0], img_rows, img_cols, 1)
    x_test = x_test.reshape(x_test.shape[0], img_rows, img_cols, 1)
    input_shape = (img_rows, img_cols, 1)

x_train = x_train.astype('float32')
x_test = x_test.astype('float32')
x_train /= 255
x_test /= 255
print('x_train shape:', x_train.shape)
print(x_train.shape[0], 'train samples')
print(x_test.shape[0], 'test samples')

# convert class vectors to binary class matrices
y_train = keras.utils.to_categorical(y_train, num_classes)
y_test = keras.utils.to_categorical(y_test, num_classes)
```

For `tensorflow` backend `K.image_data_format()` is: `channels_last` so `else` branch works. 

## Build model

```python
model = Sequential()
model.add(Conv2D(32, kernel_size=(3, 3), activation='relu', input_shape=input_shape))
model.add(Conv2D(64, (3, 3), activation='relu'))
model.add(MaxPooling2D(pool_size=(2, 2)))
model.add(Dropout(0.25))
model.add(Flatten())
model.add(Dense(128, activation='relu'))
model.add(Dropout(0.5))
model.add(Dense(num_classes, activation='softmax'))

model.compile(loss=keras.losses.categorical_crossentropy,
              optimizer=keras.optimizers.Adadelta(),
              metrics=['accuracy'])
```

For given model we have `Conv2D -> Conv2D -> MaxPool -> Dropout -> Flatten -> Dense -> Dropout -> Dense` layers. 

## Train model

```python
model.fit(x_train, y_train,
          batch_size=batch_size,
          epochs=epochs,
          verbose=1,
          validation_data=(x_test, y_test))
```

On my laptop each epoch takes almost 1 minute to complete, so overall time is arouunf 10-11 minutes:

```
Train on 60000 samples, validate on 10000 samples
Epoch 1/12
60000/60000 [==============================] - 49s 824us/step - loss: 0.2677 - acc: 0.9180 - val_loss: 0.0588 - val_acc: 0.9805
Epoch 2/12
60000/60000 [==============================] - 47s 790us/step - loss: 0.0872 - acc: 0.9738 - val_loss: 0.0388 - val_acc: 0.9864
Epoch 3/12
60000/60000 [==============================] - 50s 828us/step - loss: 0.0650 - acc: 0.9808 - val_loss: 0.0356 - val_acc: 0.9880
Epoch 4/12
60000/60000 [==============================] - 57s 950us/step - loss: 0.0533 - acc: 0.9836 - val_loss: 0.0302 - val_acc: 0.9891
Epoch 5/12
60000/60000 [==============================] - 56s 931us/step - loss: 0.0464 - acc: 0.9859 - val_loss: 0.0280 - val_acc: 0.9901
Epoch 6/12
60000/60000 [==============================] - 51s 858us/step - loss: 0.0415 - acc: 0.9875 - val_loss: 0.0291 - val_acc: 0.9900
Epoch 7/12
60000/60000 [==============================] - 56s 933us/step - loss: 0.0373 - acc: 0.9889 - val_loss: 0.0280 - val_acc: 0.9913
Epoch 8/12
60000/60000 [==============================] - 58s 966us/step - loss: 0.0341 - acc: 0.9899 - val_loss: 0.0256 - val_acc: 0.9922
Epoch 9/12
60000/60000 [==============================] - 52s 860us/step - loss: 0.0316 - acc: 0.9905 - val_loss: 0.0259 - val_acc: 0.9917
Epoch 10/12
60000/60000 [==============================] - 50s 828us/step - loss: 0.0276 - acc: 0.9915 - val_loss: 0.0268 - val_acc: 0.9910
Epoch 11/12
60000/60000 [==============================] - 48s 805us/step - loss: 0.0285 - acc: 0.9913 - val_loss: 0.0270 - val_acc: 0.9921
Epoch 12/12
60000/60000 [==============================] - 53s 888us/step - loss: 0.0252 - acc: 0.9925 - val_loss: 0.0265 - val_acc: 0.9919
```

## Save model

Let's save model and weights for further use:

```python
model.save("./mnist_cnn_trained_model.h5")
```

## Load model and use it for prediction

```python
from keras.models import load_model
model = load_model('./mnist_cnn_trained_model.h5')
```

Then let's load test data (imagine the we do not have `x_test`, `y_test` data):
```python
(x_train, y_train), (x_test, y_test) = mnist.load_data()

# preprocess test data
num_classes = 10
img_rows, img_cols = 28, 28

x_test = x_test.reshape(x_test.shape[0], img_rows, img_cols, 1)
x_test = x_test.astype('float32')
x_test /= 255
y_test = keras.utils.to_categorical(y_test, num_classes)
```

Then predict test data:
```python
score = model.evaluate(x_test, y_test, verbose=1)
print('Test loss:', score[0])
print('Test accuracy:', score[1])
```

We get following results:
```
10000/10000 [==============================] - 2s 249us/step
Test loss: 0.0264956298373
Test accuracy: 0.9919
```

Of course, after going through this example there are a lot of questions, e.g.:

* Why exactly this architecture of neural network is selected?
* Why exactly these parameters of layers are selected?
* Based on what assumptions should we tune parameters of layers?
* How test loss is calculated?
* How test accuracy is calculated?
* Why optimizer `keras.optimizers.Adadelta` was chosen?

At the moment I do not have answers to these questions, so this is subject for further study from my side. 

Also, I understand, that it would be greatly beneficial to implement CNN myself from scratch, using some general-purpose programming language (C++/Java/Python/Go/Javascript). I think this willl give me a key to understanding how Neural Networks work and a key to more clear decisions on neural network architectures and parameters of network layers.

## Links

Trying to find answers to my questions I have found a number of great posts/articles which I want to put here for further usage in my studies. 

* [Keras examples - mnist_mlp.py](https://github.com/keras-team/keras/blob/master/examples/mnist_mlp.py). This is another example of neural network applied to MNIST dataset. Good getaway from this example would be an understanding why that particular architecture was chosen, why that values of layer parameters were chosen, etc, etc. Also it is useful to look at another examples in [Keras examples folder](https://github.com/keras-team/keras/tree/master/examples) - there is a dozen of examples of solving MNIST problem with another approaches. 
* [A simple 2D CNN for MNIST digit recognition](https://towardsdatascience.com/a-simple-2d-cnn-for-mnist-digit-recognition-a998dbc1e79a). This post is exactly about the same - solving MNIST with CNN and Keras. This is just step-by-step implementation of CNN neural network with Keras, without any detailed explanations why this or that value was chosen.
* [What is the best CNN architecture for MNIST?](https://www.kaggle.com/cdeotte/how-to-choose-cnn-architecture-mnist). Great kernel on Kaggle which describes how to choose the best CNN architecture for MNIST challenge. I need to go through this, reproduce it and try to understand. 
* [Accuracy=99.75% using 25 Million Training Images](https://www.kaggle.com/cdeotte/25-million-images-0-99757-mnist). This is another MNIST example. Good points here: generating additional images, ensembling 15 CNNs for more accuracy, data augmentation. I need to reproduce this and try to understand.
* [Visualizing CNN filters with keras](https://jacobgil.github.io/deeplearning/filter-visualizations). It shows how to visualize CNN filters with Keras. Nice research!
