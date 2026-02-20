# AWS 3-Tier Web Application Architecture

<img width="971" height="601" alt="Tier3Topology" src="https://github.com/user-attachments/assets/dbd44c1f-5801-4bb7-9cf6-d7069d836d90" />


## 🏗️ Project Overview

This project implements a highly available, secure, and scalable **3-tier web application architecture** on AWS, demonstrating enterprise-grade cloud infrastructure design principles. The architecture separates concerns across presentation, application, and data layers while maintaining security through network segmentation and defense-in-depth strategies.

### Architecture Highlights

- ✅ **High Availability**: Multi-AZ deployment across 2 Availability Zones
- ✅ **Security First**: Defense-in-depth with multiple security group layers
- ✅ **Network Segmentation**: Public and private subnets with controlled access
- ✅ **Secure Access**: Bastion host pattern for administrative SSH access
- ✅ **Scalable Design**: Foundation ready for auto-scaling and load balancing
- ✅ **Cost Optimized**: Free tier eligible components for learning and testing

---

## 📋 Table of Contents

- [Architecture Components](#architecture-components)
- [Network Design](#network-design)
- [Security Model](#security-model)
- [Prerequisites](#prerequisites)
- [Deployment Guide](#deployment-guide)
- [Testing & Validation](#testing--validation)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Cost Considerations](#cost-considerations)
- [Future Enhancements](#future-enhancements)

---

## 🎯 Architecture Components

### **Tier 1: Web/Presentation Layer**
- **Web Server**: Amazon Linux 2 EC2 instance (t2.micro)
- **Software Stack**: Apache HTTP Server, PHP 7.2, MariaDB PHP extensions
- **Location**: Public subnet in Availability Zone 1
- **Purpose**: Serves web content and handles user requests
- **Access**: Public-facing via Internet Gateway

### **Tier 2: Application Layer**
- **App Server**: Amazon Linux 2 EC2 instance (t2.micro)
- **Software**: MariaDB client for database connectivity
- **Location**: Private subnet in Availability Zone 1
- **Purpose**: Business logic processing and database interaction
- **Access**: Private, accessible only from web tier and bastion host

### **Tier 3: Data Layer**
- **Database**: Amazon RDS MariaDB instance (db.t2.micro)
- **Configuration**: Single-AZ deployment with subnet group spanning 2 AZs
- **Location**: Private subnets in Availability Zones 1 & 2
- **Purpose**: Persistent data storage with managed backups
- **Access**: Isolated, accessible only from app tier and bastion host

### **Management Layer**
- **Bastion Host**: Amazon Linux 2 EC2 instance (t2.micro)
- **Location**: Public subnet in Availability Zone 1
- **Purpose**: Secure SSH gateway for administrative access
- **Pattern**: Industry-standard jump box/bastion pattern

---

## 🌐 Network Design

### VPC Configuration
```
VPC CIDR: 192.168.0.0/16
Region: us-east-1 (or your preferred region)
Availability Zones: 2
```

### Subnet Architecture

| Subnet Name | Type | CIDR Block | AZ | Purpose |
|-------------|------|------------|-----|---------|
| Public Subnet 1 | Public | 192.168.1.0/24 | AZ-1 | Web Server, Bastion Host |
| Private Subnet 1 | Private | 192.168.2.0/24 | AZ-1 | Application Server |
| Private Subnet 2 | Private | 192.168.3.0/24 | AZ-1 | Database Subnet Group |
| Private Subnet 3 | Private | 192.168.4.0/24 | AZ-2 | Database Subnet Group (HA) |

### Routing Configuration

**Public Route Table**
- Associated with: Public Subnet 1
- Routes:
  - `192.168.0.0/16` → Local (VPC internal traffic)
  - `0.0.0.0/0` → Internet Gateway (Internet access)

**Private Route Table**
- Associated with: All Private Subnets (1, 2, 3)
- Routes:
  - `192.168.0.0/16` → Local (VPC internal traffic)
  - `0.0.0.0/0` → NAT Gateway (Outbound internet for updates)

### Network Components

- **Internet Gateway**: Enables public internet access for public subnet resources
- **NAT Gateway**: Provides outbound internet access for private subnet resources
- **Elastic IP**: Static IP address assigned to NAT Gateway
- **DB Subnet Group**: Spans Private Subnets 2 & 3 for RDS high availability

---

## 🔒 Security Model

### Security Group Configuration

#### **Bastion Host Security Group**
| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| SSH | TCP | 22 | Your IP | Administrative SSH access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Web traffic (if needed) |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure web traffic (if needed) |
| MySQL/Aurora | TCP | 3306 | Database SG | Database connectivity testing |

#### **Web Server Security Group**
| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| SSH | TCP | 22 | Your IP | Direct admin access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Public web traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure public web traffic |
| ICMP | ICMP | All | App Server SG | Network connectivity testing |

#### **App Server Security Group**
| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| SSH | TCP | 22 | Bastion Host SG | Administrative access via bastion |
| ICMP | ICMP | All | Web Server SG | Network connectivity from web tier |
| MySQL/Aurora | TCP | 3306 | Database SG | Database responses |
| HTTP | TCP | 80 | 0.0.0.0/0 | Application traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure application traffic |

#### **Database Security Group**
| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| MySQL/Aurora | TCP | 3306 | App Server SG | Application database queries |
| MySQL/Aurora | TCP | 3306 | Bastion Host SG | Administrative database access |

### Security Best Practices Implemented

1. **Principle of Least Privilege**: Each security group only allows minimum required access
2. **Defense in Depth**: Multiple security layers (network ACLs, security groups, subnet isolation)
3. **Bastion Pattern**: No direct SSH to private instances; all access through bastion
4. **Private Database**: RDS instance not publicly accessible
5. **Security Group Chaining**: Resources reference each other's security groups, not IPs
6. **Restricted SSH**: SSH access limited to specific IPs (your IP)

---

## ✅ Prerequisites

### AWS Account Requirements
- Active AWS account with appropriate permissions
- Access to AWS Management Console
- IAM permissions for:
  - EC2 (create instances, security groups)
  - VPC (create VPC, subnets, route tables, gateways)
  - RDS (create database instances, subnet groups)

### Local Machine Requirements
- **SSH Client**: 
  - Windows: PuTTY or OpenSSH
  - macOS/Linux: Built-in OpenSSH
- **SCP/File Transfer**:
  - Windows: PuTTY SCP (pscp)
  - macOS/Linux: Built-in scp
- **Key Pair**: AWS EC2 key pair downloaded (.pem and .ppk for Windows)

### Knowledge Prerequisites
- Basic understanding of AWS services
- Familiarity with Linux command line
- Understanding of networking concepts (CIDR, routing, subnets)
- Basic knowledge of SSH and security groups

---

## 🚀 Deployment Guide

### Phase 1: VPC and Network Infrastructure

#### Step 1: Create VPC
```
1. Navigate to VPC Console → Your VPCs
2. Click "Create VPC"
3. Configuration:
   - Name: Tier3-VPC
   - IPv4 CIDR: 192.168.0.0/16
   - Tenancy: Default
4. Click "Create VPC"
```

#### Step 2: Create Subnets
Create four subnets with the following specifications:

**Public Subnet 1:**
```
- Name: Public-Subnet-1
- VPC: Tier3-VPC
- Availability Zone: us-east-1a (or your first AZ)
- CIDR: 192.168.1.0/24
```

**Private Subnet 1:**
```
- Name: Private-Subnet-1
- VPC: Tier3-VPC
- Availability Zone: us-east-1a (same as public)
- CIDR: 192.168.2.0/24
```

**Private Subnet 2:**
```
- Name: Private-Subnet-2
- VPC: Tier3-VPC
- Availability Zone: us-east-1a (same as above)
- CIDR: 192.168.3.0/24
```

**Private Subnet 3:**
```
- Name: Private-Subnet-3
- VPC: Tier3-VPC
- Availability Zone: us-east-1b (DIFFERENT AZ for HA)
- CIDR: 192.168.4.0/24
```

#### Step 3: Enable Auto-assign Public IP
```
1. Select Public-Subnet-1
2. Actions → Modify auto-assign IP settings
3. Enable "Auto-assign public IPv4 address"
4. Save
```

#### Step 4: Create Internet Gateway
```
1. VPC Console → Internet Gateways
2. Create Internet Gateway
   - Name: Tier3-IGW
3. Actions → Attach to VPC
   - Select: Tier3-VPC
```

#### Step 5: Allocate Elastic IP
```
1. VPC Console → Elastic IPs
2. Allocate Elastic IP address
   - Region: Verify correct region
   - (Optional) Name tag: Tier3-EIP
3. Allocate
```

#### Step 6: Create NAT Gateway
```
1. VPC Console → NAT Gateways
2. Create NAT Gateway
   - Name: Tier3-NAT-GW
   - Subnet: Public-Subnet-1
   - Elastic IP: Select the EIP created above
3. Create NAT Gateway
4. Wait for status to become "Available" (~5 minutes)
```

#### Step 7: Create Route Tables

**Public Route Table:**
```
1. VPC Console → Route Tables → Create route table
   - Name: Public-RT
   - VPC: Tier3-VPC
2. Select Public-RT → Routes tab → Edit routes
   - Add route: 0.0.0.0/0 → Internet Gateway (Tier3-IGW)
3. Subnet Associations tab → Edit subnet associations
   - Select: Public-Subnet-1
   - Save associations
```

**Private Route Table:**
```
1. Create route table
   - Name: Private-RT
   - VPC: Tier3-VPC
2. Select Private-RT → Routes tab → Edit routes
   - Add route: 0.0.0.0/0 → NAT Gateway (Tier3-NAT-GW)
3. Subnet Associations tab → Edit subnet associations
   - Select: Private-Subnet-1, Private-Subnet-2, Private-Subnet-3
   - Save associations
```

#### Step 8: Create Security Groups

**Bastion Host Security Group:**
```
Name: Bastion-SG
Description: Security group for bastion host
VPC: Tier3-VPC

Inbound Rules:
- Type: SSH, Port: 22, Source: [Your IP]/32
- Type: HTTP, Port: 80, Source: 0.0.0.0/0
- Type: HTTPS, Port: 443, Source: 0.0.0.0/0
```

**Web Server Security Group:**
```
Name: WebServer-SG
Description: Security group for web server
VPC: Tier3-VPC

Inbound Rules:
- Type: SSH, Port: 22, Source: [Your IP]/32
- Type: HTTP, Port: 80, Source: 0.0.0.0/0
- Type: HTTPS, Port: 443, Source: 0.0.0.0/0
```

**App Server Security Group:**
```
Name: AppServer-SG
Description: Security group for application server
VPC: Tier3-VPC

Inbound Rules:
- Type: SSH, Port: 22, Source: Bastion-SG
- Type: All ICMP-IPv4, Source: WebServer-SG
```

**Database Security Group:**
```
Name: Database-SG
Description: Security group for database
VPC: Tier3-VPC

Inbound Rules:
- Type: MySQL/Aurora, Port: 3306, Source: AppServer-SG
- Type: MySQL/Aurora, Port: 3306, Source: Bastion-SG
```

**Update Security Groups (Cross-references):**
```
1. Edit Bastion-SG inbound rules:
   - Add: MySQL/Aurora, Port: 3306, Source: Database-SG

2. Edit WebServer-SG inbound rules:
   - Add: All ICMP-IPv4, Source: AppServer-SG

3. Edit AppServer-SG inbound rules:
   - Add: MySQL/Aurora, Port: 3306, Source: Database-SG
   - Add: HTTP, Port: 80, Source: 0.0.0.0/0
   - Add: HTTPS, Port: 443, Source: 0.0.0.0/0
```

---

### Phase 2: Deploy EC2 Instances

#### Step 9: Launch Bastion Host
```
1. EC2 Console → Launch Instance
2. Configuration:
   - Name: Bastion-Host
   - AMI: Amazon Linux 2 AMI (HVM)
   - Instance type: t2.micro
   - Key pair: Select existing or create new
   
3. Network settings:
   - VPC: Tier3-VPC
   - Subnet: Public-Subnet-1
   - Auto-assign public IP: Enable
   - Security group: Bastion-SG
   
4. Storage: Default (8 GB gp2)
5. Launch instance
```

#### Step 10: Launch Web Server
```
1. EC2 Console → Launch Instance
2. Configuration:
   - Name: Web-Server
   - AMI: Amazon Linux 2 AMI (HVM)
   - Instance type: t2.micro
   - Key pair: Same as bastion
   
3. Network settings:
   - VPC: Tier3-VPC
   - Subnet: Public-Subnet-1
   - Auto-assign public IP: Enable
   - Security group: WebServer-SG
   
4. Advanced details → User data:
```

```bash
#!/bin/bash
sudo yum update -y
sudo amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemctl enable httpd
```

```
5. Storage: Default
6. Launch instance
```

#### Step 11: Launch App Server
```
1. EC2 Console → Launch Instance
2. Configuration:
   - Name: App-Server
   - AMI: Amazon Linux 2 AMI (HVM)
   - Instance type: t2.micro
   - Key pair: Same as previous
   
3. Network settings:
   - VPC: Tier3-VPC
   - Subnet: Private-Subnet-1
   - Auto-assign public IP: Disable
   - Security group: AppServer-SG
   
4. Advanced details → User data:
```

```bash
#!/bin/bash
sudo yum install -y mariadb-server
sudo service mariadb start
```

```
5. Storage: Default
6. Launch instance
```

---

### Phase 3: Deploy RDS Database

#### Step 12: Create DB Subnet Group
```
1. RDS Console → Subnet groups → Create DB subnet group
2. Configuration:
   - Name: tier3-db-subnet-group
   - Description: Subnet group for Tier 3 database
   - VPC: Tier3-VPC
   - Availability Zones: Select both AZs used
   - Subnets: Select Private-Subnet-2 and Private-Subnet-3
3. Create
```

#### Step 13: Create RDS Instance
```
1. RDS Console → Databases → Create database
2. Configuration:
   
   Engine options:
   - Engine type: MariaDB
   - Version: Latest (default)
   - Templates: Free tier
   
   Settings:
   - DB instance identifier: tier3-database
   - Master username: root
   - Master password: Re:Start!9
   - Confirm password: Re:Start!9
   
   DB instance class:
   - Burstable classes: db.t2.micro
   
   Storage:
   - Storage type: General Purpose SSD (gp2)
   - Allocated storage: 20 GB (default)
   
   Connectivity:
   - VPC: Tier3-VPC
   - DB subnet group: tier3-db-subnet-group
   - Public access: No
   - VPC security group: Choose existing
     - Remove default
     - Add Database-SG
   - Availability Zone: us-east-1a (your first AZ)
   
   Additional configuration:
   - Initial database name: mydb
   - Backup retention: 0 days (disable for faster creation)
   - Encryption: Disable (not needed for testing)
   
3. Create database
4. Wait for status "Available" (~10-15 minutes)
```

---

### Phase 4: Testing & Validation

#### Step 14: Upload SSH Keys to Bastion Host

**For Windows Users (PowerShell):**
```powershell
pscp -scp -P 22 -i '.\Downloads\labsuser.ppk' '.\Downloads\labsuser.pem' ec2-user@<BASTION-PUBLIC-IP>:/home/ec2-user
```

**For macOS/Linux Users:**
```bash
chmod 400 ~/Downloads/labsuser.pem
scp -i ~/Downloads/labsuser.pem ~/Downloads/labsuser.pem ec2-user@<BASTION-PUBLIC-IP>:/home/ec2-user
```

*Replace `<BASTION-PUBLIC-IP>` with your bastion host's public IP address*

#### Step 15: SSH into Bastion Host
```bash
# Windows (PuTTY) - Use GUI with .ppk file

# Windows (OpenSSH) or macOS/Linux
ssh -i labsuser.pem ec2-user@<BASTION-PUBLIC-IP>
```

#### Step 16: Verify Key Upload
```bash
ls
# Should see: labsuser.pem
```

#### Step 17: Connect to App Server via Bastion
```bash
# Set correct permissions on key
chmod 400 labsuser.pem

# SSH into app server
ssh -i labsuser.pem ec2-user@<APP-SERVER-PRIVATE-IP>
```

*Replace `<APP-SERVER-PRIVATE-IP>` with your app server's private IP*

#### Step 18: Test Web Server Connectivity
```bash
# From app server, ping web server
ping <WEB-SERVER-PRIVATE-IP>
# Press Ctrl+C to stop after successful pings
```

#### Step 19: Test Database Connectivity
```bash
# From app server, connect to database
mysql --user=root --password='Re:Start!9' --host=<DATABASE-ENDPOINT>

# Once connected, verify database
SHOW DATABASES;
# Should see: mydb

# Exit MySQL
EXIT;
```

*Replace `<DATABASE-ENDPOINT>` with your RDS endpoint (found in RDS Console)*

#### Step 20: Test Web Server (Optional)
```bash
# In browser, navigate to:
http://<WEB-SERVER-PUBLIC-IP>

# Should see Apache test page or "It works!"
```

---

## 🔍 Testing & Validation

### Connectivity Tests

**Test Matrix:**

| From | To | Method | Expected Result |
|------|-----|--------|-----------------|
| Your Computer | Bastion Host | SSH | Connected |
| Bastion Host | App Server | SSH | Connected |
| App Server | Web Server | Ping (ICMP) | Reply received |
| App Server | Database | MySQL client | Connected, see databases |
| Internet | Web Server | HTTP | Apache page loads |

### Verification Checklist

- [ ] VPC created with correct CIDR
- [ ] All 4 subnets created in correct AZs
- [ ] Internet Gateway attached to VPC
- [ ] NAT Gateway in available state
- [ ] Route tables configured correctly
- [ ] All security groups created with proper rules
- [ ] Bastion host accessible via SSH
- [ ] Web server accessible via HTTP
- [ ] Web server serving Apache page
- [ ] Can SSH to app server via bastion
- [ ] App server can ping web server
- [ ] App server can connect to database
- [ ] Database showing 'mydb' database

---

## 📚 Best Practices

### Security
1. **Use IAM Roles**: Attach IAM roles to EC2 instances instead of storing credentials
2. **Rotate Credentials**: Regularly update database passwords and SSH keys
3. **Enable MFA**: Use Multi-Factor Authentication on AWS account
4. **CloudTrail**: Enable AWS CloudTrail for audit logging
5. **VPC Flow Logs**: Enable for network traffic analysis
6. **Secrets Manager**: Store database credentials in AWS Secrets Manager

### High Availability
1. **Multi-AZ RDS**: Enable Multi-AZ deployment for production databases
2. **Auto Scaling**: Implement Auto Scaling groups for web and app tiers
3. **Load Balancer**: Add Application Load Balancer for traffic distribution
4. **Health Checks**: Configure health checks for all tiers
5. **Backup Strategy**: Enable automated backups with appropriate retention

### Performance
1. **CloudFront**: Add CDN for static content delivery
2. **ElastiCache**: Implement caching layer (Redis/Memcached)
3. **RDS Read Replicas**: Add read replicas for read-heavy workloads
4. **EBS Optimization**: Use provisioned IOPS for high-performance workloads

### Cost Optimization
1. **Right-sizing**: Monitor and adjust instance types based on usage
2. **Reserved Instances**: Purchase RIs for predictable workloads
3. **Savings Plans**: Consider compute savings plans
4. **S3 for Static Content**: Move static assets to S3
5. **Scheduled Scaling**: Scale down during off-peak hours

### Monitoring
1. **CloudWatch**: Set up CloudWatch dashboards and alarms
2. **SNS Notifications**: Configure alerts for critical events
3. **Log Aggregation**: Use CloudWatch Logs for centralized logging
4. **Third-party Tools**: Consider Datadog, New Relic, or similar

---

## 🔧 Troubleshooting

### Common Issues

#### Cannot SSH to Bastion Host
```
Symptoms: Connection timeout or refused
Solutions:
1. Verify security group allows SSH from your IP
2. Check if instance has public IP assigned
3. Verify Internet Gateway is attached
4. Confirm public route table has IGW route
5. Check NACL settings (should be default allow all)
6. Verify you're using correct key pair
```

#### Cannot Access Private Instances
```
Symptoms: Timeout when SSHing from bastion
Solutions:
1. Verify private instance security group allows SSH from Bastion-SG
2. Confirm private route table has NAT Gateway route
3. Check NAT Gateway is in "Available" state
4. Verify correct private IP address
5. Ensure key file has correct permissions (chmod 400)
```

#### Database Connection Fails
```
Symptoms: "Can't connect to MySQL server"
Solutions:
1. Verify Database-SG allows MySQL from AppServer-SG
2. Check RDS instance status is "Available"
3. Confirm correct endpoint address (no https://, no trailing port)
4. Verify credentials (username: root, password: Re:Start!9)
5. Check DB subnet group configuration
6. Ensure RDS is in same VPC
```

#### Web Server Not Responding
```
Symptoms: Cannot load Apache page
Solutions:
1. Verify httpd service is running: systemctl status httpd
2. Check security group allows HTTP from 0.0.0.0/0
3. Confirm public IP is assigned
4. Review user data logs: /var/log/cloud-init-output.log
5. Manually restart httpd: sudo systemctl restart httpd
```

#### NAT Gateway Not Working
```
Symptoms: Private instances can't reach internet
Solutions:
1. Verify NAT Gateway is in "Available" state
2. Check Elastic IP is associated
3. Confirm private route table has 0.0.0.0/0 → NAT Gateway
4. Verify NAT Gateway is in public subnet
5. Check source/destination check is enabled on NAT Gateway
```

---

## 💰 Cost Considerations

### Monthly Cost Estimate (US East 1)

| Resource | Specification | Monthly Cost* |
|----------|--------------|---------------|
| EC2 Bastion (t2.micro) | 750 hrs free tier | $0 - $8.50 |
| EC2 Web Server (t2.micro) | 750 hrs free tier | $0 - $8.50 |
| EC2 App Server (t2.micro) | Shared free tier | $8.50 |
| RDS MariaDB (db.t2.micro) | 750 hrs free tier | $0 - $13.14 |
| NAT Gateway | Data processing + hourly | $33 - $45 |
| Elastic IP (in use) | Free when attached | $0 |
| Data Transfer | First 1GB free | $0 - $9 |
| **Total Estimated** | | **~$50 - $90/month** |

*Costs vary by region and usage. Free tier applies for first 12 months of AWS account.*

### Cost Optimization Tips

**For Learning/Testing:**
1. Stop instances when not in use
2. Delete NAT Gateway when not testing (biggest cost)
3. Use VPC endpoints instead of NAT Gateway where possible
4. Delete database snapshots regularly
5. Remove unused Elastic IPs

**For Production:**
1. Use Reserved Instances (up to 72% savings)
2. Implement Auto Scaling to match demand
3. Use Spot Instances for fault-tolerant workloads
4. Consider AWS Transit Gateway instead of multiple NAT Gateways
5. Enable Cost Explorer and set budgets

---

## 🚀 Future Enhancements

### Scalability Improvements
- [ ] Add Application Load Balancer for web tier
- [ ] Implement Auto Scaling Groups for web and app tiers
- [ ] Enable Multi-AZ deployment for RDS
- [ ] Add RDS Read Replicas for read scalability
- [ ] Implement ElastiCache (Redis/Memcached) for caching

### Security Enhancements
- [ ] Enable AWS WAF on Application Load Balancer
- [ ] Implement AWS Shield for DDoS protection
- [ ] Add AWS Config for compliance monitoring
- [ ] Enable GuardDuty for threat detection
- [ ] Implement AWS Systems Manager Session Manager (eliminate bastion)
- [ ] Add AWS Secrets Manager for credential management
- [ ] Enable VPC Flow Logs for traffic analysis

### High Availability
- [ ] Deploy across 3+ Availability Zones
- [ ] Implement Route 53 health checks and failover
- [ ] Add S3 for static content with versioning
- [ ] Configure automated backups and disaster recovery
- [ ] Implement cross-region replication

### Monitoring & Logging
- [ ] Set up CloudWatch dashboards
- [ ] Configure CloudWatch Alarms and SNS notifications
- [ ] Enable AWS CloudTrail for audit logs
- [ ] Implement centralized logging with CloudWatch Logs
- [ ] Add X-Ray for distributed tracing

### DevOps & Automation
- [ ] Infrastructure as Code (Terraform or CloudFormation)
- [ ] CI/CD pipeline with CodePipeline/CodeDeploy
- [ ] Container orchestration with ECS/EKS
- [ ] Blue-green deployments
- [ ] Automated testing and validation

### Performance
- [ ] Add CloudFront CDN
- [ ] Implement database connection pooling
- [ ] Optimize security group rules
- [ ] Use Provisioned IOPS for database
- [ ] Enable Enhanced Monitoring for RDS

---

## 📖 Additional Resources

### AWS Documentation
- [VPC User Guide](https://docs.aws.amazon.com/vpc/)
- [EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [RDS User Guide](https://docs.aws.amazon.com/rds/)
- [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
- [NAT Gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html)

### Architecture Patterns
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [3-Tier Architecture Best Practices](https://aws.amazon.com/getting-started/hands-on/deploy-wordpress-with-amazon-rds/)

### Learning Resources
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS Training and Certification](https://aws.amazon.com/training/)
- [AWS re:Start Program](https://aws.amazon.com/training/restart/)

---

## 📝 License

This project is intended for educational purposes. Feel free to use, modify, and distribute for learning and demonstration purposes.

---

## 👤 Author

**Your Name**
- GitHub: [@yourusername](https://github.com/nvimah)
- LinkedIn: [Your Name](https://www.linkedin.com/in/sharon-wainaina-556206395/)
- Blog: [yourblog.com](https://medium.com/@wainainasharon15)

---

## 🙏 Acknowledgments

- AWS re:Start Program for the foundational architecture
- AWS Documentation for best practices
- Cloud computing community for continuous learning

---

## 📞 Support

If you have questions or run into issues:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review AWS documentation
3. Open an issue on GitHub
4. Contact via LinkedIn

---

**⭐ If you found this project helpful, please give it a star!**

**🔄 Contributions welcome! Feel free to submit pull requests.**

---

*Last Updated: February 2026*
