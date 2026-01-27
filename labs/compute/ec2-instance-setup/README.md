# EC2 Instance Setup Lab

## Objective
To provision and configure an Amazon EC2 instance and understand the fundamentals of compute resources in AWS.

## Key Concepts Covered
- EC2 instance creation
- Instance types and sizing
- Key pairs and security groups
- Connecting to an EC2 instance

## Outcome
Successfully launched and accessed an EC2 instance, demonstrating foundational cloud compute knowledge.


I completed a hands-on lab exploring Amazon EC2 to understand how resizable compute capacity is launched, managed, monitored, and scaled in the cloud.

In this lab, I launched an EC2 instance with termination protection enabled, monitored its performance, modified the security group to allow HTTP access, and resized the instance to observe how EC2 supports scaling. I also tested termination protection and safely terminated the instance at the end of the lab.

Below are the step-by-step screenshots documenting the process and key configurations.

## Launching, monitoring, modifying, resizing, testing, and terminating an E2 instance

<img width="468" height="76" alt="image" src="https://github.com/user-attachments/assets/2cea9178-fedb-4382-bda4-cbcb9119d27c" />

<img width="423" height="274" alt="image" src="https://github.com/user-attachments/assets/7d8926c0-627e-433a-b9e0-2a3be3e661b0" />

I proceeded to name my instance "Web Server" and chose Amazon Linux as it's AMI.

<img width="468" height="343" alt="image" src="https://github.com/user-attachments/assets/1dda2993-e4a1-48a0-9fe0-9eb82a2328d4" />

I chose t.3micro as the instance type and proceeded without a key pair.

<img width="468" height="356" alt="image" src="https://github.com/user-attachments/assets/db01329f-1d19-4ae7-b244-822a216a8872" />

Under Network Settings, I configured Lab VPC as the VPC, named the Secuirty Group as "Web Server security group" and added a description.

<img width="468" height="312" alt="image" src="https://github.com/user-attachments/assets/3099a9aa-34a5-4e32-86db-4459c03b7241" />

The next step was to add storage, where I used the default 8 GiB disk volume. 

<img width="468" height="134" alt="image" src="https://github.com/user-attachments/assets/fb4ef83a-2595-4dad-b36d-42a0d8661c9d" />

I configured the Advanced details as follows, as well as inputted the commands:

<img width="468" height="329" alt="image" src="https://github.com/user-attachments/assets/46d5e206-dfb4-4f92-b423-aff340fa0b8d" />

<img width="468" height="435" alt="image" src="https://github.com/user-attachments/assets/a056e356-e07b-4a5d-8647-37144b7fdc85" />

Once everything was setup, I launced the instance.

<img width="468" height="228" alt="image" src="https://github.com/user-attachments/assets/e5edfb8b-c8f2-4114-aafb-77a7854253c4" />
<img width="468" height="88" alt="image" src="https://github.com/user-attachments/assets/704fb289-bd67-4c88-9100-e21ad79db19c" />

The instance was visible in the console along with all the details.

<img width="468" height="303" alt="image" src="https://github.com/user-attachments/assets/cddd3ddb-e228-41ec-8fa9-73480e57c9b0" />

I was then able to monitor my instance and obtain the instance screenshot. 

<img width="468" height="305" alt="image" src="https://github.com/user-attachments/assets/c8030beb-9b6b-436b-b843-bd5abe87467b" />

I updated the security group by adding the following inbound rule:

<img width="468" height="276" alt="image" src="https://github.com/user-attachments/assets/9ceea4bb-90d3-41fc-a420-bf93d203d4a2" />

<img width="468" height="265" alt="image" src="https://github.com/user-attachments/assets/5a4137e4-bcd7-47ab-8413-7290f3af9a44" />

I could resize the instance and EBS volume under instance settings, where I changed the instance to t3.small and volume to 10Gib.

<img width="468" height="183" alt="image" src="https://github.com/user-attachments/assets/e52a51d0-a9aa-483a-96b1-f57f4136369b" />
<img width="468" height="281" alt="image" src="https://github.com/user-attachments/assets/a2c877bd-e7b0-4be9-b85c-599fa8bf2820" />

I tested termination protection.

<img width="468" height="159" alt="image" src="https://github.com/user-attachments/assets/6773d326-fe38-4d5e-94c1-9fb40107d786" />

Finally, I disabled termination protection and terminated the instance. 

<img width="468" height="312" alt="image" src="https://github.com/user-attachments/assets/1df78bcd-1662-4595-9b0d-f0afc58190e5" />
<img width="468" height="157" alt="image" src="https://github.com/user-attachments/assets/719757d2-552d-4e2c-9ea5-c81030cf49b8" />
