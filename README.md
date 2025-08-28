# CloudLaunch AWS Assessment

## Overview

CloudLaunch is a lightweight platform demonstrating AWS fundamentals including
S3 static website hosting, IAM access controls, and VPC network design. This
repository contains the documentation for the complete AWS infrastructure setup
showcasing secure, scalable cloud architecture.

## 🚀 Live Demo

- **Static Website (S3)**:
  [http://aliu-cloudlaunch-site-bucket.s3-website-eu-west-1.amazonaws.com/](http://aliu-cloudlaunch-site-bucket.s3-website-eu-west-1.amazonaws.com/)
- **CloudFront Distribution (HTTPS)**:
  [https://d32nq6wl9xcyh5.cloudfront.net](https://d32nq6wl9xcyh5.cloudfront.net)

## Architecture Overview

### Task 1: S3 Static Website Hosting + IAM

- **Public Website Hosting** on S3 with custom domain
- **Secure File Storage** with granular IAM permissions
- **CloudFront CDN** for HTTPS and global caching
- **Limited IAM User** with precise access controls

### Task 2: VPC Network Design

- **Secure VPC** with proper network segmentation
- **Multi-tier Architecture** (Public, Application, Database subnets)
- **Security Groups** with least-privilege access
- **Internet Gateway** with controlled routing

---

## 📋 Task 1 Implementation: S3 + IAM

### S3 Buckets Created

#### 1. `cloudlaunch-site-bucket` (Public Website)

- **Purpose**: Hosts the static website
- **Access**: Publicly accessible (read-only)
- **Features**:
  - Static website hosting enabled
  - Custom index.html with CloudLaunch branding
  - Bucket policy allows public GetObject access

#### 2. `cloudlaunch-private-bucket` (Private Storage)

- **Purpose**: Secure file storage for internal documents
- **Access**: IAM user only (Get/Put permissions)
- **Security**: No public access, IAM-controlled

#### 3. `cloudlaunch-visible-only-bucket` (List-Only Access)

- **Purpose**: Demonstrates granular IAM permissions
- **Access**: IAM user can list bucket but cannot access contents
- **Use Case**: Visibility without content access

### IAM User Configuration

**User**: `cloudlaunch-user`

**Custom IAM Policy**: `CloudLaunchUserPolicy`

```json
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "AllowS3ConsoleAccess",
			"Effect": "Allow",
			"Action": ["s3:ListAllMyBuckets", "s3:GetBucketLocation"],
			"Resource": "*"
		},
		{
			"Sid": "ListSpecificBuckets",
			"Effect": "Allow",
			"Action": "s3:ListBucket",
			"Resource": [
				"arn:aws:s3:::aliu-cloudlaunch-site-bucket",
				"arn:aws:s3:::aliu-cloudlaunch-private-bucket",
				"arn:aws:s3:::aliu-cloudlaunch-visible-only-bucket"
			]
		},
		{
			"Sid": "ReadOnlyAccessToSiteBucket",
			"Effect": "Allow",
			"Action": "s3:GetObject",
			"Resource": ["arn:aws:s3:::aliu-cloudlaunch-site-bucket/*"]
		},
		{
			"Sid": "ReadWriteAccessToPrivateBucket",
			"Effect": "Allow",
			"Action": ["s3:GetObject", "s3:PutObject"],
			"Resource": ["arn:aws:s3:::aliu-cloudlaunch-private-bucket/*"]
		},
		{
			"Sid": "VPCReadOnlyAccess",
			"Effect": "Allow",
			"Action": [
				"ec2:DescribeVpcs",
				"ec2:DescribeSubnets",
				"ec2:DescribeRouteTables",
				"ec2:DescribeSecurityGroups",
				"ec2:DescribeInternetGateways",
				"ec2:DescribeAvailabilityZones",
				"ec2:DescribeRegions",
				"ec2:DescribeDhcpOptions",
				"ec2:DescribeNetworkAcls",
				"ec2:DescribeAccountAttributes",
				"ec2:DescribeInstances",
				"ec2:DescribeImages",
				"ec2:DescribeKeyPairs",
				"ec2:DescribeTags"
			],
			"Resource": "*"
		}
	]
}
```

### CloudFront Configuration (Bonus)

- **Distribution**: Global CDN for enhanced performance
- **SSL/TLS**: Automatic HTTPS with AWS-managed certificates
- **Caching**: Optimized cache policies for static content
- **Security**: HTTP to HTTPS redirect enabled

---

## 🌐 Task 2 Implementation: VPC Network Design

### VPC Architecture

**VPC Name**: `cloudlaunch-vpc`  
**CIDR Block**: `10.0.0.0/16`

### Subnet Design

| Subnet Name                 | Type    | CIDR Block    | Purpose                         | Internet Access |
| --------------------------- | ------- | ------------- | ------------------------------- | --------------- |
| `cloudlaunch-public-subnet` | Public  | `10.0.1.0/24` | Load balancers, public services | ✅ Via IGW      |
| `cloudlaunch-app-subnet`    | Private | `10.0.2.0/24` | Application servers             | ❌ Private      |
| `cloudlaunch-db-subnet`     | Private | `10.0.3.0/28` | Database services               | ❌ Private      |

### Internet Gateway

- **Name**: `cloudlaunch-igw`
- **Attached to**: `cloudlaunch-vpc`
- **Purpose**: Provides internet access to public subnet

### Route Tables

#### Public Route Table (`cloudlaunch-public-rt`)

- **Associated with**: `cloudlaunch-public-subnet`
- **Routes**:
  - `10.0.0.0/16` → Local (VPC traffic)
  - `0.0.0.0/0` → `cloudlaunch-igw` (Internet traffic)

#### Application Route Table (`cloudlaunch-app-rt`)

- **Associated with**: `cloudlaunch-app-subnet`
- **Routes**:
  - `10.0.0.0/16` → Local (VPC traffic only)
  - No internet route (fully private)

#### Database Route Table (`cloudlaunch-db-rt`)

- **Associated with**: `cloudlaunch-db-subnet`
- **Routes**:
  - `10.0.0.0/16` → Local (VPC traffic only)
  - No internet route (fully private)

### Security Groups

#### Application Security Group (`cloudlaunch-app-sg`)

- **Purpose**: Controls access to application tier
- **Inbound Rules**:
  - HTTP (80) from VPC CIDR (`10.0.0.0/16`)
- **Outbound Rules**: Default (all traffic allowed)

#### Database Security Group (`cloudlaunch-db-sg`)

- **Purpose**: Controls access to database tier
- **Inbound Rules**:
  - MySQL (3306) from App Subnet only (`10.0.2.0/24`)
- **Outbound Rules**: Default (all traffic allowed)

---

## 🔐 Access Control Summary

### IAM User Permissions (`cloudlaunch-user`)

| Resource                          | List | Read | Write | Delete |
| --------------------------------- | ---- | ---- | ----- | ------ |
| `cloudlaunch-site-bucket`         | ✅   | ✅   | ❌    | ❌     |
| `cloudlaunch-private-bucket`      | ✅   | ✅   | ✅    | ❌     |
| `cloudlaunch-visible-only-bucket` | ✅   | ❌   | ❌    | ❌     |
| VPC Resources                     | ✅   | ✅   | ❌    | ❌     |

---

## 📊 Security Best Practices Implemented

### Network Security

- **Network Segmentation**: 3-tier architecture (Public, App, Database)
- **Least Privilege Routing**: Private subnets have no internet access
- **Security Groups**: Restrictive inbound rules with specific port access

### IAM Security

- **Principle of Least Privilege**: User has minimum required permissions
- **Granular S3 Access**: Different permissions per bucket
- **No Delete Permissions**: Prevents accidental data loss
- **Resource-Specific Policies**: Access limited to specific buckets only

### Web Security

- **HTTPS Enforcement**: CloudFront redirects HTTP to HTTPS
- **Static Website Hosting**: No server-side vulnerabilities
- **Public Access Controls**: Only website content is publicly accessible

---

## 🚀 Deployment Instructions

### Prerequisites

- AWS Account with appropriate permissions
- AWS CLI configured (optional)
- Basic understanding of AWS services

### Step 1: Create S3 Infrastructure

1. Create three S3 buckets as specified
2. Configure static website hosting on site bucket
3. Upload website content
4. Set up bucket policies for public access

### Step 2: Set Up IAM

1. Create `cloudlaunch-user`
2. Create custom policy `CloudLaunchUserPolicy`
3. Attach policy to user
4. Generate access keys (if needed)

### Step 3: Deploy VPC Infrastructure

1. Create VPC with specified CIDR
2. Create subnets in appropriate CIDR ranges
3. Set up Internet Gateway and routing
4. Configure security groups with proper rules

### Step 4: (Optional) CloudFront Setup

1. Create CloudFront distribution
2. Configure origin to point to S3 website endpoint
3. Enable HTTPS redirect
4. Wait for distribution deployment

---

## 🧪 Testing & Validation

### Website Testing

- [ ] Static website loads correctly via S3 endpoint
- [ ] HTTPS access works via CloudFront (if configured)
- [ ] All pages and assets load properly

### IAM Testing

- [ ] `cloudlaunch-user` can access S3 console
- [ ] Can view all three buckets in list
- [ ] Can download from site and private buckets
- [ ] Can upload to private bucket only
- [ ] Cannot delete from any bucket
- [ ] Cannot access contents of visible-only bucket
- [ ] Can view VPC resources (read-only)

### VPC Testing

- [ ] All subnets created with correct CIDR blocks
- [ ] Route tables properly associated
- [ ] Security groups have correct rules
- [ ] Internet Gateway attached and routing configured

---

## 📈 Future Enhancements

### Potential Improvements

- **Custom Domain**: Route 53 for custom domain name
- **SSL Certificate**: ACM certificate for custom domain
- **WAF Integration**: Web Application Firewall for security
- **CloudWatch**: Monitoring and logging setup
- **CI/CD Pipeline**: Automated deployment pipeline

### Scalability Considerations

- **Multi-AZ Deployment**: Distribute across availability zones
- **Auto Scaling Groups**: For application tier scaling
- **RDS Multi-AZ**: For database high availability
- **NAT Gateway**: For private subnet internet access

---

## 📝 Assessment Completion Checklist

### Task 1: S3 + IAM ✅

- [x] Created 3 S3 buckets with appropriate access levels
- [x] Enabled static website hosting
- [x] Configured IAM user with custom policy
- [x] Implemented CloudFront distribution (Bonus)
- [x] Tested all access permissions

### Task 2: VPC Design ✅

- [x] Created VPC with proper CIDR block
- [x] Implemented 3-tier subnet architecture
- [x] Configured Internet Gateway and routing
- [x] Set up security groups with least privilege
- [x] Granted VPC read access to IAM user

### Documentation ✅

- [x] Comprehensive README with all details
- [x] JSON policy documentation
- [x] Architecture diagrams and explanations
- [x] Testing and validation procedures

---

_CloudLaunch Assessment - Demonstrating AWS Fundamentals with Security Best
Practices_
