# Amazon Route 53 Failover Routing

## Overview

I completed a comprehensive hands-on lab on implementing automatic failover routing using Amazon Route 53, AWS's DNS and domain management service. Through this lab, I gained practical experience configuring health checks that monitor endpoint availability, creating DNS failover policies, and verifying automatic traffic rerouting when primary infrastructure becomes unavailable. I transformed a multi-AZ application environment into a self-healing system that automatically detects failures and reroutes traffic to standby instances without manual intervention. By the end of this lab, I demonstrated proficiency with Route 53's failover routing capabilities essential for building highly available, fault-tolerant applications.

## Topics Covered

After completing this lab, I was able to:

- Understand DNS failover routing concepts and benefits
- Verify application deployment across multiple Availability Zones
- Configure Route 53 health checks for HTTP endpoints
- Set health check monitoring intervals and failure thresholds
- Create SNS topics and email notifications for health check failures
- Configure Route 53 A records with failover routing policies
- Understand primary and secondary failover record types
- Associate health checks with primary failover records
- Verify DNS resolution through Route 53
- Test failover functionality by simulating instance failure
- Monitor health check status and failure detection
- Verify automatic traffic rerouting to secondary instances
- Understand DNS propagation and TTL (Time-To-Live)
- Receive and confirm SNS email notifications
- Implement high availability through failover routing

## Introducing the Technologies

### Amazon Route 53

Route 53 is AWS's managed Domain Name System (DNS) and domain registration service that provides highly available DNS routing with advanced features like health checks and failover.

| Route 53 Feature | Benefit | Use Case |
|-----------------|---------|----------|
| **DNS Resolution** | Translates domain names to IP addresses | Users access application by domain name |
| **Health Checks** | Monitors endpoint availability | Detect infrastructure failures |
| **Failover Routing** | Automatically reroutes traffic | Achieve automatic failover without manual intervention |
| **Multi-AZ Support** | Routes across Availability Zones | Survive single AZ failure |
| **CloudWatch Integration** | Publishes metrics and alarms | Monitor Route 53 behavior |

### Failover Routing Policy

Failover routing enables automatic traffic rerouting based on health checks.

| Component | Purpose | Configuration |
|-----------|---------|---------------|
| **Primary Record** | Main destination for traffic | Points to primary instance, monitored by health check |
| **Secondary Record** | Standby destination | Points to secondary instance, used when primary fails |
| **Health Check** | Monitors primary endpoint | Returns healthy/unhealthy status |
| **TTL** | Cache time for DNS responses | Lower TTL (15s) = faster failover, higher DNS queries |

### Health Check Configuration

Health checks periodically verify endpoint availability and trigger alarms on failure.

| Setting | Purpose | Configuration |
|---------|---------|---------------|
| **Endpoint Type** | What to monitor | IP address, domain name, CloudWatch alarm |
| **Request Interval** | Check frequency | Standard (30s) or Fast (10s) |
| **Failure Threshold** | Consecutive failures before unhealthy | 1-10 failures |
| **Notifications** | Alert on status change | SNS topic with email subscribers |

### DNS Failover Flow

```
User Request
    ↓
Route 53 (check health of primary)
    ↓
Health Check Status?
├── Healthy → Return primary IP address
│   └── Request → Primary Instance
└── Unhealthy → Return secondary IP address
    └── Request → Secondary Instance (failover)
```

## Tasks

### Task 1: Verify Application Deployment

#### Step 1.1: Copy Lab Credentials

I accessed the lab details panel and copied the following values to a text editor:
- **CafeInstance1IPAddress**: IP of primary instance
- **PrimaryWebSiteURL**: Primary website access URL
- **SecondaryWebsiteURL**: Secondary website access URL
- **CafeInstance2IPAddress**: IP of secondary instance

#### Step 1.2: Verify EC2 Instances

I navigated to **EC2** → **Instances** and confirmed:

| Instance | Availability Zone | Configuration |
|----------|-------------------|---------------|
| CafeInstance1 | us-west-2a | Primary café application |
| CafeInstance2 | us-west-2b | Secondary café application (identical) |

📝 **Note**: Both instances are in different Availability Zones, providing fault tolerance. If one AZ fails, the other continues operating.

#### Step 1.3: Test Primary Website

I opened a browser tab and navigated to the **PrimaryWebSiteURL**.

The café website loaded, displaying:
- Café menu and application interface
- **Server Information** section showing instance ID and Availability Zone (us-west-2a)

#### Step 1.4: Test Secondary Website

I opened another browser tab and navigated to the **SecondaryWebsiteURL**.

The café website loaded with identical configuration, but:
- **Server Information** displayed different instance ID
- **Availability Zone** showed us-west-2b

#### Step 1.5: Test Application Functionality

On one of the websites, I clicked **Menu** to view café items.

I selected an item, clicked **Submit Order**, and the Order Confirmation page displayed the timestamp in the local timezone of that web server.

✅ **Task complete**: Both instances running café application with identical configuration across different AZs.

### Task 2: Configure Route 53 Health Check

#### Step 2.1: Create Health Check

I navigated to **Route 53** → **Health checks** and clicked **Create health check**.

I configured the health check:

| Setting | Value |
|---------|-------|
| Name | Primary-Website-Health |
| What to monitor | Endpoint |
| Endpoint type | IP address |
| IP address | CafeInstance1IPAddress (primary instance) |
| Path | cafe |

**Health check purpose:** Route 53 periodically sends HTTP requests to the specified path on the primary instance. If the instance responds successfully, the health check status is "Healthy". If it fails to respond, status becomes "Unhealthy".

#### Step 2.2: Configure Advanced Health Check Settings

I expanded **Advanced configuration** and configured:

| Setting | Value | Rationale |
|---------|-------|-----------|
| Request interval | Fast (10 seconds) | Faster failure detection |
| Failure threshold | 2 | Requires 2 consecutive failures before marking unhealthy |

**Failure threshold explanation:** A threshold of 2 prevents false positives (temporary network glitches) while still detecting genuine failures quickly.

I clicked **Next** to configure notifications.

#### Step 2.3: Configure SNS Notification

In the notification section, I configured:

| Setting | Value |
|---------|-------|
| Create alarm | Yes |
| Send notification to | New SNS topic |
| Topic name | Primary-Website-Health |
| Recipient email | (my email address) |

This creates an SNS topic that publishes alarm notifications when the health check status changes.

I clicked **Create health check** to finalize configuration.

#### Step 2.4: Verify Health Check Status

The health check appeared in the list with status **Healthy** (after 30-60 seconds).

I selected the health check and viewed the **Monitoring** tab to see historical health check status.

💭 **Consider**: The health check will show "Healthy" as long as the primary instance responds successfully to HTTP requests to the `/cafe` path. When we stop the instance later, this status will change to "Unhealthy".

✅ **Task complete**: Health check created and monitoring primary instance.

#### Step 2.5: Confirm SNS Subscription

I checked my email and found an AWS Notifications message with a **Confirm subscription** link.

I clicked the link to activate email notifications for this health check.

📝 **Note**: Without confirming the SNS subscription, I would not receive emails when the health check fails.

### Task 3: Configure Route 53 Failover Records

#### Step 3.1: Access Hosted Zone

I navigated to **Route 53** → **Hosted zones** and selected the domain `XXXXXX_XXXXXXXXXX.vocareum.training`.

The hosted zone displayed existing records:
- **NS record**: Name servers authoritative for this domain (don't modify)
- **SOA record**: Start of Authority with base DNS information (don't modify)

#### Step 3.2: Create Primary Failover Record

I clicked **Create record** and configured:

| Setting | Value |
|---------|-------|
| Record name | www |
| Record type | A (IPv4 address) |
| Value | CafeInstance1IPAddress |
| TTL (seconds) | 15 |
| Routing policy | Failover |
| Failover record type | Primary |
| Health check ID | Primary-Website-Health |
| Record ID | FailoverPrimary |

**Primary record configuration:**
- `www` record name creates the subdomain `www.XXXXXX_XXXXXXXXXX.vocareum.training`
- Points to primary instance's IP address
- TTL of 15 seconds allows quick failover (DNS caches this record for 15s)
- Associated with health check that monitors primary instance
- Failover type "Primary" indicates this is the preferred destination

I clicked **Create records** to finalize the primary record.

#### Step 3.3: Create Secondary Failover Record

I clicked **Create record** again and configured:

| Setting | Value |
|---------|-------|
| Record name | www |
| Record type | A (IPv4 address) |
| Value | CafeInstance2IPAddress |
| TTL (seconds) | 15 |
| Routing policy | Failover |
| Failover record type | Secondary |
| Health check ID | (left empty) |
| Record ID | FailoverSecondary |

**Secondary record configuration:**
- Same record name (`www`) but different IP address
- Points to secondary instance (standby)
- No health check needed (used when primary is unhealthy)
- Failover type "Secondary" indicates this is the standby destination

I clicked **Create records** to finalize the secondary record.

✅ **Task complete**: Failover routing configured with primary and secondary records.

**Route 53 failover behavior:**
- If primary health check is **Healthy**: DNS queries return primary instance IP
- If primary health check is **Unhealthy**: DNS queries return secondary instance IP

### Task 4: Verify DNS Resolution

#### Step 4.1: Copy DNS Record Name

I selected one of the A records and copied the **Record name** value (`www.XXXXXX_XXXXXXXXXX.vocareum.training`).

#### Step 4.2: Access via Failover Domain

I opened a new browser tab and navigated to the DNS name with `/cafe` appended:

```
http://www.XXXXXX_XXXXXXXXXX.vocareum.training/cafe/
```

The café website loaded, displaying:
- Café application interface
- **Server Information** showing primary instance (us-west-2a)

📝 **Note**: Route 53 resolved the domain name to the primary instance's IP address because the health check showed "Healthy" status.

✅ **Task complete**: DNS failover domain resolving to primary instance.

### Task 5: Verify Automatic Failover

#### Step 5.1: Simulate Primary Instance Failure

I navigated to **EC2** → **Instances** and selected **CafeInstance1**.

From the **Instance state** dropdown, I selected **Stop instance** and confirmed the stop.

The instance transitioned to **Stopped** state, simulating a failure of the primary web server.

⚠️ **Caution**: Stopping the instance stops the web application but keeps the infrastructure available for restart. In production, actual hardware failures would trigger similar failover behavior.

#### Step 5.2: Monitor Health Check Status

I navigated to **Route 53** → **Health checks** and selected **Primary-Website-Health**.

I viewed the **Monitoring** tab and observed:

**Timeline of health check status:**
- Initially: **Healthy** (primary instance responding)
- After stop: **Failed** status in monitoring graph
- Within 2-3 minutes: **Unhealthy** status (2 consecutive failures exceeded failure threshold)

💭 **Consider**: The health check requires 2 consecutive failures (at 10-second intervals) before transitioning to "Unhealthy" status. This prevents immediate failover on transient network issues.

I periodically refreshed until the status changed to **Unhealthy**.

#### Step 5.3: Verify Failover in Browser

I returned to the browser tab with the failover domain and refreshed the page.

**Results after refresh:**
- Website still loaded successfully
- **Server Information** now displayed **us-west-2b** (secondary instance)
- Instance ID changed to CafeInstance2

**Failover process that occurred:**
1. Health check detected primary instance not responding
2. Health check status changed to "Unhealthy"
3. Route 53 updated DNS records
4. Browser DNS cache expired (15s TTL)
5. New DNS query returned secondary instance IP
6. Browser connected to secondary instance
7. Application continued running without interruption

✅ **Task complete**: Application automatically failed over to secondary instance.

#### Step 5.4: Monitor SNS Notification

I checked my email and found an AWS Notifications message with subject:

```
ALARM: Primary-Website-Health-awsroute53-...
```

The email contained:
- Alarm name and timestamp
- Status change notification (Healthy → Unhealthy)
- Details about the health check failure
- Link to AWS console for further investigation

📝 **Note**: SNS emails may take 2-5 minutes to arrive after the health check fails.

✅ **Task complete**: Received SNS email notification of failover event.

## High Availability Architecture

The Route 53 failover configuration provides automatic high availability:

```
User Request to www.café.com
        ↓
    Route 53 (DNS)
        ↓
    Failover Logic
    ├─ If Primary Health Check = Healthy
    │  └─ Return Primary Instance IP (us-west-2a)
    │     └─ CafeInstance1 → Serve Request
    │
    └─ If Primary Health Check = Unhealthy
       └─ Return Secondary Instance IP (us-west-2b)
          └─ CafeInstance2 → Serve Request
            (SNS → Email Notification)
```

## Failover Behavior Summary

| Scenario | Route 53 Behavior | User Experience |
|----------|-------------------|-----------------|
| Primary healthy, secondary healthy | Route to primary | Fast, normal response from primary AZ |
| Primary unhealthy, secondary healthy | Route to secondary | Automatic failover, request served from secondary AZ |
| Primary healthy, secondary unhealthy | Route to primary | Normal operation, only primary available |
| Both unhealthy | No healthy target | Application unavailable (both instances down) |

## Conclusion

I successfully implemented automatic failover routing with Route 53:

- ✅ Verified café application deployment across two Availability Zones
- ✅ Confirmed both instances running identical application configuration
- ✅ Created Route 53 health check monitoring primary instance
- ✅ Configured fast health check (10-second intervals) with 2-failure threshold
- ✅ Set up SNS topic and email notifications for health check alarms
- ✅ Confirmed SNS email subscription
- ✅ Created primary failover DNS record with health check association
- ✅ Created secondary failover DNS record as standby destination
- ✅ Verified DNS resolution to primary instance
- ✅ Simulated primary instance failure by stopping the instance
- ✅ Confirmed health check transitioned to "Unhealthy" status
- ✅ Verified automatic DNS failover to secondary instance
- ✅ Confirmed application continued functioning via secondary instance
- ✅ Received SNS alarm notification of failover event
- ✅ Demonstrated automatic failover without manual intervention

This lab demonstrated critical patterns for building fault-tolerant applications. Route 53 failover routing enables automatic recovery from infrastructure failures, ensuring application availability even when entire Availability Zones or instances fail. Combined with multi-AZ deployment, this creates resilient systems that maintain service availability during failures.

## Additional Resources

- [Amazon Route 53 Documentation](https://docs.aws.amazon.com/route53/)
- [Route 53 Failover Routing Policy](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-failover.html)
- [Route 53 Health Checks](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/health-checks-types.html)
- [Creating and Managing Health Checks](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/health-checks.html)
- [DNS Basics and Concepts](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/welcome-dns-service.html)
- [Time-To-Live (TTL) Configuration](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy-simple.html)
- [Amazon SNS for Route 53 Notifications](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html)
- [High Availability Architecture Patterns](https://docs.aws.amazon.com/whitepapers/latest/aws-well-architected-framework/high-availability-and-business-continuity.html)
- [Multi-AZ Deployment Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
- [Monitoring Route 53 Health Checks](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/health-checks-monitor.html)
