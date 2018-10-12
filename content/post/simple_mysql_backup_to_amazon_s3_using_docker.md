+++
date = "2018-10-11T03:20:39Z"
title = "Simple MySQL backup to Amazon S3 using Docker"
Tags = ["aws","mysql", "docker"]
+++

This is just a short notice about a convenient was of making daily MySQL backups to Amazon S3. Docker is used here to spin up MySQL and AWS CLI tool.

<!--more-->

The way of doing backup I am telling about in this post works good for small and medium size projects. 
For projects with big databases another approaches should be employed. 

It's very good practice to backup the database of a project at least daily. Here I suggest following plan of doing a backup:

* create and compress (with `gzip`) MySQL dump file
* upload it to S3 
* remove dump file from local filesystem
* configure cron job that will trigger this dump-and-upload operation

## Ensure working directory

Let us operate in a separate directory where we have shell file that makes backup and upload, and where we save locally our backup files. 
Let this directory name will be `/home/john/appbackup`. Now before we do any operation, let's ensure that we operate in the directory we want: 

```bash
#!/bin/bash
pushd /home/john/appbackup
```


## Creating MySQL dump

Let's imagine that MySQL runs in a Docker container named `appmysql`. Database that we want to backup is named `appdb`. So, we make mysql dump with the following command: 

```bash
DB_DUMP_FILENAME=appdb.`date +%Y-%m-%d_%H-%M`.sql.gz
echo "Gonna create dump file: $DB_DUMP_FILENAME"

docker exec appmysql mysqldump appdb | gzip > $DB_DUMP_FILENAME

if [ $? -eq 0 ]; then
    echo "App database dump created OK. File: $DB_DUMP_FILENAME"
else
    echo "App database dump FAILED!"
    exit 1
fi
```

## Upload dump file to S3

To upload dump file to Amazon S3 we will use AWS CLI. But we will not install it to our local system, rather we going to run it in a Docker container. 
Docker image we are going to employ is `mesosphere/aws-cli`. 
If you take a look at its [Dockerfile](https://hub.docker.com/r/mesosphere/aws-cli/~/dockerfile/) you will see that its working directory is `/project`. 
Since we need it to upload files from local system then we need to map our local dir to container's `/project` dir. 

So, if we want to upload database dump to S3 bucket `appbucket` to folder `dbdump`, we do it as follows:

```bash
echo "Gonna upload dump file to S3: $DB_DUMP_FILENAME"

docker run --rm -v "/home/john/appbackup:/project" \
  -e "AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXXX" \
  -e "AWS_SECRET_ACCESS_KEY=ZZZZZZZZZZZZZZZZZZZZZZZ" \
  -e "AWS_DEFAULT_REGION=us-east-1" \
  mesosphere/aws-cli s3 cp /project/$DB_DUMP_FILENAME \ 
  s3://appbucket/dbdump/$DB_DUMP_FILENAME --no-progress
```

## Remove dump file from local system

It is easy:

```bash
echo "Now remove dump file: $DB_DUMP_FILENAME"
rm -rf /home/john/appbackup/$DB_DUMP_FILENAME
```

## The whole script

Here is the script, let's name it `dump_and_upload.sh` and put it to `/home/john/appbackup`:

```bash
#!/bin/bash
pushd /home/john/appbackup

DB_DUMP_FILENAME=appdb.`date +%Y-%m-%d_%H-%M`.sql.gz
echo "Gonna create dump file: $DB_DUMP_FILENAME"

docker exec appmysql mysqldump appdb | gzip > $DB_DUMP_FILENAME

if [ $? -eq 0 ]; then
    echo "App database dump created OK. File: $DB_DUMP_FILENAME"
else
    echo "App database dump FAILED!"
    exit 1
fi

echo "Gonna upload dump file to S3: $DB_DUMP_FILENAME"

docker run --rm -v "/home/john/appbackup:/project" \
  -e "AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXXX" \
  -e "AWS_SECRET_ACCESS_KEY=ZZZZZZZZZZZZZZZZZZZZZZZ" \
  -e "AWS_DEFAULT_REGION=us-east-1" \
  mesosphere/aws-cli s3 cp /project/$DB_DUMP_FILENAME \ 
  s3://appbucket/dbdump/$DB_DUMP_FILENAME --no-progress

echo "Now remove dump file: $DB_DUMP_FILENAME"
rm -rf /home/john/appbackup/$DB_DUMP_FILENAME

echo "Operation Dump_and_Upload completed Successfully!"

popd
```

## Setup cronjob

Let's run this `dump_and_upload.sh` script at 5 minutes after midnight every day. Thus we need to add this record to the cron:

```
5 0 * * * /home/john/appbackup/dump_and_upload.sh >> /home/john/appbackup/cronlog.log 2>&1
```

