# 3D E-Commerce Platform Architecture on AWS

## Architecture Diagram

<img width="597" height="857" alt="image" src="https://github.com/user-attachments/assets/2a5f17fc-354d-47a2-b816-1cf290e49203" />

---

## 3D E-Commerce Platform Architecture Overview

The proposed architecture supports a highly available, scalable, and secure 3D web application hosted on AWS. User traffic is first directed through Amazon Route 53, which routes requests to Amazon CloudFront. CloudFront acts as a content delivery network (CDN), helping deliver content quickly to users around the world. Dynamic requests are forwarded to an Application Load Balancer (ALB) running across multiple Availability Zones, while static 3D assets are served directly from Amazon S3.

The application backend runs on Amazon EC2 instances with attached Amazon EBS volumes, which provide persistent storage for the application. AWS Lambda is used for background processing tasks such as converting 3D assets and generating different levels of detail. Data storage is split between Amazon DynamoDB, which stores fast-access metadata and session information, and Amazon Aurora (RDS), which is deployed in multiple Availability Zones to store transactional data such as customer and order information. Amazon ElastiCache (Redis) is used to temporarily store frequently accessed scene data in memory, reducing database load and improving performance.

Monitoring and cost tracking are handled using Amazon CloudWatch and AWS Trusted Advisor.

---

## Why Each AWS Service Was Chosen

- **Amazon Route 53** provides DNS routing and health checks, ensuring users are directed to available and healthy resources.

- **Amazon CloudFront** caches large 3D assets closer to users, which improves loading times and reduces traffic to the backend servers.

- **Amazon S3** is used to store 3D models, textures, thumbnails, and pre-generated levels of detail. It is highly durable and cost-effective, and lifecycle policies can be used to manage storage costs over time.

- **Application Load Balancer (ALB)** distributes incoming traffic across multiple EC2 instances and Availability Zones, improving availability and reliability.

- **Amazon EC2 with Amazon EBS** provides flexible compute resources for the backend application. EBS volumes offer persistent storage attached to EC2 instances. Spot Instances can be used for non-critical tasks to reduce costs.

- **AWS Lambda** is used for event-driven background tasks such as asset conversion and thumbnail generation.

- **Amazon DynamoDB** was chosen for its low latency and automatic scaling, making it suitable for session data, product catalogs, and other read-heavy information.

- **Amazon Aurora (RDS)** provides a relational database for transactional data such as orders and payments. A Multi-AZ setup improves availability and reliability.

- **Amazon ElastiCache (Redis)** stores frequently accessed scene metadata in memory, allowing faster data access and reducing pressure on the databases.

- **Amazon CloudWatch and AWS Trusted Advisor** are used for monitoring system metrics and providing recommendations related to cost, performance, and reliability.

---

## How the Architecture Meets the Five Requirements

### 1. High Availability

The system is designed to remain available even if individual components fail. The ALB, EC2 instances, RDS (Aurora) database, and ElastiCache are deployed across multiple Availability Zones. Route 53 health checks help route traffic away from unhealthy resources, and CloudFront and S3 provide reliable delivery of static assets.

### 2. Scalability

The architecture can scale based on demand. Auto Scaling Groups adjust the number of EC2 instances as traffic changes, DynamoDB automatically handles traffic spikes, Lambda scales for background workloads, and CloudFront reduces load on backend systems by caching content.

### 3. Performance

Performance is improved by CloudFront for global content delivery and ElastiCache for fast in-memory data access. Pre-generated levels of detail and compressed 3D formats reduce file sizes, which helps improve load times and rendering performance.

### 4. Security

Security is handled through IAM roles that limit access to only what is required. Data is encrypted both in transit and at rest, and resources are deployed within a secure VPC environment to reduce exposure.

### 5. Cost Optimization

Costs are managed using S3 lifecycle policies, serverless processing for occasional workloads, and Spot Instances for non-critical EC2 tasks. CloudWatch alarms and Trusted Advisor recommendations help identify and control unnecessary spending.

---

## Design Trade-offs and Challenges

- **Caching vs. Personalization**  
While caching improves performance, it limits personalization. This is handled by caching only static assets and serving user-specific data through dynamic API requests.

- **Lambda Cold Starts**  
Lambda cold starts can add small delays. To avoid this for real-time features, latency-sensitive services run on EC2, while Lambda is used only for background processing.

- **Asset Format and Device Support**  
Supporting different devices requires multiple asset formats and levels of detail, which increases storage and processing needs. Automating these conversions during upload reduces manual effort.

- **Cost Predictability**  
Autoscaling can cause unexpected cost increases during high traffic periods. This is managed by setting budget alerts and monitoring usage through CloudWatch.

---

## Conclusion

This architecture uses a combination of AWS managed services, EC2 compute, and serverless processing to meet the requirements of availability, scalability, performance, security, and cost efficiency. It is suitable for a global 3D web application and can be adjusted as usage grows over time.
