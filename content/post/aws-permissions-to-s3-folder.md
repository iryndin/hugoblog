+++
date = "2017-04-26T23:46:17+03:00"
draft = false
title = "AWS permissions to S3 folder"
tags = ["aws"]
+++

Let's imagine that we have a project, which actually use AWS S3 as file storage. The project consists of 2 parts: one part puts files into S3, and the other part only reads them from S3.
Moreover, files are stored not in the bucket root, but in some folder which is placed in the bucket root. 

According to best AWS practices, we need 2 security and access control policies. One policy is for that part of software which puts files into S3 folder. This will be READ/WRITE policy, 
and another policy, used by read-only part of the project, will have only READ permissions on the folder. 

So, let's create these 2 policies (which will be later assigned to 2 different users) and explain their details. Let's start with the first, READ/WRITE policy.

## Read/write policy 

Let us have following folder structure in the bucket named `bucket1`:

- zf/cloud/2017
- zf/coffee/2017
- folder1/sdsd/11
- folder1/sdsd/22

And we want to give read/write access only to `zf` folder and its subfolders. Let's see what should we do for this. We will have a policy that will contain a set of statements.
Here are these statements with brief explanation. 

```
{
  "Sid": "AllowToSeeBucketListInTheConsole",
  "Action": ["s3:GetBucketLocation", "s3:ListAllMyBuckets"],
  "Effect": "Allow",
  "Resource": ["arn:aws:s3:::*"]
}
```
These permissios are granted to allow user to navigate within the AWS account using AWS console. 

```
{
  "Sid": "AllowRootAndHomeListingOfBucket",
  "Action": ["s3:GetBucketLocation", "s3:ListBucket"],
  "Effect": "Allow",
  "Resource": ["arn:aws:s3:::bucket1"],
  "Condition":{"StringEquals":{"s3:prefix":[""],"s3:delimiter":["/"]}}
}
```
This statement grants user permission to see a list of folders within `bucket1` bucket. This is also required for navigation. 

```
{
  "Sid": "AllowListingOfZfFolder",
  "Action": ["s3:GetBucketLocation", "s3:ListBucket", "s3:PutObject"],
  "Effect": "Allow",
  "Resource": ["arn:aws:s3:::bucket1"],
  "Condition":{"StringLike":{"s3:prefix":["zf/*"]}}
}
```
And finally, this statement grants user permission to get a list of objects inside `zf` folder, as well as put objects there. So, final policy will look like: 

```
{
  "Version": "2012-10-17",
  "Statement": [
     {
       "Sid": "AllowToSeeBucketListInTheConsole",
       "Action": ["s3:GetBucketLocation", "s3:ListAllMyBuckets"],
       "Effect": "Allow",
       "Resource": ["arn:aws:s3:::*"]
     },
     {
       "Sid": "AllowRootAndHomeListingOfBucket",
       "Action": ["s3:GetBucketLocation", "s3:ListBucket"],
       "Effect": "Allow",
       "Resource": ["arn:aws:s3:::bucket1"],
       "Condition":{"StringEquals":{"s3:prefix":[""],"s3:delimiter":["/"]}}
     },
     {
       "Sid": "AllowReadWriteOfZfFolder",
       "Action": ["s3:GetBucketLocation", "s3:ListBucket", "s3:PutObject"],
       "Effect": "Allow",
       "Resource": ["arn:aws:s3:::bucket1"],
       "Condition":{"StringLike":{"s3:prefix":["zf/*"]}}
     }
  ]
}
```

Here are some commands that show how this works:
```
# this will give Access Denied because of --recursive flag (remember, we are not allowed to go in folder1!)
aws --recursive --profile user.updater s3 ls s3://bucket1

# this will work fine
aws --profile user.updater s3 ls s3://bucket1

# this will give Access Denied, because we should add final slash here
aws --profile user.updater s3 ls s3://bucket1/zf

# this will work fine
aws --profile user.updater s3 ls s3://bucket1/zf1

# this will work fine
aws --recursive --profile user.updater s3 ls s3://bucket1/zf/
``` 

## Read-only policy 

Now, when we have read/write policy above, it is much easier to construct read-only policy from it. Actually, read-only policy is like the read/write policy above,
but without `s3:PutObject` permission. Insted, we need to add `s3:GetObject` permission, so that the last statement will look like: 

```
{
  "Sid": "AllowReadOfZfFolder",
  "Action": ["s3:GetBucketLocation", "s3:ListBucket", "s3:GetObject"],
  "Effect": "Allow",
  "Resource": ["arn:aws:s3:::bucket1"],
  "Condition":{"StringLike":{"s3:prefix":["zf/*"]}}
}
```

All previous statements will be the same as in read/write policy. 

## Useful links

[Writing IAM Policies: Grant Access to User-Specific Folders in an Amazon S3 Bucket](https://aws.amazon.com/blogs/security/writing-iam-policies-grant-access-to-user-specific-folders-in-an-amazon-s3-bucket/)

[Specifying Permissions in a Policy](http://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html#using-with-s3-actions-related-to-objects)

[How to Give User Access to an Amazon S3 Folder With CloudBerry Explorer](https://www.cloudberrylab.com/blog/how-to-give-user-access-to-an-s3-folder-with-cloudberry-explorer/)
