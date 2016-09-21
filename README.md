Backup to AWS S3
================

Simple script to backup periodicaly to AWS S3.

##AWS Setup

Before installing and adding it to your cron jobs, you have to configure your AWS environment.

### Create the S3 bucket

Create & configure an S3 bucket to hold the backup files.

Change `<VALUES>` as appropriate.
Chose the same region as your server (if it's on AWS), to minimize traffic and latency.
The `versioning.json` file is provided.

```sh
aws s3 mb s3://<BUCKET NAME> --region <REGION>
aws s3api put-bucket-versioning --bucket <BUCKET NAME> --versioning-configuration Status=Enabled
aws s3api put-bucket-lifecycle-configuration --bucket <BUCKET NAME> --lifecycle-configuration  file://versioning.json
```

### Create SNS Topic
Create a SNS Topic which we'll use to send warnings in case something breaks.

```sh
aws sns create-topic --name <TOPIC NAME> --region <REGION>
```

The above command will output the ARN of the newly created SNS Topic. Save it, as we'll need it later.

### Create the IAM user

Create an IAM user who has permissions to access the required resources: the bucket and the SNS topic.

```sh
aws iam create-user --user-name backupAgent
aws iam create-access-key --user-name backupAgent
```

The last command above will return the AWS Access Keys for the new user. Save them, as we'll need them soon.

Next, configure the access permissions for the new IAM user. The policy files are providedm but they must be edited before they can be used: `bucket_policy.json` and `sns_policy.json`.

```sh
aws iam put-user-policy --user-name backupAgent    --policy-name accessS3 --policy-document file://bucket_policy.json
aws iam put-user-policy --user-name backupAgent    --policy-name accessSNS --policy-document file://sns_policy.json
```

Subscribe to the SNS Topic to receive notifications:

```sh
aws sns subscribe --topic-arn <ARN OF THE SNS TOPIC> --protocol email --notification-endpoint <YOUR EMAIL ADDRESS>  --region <REGION>
```

##Server Setup
Now that the AWS resources and perms have been setup, you can add the relevant values to the `backup_to_S3` file, make it executable and place it somewhere nice on your server, like `/usr/local/bin`

The next step is to add it to the sytem cron jobs (`/etc/cron.d`), for example like this:

```
00 5 * * * root /usr/local/bin/backup_to_s3 daily
00 6 * * 5 root /usr/local/bin/backup_to_s3 weekly
30 6 2 * * root /usr/local/bin/backup_to_s3 monthly
```

Don't forget to configure the mysql password in `/root/.my.cnf` before you back up your databases.

Enjoy.
