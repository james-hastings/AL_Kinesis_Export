# AL_Kinesis_Export
Export AL IDS data and logs to AWS Kinesis.


# Overview

This document covers implementing the AL Kinesis Export CloudFormation template in your environment as well as the information you need to provide back to Alert Logic to get real-time AL data published.

Customers that have worked with Alert Logic to setup the export of IDS and log data to an AWS Kinesis stream have the option to enable additional shard-level metrics and corresponding CloudWatch alarms to monitor and alert on stream performance.

The process entails enabling an advanced metric on each Alert Logic Kinesis stream so that monitoring the write throughput capacity becomes possible.  This gives insight into the provisioned shards, and can be used in conjunction with CloudWatch alarms, to alert you when your current shard count is not sufficient for consistent IDS and log data export.

The additional metric necessary is considered “enhanced” and does incur additional charges from AWS – although these are minimal.  At the time of document creation, the enhanced metric covered here is roughly $.30/stream/month.  For more information on cost please see: https://aws.amazon.com/cloudwatch/pricing/

Documentation on monitoring Kinesis metrics via CloudWatch is available here: 

https://docs.aws.amazon.com/streams/latest/dev/monitoring-with-cloudwatch.html#kinesis-metrics-shard


# Running the CloudFormation Template

Customers need to run one of the three CloudFormation templates to setup the Kinesis streams and IAM Role/Policy necessary for Alert Logic IDS+Log export.

There are currently three supported export regions where you can stand up your Kinesis infrastructure – US-East-1, US-West-2, and EU-West-1.  These locations represent Alert Logic AWS data center deployments and reduce the cross-region data-transfer charges.

Make sure you run the appropriate template in its respective AWS Region.

You need to enter in an external ID value – this is used to validate that the entity attempting to use the IAM Role is authorized.  This should be a string value that is at least two characters long.

Once entered click the buttons to continue through the process; you do not need to configure any other parts of the template. 

When prompted, check the box that states infrastructure will be created and click the button to run the template.

Once the template finishes running you can expand the “Outputs” section to find the Role ARN and external ID values.

# You should submit the values below back to Alert Logic:
•	CID (Alert Logic account ID)
•	Alert Logic account name
•	Kinesis Stream ARNs for both log and ids streams
•	IAM Role ARN
•	External ID value



# Enabling Shard-level Metrics

This process enables the shard-level metric, “WriteProvisionedThroughputExceeded”, for the Kinesis Streams used to publish IDS and log data from Alert Logic.  

Start by logging into your AWS management console and navigating to the Amazon Kinesis dashboard in the region you setup the Alert Logic Kinesis streams in.  
You can use this link to jump directly to your Kinesis dashboard: https://console.aws.amazon.com/kinesis/

Select the first Alert Logic Kinesis stream and scroll to the bottom of the Stream detail page until you find the “Shard level metrics” section.

Click on the edit button, and then select “WriteProvisionedThroughputExceeded” metric, and then click on “Save”.

If you have additional Streams, go back and do this process for each stream you would like to monitor.

The new metric can take up to a few hours to populate and become usable in the next step where a CloudWatch alarm is setup to provide automatic notification when the shard throughput is exceeded and needs attention.



# Setting Up a CloudWatch Alarm 

Once the enhanced Kinesis Stream metric is setup you can also create a CloudWatch alarm to notify you via email when the write capacity of the provisioned shards is exceeded.  This is most likely to happen on the Stream used to publish log data to as the log volume typically exceeds the IDS data by several orders of magnitude.

Navigate to the CloudWatch section of your AWS Management console - or jump directly there via this link: https://console.aws.amazon.com/cloudwatch/

Click on the “Alarms” section in the left-hand navigation bar, and then select “Create Alarm” to get started.

Click on the “Select metric” button.

Click on “Kinesis”, and then choose the “Stream Metrics” button.  Scroll through the options and select the entry/entries for “WriteProvisionedThroughputExceeded”, and then click “Select Metric”.

Enter an alarm name and description that will be useful – perhaps something like:

Name: Alert Logic Kinesis Alarm
Description:  Triggered when shard throughput is exceeded - add more shards.

For the whenever section, set the field as follows:
	
	is: > 0
	for: 1 out of 1 datapoints

The CloudWatch alarm can publish to an existing AWS SNS Topic, or you can select the option to create a new list and enter in one or more email addresses.

Once you complete all of the fields, click the “Create Alarm” button.
