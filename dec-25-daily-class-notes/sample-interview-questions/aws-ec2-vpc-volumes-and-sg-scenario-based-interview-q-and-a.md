# AWS EC2, VPC, Volumes \&SG Scenario-Based Interview Q\&A

## &#x20;**AWS EC2, VPC, Volumes & Security Groups Scenario-Based Interview Q\&A**

***

### **BASIC LEVEL (1–7)**

#### **1) An EC2 instance is unreachable after launch. How do you troubleshoot?**

**Answer:**\
First I check the **Security Group inbound rules** to confirm SSH (22) or application port is open from my IP.\
Next, I verify the **Subnet route table** has a route to an **Internet Gateway**.\
Then I check if the instance has a **public IP**.\
If still unreachable, I review the **Network ACL** and confirm OS-level firewall isn’t blocking access.\
This step-by-step network path validation usually isolates the issue quickly.

***

#### **2) You need to launch a web server accessible from the internet. What components do you configure?**

**Answer:**\
I launch the EC2 in a **public subnet**, attach an **Internet Gateway** to the VPC, assign a **public IP**, configure the **route table** with 0.0.0.0/0 → IGW, and open **port 80/443** in the Security Group.\
This ensures internet-facing connectivity.

***

#### **3) Your EC2 disk is full. What do you do without data loss?**

**Answer:**\
I modify the **EBS volume size** from the console, then extend the filesystem inside the instance using `resize2fs` or `xfs_growfs`.\
Since EBS supports online resizing, no downtime is required.

***

#### **4) What happens if an EC2 instance stops? Does data remain?**

**Answer:**\
Instance store data is lost, but **EBS-backed root volumes persist**.\
In real projects, we always use EBS-backed instances for data persistence.

***

#### **5) How do you restrict SSH access to EC2?**

**Answer:**\
In Security Group inbound rules, I allow **port 22 only from my office IP range**, not from 0.0.0.0/0.\
This is a standard security hardening practice.

***

#### **6) Difference between Security Group and NACL in real use?**

**Answer:**\
Security Groups are **stateful** and attached to instances.\
NACLs are **stateless** and work at subnet level.\
In real environments, we control application access using SGs and use NACLs only for extra compliance-level subnet restrictions.

***

#### **7) How do you assign a fixed public IP to an EC2?**

**Answer:**\
I allocate an **Elastic IP** and associate it with the instance.\
This ensures IP remains same even after stop/start.

***

### **INTERMEDIATE LEVEL (8–14)**

#### **8) Your private EC2 instances need internet access for updates but must not be publicly accessible. How do you design this?**

**Answer:**\
I place them in a **private subnet** and configure a **NAT Gateway** in a public subnet.\
The private subnet route table points internet-bound traffic to NAT.\
This allows outbound internet access while keeping instances private.

***

#### **9) You need high availability for EC2 application. What setup do you use?**

**Answer:**\
I deploy instances in **multiple AZs**, put them in an **Auto Scaling Group**, and place an **Application Load Balancer** in front.\
This ensures fault tolerance and automatic scaling.

***

#### **10) One EC2 instance must access another on port 3306 securely. How do you configure SG?**

**Answer:**\
Instead of allowing IP-based rules, I configure the **DB Security Group** to allow inbound 3306 **from the App Security Group**.\
This is more secure and scalable.

***

#### **11) Your instance suddenly lost connectivity after modifying route table. What could be wrong?**

**Answer:**\
Most likely the default route (0.0.0.0/0) to Internet Gateway or NAT Gateway was removed or misconfigured.\
Restoring the correct route table entry fixes it.

***

#### **12) You need encrypted storage for EC2. How do you implement?**

**Answer:**\
I create **EBS volumes encrypted using KMS keys** and attach them to EC2.\
In production, encryption at rest is mandatory for compliance.

***

#### **13) How do you migrate an EC2 instance to another AZ?**

**Answer:**\
I take an **AMI image**, then launch a new instance from that AMI in the target AZ.\
Since AMIs are AZ-independent, migration is simple.

***

#### **14) You must allow only HTTPS traffic to EC2. What steps?**

**Answer:**\
I configure SG inbound rule to allow **443 only**, install SSL certificates on ALB or instance, and redirect HTTP → HTTPS.\
This enforces secure communication.

***

### **ADVANCED LEVEL (15–20)**

#### **15) Production EC2 CPU suddenly spikes to 100%. How do you handle?**

**Answer:**\
First I check CloudWatch metrics and logs.\
If load is valid traffic, I scale using Auto Scaling.\
If abnormal, I check running processes inside instance.\
In real-time, we set **scaling policies** to handle such spikes automatically.

***

#### **16) How do you provide EC2 access to S3 without storing keys?**

**Answer:**\
I attach an **IAM Role** to the EC2 instance with required S3 permissions.\
This avoids hardcoding access keys.

***

#### **17) How do you implement a Bastion Host setup?**

**Answer:**\
I launch a small EC2 in public subnet as **Bastion Host**, allow SSH only from office IP, and allow SSH from Bastion SG to private EC2 SG.\
This provides controlled admin access.

***

#### **18) Your EC2 instances need to communicate across VPCs. How do you enable?**

**Answer:**\
I configure **VPC Peering** or **Transit Gateway**, update route tables, and allow traffic in SG/NACL.\
Transit Gateway is preferred for multiple VPCs.

***

#### **19) How do you perform zero-downtime deployment on EC2?**

**Answer:**\
I use **Auto Scaling Groups** with **rolling updates** or **Blue-Green deployment** behind ALB.\
New instances come up, pass health checks, then old ones are terminated.

***

#### **20) How do you secure EC2 metadata service against SSRF attacks?**

**Answer:**\
I enforce **IMDSv2 only** on instances.\
This prevents unauthorized metadata token access — a best practice in hardened environments.
