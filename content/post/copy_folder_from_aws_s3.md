+++
date = "2017-11-18T12:19:37+03:00"
draft = false
title = "Copy folder from AWS S3"
tags = ["aws"]
+++

How could we copy a folder from AWS S3 to local machine using AWS CLI?

<!--more-->

## Use cp command

We can use `cp` command to copy folder or file from S3 to local machine. If we add parameter `--recursive` then 
we will get folder with all its files and folder inside. Example: 

```
aws s3 cp s3://bucketname/foldername localdirname --recursive
```
This will copy folder `foldername` with all its content inside from S3 to local folder named `localdirname`.

## Use sync command

We have also `sync` command that will, by default, copy a whole directory. But it will only copy new and modified files. 
Unchanged files remain untouched. Example:  

```
aws s3 sync s3://bucketname/foldername localdirname
```
This will synchronize S3 folder `foldername` with local folder `localdirname`.

## Useful links 

* [AWS sync command](http://docs.aws.amazon.com/cli/latest/reference/s3/sync.html)
* [AWS cp command](http://docs.aws.amazon.com/cli/latest/reference/s3/cp.html)
* [StackOverflow - Downloading folders from aws s3, cp or sync?](https://stackoverflow.com/questions/27932345/downloading-folders-from-aws-s3-cp-or-sync)