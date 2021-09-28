# backstage-enabled-application

Check build btatus: https://circleci.com/gh/saksdirect/svc-mq-consumer

This service is a low level repeater between the IBM MQ queues and CNS v2.

OMS and BlueMartini post XML and JSON messages on these queues (one queue for each banner and environment), and this app picks them up and sends them to [api-customer-notification](https://github.com/gilt/api-customer-notification) using HTTP POST to `https://api.backoffice.giltaws.com/api-customer-notification/<ltqa|stqa|prod>/send_email/`.

## Technical Details

The MQ queue names are found in the config files:

```
conf/prod.conf
conf/ltqa.conf
conf/stqa.conf
```

## Development

NOTE: please name your non-integration test as "Spec" and your integration test as "IntegrationTest"

To test:

```
$ sbt test
```

To run integration test:

Before you run following command, please make sure your aws security token for profile hbc-backoffice is still valid

```
$ sbt fun:test
```


To run the service locally:

```
$ make local
```

## Monitoring

To check the logs:
Log into the hbc-backoffice AWS account > EC2 > type "svc-mq-consumer-prod" in the search bar to find the svc-mq-consumer-prod EC2 instance. Copy the private IP address:

https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Instances:search=svc-mq;sort=privateIpAddress

[AWS EC2](img/ec2.png)

Then, ssh into gandalf.gilt.com, the above EC2 machine, and then tail the docker container logs:
```
$ ssh gandalf.gilt.com

$ ssh -i push.pem ec2-user@<machine_ip_above>

$ sudo docker logs svc-mq-consumer
```

E.g.:
```
~ $ ssh gandalf.gilt.com

bastion1.prod.ec2.gilt.local:/home/candronic> ssh -i push.pem ec2-user@172.16.71.35

Last login: Wed Jul 11 17:57:01 2018 from 10.131.0.6

       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2018.03-release-notes/
4 package(s) needed for security, out of 4 available
Run "sudo yum update" to apply all updates.

[ec2-user@ip-172-16-71-35 ~]$ sudo docker logs svc-mq-consumer

Started repeater: QO.EMAIL.NOTIFICATION.QUEUE --> https://api.backoffice.giltaws.com/api-customer-notification/stqa/send_email/
Started repeater: S40_LTQA_BM_CUST_NOTIFICATION --> https://api.backoffice.giltaws.com/api-customer-notification/ltqa/send_email/
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)

 Message: {"type":"transactional_email","event":"order_confi ... :2075035}]},"banner":"saks","recipient_type":null}

Response: [200] https://api.backoffice.giltaws.com/api-customer-notification/ltqa/send_email/

Message: {"type":"transactional_email","event":"order_confi ... :2075032}]},"banner":"saks","recipient_type":null}

Response: [200] https://api.backoffice.giltaws.com/api-customer-notification/ltqa/send_email/
```

## CloudFormation deployment

```
$ make clean create-stack-dev
$ make clean update-stack-dev
```

## Code Deployment

To deploy:

```

$ make clean deploy-stqa

$ make clean deploy-ltqa

$ make clean deploy-prod

```

## AWS CloudWatch Metrics dashboard

https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#dashboards:name=Trex-CNS

## TODO

Improve monitoring:

	- Add CloudWatch alarms on these metrics that trigger a PagerDuty alert if the error count goes above a certain threshold in a given time window (the count should most likely be 1 since we shouldn't have any errors in this simple service)
	
	- Save the messages in S3/ SQS / put them back on the MQ/ do something else if there is an issue talking to api-customer-notification after the message was picked up from the queue
