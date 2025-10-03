# AWS (Amazon Web Services) - Cloud Computing Platform

## Overview

Amazon Web Services (AWS) is a comprehensive cloud computing platform provided by Amazon. It offers a wide range of services including computing power, storage options, networking, and databases that are delivered over the internet with pay-as-you-go pricing.

### Key Features
- **Global Infrastructure**: 25 geographic regions with 81 availability zones
- **Broad Services**: 200+ fully featured services
- **Pay-as-you-go**: No upfront costs, pay only for what you use
- **Scalability**: Automatically scale resources up or down
- **Security**: Comprehensive security services and features
- **Reliability**: High availability and fault tolerance

## Core Concepts

### AWS Global Infrastructure
- **Regions**: Geographic areas around the world (e.g., us-east-1, eu-west-1)
- **Availability Zones (AZs)**: One or more discrete data centers within a region
- **Edge Locations**: Sites for CloudFront and Route 53
- **Local Zones**: Extensions of AWS regions that place compute, storage, database, and other services closer to users
- **Wavelength Zones**: AWS infrastructure deployments embedded within telecommunications providers

### AWS Services Categories
- **Compute**: EC2, Lambda, Elastic Beanstalk, Batch
- **Storage**: S3, EBS, EFS, Glacier
- **Database**: RDS, DynamoDB, Redshift, ElastiCache
- **Networking**: VPC, CloudFront, Route 53, Direct Connect
- **Security**: IAM, KMS, Shield, WAF
- **Management**: CloudWatch, CloudTrail, Config, OpsWorks
- **Analytics**: EMR, Athena, Kinesis, QuickSight
- **AI/ML**: SageMaker, Rekognition, Comprehend, Polly
- **IoT**: IoT Core, IoT Device Management, IoT Analytics

### Key AWS Services

#### Compute Services
- **EC2 (Elastic Compute Cloud)**: Virtual servers in the cloud
- **Lambda**: Serverless compute service
- **Elastic Beanstalk**: Platform as a Service (PaaS)
- **Batch**: Batch computing service
- **EKS (Elastic Kubernetes Service)**: Managed Kubernetes service
- **ECS (Elastic Container Service)**: Container orchestration service

#### Storage Services
- **S3 (Simple Storage Service)**: Object storage service
- **EBS (Elastic Block Store)**: Block storage for EC2 instances
- **EFS (Elastic File System)**: Network file system
- **Glacier**: Long-term archival storage
- **Storage Gateway**: Hybrid cloud storage service

#### Database Services
- **RDS (Relational Database Service)**: Managed relational databases
- **DynamoDB**: NoSQL database service
- **Redshift**: Data warehouse service
- **ElastiCache**: In-memory caching service
- **Neptune**: Graph database service

#### Networking Services
- **VPC (Virtual Private Cloud)**: Isolated cloud network
- **CloudFront**: Content delivery network (CDN)
- **Route 53**: Domain Name System (DNS) web service
- **Direct Connect**: Dedicated network connection
- **API Gateway**: API management service

## AWS CLI (Command Line Interface)

### Installation
```bash
# Install AWS CLI on Linux (Ubuntu/Debian)
sudo apt update
sudo apt install awscli

# Install AWS CLI on macOS
brew install awscli

# Install AWS CLI on Windows
# Download from: https://aws.amazon.com/cli/

# Verify installation
aws --version
```

### Configuration
```bash
# Configure AWS CLI
aws configure

# Configure with specific profile
aws configure --profile myprofile

# Configure with environment variables
export AWS_ACCESS_KEY_ID=your_access_key
export AWS_SECRET_ACCESS_KEY=your_secret_key
export AWS_DEFAULT_REGION=us-east-1

# Configure with credentials file
# ~/.aws/credentials
[default]
aws_access_key_id = your_access_key
aws_secret_access_key = your_secret_key

[myprofile]
aws_access_key_id = your_access_key
aws_secret_access_key = your_secret_key

# ~/.aws/config
[default]
region = us-east-1

[profile myprofile]
region = us-west-2
```

### Common CLI Commands
```bash
# List all S3 buckets
aws s3 ls

# Create S3 bucket
aws s3 mb s3://my-bucket-name

# Upload file to S3
aws s3 cp file.txt s3://my-bucket-name/

# Download file from S3
aws s3 cp s3://my-bucket-name/file.txt .

# List EC2 instances
aws ec2 describe-instances

# Create EC2 instance
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-1234567890abcdef0

# Stop EC2 instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Start EC2 instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Terminate EC2 instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# List Lambda functions
aws lambda list-functions

# Create Lambda function
aws lambda create-function \
  --function-name my-function \
  --runtime python3.8 \
  --role arn:aws:iam::123456789012:role/my-lambda-role \
  --handler lambda_function.lambda_handler \
  --code S3Bucket=my-bucket,S3Key=lambda-code.zip

# Invoke Lambda function
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  response.json

# List IAM users
aws iam list-users

# Create IAM user
aws iam create-user --user-name my-user

# Create IAM access key
aws iam create-access-key --user-name my-user

# List CloudFormation stacks
aws cloudformation list-stacks

# Create CloudFormation stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-url https://s3.amazonaws.com/my-bucket/template.json

# List CloudWatch logs
aws logs describe-log-groups

# Get CloudWatch metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-02T00:00:00Z \
  --period 300 \
  --statistics Average
```

## AWS SDKs

### Python SDK (Boto3)
```python
import boto3

# Create S3 client
s3 = boto3.client('s3')

# List buckets
response = s3.list_buckets()
for bucket in response['Buckets']:
    print(bucket['Name'])

# Create bucket
s3.create_bucket(Bucket='my-bucket-name')

# Upload file
s3.upload_file('file.txt', 'my-bucket-name', 'file.txt')

# Download file
s3.download_file('my-bucket-name', 'file.txt', 'file.txt')

# Create EC2 client
ec2 = boto3.client('ec2')

# Describe instances
response = ec2.describe_instances()
for reservation in response['Reservations']:
    for instance in reservation['Instances']:
        print(instance['InstanceId'])

# Create Lambda client
lambda_client = boto3.client('lambda')

# List functions
response = lambda_client.list_functions()
for function in response['Functions']:
    print(function['FunctionName'])

# Invoke function
response = lambda_client.invoke(
    FunctionName='my-function',
    Payload='{"key": "value"}'
)
print(response['Payload'].read().decode())
```

### JavaScript SDK
```javascript
const AWS = require('aws-sdk');

// Create S3 client
const s3 = new AWS.S3();

// List buckets
s3.listBuckets((err, data) => {
    if (err) console.log(err, err.stack);
    else console.log(data.Buckets);
});

// Create bucket
const params = {
    Bucket: 'my-bucket-name'
};
s3.createBucket(params, (err, data) => {
    if (err) console.log(err, err.stack);
    else console.log(data);
});

// Upload file
const uploadParams = {
    Bucket: 'my-bucket-name',
    Key: 'file.txt',
    Body: 'Hello, World!'
};
s3.upload(uploadParams, (err, data) => {
    if (err) console.log(err, err.stack);
    else console.log(data);
});

// Create EC2 client
const ec2 = new AWS.EC2();

// Describe instances
ec2.describeInstances((err, data) => {
    if (err) console.log(err, err.stack);
    else console.log(data);
});
```

## AWS CloudFormation

### CloudFormation Template
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Simple EC2 instance with security group'

Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: 'Name of an existing EC2 KeyPair'
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
    Description: 'EC2 instance type'

Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      ImageId: ami-0abcdef1234567890
      SecurityGroups:
        - !Ref InstanceSecurityGroup
      Tags:
        - Key: Name
          Value: MyEC2Instance

  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Enable SSH access'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

Outputs:
  InstanceId:
    Description: 'Instance ID'
    Value: !Ref EC2Instance
  PublicIP:
    Description: 'Public IP address'
    Value: !GetAtt EC2Instance.PublicIp
```

### CloudFormation Commands
```bash
# Validate template
aws cloudformation validate-template --template-body file://template.yaml

# Create stack
aws cloudformation create-stack \
  --stack-name my-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=KeyName,ParameterValue=my-key-pair

# Update stack
aws cloudformation update-stack \
  --stack-name my-stack \
  --template-body file://template.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-stack

# List stacks
aws cloudformation list-stacks

# Describe stack
aws cloudformation describe-stacks --stack-name my-stack
```

## AWS EC2 (Elastic Compute Cloud)

### EC2 Instance Types
- **General Purpose**: t2, t3, t4g, m5, m6g
- **Compute Optimized**: c5, c6g, c6i
- **Memory Optimized**: r5, r6g, r6i
- **Storage Optimized**: i3, i3en, d3, h1
- **Accelerated Computing**: p3, p4, g4, inf1

### EC2 Management
```bash
# Launch EC2 instance
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --security-group-ids sg-1234567890abcdef0 \
  --subnet-id subnet-1234567890abcdef0 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=MyInstance}]'

# Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Reboot instance
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0

# Terminate instance
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Create AMI from instance
aws ec2 create-image \
  --instance-id i-1234567890abcdef0 \
  --name "My AMI" \
  --description "AMI created from my instance"

# Create security group
aws ec2 create-security-group \
  --group-name my-security-group \
  --description "My security group"

# Add security group rule
aws ec2 authorize-security-group-ingress \
  --group-id sg-1234567890abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Create key pair
aws ec2 create-key-pair \
  --key-name my-key-pair \
  --query 'KeyMaterial' \
  --output text > my-key-pair.pem

# Set key pair permissions
chmod 400 my-key-pair.pem
```

## AWS S3 (Simple Storage Service)

### S3 Operations
```bash
# Create bucket
aws s3 mb s3://my-bucket-name

# List buckets
aws s3 ls

# Upload file
aws s3 cp file.txt s3://my-bucket-name/

# Upload directory
aws s3 cp my-directory/ s3://my-bucket-name/my-directory/ --recursive

# Download file
aws s3 cp s3://my-bucket-name/file.txt .

# Download directory
aws s3 cp s3://my-bucket-name/my-directory/ my-directory/ --recursive

# List bucket contents
aws s3 ls s3://my-bucket-name/

# Delete file
aws s3 rm s3://my-bucket-name/file.txt

# Delete bucket (must be empty)
aws s3 rb s3://my-bucket-name

# Set bucket policy
aws s3api put-bucket-policy \
  --bucket my-bucket-name \
  --policy '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::my-bucket-name/*"
      }
    ]
  }'

# Enable versioning
aws s3api put-bucket-versioning \
  --bucket my-bucket-name \
  --versioning-configuration Status=Enabled

# Set bucket encryption
aws s3api put-bucket-encryption \
  --bucket my-bucket-name \
  --server-side-encryption-configuration '{
    "Rules": [
      {
        "ApplyServerSideEncryptionByDefault": {
          "SSEAlgorithm": "AES256"
        }
      }
    ]
  }'
```

## AWS Lambda

### Lambda Operations
```bash
# Create Lambda function
aws lambda create-function \
  --function-name my-function \
  --runtime python3.8 \
  --role arn:aws:iam::123456789012:role/my-lambda-role \
  --handler lambda_function.lambda_handler \
  --code S3Bucket=my-bucket,S3Key=lambda-code.zip

# Update function code
aws lambda update-function-code \
  --function-name my-function \
  --s3-bucket my-bucket \
  --s3-key lambda-code-updated.zip

# Invoke function
aws lambda invoke \
  --function-name my-function \
  --payload '{"key": "value"}' \
  response.json

# List functions
aws lambda list-functions

# Get function configuration
aws lambda get-function-configuration --function-name my-function

# Delete function
aws lambda delete-function --function-name my-function

# Create event source mapping
aws lambda create-event-source-mapping \
  --function-name my-function \
  --event-source-arn arn:aws:sqs:us-east-1:123456789012:my-queue \
  --batch-size 10
```

### Python Lambda Function
```python
import json
import boto3

def lambda_handler(event, context):
    # Get S3 client
    s3 = boto3.client('s3')
    
    # Process event
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        # Get object from S3
        response = s3.get_object(Bucket=bucket, Key=key)
        content = response['Body'].read().decode('utf-8')
        
        # Process content
        print(f"Processing file: {key}")
        print(f"Content: {content}")
    
    return {
        'statusCode': 200,
        'body': json.dumps('Processing complete!')
    }
```

## AWS IAM (Identity and Access Management)

### IAM Operations
```bash
# Create user
aws iam create-user --user-name my-user

# Create access key
aws iam create-access-key --user-name my-user

# Create group
aws iam create-group --group-name my-group

# Add user to group
aws iam add-user-to-group --user-name my-user --group-name my-group

# Create policy
aws iam create-policy \
  --policy-name my-policy \
  --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": "s3:*",
        "Resource": "*"
      }
    ]
  }'

# Attach policy to user
aws iam attach-user-policy \
  --user-name my-user \
  --policy-arn arn:aws:iam::123456789012:policy/my-policy

# List users
aws iam list-users

# List groups
aws iam list-groups

# List policies
aws iam list-policies

# Get user policy
aws iam get-user-policy --user-name my-user --policy-name my-policy
```

### IAM Policy Example
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "logs:*",
      "Resource": "arn:aws:logs:us-east-1:123456789012:log-group:/aws/lambda/my-function"
    }
  ]
}
```

## AWS CloudWatch

### CloudWatch Operations
```bash
# Create log group
aws logs create-log-group --log-group-name /aws/lambda/my-function

# Create log stream
aws logs create-log-stream \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name 2023/01/01/[$LATEST]abcdef1234567890

# Get log events
aws logs get-log-events \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name 2023/01/01/[$LATEST]abcdef1234567890

# Put metric data
aws cloudwatch put-metric-data \
  --namespace MyApplication \
  --metric-name RequestCount \
  --value 10 \
  --unit Count

# Get metric statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2023-01-01T00:00:00Z \
  --end-time 2023-01-02T00:00:00Z \
  --period 300 \
  --statistics Average

# Create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name HighCPUUtilization \
  --alarm-description "CPU utilization exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:my-topic
```

## AWS Security

### Security Best Practices
- **IAM**: Use IAM roles instead of access keys when possible
- **Multi-Factor Authentication**: Enable MFA for all IAM users
- **Least Privilege**: Grant minimum required permissions
- **Security Groups**: Use security groups to control network access
- **VPC**: Use VPC to isolate resources
- **Encryption**: Encrypt data at rest and in transit
- **Monitoring**: Monitor and audit AWS usage
- **Backup**: Regularly backup important data

### Security Services
- **IAM**: Identity and access management
- **KMS**: Key management service
- **Shield**: DDoS protection
- **WAF**: Web application firewall
- **GuardDuty**: Threat detection
- **Security Hub**: Security findings aggregation
- **Macie**: Data security and data privacy

## AWS Cost Management

### Cost Optimization
- **Right Sizing**: Use appropriate instance types
- **Reserved Instances**: Purchase reserved instances for steady workloads
- **Spot Instances**: Use spot instances for fault-tolerant workloads
- **Auto Scaling**: Automatically scale resources based on demand
- **Storage Optimization**: Use appropriate storage classes
- **Monitoring**: Monitor costs with CloudWatch and Cost Explorer

### Cost Management Commands
```bash
# Get cost and usage report
aws ce get-cost-and-usage \
  --time-period Start=2023-01-01,End=2023-01-31 \
  --granularity MONTHLY \
  --metrics "BlendedCost","UnblendedCost","UsageQuantity" \
  --group-by Type=DIMENSION,Key=SERVICE

# Get cost allocation tags
aws ce get-cost-allocation-tags \
  --status Active

# Get reservations
aws ce describe-reservations

# Get savings plans
aws ce describe-savings-plans
```

## Interview Questions

### Beginner Level
1. **What is AWS?**
   - AWS is Amazon's cloud computing platform that provides on-demand cloud computing services and APIs

2. **What are the main benefits of AWS?**
   - Scalability, cost-effectiveness, reliability, security, and global infrastructure

3. **What is EC2?**
   - EC2 (Elastic Compute Cloud) provides scalable virtual servers in the cloud

4. **What is S3?**
   - S3 (Simple Storage Service) is object storage service that offers industry-leading scalability, data availability, security, and performance

### Intermediate Level
1. **What is the difference between S3 and EBS?**
   - S3 is object storage accessible via HTTP, while EBS is block storage attached to EC2 instances

2. **What is IAM?**
   - IAM (Identity and Access Management) is a web service that helps you securely control access to AWS resources

3. **What is CloudFormation?**
   - CloudFormation is a service that helps you model and set up your Amazon Web Services resources

4. **What is the difference between Spot Instances and Reserved Instances?**
   - Spot Instances are unused EC2 instances available at up to 90% discount, while Reserved Instances provide capacity reservation and significant discount

### Advanced Level
1. **How does AWS ensure security?**
   - AWS provides security through IAM, VPC, security groups, encryption, compliance programs, and shared responsibility model

2. **What is AWS Lambda and how does it work?**
   - Lambda is a serverless compute service that runs your code in response to events and automatically manages the underlying compute resources

3. **What is the difference between AWS and other cloud providers?**
   - AWS has the largest market share, most comprehensive service offering, and mature ecosystem compared to other cloud providers

4. **How do you optimize costs in AWS?**
   - Use right sizing, reserved instances, spot instances, auto scaling, storage optimization, and monitoring with Cost Explorer

## Best Practices

### Security
- **Use IAM Roles**: Prefer IAM roles over access keys
- **Enable MFA**: Enable multi-factor authentication for all users
- **Least Privilege**: Grant minimum required permissions
- **Network Security**: Use security groups and NACLs
- **Data Encryption**: Encrypt data at rest and in transit
- **Regular Auditing**: Regularly audit permissions and access

### Performance
- **Right Sizing**: Use appropriate instance types and sizes
- **Auto Scaling**: Implement auto scaling for dynamic workloads
- **Content Delivery**: Use CloudFront for content delivery
- **Database Optimization**: Use appropriate database services and configurations
- **Caching**: Implement caching strategies

### Reliability
- **Multi-AZ Deployment**: Deploy across multiple availability zones
- **Backup Strategy**: Implement regular backup and recovery
- **Monitoring**: Monitor resources and applications
- **Disaster Recovery**: Implement disaster recovery plans
- **High Availability**: Design for high availability

## Troubleshooting

### Common Issues
```bash
# Check AWS CLI version
aws --version

# Check AWS CLI configuration
aws configure list

# Test AWS connectivity
aws sts get-caller-identity

# Check EC2 instance status
aws ec2 describe-instances --instance-ids i-1234567890abcdef0

# Check S3 bucket status
aws s3 ls s3://my-bucket-name

# Check Lambda function status
aws lambda get-function --function-name my-function

# Check CloudWatch logs
aws logs get-log-events \
  --log-group-name /aws/lambda/my-function \
  --log-stream-name 2023/01/01/[$LATEST]abcdef1234567890

# Check IAM permissions
aws iam get-user --user-name my-user
aws iam list-attached-user-policies --user-name my-user
```

### Debugging Tools
- **AWS CloudTrail**: Monitor and log API calls
- **AWS CloudWatch**: Monitor resources and applications
- **AWS X-Ray**: Debug and analyze applications
- **AWS Config**: Track resource changes
- **AWS Trusted Advisor**: Get optimization recommendations

## Resources

### Official Documentation
- [AWS Documentation](https://docs.aws.amazon.com/)
- [AWS CLI Command Reference](https://docs.aws.amazon.com/cli/latest/reference/)
- [AWS SDK Documentation](https://aws.amazon.com/documentation/sdk-for-java/)

### Learning Resources
- [AWS Training](https://aws.amazon.com/training/)
- [AWS Certification](https://aws.amazon.com/certification/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)

### Community
- [AWS Forums](https://forums.aws.amazon.com/)
- [AWS Stack Overflow](https://stackoverflow.com/questions/tagged/amazon-web-services)
- [AWS GitHub](https://github.com/aws)