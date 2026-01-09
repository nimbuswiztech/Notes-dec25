# assignment

### AWS VPC Lab Session: Step-by-Step Practice Guide

Follow these detailed instructions to replicate the AWS VPC class lab, building a functional multi-subnet VPC with routing, gateways, and EC2 instances.

{% stepper %}
{% step %}
### Preparation

* Log into AWS Console with your credentials.
* Ensure your AWS region is set (preferably the default from your organization or "N. Virginia" for practice).
{% endstep %}

{% step %}
### Create the VPC

* Go to Services > VPC.
* Click "Create VPC".
  * Name: `svm-vpc-01`
  * IPv4 CIDR block: `10.0.0.0/27` (provides 32 IPs: 10.0.0.0–10.0.0.31)
  * Leave IPv6 CIDR empty (unless needed)
  * Tenancy: Default
* Click **Create VPC**.
{% endstep %}

{% step %}
### Add Subnets

* Navigate to "Subnets" > Create subnet.
* VPC: Select `svm-vpc-01`.

For Public Subnet:

* Subnet name: `Nimbus-public`
* Availability Zone: Pick one (e.g., us-east-1a)
* IPv4 CIDR block: `10.0.0.0/28` (IPs 10.0.0.0–10.0.0.15)
* Click **Add subnet**.

For Private Subnet:

* Subnet name: `Nimbus-private`
* Availability Zone: Pick another or same
* IPv4 CIDR block: `10.0.0.16/28` (IPs 10.0.0.16–10.0.0.31)
* Click **Create subnet**.
{% endstep %}

{% step %}
### Create Route Tables

* Go to "Route Tables" > Create route table.
  * Name: `rd-public`
  * VPC: `svm-vpc-01`
* Click **Create route table**.

Repeat for private subnet:

* Name: `rd-private`
* VPC: `svm-vpc-01`
* Click **Create route table**.

You should now have two custom route tables.
{% endstep %}

{% step %}
### Set Up Internet Gateway (IGW)

* Navigate to "Internet Gateways" > Create internet gateway.
  * Name: `Nimbus-icw`
* Click **Create internet gateway**.
* Select the gateway and click **Actions > Attach to VPC**.
  * Select `svm-vpc-01`.
* Click **Attach internet gateway**.
{% endstep %}

{% step %}
### Allocate Elastic IP (EIP)

* Go to "Elastic IPs" > Allocate Elastic IP address.
  * Click **Allocate** (leave settings at default).

You now have a static EIP for NAT Gateway use.
{% endstep %}

{% step %}
### Create NAT Gateway

* Navigate to "NAT Gateways" > Create NAT gateway.
  * Subnet: Select `Nimbus-public`.
  * Elastic IP: Choose the one just allocated.
  * Name: e.g., `windows-nat`
* Click **Create NAT gateway**.
* Wait for the NAT Gateway status to become "Available".
{% endstep %}

{% step %}
### Associate Subnets to Route Tables

* Go to "Route Tables".

Select `rd-public`:

* Click **Subnet associations > Edit subnet associations**.
* Check `Nimbus-public`.
* Save.

Select `rd-private`:

* Click **Subnet associations > Edit subnet associations**.
* Check `Nimbus-private`.
* Save.
{% endstep %}

{% step %}
### Configure Route Table Routes

Public Route Table (`rd-public`):

* Select `rd-public`.
* Under Routes > Edit routes > Add route:
  * Destination: `0.0.0.0/0`
  * Target: Internet Gateway > Select `Nimbus-icw`
* Save changes.

Private Route Table (`rd-private`):

* Select `rd-private`.
* Under Routes > Edit routes > Add route:
  * Destination: `0.0.0.0/0`
  * Target: NAT Gateway > Select your NAT
* Save changes.
{% endstep %}

{% step %}
### (Optional) Review Security Groups

* Default security group allows all outbound, but restricts inbound. For practice, you can adjust or create a new group as needed:
  * Allow SSH (port 22) or RDP (port 3389) from your own IP to connect to EC2.
{% endstep %}

{% step %}
### Launch EC2 Instances

* Go to "EC2 > Instances" > Launch Instance.

For the public instance:

* Name: `public-instance`
* AMI: Amazon Linux 2 or Windows (as preferred)
* Subnet: `Nimbus-public`
* Auto-assign Public IP: Enable (for public instance)
* Security Group: Select one allowing SSH/RDP from your IP
* Click **Launch instance**

For the private instance:

* Name: `private-instance`
* Subnet: `Nimbus-private`
* Auto-assign Public IP: Disable (will not be internet-accessible directly)
* Security Group: As above
{% endstep %}

{% step %}
### Test Connectivity

* SSH/RDP into your public instance using the public IP.
* From the public instance, attempt to access the private instance (private IP).
* For outbound internet:
  * From the private instance, use CLI (`ping`, `curl`, etc.) to verify outbound connectivity (internet works via NAT Gateway).
* Private instance is not reachable directly from the internet (secured).
{% endstep %}

{% step %}
### Clean Up

* Delete Elastic IP if no longer needed (otherwise, charges may apply).
* Terminate EC2 instances and remove resources after practice.
{% endstep %}
{% endstepper %}

### Best Practices

* Always associate NAT Gateway with a public subnet.
* Never keep unused Elastic IP addresses.
* Use least privilege on security group rules.
* Map subnets deliberately to correct route tables for proper traffic flow.
