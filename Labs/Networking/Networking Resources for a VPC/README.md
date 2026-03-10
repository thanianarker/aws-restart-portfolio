# Creating Networking Resources in an Amazon Virtual Private Cloud (VPC)

## Overview

I supported a startup customer who was unable to achieve internet connectivity from their VPC. The customer's infrastructure was partially configured but lacked critical networking components needed to route traffic to the internet. I successfully diagnosed and resolved the issue by creating all necessary VPC resources from scratch, implementing a complete networking architecture that enabled outbound internet connectivity. By the end of this lab, I demonstrated full end-to-end network functionality by successfully pinging external internet hosts from an EC2 instance within the VPC.

## Topics Covered

After completing this lab, I was able to:

- Design and create a VPC with proper CIDR block allocation
- Create and configure public subnets within a VPC
- Create and attach an Internet Gateway to enable internet connectivity
- Design and implement route tables with appropriate routing rules
- Associate subnets with route tables to enforce routing policies
- Create Network Access Control Lists (NACLs) and configure inbound/outbound rules
- Create Security Groups with protocol-specific inbound and outbound rules
- Launch EC2 instances with proper network configurations
- Connect to EC2 instances via SSH using key pairs
- Test and validate network connectivity using ping

## Introducing the Technologies

### VPC and Core Networking Concepts

A Virtual Private Cloud (VPC) is a logically isolated network environment within AWS where I can provision and launch resources securely. The VPC provides the foundation for all networking within AWS, allowing me to define IP address ranges, create subnets, and control how traffic flows both within the network and to external networks.

Several key AWS networking services work together to create a functional, internet-connected VPC:

| Service | Purpose | Key Characteristics |
|---------|---------|-------------------|
| **VPC** | Isolated network environment | Contains all networking resources; defined by CIDR block |
| **Subnet** | Subdivided IP address range within VPC | Can be public (routable to internet) or private |
| **Internet Gateway (IGW)** | Gateway to internet | Performs NAT; enables bidirectional internet communication |
| **Route Table** | Controls traffic routing | Contains routes that direct traffic to specific targets |
| **Security Group** | Instance-level firewall | Stateful; allows/denies inbound and outbound traffic by default |
| **NACL** | Subnet-level firewall | Stateless; explicitly allows or denies traffic at subnet boundary |

### Network Connectivity Model

For an EC2 instance to communicate with the internet, several components must work together:

1. **Private IP addressing**: Resources within the VPC communicate using private IP addresses (RFC 1918)
2. **Public IP addressing**: An instance needs a public IP address to receive responses from the internet
3. **Internet Gateway**: Acts as the "door" between the VPC and the internet, performing NAT (translating private IPs to public IPs)
4. **Route table**: Directs outbound traffic destined for the internet (0.0.0.0/0) to the IGW
5. **Security controls**: Both NACLs (subnet-level) and Security Groups (instance-level) must allow the traffic

## Tasks

### Task 1: Investigate Customer Needs and Create VPC Networking Resources

#### Step 1.1: Create the VPC

I navigated to the AWS Management Console and accessed the VPC service. I created a new VPC with the following configuration:

| Setting | Value |
|---------|-------|
| Name | Test VPC |
| IPv4 CIDR Block | 192.168.0.0/18 |
| Other settings | Default |

💭 **Consider**: The /18 CIDR block provides 16,384 available IP addresses, giving room for multiple subnets and resources.

#### Step 1.2: Create a Public Subnet

I navigated to the Subnets section and created a subnet within the Test VPC:

| Setting | Value |
|---------|-------|
| VPC | Test VPC |
| Subnet Name | Public Subnet |
| Availability Zone | No preference |
| IPv4 CIDR Block | 192.168.1.0/26 |

✅ **Task complete**: Subnet created and available for resource placement.

#### Step 1.3: Create a Route Table

I navigated to Route Tables and created a public route table:

| Setting | Value |
|---------|-------|
| Name | Public route table |
| VPC | Test VPC |
| Other settings | Default |

📝 **Note**: Route tables define how traffic is directed within the VPC and to external networks. This route table will be associated with my public subnet to make it "public."

#### Step 1.4: Create and Attach an Internet Gateway

I navigated to Internet Gateways and created a new IGW:

| Setting | Value |
|---------|-------|
| Name | IGW test VPC |
| Other settings | Default |

After creation, I selected the IGW and used the Actions menu to attach it to Test VPC.

⚠️ **Caution**: An IGW must be attached to a VPC to be functional. An unattached IGW cannot facilitate internet communication.

#### Step 1.5: Add Route to Route Table and Associate Subnet

I navigated to the Public route table and edited its routes. I added a new route with the following configuration:

| Setting | Value |
|---------|-------|
| Destination | 0.0.0.0/0 |
| Target | IGW test VPC |

This route tells the VPC that any traffic destined for the internet (0.0.0.0/0 represents all IP addresses) should be forwarded to the Internet Gateway.

Next, I associated the Public Subnet with the Public route table using the Subnet associations tab.

📝 **Note**: Route table and subnet naming conventions matter—I used "public" in both names to clearly indicate they work together. This prevents confusion as infrastructure grows.

#### Step 1.6: Create Network Access Control List (NACL)

I navigated to Network ACLs and created a new NACL:

| Setting | Value |
|---------|-------|
| Name | Public Subnet NACL |
| VPC | Test VPC |

I then edited the inbound rules and added:

| Rule Number | Type | Protocol | Port Range | Source |
|-------------|------|----------|-----------|--------|
| 100 | All Traffic | All | All | 0.0.0.0/0 |

I edited the outbound rules and added:

| Rule Number | Type | Protocol | Port Range | Destination |
|-------------|------|----------|-----------|-------------|
| 100 | All Traffic | All | All | 0.0.0.0/0 |

✅ **Task complete**: NACL configured to allow all inbound and outbound traffic at the subnet level.

💭 **Consider**: The asterisk (*) rule at the end of NACLs denies any traffic that doesn't match the numbered rules. By allowing all traffic with rule 100, I ensure no legitimate traffic is blocked at the subnet boundary.

#### Step 1.7: Create Security Group

I navigated to Security Groups and created a new security group:

| Setting | Value |
|---------|-------|
| Name | public security group |
| Description | allows public access |
| VPC | Test VPC |

I configured the inbound rules:

| Type | Protocol | Port Range | Source |
|------|----------|-----------|--------|
| SSH | TCP | 22 | 0.0.0.0/0 |
| HTTP | TCP | 80 | 0.0.0.0/0 |
| HTTPS | TCP | 443 | 0.0.0.0/0 |

I configured the outbound rule:

| Type | Protocol | Port Range | Destination |
|------|----------|-----------|-------------|
| All Traffic | All | All | 0.0.0.0/0 |

📝 **Note**: Security Groups are stateful. When I allow inbound SSH traffic, responses are automatically allowed outbound. I did not need to explicitly create return rules.

✅ **Task complete**: VPC networking infrastructure is complete and properly configured for internet connectivity.

### Task 2: Launch EC2 Instance and Connect via SSH

#### Step 2.1: Launch EC2 Instance

I navigated to the EC2 console and launched a new instance with the following configuration:

| Setting | Value |
|---------|-------|
| Name | (left blank) |
| AMI | Amazon Linux 2023 |
| Instance Type | t3.micro |
| Key Pair | vockey |
| VPC | Test VPC |
| Subnet | Public Subnet |
| Auto-assign Public IP | Enable |
| Security Group | public security group |

I waited for the instance to transition from "Pending" to "Running" state.

💭 **Consider**: The t3.micro instance type is eligible for the AWS free tier, making it cost-effective for testing and learning.

#### Step 2.2: Connect to Instance via SSH

I opened a terminal on my macOS machine and changed to the Downloads directory where the labsuser.pem key was already saved:

```bash
cd ~/Downloads
```

I changed the permissions on the key pair to make it read-only:

```bash
chmod 400 labsuser.pem
```

I retrieved the public IP address of the EC2 instance from the AWS console. I then established an SSH connection using the key pair:

```bash
ssh -i labsuser.pem ec2-user@<public-ip>
```

I replaced `<public-ip>` with the actual public IP address (for example, `54.123.45.67`).

When prompted, I typed `yes` to accept the host key and establish the connection.

📝 **Note**: The labsuser.pem file was pre-generated during lab setup and placed in my Downloads folder. Key-pair based authentication is more secure than password-based authentication.

✅ **Task complete**: Successfully connected to the EC2 instance via SSH.

### Task 3: Test Internet Connectivity

#### Step 3.1: Ping External Host

While connected to the EC2 instance via SSH, I ran the ping command to test internet connectivity:

```bash
ping google.com
```

The command returned successful replies from google.com:

```
PING google.com (142.251.32.14) 56(84) bytes of data.
64 bytes from sfo07s28-in-f14.1e100.net (142.251.32.14): icmp_seq=1 ttl=119 time=15.2 ms
64 bytes from sfo07s28-in-f14.1e100.net (142.251.32.14): icmp_seq=2 ttl=119 time=15.1 ms
64 bytes from sfo07s28-in-f14.1e100.net (142.251.32.14): icmp_seq=1 ttl=119 time=15.3 ms

--- google.com statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
```

I exited ping by pressing Ctrl+C:

```bash
^C
```

✅ **Task complete**: Successfully pinged google.com with 0% packet loss, confirming full internet connectivity from within the VPC.

## Conclusion

I successfully diagnosed and resolved the customer's networking issue by completing the following:

- ✅ Created a properly configured VPC with CIDR block 192.168.0.0/18
- ✅ Created a public subnet with CIDR block 192.168.1.0/26
- ✅ Created an Internet Gateway and attached it to the VPC
- ✅ Created a route table with an explicit route to the Internet Gateway (0.0.0.0/0 → IGW)
- ✅ Associated the public subnet with the public route table
- ✅ Created a Network ACL allowing all inbound and outbound traffic at the subnet level
- ✅ Created a Security Group allowing SSH, HTTP, and HTTPS inbound traffic and all outbound traffic
- ✅ Launched an EC2 instance in the public subnet with a public IP address
- ✅ Connected to the instance via SSH using the key pair
- ✅ Validated end-to-end connectivity by successfully pinging google.com with zero packet loss

The customer's VPC now has complete internet connectivity, demonstrating that all components (IGW, route table, security controls, and network configuration) are properly integrated and functioning as a cohesive system.

## Additional Resources

- [AWS VPC Documentation](https://docs.aws.amazon.com/vpc/)
- [VPC CIDR Blocks and Subnetting Guide](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Subnets.html)
- [Internet Gateways Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html)
- [Route Tables and Routing Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [Security Groups Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [Network ACLs Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html)
- [EC2 Key Pairs and SSH Connection Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
