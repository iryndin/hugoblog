+++
date = "2018-06-09T00:50:44Z"
title = "How to run HUGO under Docker"
Tags = ["docker","hugo"]
Description = "Brief description how to run Hugo under Docker and operate a site with it"
+++

It is convenient to run such tool like Hugo (and many many others) under Docker, 
without installing it to a local machine. Here I will show you how to run Hugo
under Docker, create new posts for the blog, generate static content - i.e. all basic operations
that allow to add new and edit existing posts to the blog. 

<!--more-->

## What docker image for Hugo to take

A lot of Hugo images are available on [Docker Hub](https://hub.docker.com/search/?isAutomated=0&isOfficial=0&page=1&pullCount=0&q=hugo&starCount=1), let's just take the most starred one: [jojomi/hugo](https://hub.docker.com/r/jojomi/hugo/).
One nice thing for this image is that it is updated on time and all versions of Hugo are available, starting from `0.15`. I will show you how to run Hugo `0.19`, but you can easily apply this knowledge to run Hugo of more late versions.  


## How to run hugo from docker

First, let's have two folders:

* `src` - for sources of our site
* `output` - for generated output

Then, let's do following: 

 * create some simple site (let it be blog) structure
 * download theme
 * run site on `localhost`
 * then generate html from out content


## Create simple site structure

Inside our `src` folder we have following content:

 * `config.toml` - file with config
 * `content` - folder with blog content
 * `static` - folder with static assets (images/css/scripts/etc)
 * `themes` - folder which contain one or more theme we want to employ for our website

To see example of initial folder, see it on github: https://github.com/iryndin/hugoblog

Folder `themes` should contain Hugo theme which we are going to use. For this example, let it be `hugo-redlounge`.

Now, let's run Hugo docker image to run a webserver on localhost, so that we would be able to look at our website in browser:

```
docker run --rm --name "hugo1" -p 1313:1313 -v $(pwd)/src:/src -v $(pwd)/output:/output -e HUGO_THEME="hugo-redlounge" -e HUGO_WATCH="true" jojomi/hugo:0.26
```

Now we will be able to see our website in browser at `http://localhost:1313`

## Creating new post

To create new post, run following command:

```
docker exec -it hugo1 hugo new post/example3.md
```

New file for this post will be created, and you can immediately start editing it. 

## Generate static content

Now let's generate static content for our site. To generate static content in `output2` folder you need to run command similar to what we used to run webserver on localhost, but without `HUGO_WATCH` variable and without necessity to map ports. Also, you should run it under another name, if you do not want to shutdown 
container named `hugo1` (that used in this example to run local webserver).

```
docker run --rm --name "hugo2" -v $(pwd)/src:/src -v $(pwd)/output2:/output -e HUGO_THEME="hugo-redlounge" jojomi/hugo:0.26
```

And you will have your static site in `output2` folder.


