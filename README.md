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
- Min: 1 msg/sec to 10,000 msg/sec
- Retention:
  - Default: 4 days
  - Maximum: 14 days
- No of messages in a queue: no limit
- Can have duplicate messages (at least one message)
- Can have out of order messages.
- **Size of messages:** 256KB
- **Delay queue:**
  - Default delay: 0 seconds
  - Maximum delay (Consumers don't see it immediately): 15 minutes
  - Can set a default at queue level
  - Can override the default using the DelaySeconds parameter.
- **Producing messages**
  - Sent Message structure:
    - Body (up to 256KB - compulsory).
    - Message attributes (metadata - optional)
    - Delay delivery (optional)
  - Response:
    - Message identifier
    - MD5 hash of the body
- **Consuming Messages:**
  - Poll SQS for messages (receive up to 10 messages at a time).
  - Process the message within the visibility timeout.
  - Delete the message using the message ID and receipt handle
- **Dead Letter Queue:**
  - Where the messages go after the number of failures exceeds the threshold.
- **Long Polling:**
  - Decreases the number of API calls made to SQS while
  - Long polling can be enabled at the queue level or at the API level using **WaitTimeSeconds**
- **FIFO Queue:**
  - Lower throughput (up to 3,000 per second )
- **Sending Large Message:**
  - Using the SQS Extended Client (Java Library): Sending the large message to S3 and send the metadata via SQS Queue to the Consumer to let the consumer know where to get the message on S3.
- **Security:**
  - HTTPS: Encryption in flight.
  - SSE (Server-Side Encryption) using KMS:
    - CMK available
    - Can set data key reuse period (1 min -> 24 hours)
    - Only encrypt the body, not the metadata (message ID, timestamp, attributes)
  - IAM policy must allow usage of SQS.
  - No VPC Endpoint, must have internet access to access SQS.
- **Must-known API:**
  - CreateQueue, DeleteQueue
  - **PurgeQueue:** delete all the message in queue.
  - **SendMessage, ReceiveMessage, DeleteMessage**
  - Batch APIs for **SendMessage, DeleteMessage, ChangeMessageVisibility** helps decrease your costs. 
#### SNS
#### Kinesis
#### AWS CLI, IAM Roles & Policies
#### Cloudformation
- Parameters:
  - Reference a parameter: `!Ref ParameterName/ResourceName`
  - Pseudo Parameters:
    Reference Value | Example Reture Value
    |:---|:---|
    |AWS::AccountId| 1234567890
    |AWS::NotificationARNs| [arn:aws:sns:us-east-1:12345670:MyTopic]|
    |AWS::NoValue|Does not return a value|
    |AWS::Region|us-east-2|
    |AWS::StackId| arn:aws:cloudformation:us-east-1:123456789:stack/Mystack/1xw-32423-1234-5f-2323fa|
    |AWS::StackName| MyStack|
- Mappings
  - Accessing Mappings Value: `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`
- Outputs:
  - You can't delete a CloudFormation Stack if its outputs are being referenced by another CloudFormation stack.
  - Cross stack reference: `!ImportValue outputName`.
- Conditions:
  - Instrinsic function (logical):
    - Fn:Add
    - Fn::Equals
    - Fn::If
    - Fn::Not
    - Fn::Or      
- Intrinsic Functions:
  - Fn::Ref (!Ref): can be leveraged to reference
    - Parameters: returns the value of the parameter
    - Resources: returns the physical ID of the underlying resource (ex: EC2 ID)
  - Fn::GetAtt:
    - Get attributes attached to resources you create (e.g Availablility Zone/ Instance Type of an EC2 instance)
  - Fn::FindInMap:
    - Return a named value from a specific key.
    - `!FindInMap [ MapName, TopLevelKey, SecondLevelKey ]`
  - Fn::ImportValue
    - Import values that are exported in other templates.
    - `!ImportValue exportedName`
  - Fn::Join
    - Join values with a delimiter
    - `!Join [ delimiter, [ comma-delimited list of values]]`
  - Fn::Sub
    - Used to substitute variables from a text.
    - !
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
  - By name:
    - Get undecrypted params: `aws ssm get-parameters --names [parameter's name]`
    - Get dencrypted params: `aws ssm get-parameters --names [parameter's name] --with-decryption` 
  - By path:
    - One level: `aws ssm get-parameters-by-path --path [path created in SSM]`
    - Recursive: `aws ssm get-parameters-by-path --path [path created in SSM] --recursive` 


#### IAM Best Practices
- General:
  - Never use Root Credentials, enable MFA for Root Account.
  - Grant Least Privlege.
    - Each Group / Use / Role should only have the minimum level of permission it needs.
    - Never grant a policy with "*" access to a service.
    - Monitor API calls made by a user in CloudTrail (especially Denied ones).
  - Never store IAM key credentials on any machine but a personal computer or on-premise server.
  - On premise server best practice is to call STS to obtain temporary security credentials.


- IAM Roles:
  - EC2/Lambda/ECS tasks/CodeBuild should have its own service roles.
  - 1 role per application/lambda/service (do not reuse role).
  - Create least-privledged role for any service that requires it.


- Cross Account Access:
  - Define an IAM Role for another account to access.
  - Define which accounts can access this IAM. 
  - Use AWS STS to retrieve credentials and impersonate the IAM role you have access to (AssumeRole API).
  - Temporary credentials can be valid between 15 minutes to 1 hour.







