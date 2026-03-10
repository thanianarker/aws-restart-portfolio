# Monitoring Infrastructure with CloudWatch and AWS Config

## Overview

I completed a comprehensive hands-on lab on infrastructure monitoring and compliance using AWS's monitoring and auditing services. Through this lab, I gained practical experience installing the CloudWatch agent on EC2 instances, monitoring application logs in real-time, analyzing system metrics, configuring automated alerts, and evaluating infrastructure compliance. I demonstrated the ability to deploy monitoring agents at scale, create metric filters from log data, configure SNS notifications for alarms, and establish CloudWatch Events rules for operational changes. By the end of this lab, I showcased proficiency with critical monitoring and observability tools essential for maintaining reliable, compliant cloud infrastructure.

## Topics Covered

After completing this lab, I was able to:

- Install the CloudWatch agent on EC2 instances using Systems Manager
- Configure CloudWatch agent with custom metrics and log collection
- Store CloudWatch agent configuration in AWS Systems Manager Parameter Store
- Monitor application logs using CloudWatch Logs
- Create metric filters to extract data from log files
- Generate custom metrics from log data
- Create CloudWatch alarms based on custom metrics
- Configure SNS topics for alarm notifications
- Verify alarm functionality through simulated events
- Access CloudWatch Metrics dashboards
- Explore CWAgent custom metrics collected from instances
- Understand the difference between AWS-generated and agent-generated metrics
- Configure CloudWatch Events rules for operational changes
- Create SNS subscriptions for real-time notifications
- Understand infrastructure compliance monitoring with AWS Config
- Apply AWS Config rules to enforce compliance standards
- Evaluate resource tagging compliance

## Introducing the Technologies

### AWS Monitoring and Observability Stack

AWS provides a comprehensive suite of services for monitoring, logging, and compliance tracking across infrastructure and applications.

| Service | Purpose | Use Case |
|---------|---------|----------|
| **CloudWatch** | Centralized monitoring, logging, and metrics | Application and infrastructure monitoring |
| **CloudWatch Agent** | Collects detailed metrics and logs from instances | System-level and custom metrics |
| **CloudWatch Logs** | Log aggregation and analysis | Application log monitoring and troubleshooting |
| **CloudWatch Events** | Real-time event routing and automation | Operational change notifications and automation |
| **CloudWatch Alarms** | Automated alerts based on metrics | Incident notification and auto-remediation |
| **AWS Config** | Configuration tracking and compliance | Infrastructure compliance auditing |
| **AWS Systems Manager** | Fleet management and automation | Agent deployment and configuration |

### CloudWatch Agent Capabilities

| Metric Type | Sources | Purpose |
|-------------|---------|---------|
| **System Metrics** | CloudWatch (default) | EC2 CPU, network, disk I/O (external view) |
| **Detailed System Metrics** | CloudWatch Agent | Memory, disk space, swap usage (internal view) |
| **Application Logs** | CloudWatch Agent | Apache, custom applications, system logs |
| **Custom Metrics** | CloudWatch Agent | StatsD, collectd protocol support |

### Log Filtering and Metric Creation

CloudWatch Logs enables creating custom metrics from log data using filter patterns:

| Component | Purpose | Example |
|-----------|---------|---------|
| **Filter Pattern** | Defines log field structure and conditions | `[ip, id, user, timestamp, request, status_code=404, size]` |
| **Metric Filter** | Extracts data matching the pattern | Counts 404 errors in web server logs |
| **Custom Metric** | Result metric published to CloudWatch | `LogMetrics/404Errors` |
| **Alarm** | Triggers notification when threshold exceeded | Alert when 404 errors ≥ 5 per minute |

## Tasks

### Task 1: Install CloudWatch Agent on EC2 Instance

#### Step 1.1: Access Systems Manager Run Command

I navigated to the AWS Management Console and selected **Systems Manager** from the Services menu.

In the left navigation pane, I selected **Run Command** to access the command execution interface.

📝 **Note**: Run Command is part of AWS Systems Manager, which enables remote command execution on EC2 instances without SSH access. This requires an IAM role with Systems Manager permissions (pre-configured on the instance).

#### Step 1.2: Install CloudWatch Agent Package

I clicked **Run a command** and selected the **AWS-ConfigureAWSPackage** document from the list.

In the **Command parameters** section, I configured:

| Parameter | Value |
|-----------|-------|
| Action | Install |
| Name | AmazonCloudWatchAgent |
| Version | latest |

In the **Targets** section, I selected **Choose instances manually** and checked the **Web Server** instance.

I clicked **Run** to execute the installation command.

#### Step 1.3: Verify Installation Success

I waited for the **Overall status** to change to **Success**. I refreshed the page to check progress.

I clicked on the Web Server instance under **Targets and outputs** and selected **View output**.

I expanded **Step 1 - Output** and confirmed the success message:

```
Successfully installed arn:aws:ssm:::package/AmazonCloudWatchAgent
```

✅ **Task complete**: CloudWatch agent successfully installed on Web Server instance.

💭 **Consider**: The installation process uses Systems Manager's document execution framework, which tracks command status, output, and errors. This provides audit trails and error diagnostics without requiring direct SSH access.

#### Step 1.4: Create CloudWatch Agent Configuration

I navigated to **Systems Manager** → **Parameter Store** and clicked **Create parameter**.

I configured the parameter:

| Setting | Value |
|---------|-------|
| Name | Monitor-Web-Server |
| Description | Collect web logs and system metrics |
| Type | String |
| Value | (JSON configuration below) |

I pasted the CloudWatch agent configuration JSON:

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "log_group_name": "HttpAccessLog",
            "file_path": "/var/log/httpd/access_log",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%b %d %H:%M:%S"
          },
          {
            "log_group_name": "HttpErrorLog",
            "file_path": "/var/log/httpd/error_log",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%b %d %H:%M:%S"
          }
        ]
      }
    }
  },
  "metrics": {
    "metrics_collected": {
      "cpu": {
        "measurement": [
          "cpu_usage_idle",
          "cpu_usage_iowait",
          "cpu_usage_user",
          "cpu_usage_system"
        ],
        "metrics_collection_interval": 10,
        "totalcpu": false
      },
      "disk": {
        "measurement": [
          "used_percent",
          "inodes_free"
        ],
        "metrics_collection_interval": 10,
        "resources": [
          "*"
        ]
      },
      "diskio": {
        "measurement": [
          "io_time"
        ],
        "metrics_collection_interval": 10,
        "resources": [
          "*"
        ]
      },
      "mem": {
        "measurement": [
          "mem_used_percent"
        ],
        "metrics_collection_interval": 10
      },
      "swap": {
        "measurement": [
          "swap_used_percent"
        ],
        "metrics_collection_interval": 10
      }
    }
  }
}
```

**Configuration breakdown:**

| Section | Collects | Purpose |
|---------|----------|---------|
| **logs.files** | Apache access and error logs | Track HTTP requests, errors, and anomalies |
| **metrics.cpu** | CPU usage metrics (idle, iowait, user, system) | Detect CPU utilization and contention |
| **metrics.disk** | Disk usage and I/O operations | Monitor storage capacity and performance |
| **metrics.mem** | Memory utilization percentage | Track RAM usage patterns |
| **metrics.swap** | Swap usage percentage | Detect memory pressure situations |

I clicked **Create parameter** to save the configuration.

✅ **Task complete**: CloudWatch agent configuration stored in Parameter Store.

#### Step 1.5: Start CloudWatch Agent with Configuration

I returned to **Run Command** and clicked **Run command**.

I filtered for the **AmazonCloudWatch-ManageAgent** document by:
1. Selecting the document name prefix filter
2. Entering "AmazonCloudWatch-ManageAgent"
3. Pressing Enter

I selected the **AmazonCloudWatch-ManageAgent** document.

I configured the command parameters:

| Parameter | Value |
|-----------|-------|
| Action | configure |
| Mode | ec2 |
| Optional Configuration Source | ssm |
| Optional Configuration Location | Monitor-Web-Server |
| Optional Restart | yes |

In the **Targets** section, I selected **Choose instances manually** and checked **Web Server**.

I clicked **Run** to start the CloudWatch agent.

#### Step 1.6: Verify Agent Configuration

I waited for the **Overall status** to change to **Success**.

📝 **Note**: The agent now retrieves its configuration from Parameter Store and begins collecting logs and metrics from the Apache web server and system. This centralized configuration approach allows managing agent settings across many instances without re-deploying the agent package.

✅ **Task complete**: CloudWatch agent configured and running with log and metric collection active.

### Task 2: Monitor Application Logs Using CloudWatch Logs

#### Step 2.1: Generate Test Log Data

I copied the **WebServerIP** from the lab details panel.

I opened a new browser tab and navigated to the WebServerIP to access the web server test page.

I then appended `/start` to the URL and pressed Enter to trigger a 404 error.

This request generated log entries in the Apache access logs that would be sent to CloudWatch.

📝 **Note**: The CloudWatch agent running on the instance continuously reads these log files and ships them to CloudWatch Logs. This happens automatically without logging into the server.

#### Step 2.2: Access CloudWatch Logs

I navigated to **CloudWatch** in the AWS Management Console.

In the left navigation pane, I selected **Log groups**.

I observed two log groups:
- **HttpAccessLog**: Contains HTTP request details (method, path, status code, user agent)
- **HttpErrorLog**: Contains server errors and warnings

I clicked on **HttpAccessLog** to open it.

#### Step 2.3: Examine Log Data

In the **Logs streams** section, I selected the log stream (identified by the instance ID).

The log data displayed GET requests sent to the web server, including my `/start` request with a **404** status code (page not found).

Each log entry contained:
- Client IP address
- Timestamp
- HTTP request (method, path, protocol)
- HTTP status code
- Response size
- Referring page and user agent

💭 **Consider**: Log streams organize logs by source (instance ID in this case). This hierarchical structure allows filtering and analyzing logs from specific instances or groups of instances.

✅ **Task complete**: Web server logs successfully streamed to CloudWatch Logs.

#### Step 2.4: Create Metric Filter for 404 Errors

I returned to **Log groups** and selected the checkbox next to **HttpAccessLog**.

From the **Actions** dropdown, I selected **Create metric filter**.

I pasted the filter pattern into the **Filter pattern** box:

```
[ip, id, user, timestamp, request, status_code=404, size]
```

**Pattern explanation:**
- `[ip, id, user, timestamp, request, status_code=404, size]` defines the log line structure
- `status_code=404` specifies that only lines with a 404 status code match the filter
- Other fields are captured but not filtered

In the **Test pattern** section, I selected my EC2 instance ID from the dropdown.

I clicked **Test pattern** and confirmed results showed at least one 404 entry in the **Results** section.

I clicked **Next** to proceed to naming the filter.

#### Step 2.5: Create Custom Metric

In the **Create filter name** section, I entered:

| Setting | Value |
|---------|-------|
| Filter name | 404Errors |

In the **Metric details** section, I configured:

| Setting | Value |
|---------|-------|
| Metric namespace | LogMetrics |
| Metric name | 404Errors |
| Metric value | 1 |

**Metric value explanation:** Each log entry matching the filter increments the metric by 1. This allows counting occurrences of 404 errors.

I clicked **Next** and then **Create metric filter** on the review page.

✅ **Task complete**: Custom metric filter created to track 404 errors from application logs.

#### Step 2.6: Create CloudWatch Alarm

I returned to the **404Errors** filter and selected the checkbox in the top-right corner of the filter panel.

I clicked **Create alarm** in the **Metric filters** section.

I configured the alarm settings:

**Metrics section:**
- Period: 1 minute

**Conditions section:**
- Whenever 404Errors is: **Greater/Equal**
- than: **5**

This creates an alarm that triggers when 5 or more 404 errors occur within a 1-minute window.

I clicked **Next** to configure notifications.

#### Step 2.7: Configure SNS Notification

In the **Notification** section:
- Notification type: SNS Topic
- Action: Create new topic

I entered an email address to receive alarm notifications.

I clicked **Create topic** to establish the SNS topic.

I clicked **Next** to configure alarm metadata.

#### Step 2.8: Name and Create Alarm

In the **Name and description** section, I configured:

| Setting | Value |
|---------|-------|
| Alarm name | 404 Errors |
| Alarm description | Alert when too many 404s detected on an instance |

I clicked **Next** and then **Create alarm**.

#### Step 2.9: Confirm SNS Subscription

I checked my email for a confirmation message from AWS SNS.

I clicked the **Confirm subscription** link to activate notifications for this topic.

✅ **Task complete**: Alarm and SNS notification configured for 404 errors.

#### Step 2.10: Trigger Alarm with Test Requests

I returned to the web server browser tab and made multiple requests to non-existent pages by appending different paths to the URL (e.g., `/start2`, `/test`, `/missing`).

I repeated this at least 5 times to generate 5+ 404 errors within the monitoring window.

I waited 1-2 minutes for CloudWatch to evaluate the alarm condition.

#### Step 2.11: Verify Alarm Triggered

I returned to the CloudWatch console and observed the alarm status changed from orange (Insufficient Data) to red (Alarm state).

I checked my email and received an alarm notification with the subject "ALARM: 404 Errors".

✅ **Task complete**: Alarm successfully triggered and notification received.

**Alarm notification contained:**
- Alarm name and description
- Timestamp the alarm entered alarm state
- Metric value that triggered the alarm
- Link to CloudWatch console for investigation

### Task 3: Monitor Instance Metrics Using CloudWatch

#### Step 3.1: Access EC2 Monitoring

I navigated to **EC2** in the AWS Management Console.

I selected **Instances** and checked the **Web Server** instance.

I clicked the **Monitoring** tab to view CloudWatch metrics.

The tab displayed graphs for:
- **CPU Utilization**: Percentage of available CPU being used
- **Network In/Out**: Data transferred to/from the instance
- **Disk Read/Write Operations**: Storage I/O operations

📝 **Note**: These metrics are captured by CloudWatch automatically and reflect the instance from an external perspective (hypervisor-level monitoring).

#### Step 3.2: Explore CloudWatch Metrics

I navigated to **CloudWatch** and selected **Metrics** in the left navigation pane.

I expanded **Metrics** and selected **All metrics** to view the full metric catalog.

#### Step 3.3: Explore CWAgent Custom Metrics

I selected **CWAgent** from the metric namespaces.

I navigated through the metric hierarchy: **CWAgent** → **device, fstype, host, path**

The metrics displayed disk space information collected by the CloudWatch agent running on the instance.

I clicked **CWAgent** (in the breadcrumb) to navigate back and then selected **host** to view memory metrics.

💭 **Consider**: CWAgent metrics provide visibility into what's happening *inside* the instance (memory, disk space), while default CloudWatch metrics provide external visibility (CPU, network). Together, they provide complete infrastructure observability.

#### Step 3.4: Explore AWS-Generated Metrics

I clicked **All** to view all available metrics.

I explored various AWS service metrics that CloudWatch automatically collects from services in use (EC2, EBS, etc.).

✅ **Task complete**: Successfully explored both agent-generated and AWS-generated metrics.

### Task 4: Create Real-Time Notifications with CloudWatch Events

#### Step 4.1: Access CloudWatch Events Rules

I navigated to **CloudWatch** and expanded **Events** in the left navigation pane.

I selected **Rules** to access the event rule management interface.

I clicked **Create rule** to begin creating a new event rule.

#### Step 4.2: Configure Event Rule

I configured the rule:

| Setting | Value |
|---------|-------|
| Name | Instance_Stopped_Terminated |

I clicked **Next** to proceed to event pattern configuration.

#### Step 4.3: Define Event Pattern

In the **Build event pattern** section, I configured:

| Setting | Value |
|---------|-------|
| Event source | AWS Services |
| AWS service | EC2 |
| Event Type | EC2 Instance State-change Notification |

I selected the checkbox for **Specific state(s)** and chose:
- **stopped**
- **terminated**

This rule now matches only events where an EC2 instance enters the stopped or terminated state.

I clicked **Next** to configure targets.

#### Step 4.4: Configure SNS Notification Target

In the **Select target(s)** section, I configured:

| Setting | Value |
|---------|-------|
| Target type | AWS Service |
| Target | SNS topic |
| Topic | Default_CloudWatch_Alarms_Topic |

I unchecked **Use execution role** (recommended) to use the existing role.

I clicked **Next** and then **Next** again to proceed to review.

I clicked **Create rule** to finalize the event rule.

✅ **Task complete**: CloudWatch Events rule created to notify on instance state changes.

#### Step 4.5: Verify Notification Infrastructure

I navigated to **Simple Notification Service** and selected **Topics**.

I clicked on the topic to view its subscriptions.

I confirmed my email address was subscribed to the topic (configured during alarm setup in Task 2).

✅ **Task complete**: SNS topic configured to deliver state-change notifications.

### Task 5: Monitoring Infrastructure Compliance (Partial)

#### Step 5.1: Access AWS Config

I navigated to **AWS Config** in the AWS Management Console.

If a **Get started** button appeared, I clicked it and followed the setup steps (Next → Next → Confirm).

📝 **Note**: AWS Config requires initial setup to enable resource monitoring and compliance tracking. The setup configures S3 buckets and IAM roles for audit logging.

#### Step 5.2: Create Required Tags Rule

In the left navigation pane, I selected **Rules** (toward the top).

I clicked **Add rule** and searched for `required-tags` in the AWS Managed Rules section.

I selected the **required-tags** rule.

I clicked **Next** and configured the rule on the **Configure rule** page.

Under **Parameters**, I set:
- **tag1Key**: `project` (replacing any existing value)

This rule requires all resources to have a "project" tag for compliance.

I clicked **Next** and then **Add rule**.

💭 **Consider**: Tagging compliance rules ensure consistent metadata across infrastructure, enabling cost allocation, resource organization, and automated policy enforcement.

#### Step 5.3: Create EBS Volume Attachment Rule

I clicked **Add rule** again and searched for `ec2-volume-inuse-check`.

I selected the **ec2-volume-inuse-check** rule.

I clicked **Next** twice to proceed to the add rule step.

I clicked **Add rule** to create the rule.

📝 **Note**: This rule checks that all EBS volumes are attached to EC2 instances, preventing orphaned storage that wastes costs.

#### Step 5.4: Wait for Rule Evaluation

I waited for AWS Config to scan and evaluate resources against these rules. This process takes a few minutes.

I refreshed the page if necessary to see rule evaluation results.

✅ **Task complete**: Compliance rules created and evaluation initiated.

**Rule evaluation will show:**
- **required-tags rule**: Web Server instance (compliant because it has a project tag) and other resources (non-compliant if they lack the project tag)
- **ec2-volume-inuse-check rule**: Attached volumes (compliant) and unattached volumes (non-compliant)

## Monitoring and Observability Architecture

The complete monitoring architecture I implemented:

```
EC2 Instance
├── CloudWatch Agent
│   ├── Collects logs → CloudWatch Logs
│   │   └── Metric filters → Custom metrics
│   │       └── Alarms → SNS notifications
│   └── Collects metrics → CloudWatch Metrics
│       └── Dashboards & analysis
│
├── System changes
│   └── CloudWatch Events
│       └── SNS notifications
│
└── Configuration
    └── AWS Config rules
        └── Compliance reporting
```

## Conclusion

I successfully demonstrated comprehensive infrastructure monitoring and compliance capabilities:

- ✅ Installed CloudWatch agent on EC2 instances using Systems Manager
- ✅ Stored agent configuration in Parameter Store for centralized management
- ✅ Configured agent to collect Apache web server logs
- ✅ Configured agent to collect system metrics (CPU, disk, memory, swap)
- ✅ Monitored application logs in CloudWatch Logs
- ✅ Created metric filters to extract 404 errors from web logs
- ✅ Generated custom metrics from log data
- ✅ Created CloudWatch alarms based on custom metrics
- ✅ Configured SNS topics and email subscriptions
- ✅ Triggered and verified alarm notifications
- ✅ Explored CloudWatch Metrics dashboards
- ✅ Differentiated between AWS-generated and agent-generated metrics
- ✅ Created CloudWatch Events rules for operational changes
- ✅ Configured real-time SNS notifications for instance state changes
- ✅ Began AWS Config compliance rule implementation
- ✅ Understood resource tagging and volume attachment compliance checks

This lab demonstrated essential skills for operating production infrastructure: real-time monitoring, automated alerting, log analysis, and compliance tracking. These capabilities enable rapid issue detection, root cause analysis, and compliance auditing at scale.

## Additional Resources

- [CloudWatch Documentation](https://docs.aws.amazon.com/cloudwatch/)
- [CloudWatch Agent Configuration Reference](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch-Agent-Configuration-File-Details.html)
- [CloudWatch Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_QuerySyntax.html)
- [CloudWatch Alarms Best Practices](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Best_Practice_Recommended_Alarms_AWS_Services.html)
- [CloudWatch Events User Guide](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/)
- [AWS Config Rules](https://docs.aws.amazon.com/config/latest/developerguide/managed-rules-by-aws-config.html)
- [AWS Systems Manager Run Command](https://docs.aws.amazon.com/systems-manager/latest/userguide/run-command.html)
- [AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)
- [Amazon SNS Configuration](https://docs.aws.amazon.com/sns/latest/dg/welcome.html)
- [Infrastructure Monitoring Best Practices](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Best_Practice_Recommended_Alarms_AWS_Services.html)
