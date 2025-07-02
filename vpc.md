# Experiment 6: AWS VPC Setup

## Overview
Create and configure a Virtual Private Cloud (VPC) with public and private subnets, including Internet Gateway, Route Tables, and bastion host setup for secure access to private resources.

## Architecture
- **VPC**: `12.0.0.0/16`
- **Public Subnet**: `12.0.0.0/20`
- **Private Subnet**: `12.0.16.0/20`
- **Internet Gateway**: For public internet access
- **Bastion Host**: EC2 in public subnet for accessing private subnet

## Step-by-Step Instructions

### Step 1: Create VPC
1. Go to AWS Console → Search for "VPC"
2. Click "Create VPC"
3. **Configure VPC Settings:**
   - Name tag: `Rollno-VPC`
   - IPv4 CIDR block: `12.0.0.0/16`
   - IPv6 CIDR block: No IPv6 CIDR block
   - Tenancy: Default
   - Tags: Key: `Your Name`, Value: `Rollno-VPC`
4. Click "Create VPC"

### Step 2: Create Internet Gateway
1. Go to Internet Gateways → Click "Create Internet Gateway"
2. **Settings:**
   - Name tag: `Rollno-IG`
3. Click "Create Internet Gateway"
4. **Attach to VPC:**
   - Select the created Internet Gateway
   - Actions → Attach to VPC
   - Select your VPC (`Rollno-VPC`)
   - Click "Attach Internet Gateway"

### Step 3: Create Subnets
1. Go to Subnets → Click "Create Subnet"
2. Select your VPC created earlier

**Public Subnet:**
- Subnet name: `Rollno-public-SB1`
- Availability Zone: `us-east-1a`
- IPv4 CIDR block: `12.0.0.0/20`

**Private Subnet:**
- Subnet name: `Rollno-private-SB2`
- Availability Zone: `us-east-1a`
- IPv4 CIDR block: `12.0.16.0/20`

3. Click "Create Subnet"

### Step 4: Create Route Tables

#### Public Route Table
1. Go to Route Tables → Click "Create Route Table"
2. **Settings:**
   - Name: `Rollno-Public-RT`
   - VPC: Select your VPC
3. Click "Create Route Table"
4. **Add Internet Route:**
   - Select the route table → Routes tab → Edit routes
   - Add route: Destination `0.0.0.0/0`, Target: Internet Gateway
   - Save changes
5. **Associate with Public Subnet:**
   - Subnet Associations tab → Edit subnet associations
   - Select `Rollno-public-SB1`
   - Save associations

#### Private Route Table
1. Click "Create Route Table"
2. **Settings:**
   - Name: `Rollno-private-RT`
   - VPC: Select your VPC
3. Click "Create Route Table"
4. **Associate with Private Subnet:**
   - Subnet Associations tab → Edit subnet associations
   - Select `Rollno-private-SB2`
   - Save associations

### Step 5: Create EC2 Instance in Public Subnet (Bastion Host)
1. Go to EC2 → Instances → Launch Instance
2. **Instance Configuration:**
   - Name: `EC1_Rollno_Public_subnet-Instance`
   - AMI: Ubuntu
   - Instance type: t2.micro
   - Key pair: Use existing or create new
3. **Network Settings:**
   - VPC: Select `Rollno-VPC`
   - Subnet: Select `Rollno-public-SB1`
   - Auto-assign public IP: Enable
4. **Security Group:**
   - Name: `Public-Rollno-ec2-SG`
   - Rules: SSH (Port 22) from Anywhere (0.0.0.0/0)
5. Launch Instance

### Step 6: Create EC2 Instance in Private Subnet
1. Launch another instance with these settings:
2. **Instance Configuration:**
   - Name: `EC2_Rollno_Private_subnet-Instance`
   - AMI: Ubuntu
   - Instance type: t2.micro
   - Key pair: Same as public instance
3. **Network Settings:**
   - VPC: Select `Rollno-VPC`
   - Subnet: Select `Rollno-private-SB2`
   - Auto-assign public IP: Disable
4. **Security Group:**
   - Name: `Private-Rollno-ec2-SG`
   - Rules: SSH (Port 22) from Custom (12.0.0.0/20)
5. Launch Instance

### Step 7: Connect to Private Instance via Bastion Host

#### Connect to Bastion Host
1. Use PuTTY or SSH to connect to the public instance
2. Copy your private key to the bastion host

#### Setup Private Key on Bastion Host
```bash
# Create private key file
scp -i /path/to/your-key.pem /path/to/local/file ubuntu@<EC2-PUBLIC-IP>:/home/ubuntu/



# Paste your private key content here, then save

# Set correct permissions
chmod 400 aws_privatekey_ec2.pem
```

#### Connect to Private Instance
```bash
# SSH to private instance from bastion host
ssh -i aws_privatekey_ec2.pem ubuntu@<PRIVATE_INSTANCE_IP>

# Replace <PRIVATE_INSTANCE_IP> with actual private IP of your private instance
```

## Verification
- Public instance should have internet access
- Private instance should only be accessible from bastion host
- Private instance should not have direct internet access

## Security Best Practices
- Keep private keys secure
- Use specific security group rules
- Regularly update and patch instances
- Monitor access logs

## Troubleshooting
- Ensure security groups allow necessary traffic
- Verify route table associations
- Check NACL settings if connectivity issues persist
- Confirm key pair permissions (chmod 400)

## Clean Up
To avoid charges:
1. Terminate EC2 instances
2. Delete VPC (this will delete associated subnets, route tables, etc.)
3. Delete Internet Gateway if not attached to VPC
