Not monitoring MEMORY usage can cause serious production issues, adding more time/cost if its not being monitored. Leaving your application with further 
delays and even failures, causing frustration between your company trust partners/clients. MEM should always be monitored as it's a critical factor.
Please note, you can always gain MEM information and more by SSH-ing to your Instance(s) and issuing the following cmds below if metrics are not readily available via console.

free -mh 
top 
cat /proc/meminfo
glances (requires install) 
htop (requires install) 
 
Before creating a custom metrics on AWS Instances, we need to understand why AWS does not provide this metrics in EC2 monitoring tab or
in CloudWatch. The default metrics provided by AWS are collected by the Hypervisor/host level( the physical hardware), whereas the MEMORY metrics is at the 
Operating system level, therefore it becomes a custom metric that we have to create and push to CloudWatch.


"The default namespace for metrics collected by the CloudWatch agent is CWAgent, although you can specify a different namespace when you configure the agent.
Metrics collected by the CloudWatch agent are billed as custom metrics. The CloudWatch agent is open source under the MIT license, and is hosted on GitHub"

The following steps walk you through how to create and attach IAM role to your EC2, Install CloudWatch Agent

Step 1) Create an IAM role to send custom metrics to CloudWatch and then attach role to our EC2 instance(s).

IAM → Create Role → Select Ec2 and then click on Next permissions as shown below
"CustomMetricCloudWatch"

Step 2) Now go to EC2 instance → select EC2 instance → select action → and then select security and then select Modify IAM role


Step 3) Login to the EC2 instance and Install the agent and configure the metrics config file.
   Please view the URL to download appropriate agent for your Linux distro
   https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html

> wget https://s3.amazonaws.com/amazoncloudwatch-agent/redhat/amd64/latest/amazon-cloudwatch-agent.rpm
> sudo rpm -U ./amazon-cloudwatch-agent.rpm

Create and add the following content. The agent to collect memory usage data in percentage format every 60 seconds.
With "InstanceId" appended, you will receive a recommendation from AWS based on MEM usage.
vi /opt/aws/amazon-cloudwatch-agent/bin/config.json

```
{
   "metrics":{
      "metrics_collected":{
         "mem":{
            "measurement":[
               "mem_used_percent"
            ],
            "metrics_collection_interval":60
         }
      },
      "append_dimensions": {
        "InstanceId": "${aws:InstanceId}"
      }
   }
}
```

(Another way of creating the .Json config with your metrics details is to use the CloudWatch Wizard, cmd list below.)
(sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard)

Step 4) Start CloudWatch Agent with the command following command

> sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

Step 5) check the status of agent
**sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a status**
> sudo systemctl status amazon-cloudwatch-agent


Step 6) View the logs if you run into any issues

> cat /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log



Step 7) Go to CloudWatch > under Metrics > All metrics and you'll see a custom namespaces "CWAgent", ready to be viewed
   in a graph. Allow some time to process collection of the metric.


Summary

We have gone through the steps to create and attach IAM role and install CloudWatch Agent and basic .json config to report MEM usage.
Please note this can be automated using IAC and other tools to automatically push to EC2. MY goal was to show you in a simple way to
understand. If I missed anything, please let me know.


The following AWS resources are useful for further information

https://aws.amazon.com/premiumsupport/knowledge-center/cloudwatch-push-metrics-unified-agent/
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/download-cloudwatch-agent-commandline.html
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html

There is another method of doing this by install scripts, but it's deprecated, and recommend to use CloudWatch agent.
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-scripts-intro.html
