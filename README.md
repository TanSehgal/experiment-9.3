# Experiment 9.3: Deploy Full Stack App on AWS with Load Balancing

## 🎯 Objective
Gain hands-on experience deploying a full stack application to AWS and configure load balancing for scalability and high availability. Master EC2, Application Load Balancer (ALB), VPC, and security group configuration.

## 📦 Project Structure
```
experiment-9.3/
├── backend/
│   ├── server.js           # Express server
│   ├── package.json
│   └── .env.example
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── App.js
│   │   └── index.js
│   └── package.json
├── aws-config/
│   ├── ec2-user-data.sh    # EC2 initialization script
│   ├── alb-config.json     # Load balancer configuration
│   └── security-groups.md   # Security group rules
├── deployment/
│   ├── deploy-backend.sh
│   ├── deploy-frontend.sh
│   └── setup-aws.md
└── README.md
```

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                    Internet                          │
└──────────────────┬──────────────────────────────────┘
                   │
         ┌─────────▼──────────┐
         │   Route 53 (DNS)   │  (Optional)
         └─────────┬──────────┘
                   │
         ┌─────────▼──────────┐
         │  Application Load  │
         │    Balancer (ALB)  │
         └────┬─────────┬─────┘
              │         │
     ┌────────▼───┐ ┌──▼─────────┐
     │  EC2       │ │  EC2       │
     │  Backend 1 │ │  Backend 2 │
     │  (Node.js) │ │  (Node.js) │
     └────────────┘ └────────────┘
              │         │
         ┌────▼─────────▼────┐
         │   EC2 Frontend    │
         │   (React/Nginx)   │
         └───────────────────┘
```

## ☁️ AWS Services Used

### 1. **Amazon EC2** (Elastic Compute Cloud)
- Virtual servers for hosting application
- Multiple instances for high availability
- Auto-scaling capabilities

### 2. **Application Load Balancer (ALB)**
- Distributes traffic across multiple EC2 instances
- Health checks for fault tolerance
- Path-based routing

### 3. **VPC** (Virtual Private Cloud)
- Isolated network environment
- Public and private subnets
- Security and access control

### 4. **Security Groups**
- Virtual firewalls for EC2 instances
- Control inbound and outbound traffic
- Port-specific rules

### 5. **Route 53** (Optional)
- DNS service for domain management
- Route traffic to ALB

## 🚀 Deployment Steps

### Phase 1: AWS Account Setup

1. **Create AWS Account**
   - Sign up at https://aws.amazon.com
   - Complete billing setup
   - Enable MFA for security

2. **Configure IAM User**
   ```bash
   # Create IAM user with permissions:
   - AmazonEC2FullAccess
   - ElasticLoadBalancingFullAccess
   - AmazonVPCFullAccess
   ```

3. **Install AWS CLI**
   ```bash
   # Install AWS CLI
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   unzip awscliv2.zip
   sudo ./aws/install
   
   # Configure credentials
   aws configure
   ```

### Phase 2: VPC and Network Setup

1. **Create VPC**
   ```bash
   # CIDR: 10.0.0.0/16
   aws ec2 create-vpc --cidr-block 10.0.0.0/16
   ```

2. **Create Subnets**
   ```bash
   # Public Subnet 1 (us-east-1a)
   aws ec2 create-subnet --vpc-id <VPC_ID> --cidr-block 10.0.1.0/24 --availability-zone us-east-1a
   
   # Public Subnet 2 (us-east-1b)  
   aws ec2 create-subnet --vpc-id <VPC_ID> --cidr-block 10.0.2.0/24 --availability-zone us-east-1b
   ```

3. **Create Internet Gateway**
   ```bash
   aws ec2 create-internet-gateway
   aws ec2 attach-internet-gateway --vpc-id <VPC_ID> --internet-gateway-id <IGW_ID>
   ```

### Phase 3: Security Groups Configuration

1. **ALB Security Group**
   ```bash
   # Allow HTTP (80) and HTTPS (443) from internet
   Inbound Rules:
   - Type: HTTP, Port: 80, Source: 0.0.0.0/0
   - Type: HTTPS, Port: 443, Source: 0.0.0.0/0
   ```

2. **Backend EC2 Security Group**
   ```bash
   # Allow traffic from ALB only
   Inbound Rules:
   - Type: Custom TCP, Port: 5000, Source: ALB Security Group
   - Type: SSH, Port: 22, Source: Your IP
   ```

3. **Frontend EC2 Security Group**
   ```bash
   Inbound Rules:
   - Type: HTTP, Port: 80, Source: 0.0.0.0/0
   - Type: SSH, Port: 22, Source: Your IP
   ```

### Phase 4: EC2 Instance Setup

1. **Launch Backend EC2 Instances (2x)**
   ```bash
   # Instance specifications:
   - AMI: Amazon Linux 2
   - Instance Type: t2.micro (Free tier)
   - Key Pair: Create new or use existing
   - Security Group: Backend SG
   - User Data: backend-setup.sh
   ```

2. **Launch Frontend EC2 Instance**
   ```bash
   # Instance specifications:
   - AMI: Amazon Linux 2
   - Instance Type: t2.micro
   - Security Group: Frontend SG
   - User Data: frontend-setup.sh
   ```

### Phase 5: Application Deployment

1. **Deploy Backend**
   ```bash
   # SSH into backend instances
   ssh -i your-key.pem ec2-user@<BACKEND_IP>
   
   # Install Node.js
   curl -sL https://rpm.nodesource.com/setup_18.x | sudo bash -
   sudo yum install -y nodejs
   
   # Clone and setup
   git clone <YOUR_REPO>
   cd backend
   npm install
   npm start
   ```

2. **Deploy Frontend**
   ```bash
   # SSH into frontend instance
   ssh -i your-key.pem ec2-user@<FRONTEND_IP>
   
   # Install Node.js and Nginx
   sudo yum install -y nodejs nginx
   
   # Build React app
   cd frontend
   npm install
   npm run build
   
   # Configure Nginx
   sudo cp build/* /usr/share/nginx/html/
   sudo systemctl start nginx
   ```

### Phase 6: Load Balancer Configuration

1. **Create Target Group**
   ```bash
   aws elbv2 create-target-group \
     --name backend-targets \
     --protocol HTTP \
     --port 5000 \
     --vpc-id <VPC_ID> \
     --health-check-path /health
   ```

2. **Register Targets**
   ```bash
   aws elbv2 register-targets \
     --target-group-arn <TG_ARN> \
     --targets Id=<INSTANCE_ID_1> Id=<INSTANCE_ID_2>
   ```

3. **Create Application Load Balancer**
   ```bash
   aws elbv2 create-load-balancer \
     --name my-app-alb \
     --subnets <SUBNET_ID_1> <SUBNET_ID_2> \
     --security-groups <ALB_SG_ID>
   ```

4. **Create Listener**
   ```bash
   aws elbv2 create-listener \
     --load-balancer-arn <ALB_ARN> \
     --protocol HTTP \
     --port 80 \
     --default-actions Type=forward,TargetGroupArn=<TG_ARN>
   ```

## 🧪 Testing the Deployment

### 1. Health Check
```bash
# Check target health
aws elbv2 describe-target-health --target-group-arn <TG_ARN>
```

### 2. Access Application
```bash
# Get ALB DNS name
aws elbv2 describe-load-balancers --names my-app-alb

# Access in browser
http://<ALB_DNS_NAME>
```

### 3. Test Load Balancing
```bash
# Multiple requests should be distributed
for i in {1..10}; do
  curl http://<ALB_DNS_NAME>/api/health
  echo ""
done
```

### 4. Test Fault Tolerance
```bash
# Stop one backend instance
aws ec2 stop-instances --instance-ids <INSTANCE_ID>

# Verify app still works (ALB routes to healthy instance)
curl http://<ALB_DNS_NAME>
```

## 📊 Monitoring and Maintenance

### CloudWatch Metrics
- Request count
- Target response time
- Healthy/Unhealthy host count
- HTTP error rates

### ALB Access Logs
```bash
# Enable access logs
aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn <ALB_ARN> \
  --attributes Key=access_logs.s3.enabled,Value=true
```

## 💰 Cost Optimization

### Free Tier Usage
- 750 hours EC2 t2.micro per month
- 15 GB data transfer out
- 1 GB ALB usage

### Cost-Saving Tips
1. Use Reserved Instances for production
2. Stop instances when not needed
3. Use Auto Scaling to match demand
4. Monitor usage with AWS Cost Explorer

## 🔒 Security Best Practices

1. **Never expose SSH to 0.0.0.0/0**
2. **Use IAM roles instead of access keys**
3. **Enable encryption at rest and in transit**
4. **Regular security group audits**
5. **Implement WAF for additional protection**
6. **Use HTTPS with SSL/TLS certificates**

## 📝 Troubleshooting

### ALB returns 502 Bad Gateway
- Check backend instances are running
- Verify security group allows ALB → Backend
- Check health check endpoint

### Cannot SSH into instances
- Verify security group allows SSH from your IP
- Check key pair permissions (chmod 400)
- Ensure instance is in public subnet with IGW

### High latency
- Add more backend instances
- Use CloudFront CDN
- Optimize application code
- Check database queries

## ✨ Key Features Achieved

✅ **Scalability** - Horizontal scaling with multiple EC2 instances
✅ **High Availability** - ALB distributes traffic across AZs
✅ **Fault Tolerance** - Automatic failover to healthy instances  
✅ **Load Distribution** - Even traffic distribution via ALB
✅ **Health Monitoring** - Continuous health checks
✅ **Security** - VPC isolation and security groups

## 📚 Learning Outcomes

1. AWS EC2 instance management
2. VPC and subnet configuration
3. Security group design
4. Application Load Balancer setup
5. Target group and listener configuration
6. Health check implementation
7. Multi-AZ deployment strategies
8. AWS CLI usage
9. Infrastructure as Code concepts
10. Cloud cost optimization

## 👨‍💻 Author
TanSehgal

## 📝 License
This project is for educational purposes.

## 🔗 Useful Resources

- [AWS EC2 Documentation](https://docs.aws.amazon.com/ec2/)
- [ALB Guide](https://docs.aws.amazon.com/elasticloadbalancing/)
- [VPC Best Practices](https://docs.aws.amazon.com/vpc/)
- [AWS Free Tier](https://aws.amazon.com/free/)

---

**Happy Deploying! ☁️✨**
