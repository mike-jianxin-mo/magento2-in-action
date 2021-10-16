---
title: CoudWatch Setup (1)
author: Mike Mo
date: 2021-10-14
category: [AWS, Data]
layout: post
---

##### Great AWS Knowledge Source: [1]

#### check server type ARM or X86

uname -m

#### set up role

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-cloudwatch-agent.html

#### download and install

wget https://s3.amazonaws.com/amazoncloudwatch-agent/ubuntu/amd64/latest/amazon-cloudwatch-agent.deb

sudo dpkg -i -E ./amazon-cloudwatch-agent.deb

ls /opt/aws/amazon-cloudwatch-agent/bin/config.json

#### install aws

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
aws --version

The following command fail! In fact, we don't need the aws, the cloudwatchagent can configure itself.
However, cloudwatchagent is base on aws cli functions.

sudo aws configure --profile AmazonCloudWatchAgent

#### configure the cloud watch agent

This configuration works fine.
The option details can be reference(for Yes or No):

https://www.wellarchitectedlabs.com/cost/200_labs/200_aws_resource_optimization/4_memory_plugin/

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard

#### run the agent

start cloudwatch

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

#### errors

tail /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log

2021-10-14T04:59:43Z E! [outputs.cloudwatchlogs] Aws error received when sending logs to staging_varnish_test/i-09d213ee4b5206efd: UnrecognizedClientException: The security token included in the request is invalid.

2021-10-14T04:59:43Z W! [outputs.cloudwatchlogs] Retried 441 time, going to sleep 57.148066432s before retrying.

2021-10-14T05:00:40Z E! [outputs.cloudwatchlogs] Aws error received when sending logs to staging_varnish_test/i-09d213ee4b5206efd: UnrecognizedClientException: The security token included in the request is invalid.

2021-10-14T05:00:40Z W! [outputs.cloudwatchlogs] Retried 442 time, going to sleep 36.606756923s before retrying.

#### reinstall aws cli

aws is the base of the cloud agent
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-cloudwatch-agent.html

#### create aws user permission

https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-iam-roles-for-cloudwatch-agent.html

#### set up the aws keys

sudo aws configure --profile AmazonCloudWatchAgent

#### no logs have been sent to cloud

tail -n 50 /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log

2021-10-14T10:54:51Z E! [outputs.cloudwatchlogs] Aws error received when sending logs to staging_varnish_test/i-09d213ee4b5206efd: NoCredentialProviders: no valid providers in chain

caused by: EnvAccessKeyNotFound: failed to find credentials in the environment.

SharedCredsLoad: failed to load profile, .

EC2RoleRequestError: no EC2 instance role found

caused by: EC2MetadataError: failed to make EC2Metadata request

status code: 404, request id:

caused by: <?xml version="1.0" encoding="iso-8859-1"?>

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
	"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">

 <head>
  <title>404 - Not Found</title>
 </head>
 <body>
  <h1>404 - Not Found</h1>
 </body>
</html>

2021-10-14T10:54:51Z W! [outputs.cloudwatchlogs] Retried 415 time, going to sleep 51.471788249s before retrying.

Error: Caused by the IAM roles setting of the EC2 instance is missing
Solution: Set the role.

##### No log has been synced to cloud

No log has been synced to cloud, but the server memory usage data has been synced successfully.

Ref: https://stackoverflow.com/questions/58658744/cloudwatch-agent-not-sending-logs-to-cloudwatch

This is caused by the permission error. Solved it by followed the above instruction

sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json

At the top of the file you will see:

"agent": {
"metrics_collection_interval": 60,
"run_as_user": "cwagent"
},
You need to change run_as_user to root, like:

"agent": {
"metrics_collection_interval": 60,
"run_as_user": "root"
},
Once you have changed that, you simply reload the config file:

sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

And then restart the service:

sudo systemctl restart amazon-cloudwatch-agent.service

It works!!!!

[1]: https://www.wellarchitectedlabs.com/
