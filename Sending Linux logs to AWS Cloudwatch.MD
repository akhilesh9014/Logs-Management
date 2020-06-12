## Sending Linux logs to AWS Cloudwatch
Amazon CloudWatch is a monitoring service for AWS cloud resources and the applications you run on AWS. You can use Amazon CloudWatch to collect and track metrics, collect and monitor log files, set alarms, and automatically react to changes in your AWS resources. You can use Amazon CloudWatch to gain system-wide visibility into resource utilization, application performance, and operational health. Configuration for sending OS logs to CloudWatch involves, Create IAM Role with relevant permission and attach to Linux instance. Install the CloudWatch agent in the instance. Prepare the configuration file in the instance. Start the CloudWatch agent service in the instance. Monitor the logs using CloudWatch web console. Create AWS Role for CloudWatch Create AWS Policy of type (Service) “CloudWatch Logs” in the AWS console and add following permissions for all resources. Name the policy Eg. CWPol-LinuxLog. This is done in the visual editor.

* CreateLogStream
* DescribeLogStreams
* CreateLogGroup
* PutLogEvents

copy and paste below scripe to create new policy, and name it as a (aws-log-aget-policy),
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1591785228926",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:DescribeLogStreams",
        "logs:PutLogEvents"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}

```
### 2.Create AWS Role of type EC2 and name the role Eg, CWRole-LinuxLog
### 3.Add the above policy aws-log-aget-policy to Role CWRole-LinuxLog. 
Note: Creating the AWS Role is a one time process and this can be attached to any Linux instance you want to send the logs to CloudWatch. And we are doing it for all of the instances we discuss here. Linux flavor looks similar. But there are some differences the way they manage packages and applications. We see how the logs can be sent to CloudWatch from the following variants.

* Amazon Linux 2
* Amazon Linux 1
* Centos 7/RHEL 7

### Amazon Linux 2 

Amazon Linux 2 is a newer version of Linux from Amazon. It includes systemd service and systems manager. I have executed on the following specifications. 
```

AMI: amzn2-ami-hvm-2017.12.0.20180328.1-x86_64-gp2 (ami-e5441e8a)
Cloud Platform: Amazon 
Operating System: Amazon Linux 2 
Kernel: Linux 4.14.26–54.32.amzn2.x86_64 
Architecture: x86–64 
RPMS : awslogs-1.1.4–2.amzn2.noarch 

```

1. Launch instance using amazon AMI2 image and attach the role CWRole-LinuxLog to it.
2. Log in to the instance and install awslogs package.

```
$ sudo yum install awslogs

```
3. Edit file /etc/awslogs/awscli.conf and change your AWS Region.
4. Edit file /etc/awslogs/awslogs.conf and verify following lines 
```
[/var/log/messages] 
datetime_format = %b %d %H:%M:%S 
file = /var/log/messages 
buffer_duration = 5000 log_stream_name = {instance_id} 
initial_position = start_of_file 
log_group_name = <custom-name> # enter any name what you want 

```
5. The above configuration will create a hierarchical structure in the CloudWatch interface with log_group_name AMZ-2 ( an arbitrary name) as a container and respective instance id as the object where contents of “/var/log/messages” file is redirected. buffer_duration specifies the time duration for the batching of log events. The minimum value is 5000ms and the default value is 5000ms.
6. Start and enable awslogsd service by typing the command:

```
$ sudo service awslogsd start
$ sudo systemctl  enable awslogsd

```

## Amazon Linux 1
This is older version Linux from Amazon, still popular and many organization has this version installed. This is sys V based Operating System. I carried out the experiment with the following specifications. 
AMI: amzn-ami-hvm-2018.03.0.20180508-x86_64-gp2 (ami-76d6f519) 
Cloud Platform: Amazon 
Operating System: Amazon Linux 1 
Kernel: 4.14.33-51.37.amzn1.x86_64 Architecture: x86–64 
RPMS : awslogs-1.1.4-1.12.amzn1.noarch 
  1. Launch instance using Amazon AMI-1 image and attach the Role CWRole-LinuxLog to it. 2. Log in to the instance and install awslogs package
```
$ sudo yum install awslogs

```
 3. Edit file /etc/awslogs/awscli.conf and change your AWS Region.
 4. Edit file /etc/awslogs/awslogs.conf and verify the following lines.
```
[/var/log/messages]
datetime_format = %b %d %H:%M:%S
file = /var/log/messages
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name =  AMZ-1

```

5. The above configuration will create a hierarchical structure in the CloudWatch interface with log_group_name AMZ-1 ( an arbitrary name) as a container and respective instance id as the object where contents of “/var/log/messages” file is redirected.
6. Start and enable awslogsd service by typing the command:

```

$ sudo service awslogs start
$ sudo chkconfig  awslogs on

```

## RHEL 7/CentOS 7
Unfortunately, RPM packages are not available. So you cannot use yum command to install the CloudWatch agent package. You can follow the same steps for CentOS 7 also. 
AMI: RHEL-7.5_HVM_GA-20180322-x86_64-1-Hourly2-GP2 (ami-5b673c34) 
Cloud Platform: Amazon 
Operating System: RHEL-7.5 
Kernel: 3.10.0-862.el7.x86_64 
Architecture: x86–64 
 1. Launch Instance using amazon RHEL 7 image and attach the role CWRole-LinuxLog to it. 
 2. Login to the instance and download awslogs package. You dont have RPM packages available here.

```

curl https://s3.amazonaws.com/aws-cloudwatch/downloads/latest/awslogs-agent-setup.py -O

```
 3.Configure CloudWatch agent by executing the command below. Follow the screen shot to set up the agent configuration file.
```
# python ./awslogs-agent-setup.py --region ap-south-1

```
![](https://medium.com/tensult/to-send-linux-logs-to-aws-cloudwatch-17b3ea5f4863)


 4.The command above creates the configuration file /var/awslogs/etc/awslogs.conf like below.

```

[/var/log/messages]
datetime_format = %b %d %H:%M:%S
file = /var/log/messages
buffer_duration = 5000
log_stream_name = {instance_id}
initial_position = start_of_file
log_group_name = RHEL-7

```

 5.Execute the command below to start and enable awslogs the service.
```

$ systemctl start awslogs
$ chkconfig awslogs on

```
### Testing the configuration 
### Open CloudWatch web console and click Logs in the left pane to view logs.

![](https://medium.com/tensult/to-send-linux-logs-to-aws-cloudwatch-17b3ea5f4863)

##### Note: Configuration can be repeated for other log files like /var/log/secure by adding parameters in awslogs.conf. Note that path of awslogs.conf is different for different flavors of Operating Systems.

### Conclusion
We have understood the importance of centralized loggings solution and how CloudWatch can be used for the same. Configuration steps are different in some of the Linux flavors.
