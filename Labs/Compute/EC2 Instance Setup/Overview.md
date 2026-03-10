# Amazon EC2 Instance Setup

## Overview

I completed a comprehensive hands-on lab exploring the core capabilities of Amazon Elastic Compute Cloud (EC2), AWS's foundational compute service. Through this lab, I gained practical experience launching a fully functional web server, implementing security controls, monitoring instance health, and performing common operational tasks. I demonstrated proficiency with termination protection, instance resizing, and security group configuration—all critical skills for managing production EC2 infrastructure. By the end of this lab, I successfully launched, monitored, secured, resized, and terminated an EC2 instance, understanding the complete lifecycle of cloud-based virtual servers.

## Topics Covered

After completing this lab, I was able to:

- Launch EC2 instances with termination protection enabled
- Configure security groups to control inbound and outbound traffic
- Deploy web applications using EC2 user data scripts
- Monitor instance health using status checks and CloudWatch metrics
- Access instance screenshots for troubleshooting without SSH
- Modify security group rules to enable service accessibility
- Resize EC2 instances by changing instance types
- Resize Amazon EBS volumes to increase storage capacity
- Test and understand termination protection functionality
- Terminate EC2 instances and clean up resources
- Interpret instance states and status check results

## Introducing the Technologies

### Amazon EC2: Core Concepts

Amazon Elastic Compute Cloud (EC2) is AWS's primary compute service, providing resizable virtual servers called instances. EC2 abstracts away the complexity of managing physical servers while providing complete control over the computing environment. The key value proposition is pay-as-you-go pricing—I only pay for the compute capacity I actually use.

| EC2 Component | Purpose | Key Details |
|---------------|---------|------------|
| **Instance** | Virtual server in the cloud | Runs an operating system and applications |
| **AMI** (Amazon Machine Image) | Template for launching instances | Contains OS, applications, and configurations |
| **Instance Type** | Determines CPU, memory, and networking capacity | Examples: t3.micro (1 GiB RAM), t3.small (2 GiB RAM) |
| **EBS Volume** | Network-attached storage | Provides persistent block storage for instances |
| **Security Group** | Virtual firewall at instance level | Controls inbound and outbound traffic by port/protocol |
| **Public IP/DNS** | Internet-accessible address | Allows external access to the instance |

### Instance Types and Use Cases

EC2 offers multiple instance types optimized for different workloads. Instance types are identified by a family prefix (t, m, c, r, etc.) and size (micro, small, medium, large).

| Instance Type | vCPU | Memory | Best For | Cost Model |
|---------------|------|--------|----------|------------|
| t3.micro | 2 | 1 GiB | Testing, development, low-traffic applications | Lowest cost, free tier eligible |
| t3.small | 2 | 2 GiB | Small production workloads, web servers | Low cost, burstable performance |
| m5.medium | 1 | 4 GiB | General-purpose applications, balanced workloads | Moderate cost |
| c5.large | 2 | 4 GiB | Compute-intensive workloads, batch processing | Higher cost, optimized CPU |

💭 **Consider**: The t3 family uses "burstable" performance, meaning they accumulate CPU credits during idle time and consume them during high usage. This makes them cost-effective for workloads with variable demand.

### Monitoring and Health Checks

EC2 provides automated monitoring to detect instance issues. Two key status checks run continuously:

| Status Check | What It Tests | Indicates |
|--------------|---------------|-----------|
| **System Reachability** | EC2 host hardware and network connectivity | Whether the underlying infrastructure is healthy |
| **Instance Reachability** | Operating system and application responsiveness | Whether the instance OS can process requests |

Both checks must pass (2/2 checks passed) for the instance to be considered fully operational.

## Tasks

### Task 1: Launch EC2 Instance with Termination Protection

#### Step 1.1: Access EC2 Console and Begin Launch

I navigated to the AWS Management Console and accessed the EC2 service. I clicked **Launch instance** to begin configuring a new EC2 instance.

#### Step 1.2: Configure Instance Name and Tags

In the **Name and tags** pane, I entered:

| Setting | Value |
|---------|-------|
| Name | Web Server |

📝 **Note**: AWS automatically creates a key-value pair where the key is "Name" and the value is "Web Server". This name appears in the console and helps identify the instance.

#### Step 1.3: Select Amazon Machine Image (AMI)

Under **Application and OS Images**, I selected the default AMI:

| Setting | Value |
|---------|-------|
| AMI | Amazon Linux 2023 |

✅ **Task complete**: AMI selection confirms the instance will run Amazon Linux 2023, which includes the tools needed for a web server.

#### Step 1.4: Choose Instance Type

In the **Instance type** pane, I selected:

| Setting | Value |
|---------|-------|
| Instance Type | t3.micro |

💭 **Consider**: The t3.micro instance type provides 2 vCPU and 1 GiB of memory, making it suitable for testing and development. It's also eligible for the AWS free tier, making it cost-effective for learning.

#### Step 1.5: Configure Key Pair

In the **Key pair (login)** pane, I selected:

| Setting | Value |
|---------|-------|
| Key Pair | Proceed without a key pair (Not recommended) |

📝 **Note**: Since I deployed a web server that I would access via HTTP rather than SSH, I proceeded without a key pair. This improves security by preventing SSH access.

#### Step 1.6: Configure Network Settings

In the **Network settings** pane, I clicked **Edit** and configured:

| Setting | Value |
|---------|-------|
| VPC | Lab VPC |
| Security Group Name | Web Server security group |
| Security Group Description | Security group for my web server |

For inbound rules, I removed the default SSH rule by clicking **Remove**, as SSH access was not needed.

⚠️ **Caution**: Removing unnecessary inbound rules improves security by reducing the attack surface. Only open ports that the application requires.

#### Step 1.7: Configure Storage

In the **Configure storage** pane, I kept the default configuration:

| Setting | Value |
|---------|-------|
| Root Volume Size | 8 GiB |
| Root Volume Type | gp2 (General Purpose SSD) |

📝 **Note**: This root volume serves as the boot volume and stores the operating system. Additional volumes can be attached if needed for additional storage.

#### Step 1.8: Enable Termination Protection and Add User Data

I expanded the **Advanced details** pane and configured:

| Setting | Value |
|---------|-------|
| Termination Protection | Enable |

I then pasted the following user data script into the **User data** text box:

```bash
#!/bin/bash
yum -y install httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
```

**What this script does:**
- Installs Apache web server (httpd package)
- Enables httpd to start automatically on system boot
- Starts the httpd service immediately
- Creates a simple HTML homepage at `/var/www/html/index.html`

✅ **Task complete**: Termination protection enabled and user data script configured.

💭 **Consider**: User data scripts run once during instance launch with root privileges, making them ideal for initial configuration. This is more efficient than manually configuring each instance after launch.

#### Step 1.9: Launch the Instance

I clicked **Launch instance** to create the EC2 instance. The instance appeared in **Pending** state initially, then transitioned to **Running** state within seconds.

I waited for the status checks to display **2/2 checks passed**, confirming the instance and underlying infrastructure were healthy.

✅ **Task complete**: EC2 instance successfully launched and running.

### Task 2: Monitor Instance Health and Performance

#### Step 2.1: Review Status Checks

I selected the Web Server instance and navigated to the **Status checks** tab. I confirmed both status checks had passed:

| Status Check | Result |
|--------------|--------|
| System Reachability Check | ✅ Passed |
| Instance Reachability Check | ✅ Passed |

📝 **Note**: These automated checks run every minute. If either check fails, it indicates a hardware issue (System reachability) or an OS/application issue (Instance reachability).

#### Step 2.2: Review CloudWatch Metrics

I selected the **Monitoring** tab to view CloudWatch metrics for the instance. CloudWatch displayed graphs for:

- **CPU Utilization**: Shows how much of the available CPU capacity is being used
- **Network In/Out**: Shows data transferred in and out of the instance
- **EBS Read/Write**: Shows disk I/O operations

💭 **Consider**: Basic monitoring (5-minute granularity) is enabled by default. Detailed monitoring (1-minute granularity) can be enabled for more precise metrics, though it incurs additional charges.

#### Step 2.3: Capture Instance Screenshot

In the **Actions** menu, I selected **Monitor and troubleshoot** → **Get Instance Screenshot**. The screenshot displayed the instance's console output, showing the Linux boot sequence and confirming the httpd web server was running.

📝 **Note**: Instance screenshots are invaluable for troubleshooting without SSH access. They provide visibility into what the instance's console would show if a monitor were attached to the server.

✅ **Task complete**: Successfully reviewed multiple monitoring methods.

### Task 3: Update Security Group and Access Web Server

#### Step 3.1: Retrieve Instance Public IP Address

I selected the Web Server instance and navigated to the **Details** tab. I copied the **Public IPv4 address** (for example, `54.123.45.67`).

#### Step 3.2: Attempt to Access Web Server (Expected to Fail)

I opened a new browser tab, pasted the public IP address in the address bar, and pressed Enter. The request timed out because the security group did not allow inbound HTTP traffic on port 80.

💭 **Consider**: This demonstrates security groups functioning as a firewall. Even though the web server was running and operational, the network security controls prevented external access. This is the expected and desired behavior—applications must explicitly allow traffic, rather than denying it by default.

#### Step 3.3: Modify Security Group Inbound Rules

I navigated to the **Security Groups** section under **Network & Security**. I selected the **Web Server security group** and clicked the **Inbound rules** tab.

I clicked **Edit inbound rules**, then **Add rule** and configured:

| Setting | Value |
|---------|-------|
| Type | HTTP |
| Protocol | TCP |
| Port Range | 80 |
| Source | 0.0.0.0/0 (Anywhere-IPv4) |

I clicked **Save rules** to apply the new rule.

📝 **Note**: Setting the source to `0.0.0.0/0` allows HTTP traffic from any IP address. For production systems, you would restrict this to specific IP ranges to reduce the attack surface.

#### Step 3.4: Access Web Server Successfully

I returned to the browser tab with the instance's public IP address and refreshed the page. The page now displayed:

```
Hello From Your Web Server!
```

✅ **Task complete**: Successfully modified security group to allow HTTP traffic and confirmed the web server was accessible.

### Task 4: Resize EC2 Instance

#### Step 4.1: Stop the Instance

I selected the Web Server instance and clicked **Instance state** → **Stop instance**. The instance performed a normal shutdown and transitioned to **Stopped** state.

📝 **Note**: Stopping an instance differs from terminating it. A stopped instance can be restarted later, and you retain the instance ID and configuration. However, you are not charged for stopped instances (only for attached EBS volumes).

⚠️ **Caution**: Stopping an instance causes a brief service interruption. For production applications, plan stopping during maintenance windows.

#### Step 4.2: Change Instance Type

While the instance was stopped, I opened the **Actions** menu and selected **Instance Settings** → **Change Instance Type**. I changed:

| Setting | Original | New |
|---------|----------|-----|
| Instance Type | t3.micro | t3.small |

The t3.small instance type doubles the memory from 1 GiB to 2 GiB while maintaining 2 vCPU.

✅ **Task complete**: Instance type change queued; will apply when instance restarts.

💭 **Consider**: You can only change instance type while the instance is stopped. The instance must be compatible with the target instance type (same family or family with compatible capabilities). Some instance types cannot be mixed.

#### Step 4.3: Resize EBS Volume

I navigated to **Volumes** under **Elastic Block Store**. I selected the root volume attached to the Web Server instance and clicked **Actions** → **Modify Volume**.

I changed the volume size from 8 GiB to 10 GiB and clicked **Modify** twice to confirm.

📝 **Note**: EBS volumes can be resized while the instance is running or stopped. However, after resizing, you may need to extend the filesystem within the OS to utilize the new space. For this lab, the volume modification was sufficient.

✅ **Task complete**: EBS volume size increased from 8 GiB to 10 GiB.

#### Step 4.4: Start the Resized Instance

I selected the Web Server instance and clicked **Instance state** → **Start instance**. The instance restarted with the new configuration (t3.small instance type and 10 GiB EBS volume).

I waited for the instance to return to **Running** state with **2/2 checks passed**.

✅ **Task complete**: Instance successfully resized and restarted with new configuration.

### Task 5: Test and Verify Termination Protection

#### Step 5.1: Attempt to Terminate Instance (Expected to Fail)

I selected the Web Server instance and clicked **Instance state** → **Terminate (delete) instance**. A confirmation dialog appeared, but clicking **Terminate** failed with an error message:

```
Failed to terminate an instance: The instance may not be terminated.
```

This failure occurred because termination protection was enabled.

💭 **Consider**: Termination protection is a safety mechanism that prevents accidental deletion of critical instances. It must be explicitly disabled before the instance can be terminated.

#### Step 5.2: Disable Termination Protection

I opened the **Actions** menu and selected **Instance Settings** → **Change termination protection**. I unchecked the **Enable** checkbox and clicked **Save**.

#### Step 5.3: Terminate Instance

With termination protection disabled, I clicked **Instance state** → **Terminate instance** and confirmed the termination. The instance transitioned to **Terminated** state.

⚠️ **Caution**: Termination is irreversible. Once terminated, the instance cannot be restarted. All data stored on EBS volumes marked for deletion is lost.

✅ **Task complete**: Successfully demonstrated termination protection and terminated the instance.

## Conclusion

I successfully completed a comprehensive EC2 lifecycle management lab, demonstrating proficiency with:

- ✅ Launched an EC2 instance with termination protection and user data script
- ✅ Configured security groups to allow/deny inbound traffic
- ✅ Deployed a web server via user data automation
- ✅ Verified instance health using status checks (2/2 passed)
- ✅ Monitored instance performance using CloudWatch metrics
- ✅ Captured instance screenshots for console-level troubleshooting
- ✅ Modified security group inbound rules to enable HTTP access
- ✅ Accessed the running web server from a browser
- ✅ Resized an instance type from t3.micro (1 GiB) to t3.small (2 GiB)
- ✅ Increased EBS volume size from 8 GiB to 10 GiB
- ✅ Tested and understood termination protection functionality
- ✅ Terminated the instance and cleaned up resources

This lab demonstrated the core capabilities of EC2 and the typical workflows for managing cloud-based virtual servers throughout their lifecycle—from launch through scaling and termination.

## Additional Resources

- [Amazon EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [EC2 Instance Types Documentation](https://aws.amazon.com/ec2/instance-types/)
- [Amazon Machine Images (AMI) Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [EC2 User Data and Shell Scripts](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html)
- [Security Groups User Guide](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [EC2 Key Pairs Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
- [Status Checks for Your Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring-system-instance-checks-ec2.html)
- [CloudWatch Metrics for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/monitoring_ec2.html)
- [Resizing Your Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-resize.html)
- [Termination Protection for Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html#Using_termination_protection)
- [Amazon EBS Volumes User Guide](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes.html)
