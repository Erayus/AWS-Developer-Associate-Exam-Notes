# AWS Developer Associate Exam Notes

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
- 
#### ELB
#### ASG
#### EBS
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
#### SSM Parameter Store
#### IAM Best Practices






