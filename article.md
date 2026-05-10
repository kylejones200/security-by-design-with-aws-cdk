---
author: "Kyle Jones"
date_published: "October 18, 2024"
date_exported_from_medium: "November 10, 2025"
canonical_link: "https://medium.com/@kyle-t-jones/security-by-design-with-aws-cdk-5187da3a2c74"
---

# Security by design with AWS CDK Security is job zero when building any application. AWS CDK makes it
easier for developers to incorporate security best practices while...

### Security by design with AWS CDK
Security is job zero when building any application. AWS CDK makes it easier for developers to incorporate security best practices while building scalable systems. AWS offers various services and tools to enhance security, including IAM roles, encryption, logging, monitoring, and compliance tools. These tools reduce the risk of misconfigurations and security issues when using CDK.

### Implementing security into CDK projects
Security must be at the forefront of every resource definition and interaction when working with CDK. AWS CDK enables secure defaults for AWS resources while allowing developers to enforce strict security policies programmatically. This includes enforcing encryption, setting appropriate permissions, using the principle of least privilege, and ensuring that access control is granular and explicit.

**Secure Defaults in CDK Constructs**

Many AWS resources have security features that must be configured explicitly by default. AWS CDK provides high-level constructs (L2 constructs) that often come with secure defaults, such as enabling encryption for S3 buckets or enforcing HTTPS-only communication for API Gateway. While these defaults are a good starting point, you can further tighten the security posture by customizing them.

For example, let's define an S3 bucket with additional security controls, such as:

- Server-Side Encryption (SSE) using AWS KMS
- Bucket policies that restrict public access
- Access logging for auditing purposes

In this example:

```python
import * as cdk from '@aws-cdk/core';
import * as s3 from '@aws-cdk/aws-s3';
import * as iam from '@aws-cdk/aws-iam';
import * as kms from '@aws-cdk/aws-kms';

class SecureS3Stack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create a KMS key for encryption
    const encryptionKey = new kms.Key(this, 'S3EncryptionKey', {
      enableKeyRotation: true, // Rotating keys for added security
    });

    // Define a secure S3 bucket
    const secureBucket = new s3.Bucket(this, 'SecureBucket', {
      versioned: true, // Enable versioning for data recovery
      encryption: s3.BucketEncryption.KMS, // Use KMS encryption
      encryptionKey: encryptionKey, // Specify the KMS key to use
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL, // Block all public access
      serverAccessLogsBucketPrefix: 'logs/', // Enable server access logging
      removalPolicy: cdk.RemovalPolicy.RETAIN, // Retain the bucket when the stack is deleted
    });

    // Add a bucket policy to restrict access
    secureBucket.addToResourcePolicy(new iam.PolicyStatement({
      effect: iam.Effect.DENY,
      principals: [new iam.AnyPrincipal()],
      actions: ['s3:GetObject'],
      resources: [`${secureBucket.bucketArn}/*`],
      conditions: {
        'Bool': { 'aws:SecureTransport': 'false' } // Deny requests that are not using HTTPS
      }
    }));

    // Output the bucket name for reference
    new cdk.CfnOutput(this, 'BucketName', {
      value: secureBucket.bucketName,
      description: 'The name of the secure S3 bucket',
    });
  }
}

const app = new cdk.App();
new SecureS3Stack(app, 'SecureS3Stack');
```

We create an S3 bucket that is encrypted using a KMS key.

The bucket restricts all public access and enforces HTTPS for all data transfers.

Logging is enabled to track all access to the bucket.

Finally, we ensure the bucket remains even after the stack is deleted (essential for preserving data in production environments).

### Automating the creation of security resources
One of the main advantages of AWS CDK is that it allows developers to automate the provisioning of secure infrastructure. This reduces the likelihood of human error and ensures that security controls are consistently applied across environments.import \* as cdk from '@aws-cdk/core';

In this example:

```python
import * as lambda from '@aws-cdk/aws-lambda';
import * as iam from '@aws-cdk/aws-iam';

class SecureLambdaStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Define a Lambda execution role with limited permissions
    const lambdaExecutionRole = new iam.Role(this, 'LambdaExecutionRole', {
      assumedBy: new iam.ServicePrincipal('lambda.amazonaws.com'),
      managedPolicies: [
        iam.ManagedPolicy.fromAwsManagedPolicyName('service-role/AWSLambdaBasicExecutionRole')
      ]
    });

    // Grant additional permissions for reading from S3
    lambdaExecutionRole.addToPolicy(new iam.PolicyStatement({
      actions: ['s3:GetObject'],
      resources: ['arn:aws:s3:::my-secure-bucket/*']
    }));

    // Define a Lambda function that uses the secure role
    const mySecureLambda = new lambda.Function(this, 'MySecureLambda', {
      runtime: lambda.Runtime.NODEJS_14_X,
      code: lambda.Code.fromAsset('lambda'), // Path to your Lambda code
      handler: 'index.handler',
      role: lambdaExecutionRole // Attach the restricted role to the Lambda function
    });

    // Output the Lambda function name for reference
    new cdk.CfnOutput(this, 'LambdaName', {
      value: mySecureLambda.functionName,
      description: 'The name of the secure Lambda function',
    });
  }
}

const app = new cdk.App();
new SecureLambdaStack(app, 'SecureLambdaStack');
```

We create an IAM role restricted to only the permissions necessary for the Lambda function to execute and access specific S3 objects.

The AWSLambdaBasicExecutionRole allows the Lambda to log to CloudWatch.

We define additional permissions to allow access to a specific S3 bucket (by ARN) and no more.

**Automating Security Groups**

Security Groups (SGs) are essential for controlling traffic to and from your resources, such as EC2 instances, Lambda functions, or RDS databases. AWS CDK allows you to define security groups programmatically, ensuring that access control is tightly managed.

How to create a secure EC2 instance with a tightly configured security group:

In this example:

```python
import * as cdk from '@aws-cdk/core';
import * as ec2 from '@aws-cdk/aws-ec2';
import * as iam from '@aws-cdk/aws-iam';

class SecureEC2Stack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create a VPC
    const vpc = new ec2.Vpc(this, 'MyVpc', {
      maxAzs: 2, // Limit to 2 availability zones for cost management
    });

    // Create a security group that allows SSH access from a specific IP range
    const securityGroup = new ec2.SecurityGroup(this, 'SecurityGroup', {
      vpc,
      description: 'Allow SSH access from a specific IP range',
      allowAllOutbound: true, // Allow outbound traffic
    });

    // Restrict inbound SSH to a specific IP range
    securityGroup.addIngressRule(ec2.Peer.ipv4('203.0.113.0/24'), ec2.Port.tcp(22), 'Allow SSH access from IP range 203.0.113.0/24');

    // Define an EC2 instance with the security group
    const ec2Instance = new ec2.Instance(this, 'MyInstance', {
      instanceType: new ec2.InstanceType('t3.micro'),
      machineImage: new ec2.AmazonLinuxImage(),
      vpc,
      securityGroup,
      keyName: 'my-key-pair' // Specify the key pair for SSH access
    });

    // Output the public IP of the instance for easy access
    new cdk.CfnOutput(this, 'InstancePublicIp', {
      value: ec2Instance.instancePublicIp,
      description: 'The public IP of the EC2 instance',
    });
  }
}

const app = new cdk.App();
new SecureEC2Stack(app, 'SecureEC2Stack');
```

We create a VPC with two availability zones.

We define a security group that only allows inbound SSH traffic from a specific IP range (203.0.113.0/24) and restricts all other traffic.

An EC2 instance is launched within this secure group, and the public IP is output for quick access.

### Logging and monitoring security-related events in AWS
Proper logging and monitoring are critical components of any security strategy. AWS provides several services for logging and monitoring security-related events, such as CloudTrail, CloudWatch, and VPC Flow Logs. AWS CDK lets you easily configure these services to track and analyze security events.

Enabling CloudTrail for API Activity Logging

AWS CloudTrail records API calls made on your AWS account, including who made the call, when the call was made, and which resources were affected. You can use CloudTrail logs to detect unauthorized access, unusual activity, or changes to security configurations.

```python
import * as cdk from '@aws-cdk/core';
import * as cloudtrail from '@aws-cdk/aws-cloudtrail';
import * as s3 from '@aws-cdk/aws-s3';

class CloudTrailStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create an S3 bucket to store CloudTrail logs
    const trailBucket = new s3.Bucket(this, 'CloudTrailLogsBucket', {
      versioned: true,
      encryption: s3.BucketEncryption.S3_MANAGED,
      blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL, // Ensure the bucket is private
    });

    // Create a CloudTrail to monitor all API activity
    new cloudtrail.Trail(this, 'MyCloudTrail', {
      bucket: trailBucket,
      isMultiRegionTrail: true, // Enable multi-region logging for extra security
    });

    // Output the bucket name for reference
    new cdk.CfnOutput(this, 'CloudTrailBucketName', {
      value: trailBucket.bucketName,
      description: 'The name of the S3 bucket storing CloudTrail logs',
    });
  }
}

const app = new cdk.App();
new CloudTrailStack(app, 'CloudTrailStack');
```

We enable AWS CloudTrail to capture all API activity within the AWS account.

The logs are stored in an encrypted S3 bucket, and public access is completely blocked.

Multi-region logging ensures that API activity in other AWS regions is captured.

### Monitoring VPC traffic using VPC flow logs
VPC Flow Logs capture information about the IP traffic going to and from network interfaces in your VPC. This is useful for monitoring and troubleshooting network activity and can also help detect potential security threats, such as unexpected traffic patterns or unauthorized access attempts.

How to enable VPC Flow Logs for a VPC using AWS CDK:

```python
import * as cdk from '@aws-cdk/core';
import * as ec2 from '@aws-cdk/aws-ec2';
import * as logs from '@aws-cdk/aws-logs';

class VPCFlowLogsStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create a VPC
    const vpc = new ec2.Vpc(this, 'MyVpc', {
      maxAzs: 2,
    });

    // Create a CloudWatch Log Group to store VPC Flow Logs
    const flowLogsGroup = new logs.LogGroup(this, 'FlowLogsGroup', {
      retention: logs.RetentionDays.ONE_MONTH, // Retain logs for 1 month
    });

    // Enable VPC Flow Logs
    new ec2.FlowLog(this, 'VpcFlowLogs', {
      resourceType: ec2.FlowLogResourceType.fromVpc(vpc),
      destination: ec2.FlowLogDestination.toCloudWatchLogs(flowLogsGroup),
    });

    // Output the VPC ID for reference
    new cdk.CfnOutput(this, 'VpcId', {
      value: vpc.vpcId,
      description: 'The ID of the VPC with Flow Logs enabled',
    });
  }
}

const app = new cdk.App();
new VPCFlowLogsStack(app, 'VPCFlowLogsStack');
```

We enable VPC Flow Logs for a VPC and send the log data to CloudWatch Logs for further analysis.

Logs are stored in a CloudWatch Log Group with a retention period of one month.

### Integrating compliance tools
Compliance with industry standards and regulations is critical for many organizations, particularly in the healthcare, finance, and government sectors. AWS provides tools such as AWS Config, Amazon Macie, and AWS Security Hub to help manage compliance, which can be automated using AWS CDK.

**Using AWS Config for Compliance**

AWS Config tracks the configuration of AWS resources and ensures they comply with specified rules. You can define custom compliance rules or use AWS Config's managed rules to ensure that your resources meet security standards, such as that all S3 buckets are encrypted.

How to set up AWS Config using CDK:

```python
import * as cdk from '@aws-cdk/core';
import * as config from '@aws-cdk/aws-config';

class ConfigComplianceStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Create an AWS Config rule to check that S3 buckets are encrypted
    new config.ManagedRule(this, 'S3EncryptionRule', {
      identifier: 'S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED',
    });

    // Output a message indicating that AWS Config is monitoring
    new cdk.CfnOutput(this, 'ConfigMonitoringMessage', {
      value: 'AWS Config is monitoring S3 bucket encryption',
      description: 'AWS Config rule to monitor S3 encryption enabled',
    });
  }
}

const app = new cdk.App();
new ConfigComplianceStack(app, 'ConfigComplianceStack');
```

We set up AWS Config to monitor whether all S3 buckets enable server-side encryption.

The managed rule S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED ensures that any non-compliant buckets are flagged for remediation.

Using AWS Security Hub for Centralized Compliance

AWS Security Hub provides a comprehensive view of your security posture across AWS accounts. It integrates with AWS services such as AWS Config, GuardDuty, and Inspector to aggregate and analyze security findings, making it easier to maintain compliance.

How to enable AWS Security Hub using AWS CDK:

```python
import * as cdk from '@aws-cdk/core';
import * as securityhub from '@aws-cdk/aws-securityhub';

class SecurityHubStack extends cdk.Stack {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // Enable AWS Security Hub
    new securityhub.CfnHub(this, 'SecurityHub', {
      tags: {
        'Environment': 'Production'
      }
    });

    // Output a message indicating that Security Hub is enabled
    new cdk.CfnOutput(this, 'SecurityHubMessage', {
      value: 'AWS Security Hub is enabled for this account',
      description: 'Security Hub enabled',
    });
  }
}

const app = new cdk.App();
new SecurityHubStack(app, 'SecurityHubStack');
```

In this example, AWS Security Hub is enabled, and findings from various AWS security services will be aggregated into a centralized dashboard.

This helps provide a holistic view of your AWS security posture and ensures compliance with best practices.

#### **How to Ensure Your Infrastructure Follows Security Best Practices**
Security best practices with AWS CDK include encryption, access control, monitoring, and following the principle of least privilege.

Automating IAM Role Creation and Policies

When building cloud applications, managing access permissions is one of the most critical aspects of securing your resources. The principle of least privilege dictates that each service or user should have the minimum access necessary to perform their task.

AWS CDK allows you to create IAM roles and attach policies with specific permissions automatically.
