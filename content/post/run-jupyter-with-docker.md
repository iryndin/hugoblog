+++
date = "2018-11-02T17:20:39Z"
title = "Run Jupyter notebooks with Docker"
Tags = ["machine learning", "docker"]
+++

Let me show the way I am running Jupyter notebooks on my laptop. No surprise, I am doing using Docker. Why? Because this is fast, convenient, platform-agnostic and allows to have clean host system. 

<!--more-->

## Jupyter Docker stacks

There is a great project on Github - [Jupyter Docker stacks](https://github.com/jupyter/docker-stacks/). Jupyter Docker Stacks are a set of ready-to-run Docker images containing Jupyter applications and interactive computing tools. I am using exactly this project for running my Jupyter notebooks. 

### What image to choose

Since Jupyter Docker Stacks has a number of images (that are organized in a hierarchy), one reasonable question appears - what image to use? There is a great document explaining this - [Selecting an Image](https://jupyter-docker-stacks.readthedocs.io/en/latest/using/selecting.html). Based on this document, I usually run `scipy-notebook` image, which includes all necessary libs: `numpy`, `pandas`, `scikit-learn` and many others.

Here is the command to run this image:

```
docker run --rm -p 8888:8888 --name myds1 jupyter/scipy-notebook:latest
```
Then you open your browser, and copy URL that running Jupyter shows you. That is it!




