---
title: "CloudWatch Agent Manual Install"
date: 2020-04-24T11:16:09-04:00
chapter: false
pre: "<b>4. </b>"
weight: 4
---

{{% notice note %}}
There are multiple ways to install the CloudWatch agent. This lab will walk through a manual install on a single instance. Please visit the [CloudWatch installation documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-on-EC2-Instance.html) for a comprehensive list of ways to install CloudWatch.
{{% /notice %}}

1. On the left bar in the EC2 console, click on **Instances** and select the **EC2 Instance** with the **CloudWatchAgentServerRole** IAM role.
![Images/MemInstall02.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall02.png?classes=lab_picture_small)

2. Connect into the EC2 Instance using the **browser-based SSH connection tool**.
![Images/MemInstall03.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall03.png?classes=lab_picture_small)
![Images/MemInstall04.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall04.png?classes=lab_picture_small)
![Images/MemInstall05.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall05.png?classes=lab_picture_small)

3. Download the **Amazon Cloudwatch** agent package, the instructions below are for Amazon Linux, for other OS please check [here](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html)

```
wget https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/AmazonCloudWatchAgent.zip
```

![Images/MemInstall06.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall06.png?classes=lab_picture_small)

4. Unzip and Install the package

```
unzip AmazonCloudWatchAgent.zip
sudo ./install.sh
```

![Images/MemInstall07.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall07.png?classes=lab_picture_small)

5. Configure the AmazonCloudWatchAgent profile

Before running the CloudWatch agent on any servers, you must create a CloudWatch agent configuration file, which is a JSON file that specifies the metrics and logs that the agent is to collect, including custom metrics. You can create it by using the wizard or by writting it yourself from scratch. Any time you change the agent configuration file, you must then restart the agent to have the changes take effect.

The wizard can autodetect the credentials and AWS Region to use if you have the AWS credentials and configuration files in place. For more information about these files, see [Configuration and Credential Files](https://docs.aws.amazon.com/cli/latest/userguide/cli-config-files.html) in the *AWS Systems Manager User Guide* and the [AWS documentation page](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file.html).

For now, let's start the CloudWatch agent configuration file wizard executing the command below at the selected EC2 instance.

```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

![Images/MemInstall08.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall08.png?classes=lab_picture_small)

For this lab we want to keep the following structure:

| CloudWatch Agent Configutation File Wizard                  | Parameter    |
| ----------------------------------------------------------- |:------------:|
| On which OS are you planning to use the agent?              | 1. Linux     |
| Are you using EC2 or On-Premises hosts?                     | 1. EC2       |
| Which user are you planning to run the agent?               | 2. cwagent   |
| Do you want to turn on StatsD daemon?                       | 2. No        |
| Do you want to monitor metrics from CollectD?               | 2. No        |
| Do you want to monitor any host metrics?                    | 1. Yes       |
| Do you want to monitor cpu metrics per core?                | 2. No        |
| Do you want to add ec2 dimensions?                          | 1. Yes       |
| Would you like to collect your metrics at high resolution?  | 4. 60s       |
| Which default metrics config do you want?                   | 1. Basic     |
| Are you satisfied with the above config?                    | 1. Yes       |
| Do you have any existing CloudWatch Log Agent?              | 2. No        |
| Do you want to monitor any log files?                       | 2. No        |
| Do you want to store the config in the SSM parameter store? | 2. No        |

The CloudWatch Agent config file should look like the following:

```
{
	"agent": {
			"metrics_collection_interval": 60,
			"run_as_user": "cwagent"
	},
	"metrics": {
			"append_dimensions": {
				"AutoScalingGroupName": "${aws:AutoScalingGroupName}",
				"ImageId": "${aws:ImageId}",
				"InstanceId": "${aws:InstanceId}",
				"InstanceType": "${aws:InstanceType}"
			},
			"metrics_collected": {
				"disk": {
					"measurement": [
						"used_percent"
				],
				"metrics_collection_interval": 60,
					"resources": [
							"*"
				]
			},
			"mem": {
					"measurement": [
						"mem_used_percent"
					],
					"metrics_collection_interval": 60
			}
		}
	}
}
```

6. Start the CloudWatch Agent

```
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s
```

![Images/MemInstall09.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall09.png?classes=lab_picture_small)

7. It may take up to 5 minutes for the metrics to become available, go back to the **Amazon CloudWatch** console page, under the **Metrics** session to validate that you are getting Memory information.

8. Click **CWAgent**:
![Images/MemInstall10.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall10.png?classes=lab_picture_small)

9. Click **ImageID,InstanceID,InstanceType**:
![Images/MemInstall11.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall11.png?classes=lab_picture_small)

10. Select the **Instance** from the list below:
![Images/MemInstall12.png](/Cost/200_AWS_Resource_Optimization/Images/AgentInstall12.png?classes=lab_picture_small)

You have now completed the CloudWatch agent installation and will be able to monitor on Amazon CloudWatch the memory utilization of that instance.

#### Automation Option
{{% notice note %}}
The next step is not mandatory to complete this lab.
{{% /notice %}}

If you have to install and start the CloudWatch agent on several instances at once doing it manually might not be a scalable option. Consider using [AWS Systems Manager](https://aws.amazon.com/systems-manager/) or a pre-configured AWS CloudFormation template to automatically install the CloudWatch agent by default on all your stacks.

[AWS Systems Manager](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/installing-cloudwatch-agent-ssm.html) Steps:

- Create IAM Roles and Users for Use with CloudWatch Agent
- Download and Configure the CloudWatch Agent
- Install the CloudWatch Agent on EC2 Instances Using Your Agent Configuration
- Install the CloudWatch Agent on On-Premises Servers

CloudFormation Steps:

- Right-click and save link as: [here](https://raw.githubusercontent.com/awslabs/aws-cloudformation-templates/master/aws/solutions/AmazonCloudWatchAgent/inline/amazon_linux.template) to download the **AWS Cloudformation** template
- Go to the **AWS CloudFormation** console
- Click to **Create Stack** and select **Upload a template file** and point to the downloaded file
- Enter a **Stack Name** and select a **KeyName**
- Enter the tag **Key: Event | Value: myStackforWACostLab**
- Click **Next** and **Create stack**

{{< prev_next_button link_prev_url="../3_attach_iamrole/" link_next_url="../5_ec2_computer_opt/" />}}
