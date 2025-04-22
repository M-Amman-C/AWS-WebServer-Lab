# Title: AWS - Launch and Configure Amazon Linux Web Server via SSM
**Description:** Learn to launch and configure a web instance with Amazon EC2

---

## Step 1: Create a Security Group and IAM Role

**Note:** Make sure all steps are performed in region US-EAST-1

- Create a security group titled “web-server1-sg”
- Allow HTTP and HTTPS as inbound rules
- Navigate to the IAM tab and go to “Roles”
- Create a Role “AmazonSSMManagedInstanceCore” with policy for SSM 

### Solution Box:

**Security Group:**
- Navigate to EC2 Console → Security Groups → Create security group
- Name: web-server1-sg  
- Description: Allow HTTP/HTTPS only

**Inbound Rules:**
- HTTP (port 80) – Source: 0.0.0.0/0  
- HTTPS (port 443) – Source: 0.0.0.0/0

**Outbound Rule:** Leave default (allow all)  
- Click “Create security group”

**IAM Role:**
- Navigate to the IAM tab and go to “Roles”
- Click on “Create Role”
- Keep trust entity as AWS Service and select EC2 under Use Case
- Click on “EC2 Role for AWS Systems Manager”
- Press Next
- Skip permissions tab
- Enter “SSMEC2Role” as Role Name and create the role

---

## Step 2: Launch Amazon Linux EC2 Instance

**Launch an EC2 Instance with Following configuration:**
- Name tag: web-server1
- Architecture: arm
- Size: t4g.micro
- Security group: web-server1-sg

### Solution Box:

- Go to EC2 > Instances > Launch Instance  
- Set:
  - Name tag: web-server1
  - AMI: Amazon Linux 2023
  - Keep architecture as arm
  - Under Instance type, click on “t4g.micro”
  - Key pair: Select “Proceed without a key pair”
- Under the Network Settings, click on edit:
  - choose default VPC
  - Make sure Auto-assign public IP is set to enabled
  - Click on existing security group and select the security group created in first step
- Under Advanced Details:
  - Select IAM instance profile as the one created in last step (SSMEC2Role)
- Click Launch Instance
- Wait for instance to launch successfully and come in running state

---

## Step 3: Connect to Instance using SSM

**Use Systems Manager service to connect to instance through SSM**

### Solution Box:

- Go to Systems Manager > Session Manager
- Click Start Session
- Choose the instance web-server1 and click Start

---

## Step 4: Install Apache Web Server (httpd)

- Switch to root user
- Install the httpd package
- Configure it to display the web page index for http requests
- Access the page from public ip of your instance

### Solution Box:

- Install the “httpd” package
- Start and enable the service
- Create an index in “/var/www/html”
- Change the content to “Hello from CloudKida Web Server”
- Return to the AWS Management Console
- Navigate to the EC2 page, and find the instance’s public IP
- Enter “http://<public ip>/” in your browser to access the page

**Commands:**
```bash
sudo -i
# Install package
sudo yum update -y
sudo yum install -y httpd

# Start service
sudo systemctl start httpd
sudo systemctl enable httpd

# Create index file
echo "<h1>Hello from CloudKida Web Server</h1>" | sudo tee /var/www/html/index.html
```

## Step 5: Cleanup all the resources

Delete all the resources created in this session  
- EC2 Instance  
- Security group  
- IAM Role  

### Solution Box:

**EC2 Instance**  
- Navigate to the EC2 Page and go to instances section  
- Select the instance with tag “web-server1”  
- Click on instance state and then press terminate instance  

**Security Group:**  
- In the left navigation pane, find and press Security Groups  
- Select “web-server1-sg” group and click delete under Actions tab  

**IAM Role:**  
- Go to IAM > Roles section  
- Enter “SSMEC2Role” in search and select it  
- Click on Delete  

## Hardening Script:

```bash
mkfs.ext4 /dev/xvdb
mkfs.ext4 /dev/xvdc
mkfs.ext4 /dev/xvdd

mount /dev/xvdb /mnt
rm -rf /mnt/*
rsync -avrH /home/ /mnt/
umount /mnt/

mount /dev/xvdc /mnt
rm -rf /mnt/*
rsync -avrH /var/ /mnt/
umount /mnt/

mount /dev/xvdd /mnt
rm -rf /mnt/*
rsync -avrH /tmp/ /mnt/
umount /mnt/

mkswap /dev/xvde
swapon /dev/xvde

echo /dev/xvdb /home ext4 defaults 0 0 >> /etc/fstab
echo /dev/xvdc /var ext4 defaults 0 0 >> /etc/fstab
echo /dev/xvdd /tmp ext4 defaults 0 0 >> /etc/fstab
echo /dev/xvde swap swap defaults 0 0 >> /etc/fstab

mount -a
systemctl daemon-reload
```
