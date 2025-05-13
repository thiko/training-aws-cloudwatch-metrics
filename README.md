# Exercise: Creating CloudWatch Metric Filters and Alarms for S3 Access

## Objective
Learn how to create CloudWatch Metric Filters to monitor specific patterns in logs, and how to set up alarms based on these metrics. In this exercise, we'll focus on monitoring access to your S3 bucket using CloudTrail logs.

## Overview
CloudWatch Metric Filters allow you to extract specific data from log events and convert them into CloudWatch metrics. These metrics can then be used to create dashboards and alarms. In this exercise, we'll track S3 object access patterns and create relevant alerts.

## Steps

### Part 1: Enable CloudTrail Logging for S3

1. **Enable CloudTrail for S3 events**
   - Navigate to the CloudTrail service in the AWS Management Console
   - Click "Create trail"
   - Enter a name for your trail: `s3-access-trail`
   - For "Storage location", create a new S3 bucket or use an existing one
   - Activate CloudWatch Logs ingestion - enter a suitable name for the log group (you can also name it like the trail `/aws/cloudtrail/s3-access-trail`)
   - Under "Events", select "Management events"
   - Ensure "Read" and "Write" are selected
   - Click "Next" and then "Create trail"

2. **Configure S3 data events**
   - In your new trail, click on the "Data events" tab
   - Click "Edit"
   - Add an S3 data event
   - Select the S3 bucket you created in the previous exercise
   - Enable logging for "All object-level API activity"
   - Save your changes

3. **Generate some S3 activity**
   - Return to your S3 bucket
   - Upload a few new files
   - Download some existing files
   - Delete one file
   - These actions will generate CloudTrail logs

### Part 2: Create a Metric Filter and Alarm

1. **Navigate to CloudWatch Logs**
   - Go to the CloudWatch service
   - Select "Log groups" from the left navigation
   - Find the log group that corresponds to your CloudTrail (typically `/aws/cloudtrail/s3-access-trail`)

2. **Create a Metric Filter for S3 Object Downloads**
   - Select your CloudTrail log group
   - Click "Actions" and select "Create metric filter"
   - Define the filter pattern:
     ```
     { $.eventName = "GetObject" && $.requestParameters.bucketName = "training-[YourName]-[Date]" }
     ```
     (Replace with your actual bucket name)
   - Click "Next"
   - Name your filter: `S3ObjectDownloads`
   - For the metric namespace, enter: `S3AccessMetrics`
   - For the metric name, enter: `ObjectDownloads`
   - For the metric value, enter: `1` (to count each occurrence)
   - Leave "Default value" empty
   - Click "Next" and then "Create metric filter"

3. **Create a Metric Filter for S3 Object Uploads**
   - Repeat the process above but with this filter pattern:
     ```
     { $.eventName = "PutObject" && $.requestParameters.bucketName = "training-[YourName]-[Date]" }
     ```
   - Name the filter: `S3ObjectUploads`
   - Use the same namespace: `S3AccessMetrics`
   - For the metric name, enter: `ObjectUploads`

4. **Create a CloudWatch Alarm**
   - Navigate to CloudWatch "Metrics" on the left navigation
   - Find your custom namespace `S3AccessMetrics`
   - Select the metrics you created
   - Click "Create alarm"
   - Set the threshold to be "Greater than or equal to 5" for the `ObjectDownloads` metric (meaning 5 downloads within the time period)
   - Set the evaluation period to 5 minutes
   - Click "Next"
   - For notification:
     - Select "Create new topic" if you don't have one
     - Enter a topic name: `S3AccessAlerts`
     - Enter your email address
     - Click "Create topic"
   - Click "Next"
   - Name your alarm: `S3HighDownloadActivity`
   - Add a description: `Alert when more than 5 downloads occur within 5 minutes`
   - Click "Next" and then "Create alarm"

5. **Confirm the SNS subscription**
   - Check your email
   - Click the confirmation link from AWS SNS to confirm your subscription

### Part 3: Test Your Metric Filter and Alarm

1. **Generate test data**
   - Return to your S3 bucket
   - Download the same file multiple times in succession (at least 5 times)
   - Note: CloudTrail logs may take a few minutes to appear

2. **Monitor the metric**
   - Navigate to CloudWatch "Metrics"
   - Find your custom metrics under the `S3AccessMetrics` namespace
   - Create a simple graph showing both upload and download activity
   - Adjust the time range to the last hour to see your recent activity

3. **Verify the alarm**
   - Check the "Alarms" section in CloudWatch
   - Verify if your alarm triggered based on the download activity
   - Check your email for any alarm notifications

### Part 4: Create a CloudWatch Dashboard

1. **Create a new dashboard**
   - In CloudWatch, navigate to "Dashboards"
   - Click "Create dashboard"
   - Name your dashboard: `S3-Activity-Dashboard`
   - Click "Create dashboard"

2. **Add metrics to your dashboard**
   - Select "Line" as the widget type
   - Find your `S3AccessMetrics` namespace
   - Select both metrics (`ObjectDownloads` and `ObjectUploads`)
   - Click "Create widget"
   - Adjust the time range to show the last few hours

3. **Add a second widget for alarm status**
   - Click "Add widget"
   - Select "Alarm status"
   - Select your `S3HighDownloadActivity` alarm
   - Click "Create widget"

4. **Save your dashboard**
   - Arrange the widgets as desired
   - Click "Save dashboard"

## Verification

Your setup is complete when:
- CloudTrail is logging S3 object-level activities
- Two metric filters are capturing download and upload events
- An alarm is configured to trigger when download activity exceeds your threshold
- A dashboard is displaying your custom metrics and alarm status
- You've observed the metrics changing in response to your S3 activity

## Cleanup

To avoid ongoing charges:
- Consider disabling the CloudTrail trail when you're done with the exercise
- Delete the CloudWatch alarm if no longer needed
- Delete the CloudWatch dashboard you created
- Delete any SNS topics created for this exercise
- Note that CloudWatch metric filters themselves don't incur charges, but storing the logs does

## Extended Learning

- Try creating additional metric filters to track other S3 operations like DeleteObject
- Experiment with different statistic types (Sum, Average, Maximum) for your metrics
- Create a composite alarm that triggers based on both upload and download activity

## Key AWS Metrics Concepts

- **Namespace:** A container for metrics that helps organize metrics from different services. Example: AWS/EC2 is the namespace for all EC2 metrics.
- **Metric:** A time-ordered set of data points representing a variable you want to monitor. Example: CPUUtilization tracks processor usage over time.
- **Dimension:** A name/value pair that is part of a metric's identity, used to uniquely identify and filter metrics. Example: InstanceId=i-1234567890abcdef0 specifies which instance the metric applies to.
-  **Data Point:** A value of a metric at a specific point in time. Example: A CPU utilization reading of 75% at 14:05:30.
- Statistic: Aggregated metric data calculated over a specified time period. Example: Maximum CPU usage across a fleet of instances.
-  **Period:** The length of time over which statistics are aggregated. Example: A 5-minute period creates data points at 5-minute intervals.
- **Unit:** The standard of measurement for a metric. Example: Bytes for data transfer or Percent for utilization metrics.