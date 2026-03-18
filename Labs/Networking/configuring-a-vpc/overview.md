# Configuring a VPC

## Overview

I completed a comprehensive hands-on lab on building and configuring Amazon Virtual Private Cloud (VPC) infrastructure from the ground up. Through this lab, I gained practical experience creating VPCs with custom CIDR blocks, designing multi-subnet architectures with public and private tiers, implementing internet connectivity through gateways, and establishing bastion host patterns for secure access to private resources. I demonstrated the ability to design secure network topologies, configure routing policies, and implement defense-in-depth security patterns. By the end of this lab, I showcased proficiency with VPC fundamentals essential for architecting enterprise-grade cloud infrastructure.

## Topics Covered

After completing this lab, I was able to:

- Create and configure Virtual Private Clouds (VPCs)
- Design CIDR block allocation for scalable networks
- Enable DNS hostnames within VPCs
- Create public and private subnets with appropriate CIDR ranges
- Configure automatic public IP assignment for public subnets
- Create and attach Internet Gateways for internet connectivity
- Create and configure route tables with static routes
- Route internet-bound traffic (0.0.0.0/0) through gateways
- Associate subnets with route tables
- Differentiate between public and private subnets based on routing
- Create NAT Gateways for private subnet outbound internet access
- Allocate Elastic IPs for NAT Gateways
- Configure private route tables for NAT Gateway routing
- Implement bastion server patterns for secure private access
- Configure security groups for SSH access
- Launch EC2 instances in specific subnets with network settings
- Understand security group rules and inbound/outbound filtering
- Implement defense-in-depth network security architecture

## Introducing the Technologies

### Amazon VPC Components

Amazon VPC provides isolated network environments with complete control over IP addressing, routing, and security.

| VPC Component | Purpose | Configuration |
|---------------|---------|---------------|
| **VPC** | Isolated network container | CIDR block (e.g., 10.0.0.0/16) |
| **Subnet** | Subdivided IP range | Public or private, in specific AZ |
| **Internet Gateway** | Connection to public internet | Attached to VPC, routes 0.0.0.0/0 traffic |
| **Route Table** | Routing rules for traffic | Local routes + custom routes (IGW, NAT, etc.) |
| **NAT Gateway** | Outbound internet for private subnets | Placed in public subnet, uses Elastic IP |
| **Bastion Host** | Secure jump box to private resources | EC2 instance in public subnet |

### CIDR Block Design

CIDR notation specifies IP ranges using network/prefix format. The prefix indicates how many bits are network bits.

| CIDR Block | IP Range | Host Count | Use Case |
|-----------|----------|-----------|----------|
| 10.0.0.0/16 | 10.0.0.0 - 10.0.255.255 | 65,536 | Entire VPC network |
| 10.0.0.0/24 | 10.0.0.0 - 10.0.0.255 | 256 | Public subnet (web servers) |
| 10.0.2.0/23 | 10.0.2.0 - 10.0.3.255 | 512 | Private subnet (databases, app servers) |

### Public vs Private Subnets

| Characteristic | Public Subnet | Private Subnet |
|----------------|---------------|----------------|
| **Internet Access** | Yes (via IGW) | Yes (via NAT) for outbound only |
| **Public IP** | Auto-assigned | Elastic IP only (via NAT) |
| **Route Table** | Routes 0.0.0.0/0 → IGW | Routes 0.0.0.0/0 → NAT |
| **Typical Resources** | Web servers, bastion hosts | Databases, app servers |
| **Inbound from Internet** | Allowed via security groups | Blocked by default |

### Security Layers

```
Internet
    ↓
Internet Gateway
    ↓
Security Group (instance level)
    ↓
Network ACL (subnet level)
    ↓
EC2 Instance
```

## Tasks

### Task 1: Create VPC

#### Step 1.1: Create VPC with Custom CIDR

I navigated to **VPC** → **Your VPCs** and clicked **Create VPC**.

I configured the VPC:

| Setting | Value |
|---------|-------|
| Resources to create | VPC only |
| Name tag | Lab VPC |
| IPv4 CIDR block | 10.0.0.0/16 |
| IPv6 CIDR block | No IPv6 CIDR block |
| Tenancy | Default |

**CIDR block rationale:** 10.0.0.0/16 provides 65,536 IP addresses, supporting a scalable VPC with multiple subnets.

I clicked **Create VPC** to provision the VPC.

✅ **Task complete**: VPC created with confirmation message showing VPC ID.

#### Step 1.2: Enable DNS Hostnames

I selected the Lab VPC and clicked **Actions** → **Edit VPC settings**.

I selected **Enable DNS hostnames** in the DNS settings section.

I clicked **Save** to apply the setting.

📝 **Note**: Enabling DNS hostnames ensures EC2 instances launched in this VPC receive public DNS names, enabling easier access and management.

✅ **Task complete**: DNS hostnames enabled for the VPC.

### Task 2: Create Public and Private Subnets

#### Step 2.1: Create Public Subnet

I navigated to **VPC** → **Subnets** and clicked **Create subnet**.

I configured the public subnet:

| Setting | Value |
|---------|-------|
| VPC ID | Lab VPC |
| Subnet name | Public Subnet |
| Availability Zone | First AZ (specific, not "No preference") |
| IPv4 CIDR block | 10.0.0.0/24 |

**Subnet CIDR rationale:** 10.0.0.0/24 provides 256 addresses, sufficient for web servers and bastion hosts.

I clicked **Create subnet** to create the subnet.

#### Step 2.2: Enable Auto-Assign Public IP

I selected **Public Subnet** and clicked **Actions** → **Edit subnet settings**.

I selected **Enable auto-assign public IPv4 address**.

I clicked **Save** to enable automatic public IP assignment.

💭 **Consider**: Auto-assigning public IPs simplifies instance management—instances automatically receive internet-accessible addresses without manual EIP allocation.

✅ **Task complete**: Public subnet created with auto-assigned public IPs enabled.

#### Step 2.3: Create Private Subnet

I clicked **Create subnet** again and configured the private subnet:

| Setting | Value |
|---------|-------|
| VPC ID | Lab VPC |
| Subnet name | Private Subnet |
| Availability Zone | First AZ (specific) |
| IPv4 CIDR block | 10.0.2.0/23 |

**Subnet CIDR rationale:** 10.0.2.0/23 provides 512 addresses (double the public subnet), accommodating more internal resources like databases and application servers that don't require internet connectivity.

I clicked **Create subnet** to create the private subnet.

📝 **Note**: The /23 CIDR block encompasses both 10.0.2.x and 10.0.3.x ranges, providing growth capacity for database and application resources.

✅ **Task complete**: Private subnet created without auto-assigned public IPs.

### Task 3: Create Internet Gateway

#### Step 3.1: Create IGW

I navigated to **VPC** → **Internet gateways** and clicked **Create internet gateway**.

I entered:

| Setting | Value |
|---------|-------|
| Name tag | Lab IGW |

I clicked **Create internet gateway** to provision the gateway.

#### Step 3.2: Attach IGW to VPC

I selected **Lab IGW** and clicked **Actions** → **Attach to a VPC**.

From the dropdown, I selected **Lab VPC**.

I clicked **Attach internet gateway** to attach the gateway.

✅ **Task complete**: Internet Gateway created and attached to VPC.

**IGW purpose:** Enables resources in the VPC to communicate with the internet. Routes traffic destined for external IP addresses to/from the internet.

### Task 4: Configure Route Tables

#### Step 4.1: Rename Default Route Table

I navigated to **VPC** → **Route tables**.

I selected the route table with **Lab VPC** in the VPC column.

I clicked the edit icon in the Name column and renamed it to **Private Route Table**.

I clicked **Save** to apply the name change.

📝 **Note**: The default route table created with the VPC serves as the private route table. This name clarifies its purpose.

#### Step 4.2: Create Public Route Table

I clicked **Create route table** and configured:

| Setting | Value |
|---------|-------|
| Name | Public Route Table |
| VPC | Lab VPC |

I clicked **Create route table** to create the new route table.

#### Step 4.3: Add Internet Route to Public Route Table

I selected **Public Route Table** and clicked the **Routes** tab.

I clicked **Edit routes** and then **Add route**.

I configured the route:

| Setting | Value |
|---------|-------|
| Destination | 0.0.0.0/0 |
| Target | Internet Gateway → Lab IGW |

**Route explanation:**
- **Destination 0.0.0.0/0**: All IP addresses not matching more specific routes
- **Target Lab IGW**: Route all internet-bound traffic through the Internet Gateway

I clicked **Save changes** to add the route.

✅ **Task complete**: Public route table configured with internet-bound routing.

#### Step 4.4: Associate Public Route Table with Public Subnet

I selected **Public Route Table** and clicked the **Subnet associations** tab.

I clicked **Edit subnet associations** and selected **Public Subnet**.

I clicked **Save associations** to associate the route table with the subnet.

✅ **Task complete**: Public subnet now "public" because it has a route to the internet.

**Subnet publicity definition:** A subnet is "public" if its route table contains a route to an Internet Gateway. The Public Subnet now routes internet-bound traffic to the IGW, making it publicly routable.

### Task 5: Launch Bastion Server

#### Step 5.1: Configure Bastion Instance

I navigated to **EC2** → **Instances** and clicked **Launch instances**.

I configured the instance:

| Setting | Value |
|---------|-------|
| Name | Bastion Server |
| AMI | Amazon Linux 2023 |
| Instance type | t3.micro |
| Key pair | Proceed without a key pair |

#### Step 5.2: Configure Network Settings

In the **Network settings** section, I clicked **Edit** and configured:

| Setting | Value |
|---------|-------|
| VPC | Lab VPC |
| Subnet | Public Subnet |
| Auto-assign public IP | Enable |

#### Step 5.3: Create and Configure Security Group

In the **Firewall (security groups)** section, I selected **Create security group** and configured:

| Setting | Value |
|---------|-------|
| Security group name | Bastion Security Group |
| Description | Allow SSH |

**Inbound rule:**
- Type: SSH
- Source type: Anywhere (0.0.0.0/0)

📝 **Note**: Allowing SSH from "Anywhere" is acceptable for a lab bastion host. In production, restrict SSH to specific IP ranges.

I clicked **Launch instance** to create the bastion server.

✅ **Task complete**: Bastion server launched in public subnet with public IP and SSH access.

**Bastion server purpose:** Provides secure jumpbox access to resources in private subnets. Users SSH to the bastion, then SSH to private instances from there.

### Task 6: Create NAT Gateway

#### Step 6.1: Create NAT Gateway

I searched for **NAT gateways** and navigated to the NAT Gateways service.

I clicked **Create NAT gateway** and configured:

| Setting | Value |
|---------|-------|
| Name | Lab NAT gateway |
| Subnet | Public Subnet |
| Elastic IP | Allocate (click "Allocate Elastic IP") |

**NAT Gateway placement:** Located in the public subnet, uses an Elastic IP as its address for outbound traffic.

I clicked **Create a NAT gateway** to provision the gateway.

⚠️ **Caution**: NAT Gateways incur hourly charges plus data processing fees. They're necessary for private instances to access the internet, but understand the cost implications.

✅ **Task complete**: NAT Gateway created in public subnet.

#### Step 6.2: Configure Private Route Table for NAT

I navigated to **VPC** → **Route tables** and selected **Private Route Table**.

I clicked the **Routes** tab and clicked **Edit routes**.

I clicked **Add route** and configured:

| Setting | Value |
|---------|-------|
| Destination | 0.0.0.0/0 |
| Target | NAT Gateway → Lab NAT gateway |

I clicked **Save changes** to add the route.

✅ **Task complete**: Private route table configured for outbound internet via NAT.

**NAT Gateway routing explanation:**
- Private subnet instances send internet-bound traffic to the NAT Gateway
- NAT Gateway replaces private instance IP with its Elastic IP (source IP translation)
- NAT Gateway forwards request to internet
- Response returns to NAT Gateway
- NAT Gateway returns response to original private instance
- Private instances remain unreachable from the internet (outbound only)

## VPC Architecture Summary

The complete VPC architecture I created:

```
        Internet (0.0.0.0/0)
              ↓
    ┌─────────────────────┐
    │  Internet Gateway   │
    │   (Lab IGW)         │
    └──────────┬──────────┘
               ↓
    ┌──────────────────────────────┐
    │     Lab VPC (10.0.0.0/16)    │
    │                              │
    │  ┌─────────────────────────┐ │
    │  │ Public Subnet           │ │
    │  │ (10.0.0.0/24)           │ │
    │  │                         │ │
    │  │ Bastion Server ◄────────┼─┼─── SSH from Internet
    │  │ NAT Gateway             │ │
    │  │                         │ │
    │  │ Route: 0.0.0.0/0 → IGW │ │
    │  └─────────────────────────┘ │
    │              ↓                │
    │  ┌─────────────────────────┐ │
    │  │ Private Subnet          │ │
    │  │ (10.0.2.0/23)           │ │
    │  │                         │ │
    │  │ EC2 Instances           │ │
    │  │ Databases               │ │
    │  │                         │ │
    │  │ Route: 0.0.0.0/0 → NAT │ │
    │  └─────────────────────────┘ │
    │              ↑                │
    └──────────────┼────────────────┘
                   ↓
        ┌──────────────────────┐
        │   NAT Gateway        │
        │ (outbound internet)  │
        └──────────────────────┘
```

## Network Security Layers

The VPC implements defense-in-depth with multiple security layers:

| Layer | Control | Protection |
|-------|---------|-----------|
| **Subnet Isolation** | Route tables | Private subnet unreachable from internet |
| **Security Groups** | Inbound/outbound rules | Allow specific ports and protocols |
| **NACLs** | Stateless filtering | Additional subnet-level filtering |
| **Bastion Pattern** | Jump box | Access private instances through bastion |
| **NAT Gateway** | Outbound translation | Private instances hidden from internet |

## Conclusion

I successfully designed and implemented a complete VPC infrastructure:

- ✅ Created VPC with custom 10.0.0.0/16 CIDR block
- ✅ Enabled DNS hostnames for EC2 instance DNS names
- ✅ Created public subnet (10.0.0.0/24) with auto-assigned public IPs
- ✅ Created private subnet (10.0.2.0/23) for internal resources
- ✅ Created Internet Gateway for public internet connectivity
- ✅ Renamed default route table to "Private Route Table"
- ✅ Created public route table with internet-bound routing (0.0.0.0/0 → IGW)
- ✅ Associated public route table with public subnet
- ✅ Launched bastion server in public subnet with SSH access
- ✅ Created bastion security group allowing SSH from anywhere
- ✅ Created NAT Gateway in public subnet with Elastic IP
- ✅ Configured private route table for outbound internet (0.0.0.0/0 → NAT)
- ✅ Implemented defense-in-depth security with public/private tiers
- ✅ Designed scalable subnet CIDR ranges
- ✅ Created bastion host pattern for secure private access

This lab demonstrated enterprise-grade VPC design with proper segmentation, internet connectivity, and security controls. The bastion pattern enables secure access to private resources, NAT Gateways provide outbound internet for private instances, and route tables enforce traffic policies across the network.

## Additional Resources

- [Amazon VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [VPC CIDR Blocks and Subnetting](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)
- [Internet Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)
- [Route Tables and Routing](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [Network ACLs](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [Bastion Host Pattern](https://docs.aws.amazon.com/quickstart/latest/linux-bastion/architecture.html)
- [VPC Best Practices](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-best-practices.html)
- [CIDR Notation and IP Addressing](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)
