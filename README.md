# AWS Developer Associate 2020 Exam Notes

## Table of contents: 
  - [Exam Domain and % of Examination](#exam-domain-and--of-examination)


## Exam Domain and % of Examination

Domain|% of Examinations
|:----|:----------------:|
| Domain 1: Deployment | 22% |
| Domain 2: Security | 26% |
| Domain 3: Developement with AWS Services | 30% |
| Domain 4: Refractoring | 10% |
| Domain 5: Monitoring and Troubleshooting | 12 % |
| **TOTAL** | **100%** |

**Domain 1: Deployment**
- Deploy written code in AWS using existing CI/CD pipelines, processes, and patterns. 
- Deploy applications using Elastic Beanstalk.  
- Prepare the application deployment package to be deployed to AWS.  
- Deploy serverless applications.  
 
**Domain 2: Security**  
- Make authenticated calls to AWS services.  
- Implement encryption using AWS services.  
- Implement application authentication and authorization. 
 
**Domain 3: Development with AWS Services**  
- Write code for serverless applications. 
- Translate functional requirements into application design. 
- Implement application design into application code. 
- Write code that interacts with AWS services by using APIs, SDKs, and AWS CLI. 
 
**Domain 4: Refactoring**  
- Optimize application to best use AWS services and features. 
- Migrate existing application code to run on AWS. 
 
**Domain 5: Monitoring and Troubleshooting**  
- Write code that can be monitored. 
- Perform root cause analysis on faults found in testing or production

## Notes
---
### **AWS Fundamentals**
---
#### EC2
- **SSH:**
  - Programmatic way to access your instance (port 22 needs to be open in a security group).
  - .pem file's permissions are required to change before being used as key to access the instance.
- **Instances Metadata**: SSH into intance and then `curl http://169.254.169.254/latest/meta-data/`
- **User data section**: Input scripts to this section to setup the instance when it's initiated (e.g update packages, install web servers...)
- **Launch Types:**
  - ***On Demand***: short workload, predictable pricing, pay as you go. 
  - ***Reserved Instances***: long workloads (>= 1 year), pay upfront.
  - ***Convertible Reserved Instances***: long workloads with flexible instances.
  - ***Schedule Reserved Instances***: launch within time window you reserve.
  - ***Spot Instances***: short workloads, for cheap, can lose instances.
  - ***Dedicated Instances***: no other customers will share your hardware, expensive.
  - ***Dedicated Hosts***: book an entire physical server, control instance placement, highly expensive. 
- **Security group:**
  - Act as a fireware that controls traffic that goes into (inboud) and goes out (outbound) the instance.
  - Can be attached to multiple instances.
  - By default, all **inboud** is *blocked*, **outbound** is *authorized*.
  - **SG Referencing**: SGs can be used to authorize access between instances.
- **Billing:**
  - Billed by seconds
  - Billed for storage, data transfer, fixed IP public address, load balancing.
- **Custom AMI**:
  - Pre-installed packages needed.
  - Faster boot time (no need for long ec2 user data at boot time).
  - Machine comes configured with monitoring / enterprise software.
  - Security concerns - control over the machines in the network.
  - Control of maintenance and updates of AMIs over time.
  - Active Directory Integration out of the box.
  - *Installing your app ahead of time (for faster deploys when auto-scaling)*
  - *Built for a specific AWS region*
- **CPU Credits:** allow your instance to burst, credits are lost when bursting and earned when not bursting.  
#### IAM
- Group > User > Role > Policy.
- Use *Role* that has the only neccessary policies to give permissions to EC2 instances.
- For the on-premise appliations that needs permissions to use AWS Resources, create a dedicated user for it and store the credentials into environment variables (or use AWS CLI `aws configure` to store he credentials).
- You can retrieve the role name attached to your EC2 instance using the metadata service but not the policy itself.
- Use *STS decode-authorization-message* API to decode the cryptic error message.

#### ELB
- Load balancers are servers that forward internet traffic to multiple servers (EC2 Instances) downstream.
- Why?:
  - Spread load across multiple downstream instances.
  - Expose a single point of access (DNS) to your application.
  - Seamlessly handle failures of downstream instances: LB does regular health checks to your instances and stop sending traffic to unhealthy instances.
  - Provide SSL Termination (HTTPS) for your website:
  - Enforce stickness with cookies: allows the user to talk to the same instance.
  - High availability cross zones.
  - Separate public traffic from private traffic.
- Types:
  - Application Load Balancer (v2 - new generation)
    - Load balance to multiple HTTP applications across machines (target groups).
    - Load balance to multiple applications on the same machine (ex: containers).
    - Load balance based on route in URL.
    - Load balance based on hostname in URL
    - The application servers don't see the IP of the client directly
      - The true IP of the client is inserted in the header **X-Forwarded-For**
      - We can also get Port (X-Forwarded-Port) and Proto (X-Forwarded-Proto).
  - Network load balancers (Layer 4) allow to do:
    - Forward TCP traffic to your instances.
    - Handle millions of request per seconds.
    - Support for static IP or elastic IP addresses.
    - Less latency ~ 100 ms (vs 400ms for ALB)
    - Mostly used for extreme performance.
- Good to know:
  - Any LB has a static host name. Do not resolve and use underlying IP.
  - LBs can scale but not instantaneously - contact AWS for a "warm-up".
  - NLB directly see the client IP.
  - 4xx errors are client induced errors.
  - 5xx erros are application induced errors.
    - Load Balancer Errors 503 meants at capacity or no registered target.
  - If the LB can't connect to your application, check your security groups.
#### ASG
- What:
  - Scale out (add EC2 instances) to match an increased load.
  - Scale in (remove EC2 instances) to match a decreased load.
  - Ensure we have a minimum and a maximum number of machines running.
  - Automatically Register new instances to a load balancer.
- Launch confguration:
  - An "instruction" the ASG use to create the instances.
  - Details:
    - AMI + Instance Type
    - EC2 User Data
    - EBS Volumes
    - Security Groups
    - SSH Key Pairs
- Auto Scaling Alarms:
  - Use CloudWatch Alarms.
  - An alarm monitors a metric (such as Average CPU).
  - Metrics are computed for the overall ASG instances.
- Auto Scaling Custom Metric
  - We can auto scale based on a custom metric (ex: number  of connected users).
    1. Send custom metric from application on EC2 to CloudWatch (PutMetric API).
    2. Create CloudWatch alarm to react to low/high values.
    3. Use the CloudWatch alarm as the scaling policy for ASG.
- Extra notes:
  - Update ASG by providing a new launch configuration.
  - IAM roles attached to an ASG will get assigned to EC2 instances.
  - ASG are free. You pay for the underlying resourcse being launched.
#### EBS
- Network drive
  - Uses the network to communicate with teh instance, which means there might be a bit of latency.
  - Can be detached from an EC2 instance and attached to to another one quickly.
- Locked to an AZ
  - An EBS Volume in us-east-1a cannot be attached to us-east-1b
  - To move a volume accross, you first need to snapshot it.
- Have a provisioned capacity (size in GBs, and IOPS)
  - You get billed for all the provisioned capacity.
  - You can increase the capcity of the drive over time.
- Types:
  - GP2(SSD): General purpose SSD volume that balances price and performance for a wide variety of workloads.
  - IOI (SSD): Highest-performance SSD volume for mission-critical low-latency or high-throughput workloads.
  - STI (HDD): Low cost HDD volume designed for frequently accessed, throughput-intensive workloads.
  - SCI (HDD): Lowest cost HDD volume designed for less frequently accessed workloads.
- Snapshots:
  - Used to backup EBS Volumes.
  - Only tak the actual space of the blocks on the volume.
    - If you snapshot a 100GB drive that only has 5 GB of data, then your EBS snapshot will only be 5GB.
- Extra notes:
  - Can only be attached to only one instance at a time.
  - Migrating across AZ: Snapshot -> recreate in a different AZ.
  - Root EBS Volume of instances get terminated by default if the EC2 instance gets terminated (You can disable that). 
#### Route53
#### RDS
#### Elastic Cache
#### VPC
#### Amazon S3
---
### **Deployment**
---
#### CodeCommit
#### CodePipeline
#### CodeBuild
#### CodeDeploy
#### AWS Elastic Beanstalk
#### SAM
#### Docker in AWS (ECS, ECR & Fargate)
---

### **Development**
---
#### Lambda
#### AWS Step Functions && AWS SWF (Legacy)
#### DynamoDB
#### API Gateway & Cognito
#### SQS
#### SNS
#### Kinesis
#### AWS CLI, IAM Roles & Policies
---

### **Monitoring**
---
#### CloudWatch
#### AWS X-Ray
#### AWS CloudTrail
---

### **Security**
---
#### KMS
- Limit of data to be encrypted: 4KB.
- If data > 4KB, use Envelope Encryptionv === Encryption SDK === GenerateDataKey API.
#### SSM Parameter Store
- Secure storage for configuration and secrets.
- Optional Seamless Encryption using KMS.
- Severless, scalable, durable, easy SDK free.
- Version tracking of configurations/secrets.
- Configuration management using path & IAM.
- Notifications with CloudWatch Events.
- Integration with CloudFormation.
- Store Hierarchy
- CLI
  - Get undecrypted params: `aws ssm get-parameters --names [parameter name/path]`
  - Get dencrypted params: `aws ssm get-parameters --names [parameter name/path] --with-decryption` 
#### IAM Best Practices






