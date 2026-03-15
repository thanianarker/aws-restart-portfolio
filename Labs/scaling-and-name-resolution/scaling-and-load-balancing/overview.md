# Scaling and Load Balancing 

## Overview

I completed a comprehensive hands-on lab on building scalable, highly available infrastructure using AWS Elastic Load Balancing (ELB) and EC2 Auto Scaling. Through this lab, I gained practical experience creating machine images for repeatability, configuring application load balancers to distribute traffic, implementing launch templates for infrastructure-as-code, and establishing auto-scaling policies based on performance metrics. I transformed a single-instance architecture into a multi-tier, highly available system spanning multiple Availability Zones. By the end of this lab, I demonstrated proficiency with the core services for building resilient, scalable cloud architectures capable of handling variable demand while maintaining optimal performance.

## Topics Covered

After completing this lab, I was able to:

- Create Amazon Machine Images (AMIs) from running EC2 instances
- Understand AMI contents and use cases for standardization
- Create and configure Application Load Balancers
- Configure load balancer target groups for traffic routing
- Distribute traffic across multiple Availability Zones
- Create and configure launch templates for Auto Scaling
- Specify AMI, instance type, security groups in launch templates
- Create Auto Scaling groups with capacity policies
- Configure desired, minimum, and maximum capacity
- Implement target tracking scaling policies based on CPU utilization
- Monitor Auto Scaling group instance launches
- Verify load balancer health checks and traffic distribution
- Access applications through load balancer DNS names
- Understand the relationship between ELB and Auto Scaling
- Build multi-tier, highly available architecture
- Implement automatic failover across Availability Zones

## Introducing the Technologies

### Elastic Load Balancing (ELB)

Elastic Load Balancing automatically distributes incoming application traffic across multiple targets and Availability Zones, providing fault tolerance and scalability.

| ELB Feature | Benefit | Use Case |
|------------|---------|----------|
| **Traffic Distribution** | Distributes requests across multiple instances | Prevents any single instance from being overwhelmed |
| **Health Checks** | Monitors instance health | Stops sending traffic to unhealthy instances |
| **Multi-AZ** | Spans multiple Availability Zones | Survives single AZ failure |
| **Auto Scaling Integration** | Works seamlessly with Auto Scaling | Automatically registers/deregisters instances |
| **DNS Name** | Single entry point for applications | Abstracts underlying instance infrastructure |

### Application Load Balancer (ALB) Types

| Load Balancer Type | Layer | Best For |
|-------------------|-------|----------|
| **Application Load Balancer (ALB)** | Layer 7 (Application) | HTTP/HTTPS, microservices, API traffic |
| **Network Load Balancer (NLB)** | Layer 4 (Transport) | Extreme performance, gaming, IoT |
| **Classic Load Balancer (CLB)** | Layer 4/7 (Hybrid) | Legacy applications, backward compatibility |

### EC2 Auto Scaling

Auto Scaling automatically adjusts the number of EC2 instances to maintain desired capacity based on scaling policies.

| Auto Scaling Component | Purpose | Example |
|----------------------|---------|---------|
| **Launch Template** | Blueprint for instance configuration | Specifies AMI, instance type, security group |
| **Desired Capacity** | Target number of instances | 2 instances |
| **Minimum Capacity** | Lowest instance count allowed | 2 instances |
| **Maximum Capacity** | Highest instance count allowed | 4 instances |
| **Scaling Policy** | Trigger for adding/removing instances | Scale at 50% average CPU |

### Scaling Policy Types

| Policy Type | Trigger | Use Case |
|------------|---------|----------|
| **Target Tracking** | Maintain specific metric target | CPU at 50%, response time at 1s |
| **Step Scaling** | Respond to metric in bands | Add 2 instances if CPU >70%, remove if <30% |
| **Simple Scaling** | Single threshold-based action | Add 1 instance when CPU >80% |
| **Scheduled** | Time-based scaling | Scale up at 8 AM, down at 6 PM |

## Tasks

### Task 1: Create Amazon Machine Image (AMI)

#### Step 1.1: Identify Web Server Instance

I navigated to the AWS Management Console and accessed **EC2** → **Instances**.

I located the **Web Server 1** instance in **Running** state.

📝 **Note**: This instance is a pre-configured web server that serves as the template for all instances launched by Auto Scaling. Creating an AMI captures its entire configuration.

#### Step 1.2: Create AMI from Instance

I selected the **Web Server 1** instance.

From the **Actions** dropdown, I selected **Image and templates** → **Create image**.

I configured the AMI:

| Setting | Value |
|---------|-------|
| Image name | Web Server AMI |
| Image description | Lab AMI for Web Server |

I clicked **Create image** to initiate AMI creation.

#### Step 1.3: Verify AMI Creation

The console displayed an AMI ID (e.g., `ami-0a1b2c3d4e5f6g7h8`), confirming successful AMI creation.

💭 **Consider**: An AMI is a blueprint containing the OS, applications, configuration, and data from the source instance. Creating an AMI from a configured server allows launching identical instances without manual configuration.

✅ **Task complete**: Web Server AMI successfully created and ready for use in Auto Scaling group.

### Task 2: Create Application Load Balancer

#### Step 2.1: Access Load Balancer Creation

I navigated to **EC2** → **Load Balancing** → **Load Balancers**.

I clicked **Create load balancer** and selected **Application Load Balancer** → **Create**.

#### Step 2.2: Configure Basic Load Balancer Settings

On the create page, I configured:

| Setting | Value |
|---------|-------|
| Load balancer name | LabELB |
| Scheme | Internet-facing (default) |
| IP address type | IPv4 (default) |

**Load balancer name purpose:** Identifies the load balancer in the console and logs.

#### Step 2.3: Configure Network Mapping

In the **Network mapping** section, I configured:

| Setting | Value |
|---------|-------|
| VPC | Lab VPC |
| Availability Zone 1 | Public Subnet 1 |
| Availability Zone 2 | Public Subnet 2 |

**Multi-AZ configuration:** Placing the load balancer in public subnets across two AZs ensures fault tolerance—if one AZ becomes unavailable, the load balancer continues operating from the other AZ.

#### Step 2.4: Configure Security Group

I removed the default security group and selected **Web Security Group**, which permits HTTP (port 80) traffic.

📝 **Note**: Security groups control which traffic is allowed to reach the load balancer. The Web Security Group was pre-created with HTTP rules.

#### Step 2.5: Create Target Group

In the **Listeners and routing** section, I clicked **Create target group** link.

A new tab opened with target group configuration. I configured:

| Setting | Value |
|---------|-------|
| Target type | Instances |
| Target group name | lab-target-group |

I clicked **Next** and then **Create target group** without registering targets manually (Auto Scaling will register instances automatically).

#### Step 2.6: Complete Load Balancer Creation

I returned to the load balancer tab and clicked **Refresh** next to the Forward to dropdown.

I selected **lab-target-group** as the target group to receive traffic.

I clicked **Create load balancer** to finalize creation.

#### Step 2.7: Copy Load Balancer DNS Name

After creation, I clicked **View load balancer** and copied the **DNS name** (e.g., `LabELB-123456789.us-west-2.elb.amazonaws.com`).

I saved the DNS name for later use when testing the application.

✅ **Task complete**: Application Load Balancer created, configured across multiple AZs, and ready to distribute traffic.

### Task 3: Create Launch Template

#### Step 3.1: Access Launch Template Creation

I navigated to **EC2** → **Instances** → **Launch Templates**.

I clicked **Create launch template** to define the instance configuration for Auto Scaling.

#### Step 3.2: Configure Launch Template Metadata

I configured:

| Setting | Value |
|---------|-------|
| Launch template name | lab-app-launch-template |
| Template version description | A web server for the load test app |
| Auto Scaling guidance | Checked (provides guidance for ASG setup) |

#### Step 3.3: Select AMI

In the **Application and OS Images** section, I selected the **My AMIs** tab.

**Web Server AMI** (created in Task 1) was already selected, confirming that new instances will be launched with identical configuration to the original server.

#### Step 3.4: Configure Instance Type

In the **Instance type** section, I selected **t3.micro** from the dropdown.

💭 **Consider**: t3.micro provides 2 vCPU and 1 GiB memory—sufficient for the test application. For production, instance type would be chosen based on application requirements.

#### Step 3.5: Configure Key Pair and Security

In the **Key pair (login)** section, I confirmed the dropdown was set to **Don't include in launch template**.

📝 **Note**: Since instances are accessed through the load balancer (not SSH), key pairs are unnecessary. This improves security by preventing SSH access.

In the **Network settings** section, I selected **Web Security Group** from the security groups dropdown.

#### Step 3.6: Create Launch Template

I clicked **Create launch template** to finalize the template.

The console confirmed successful creation with message: "Successfully created lab-app-launch-template".

✅ **Task complete**: Launch template created with all configuration needed by Auto Scaling group.

### Task 4: Create Auto Scaling Group

#### Step 4.1: Create ASG from Launch Template

I clicked **View launch templates** and selected **lab-app-launch-template**.

From the **Actions** dropdown, I selected **Create Auto Scaling group**.

#### Step 4.2: Configure Auto Scaling Group Name

On the choose launch template page, I entered:

| Setting | Value |
|---------|-------|
| Auto Scaling group name | Lab Auto Scaling Group |

I clicked **Next** to proceed to instance launch options.

#### Step 4.3: Configure Network and Subnets

On the instance launch options page, I configured:

| Setting | Value |
|---------|-------|
| VPC | Lab VPC |
| Availability Zones and subnets | Private Subnet 1 (10.0.1.0/24) and Private Subnet 2 (10.0.3.0/24) |

**Private subnet placement:** Instances are placed in private subnets (accessible only through the load balancer), not directly from the internet. This improves security.

**Multi-AZ placement:** Instances span two Availability Zones, providing fault tolerance—if one AZ fails, instances in the other AZ continue running.

I clicked **Next** to configure advanced options.

#### Step 4.4: Configure Load Balancer Integration

On the advanced options page, I configured:

| Setting | Value |
|---------|-------|
| Load balancing | Attach to an existing load balancer |
| Target groups | lab-target-group \| HTTP |
| Health check type | ELB |

**Health check type:** ELB (Elastic Load Balancer) health checks verify instances pass the load balancer's health check before receiving traffic. If an instance fails health checks, Auto Scaling removes it and launches a replacement.

I clicked **Next** to configure scaling policies.

#### Step 4.5: Configure Capacity and Scaling Policies

On the group size page, I configured:

| Setting | Value |
|---------|-------|
| Desired capacity | 2 |
| Minimum capacity | 2 |
| Maximum capacity | 4 |

**Capacity configuration:**
- **Desired: 2** - Auto Scaling launches 2 instances immediately
- **Minimum: 2** - Never scale below 2 instances (maintains availability)
- **Maximum: 4** - Never scale beyond 4 instances (controls costs)

In the **Scaling policies** section, I configured:

| Setting | Value |
|---------|-------|
| Policy type | Target tracking scaling policy |
| Metric type | Average CPU utilization |
| Target value | 50 |

**Scaling policy explanation:**
- Auto Scaling monitors average CPU across all instances
- If CPU rises above 50%, new instances are added
- If CPU drops below 50%, instances are removed
- Target tracking automatically adjusts instance count to maintain the 50% target

💭 **Consider**: A 50% CPU target provides good balance—high enough to utilize instances efficiently, but with headroom before performance degrades.

I clicked **Next** to proceed.

#### Step 4.6: Configure Notifications and Tags

On the notifications page, I clicked **Next** (no notifications configured).

On the tags page, I added:

| Key | Value |
|-----|-------|
| Name | Lab Instance |

This tag appears on all instances launched by Auto Scaling, making them easy to identify.

I clicked **Next** and then **Create Auto Scaling group** to finalize configuration.

✅ **Task complete**: Auto Scaling group created and launching instances.

**Initial state:** The Auto Scaling group shows 0 instances initially, but quickly launches 2 instances to reach the desired capacity.

### Task 5: Verify Load Balancing and Auto Scaling

#### Step 5.1: Verify Instance Launch

I navigated to **EC2** → **Instances** and observed two new **Lab Instance** instances launching.

💭 **Consider**: It may take 30-60 seconds for instances to appear and boot. If instances aren't visible, the Auto Scaling group is still launching them.

#### Step 5.2: Verify Load Balancer Health Checks

I navigated to **EC2** → **Load Balancing** → **Target Groups**.

I selected **lab-target-group** to view registered targets.

I observed two **Lab Instance** targets listed in the **Registered targets** section.

#### Step 5.3: Wait for Healthy Status

I refreshed the page periodically and waited for both instances to display **Health status: healthy**.

A "healthy" status indicates:
- Instance has booted completely
- Instance has passed the load balancer's health check
- Load balancer will send traffic to this instance

⚠️ **Caution**: Instances may show "Initial" status temporarily while booting. This is normal and transitions to "Healthy" once the health check passes.

✅ **Task complete**: Both instances healthy and ready to receive traffic.

#### Step 5.4: Test Load Balancer with Application

I opened a new browser tab and navigated to the load balancer's DNS name (e.g., `http://LabELB-123456789.us-west-2.elb.amazonaws.com/`).

The **Load Test application** appeared in the browser.

**Request flow:**
1. Browser sends request to load balancer DNS name
2. Load balancer receives request
3. Load balancer selects one of the two healthy instances
4. Instance processes request and returns response
5. Load balancer returns response to browser

✅ **Task complete**: Load balancer successfully distributing traffic to Auto Scaled instances.

## Architecture Transformation

I transformed the infrastructure from a single-instance to a highly available, scalable architecture:

**Starting Architecture:**
```
Web Server 1 (single instance)
└── Public Subnet (AZ: us-west-2a)
    └── Single point of failure
```

**Final Architecture:**
```
                    Load Balancer (LabELB)
                    └── Public Subnets (AZ: us-west-2a, us-west-2b)

Auto Scaling Group (Lab Auto Scaling Group)
├── Lab Instance 1
│   └── Private Subnet 1 (AZ: us-west-2a)
├── Lab Instance 2
│   └── Private Subnet 2 (AZ: us-west-2b)
└── (Can scale to 4 instances based on CPU)

Scaling Policy:
└── Target Tracking (Average CPU 50%)
    ├── Scale up when CPU > 50%
    └── Scale down when CPU < 50%
```

## Scalability and Fault Tolerance

The final architecture provides:

| Capability | Benefit | How It Works |
|-----------|---------|------------|
| **Load Distribution** | Prevents bottleneck | ELB spreads requests across instances |
| **Automatic Scaling** | Handles traffic spikes | ASG adds instances when CPU rises |
| **Cost Optimization** | Reduces idle capacity | ASG removes instances when demand drops |
| **Fault Tolerance** | Survives instance failure | ELB removes failed instances, ASG launches replacement |
| **Multi-AZ** | Survives AZ failure | Instances in 2 AZs, can operate with 1 AZ down |

## Conclusion

I successfully demonstrated comprehensive load balancing and auto-scaling architecture design:

- ✅ Created an AMI from a configured EC2 instance for standardization
- ✅ Created an Application Load Balancer across multiple Availability Zones
- ✅ Configured target groups for traffic routing
- ✅ Selected appropriate security groups for load balancer access
- ✅ Created a launch template with AMI, instance type, and security group
- ✅ Created an Auto Scaling group with specific capacity constraints
- ✅ Configured instances to launch in private subnets across multiple AZs
- ✅ Integrated Auto Scaling group with load balancer
- ✅ Configured ELB health checks for automatic failure detection
- ✅ Implemented target tracking scaling policy based on CPU utilization
- ✅ Set desired (2), minimum (2), and maximum (4) capacity
- ✅ Tagged all Auto Scaled instances for identification
- ✅ Verified instances launched and passed health checks
- ✅ Accessed application through load balancer DNS name
- ✅ Demonstrated load distribution across multiple instances
- ✅ Created highly available, fault-tolerant architecture

This lab demonstrated essential patterns for building production-grade cloud applications. ELB and Auto Scaling work together to create self-healing, auto-scaling infrastructure that maintains performance during traffic spikes and reduces costs during low-demand periods.

## Additional Resources

- [Elastic Load Balancing Documentation](https://docs.aws.amazon.com/elasticloadbalancing/)
- [Application Load Balancer User Guide](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/)
- [EC2 Auto Scaling User Guide](https://docs.aws.amazon.com/autoscaling/ec2/userguide/)
- [Launch Templates Documentation](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html)
- [Target Groups for Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)
- [Auto Scaling Scaling Policies](https://docs.aws.amazon.com/autoscaling/ec2/userguide/scaling_plan_overview.html)
- [CloudWatch Metrics for Auto Scaling](https://docs.aws.amazon.com/autoscaling/ec2/userguide/as-instance-monitoring.html)
- [Creating Amazon Machine Images (AMIs)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-an-ami-ebs.html)
- [Health Checks for Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)
- [Multi-AZ Deployments Best Practices](https://docs.aws.amazon.com/whitepapers/latest/aws-well-architected-framework/high-availability-and-business-continuity.html)
