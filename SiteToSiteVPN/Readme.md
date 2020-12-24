### Two VPCs in different Availability Zones:
VPC-MainOffice with CIDR block 10.10.0.0/16 - VPC-BranchOffice with CIDR block 10.20.0.0/16
A public subnet in each VPC: - Subnet-MainOffice-Public with CIDR block 10.10.1.0/24 - Subnet-BranchOffice-Public with CIDR block 10.20.1.0/24
An internet gateway (IGW) in each VPC
Two route tables, each attached to the appropriate subnet:
Names: RT-MainOffice and RT-BranchOffice - Routes: Local, and 0.0.0.0/0 pointing to the IGW
### Create Two EC2 Instances
Create two new EC2 instances: one in VPC-MainOffice and one in VPC-BranchOffice.

#### EC2-MainOffice
AMI: Amazon Linux 2
Instance type: t2.medium
Network: VPC-MainOffice
Subnet: Subnet-MainOffice-Public
Auto-assign Public IP: Enable
Tags:
Key: Name; Value: EC2-MainOffice
Security group: Create a new security group:
Type: SSH; Source: My IP
Type: All TCP; Source: Custom, 10.20.0.0/16
Type: All UDP; Source: Custom, 10.20.0.0/16
Type: All ICMP - IPv4; Source: Custom, 10.20.0 0/16
Key pair: Create a new key pair:
Key pair name: Key-MainOffice
Download and save key pair.
#### EC2-BranchOffice
AMI: Amazon Linux 2
Instance type: t2.medium
Network: VPC-BranchOffice
Subnet: Subnet-BranchOffice-Public
Auto-assign Public IP: Enable
Tags:
Key: Name; Value: EC2-BranchOffice
Security group: Create a new security group:
Type: SSH; Source: My IP
Type: All TCP; Source: Custom, 10.10.0.0/16
Type: All UDP; Source: Custom, 10.10.0.0/16
Type: All ICMP - IPv4; Source: Custom, 10.10.0.0/16
Key pair: Create a new key pair:
Key pair name: Key-BranchOffice
Download and save key pair.
Once EC2-BranchOffice is created, disable the source/destination checks.

### Create Virtual Private Network Resources
Create the following resources:

Virtual private gateway attached to VPC-MainOffice
Name: VPG-MainBranch
### Customer gateway
Name: CGW-MainBranch
Routing: Static
IP Address: Public IP address of EC2-BranchOffice
Site-to-Site VPN connection
Name: VPN-MainBranch
Virtual Private Gateway: VPG-MainBranch
Customer Gateway: CGW-MainBranch
Routing Options: Static
IP Prefixes: 10.20.0.0/16
It may take several minutes for the VPN connection to move from <span style="color:gold">pending</span> to <span style="color:green">available</span>.

### Install and Configure Openswan
Connect via SSH to EC2-BranchOffice.

Install Openswan:

sudo su
yum install openswan
Configure /etc/ipsec.conf — if there is a # in front of this line, remove it:

include /etc/ipsec.d/*.conf
Configure /etc/sysctl.conf, adding these lines to the file:

net.ipv4.ip_forward = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.default.accept_source_route = 0
Configure /etc/ipsec.d/aws.conf, adding these lines to the file:

conn Tunnel1
  authby=secret
  auto=start
  left=%defaultroute
  leftid=<CUSTOMER_GATEWAY_IP_ADDRESS>
  right=<VIRTUAL_PRIVATE_GATEWAY_IP_ADDRESS>
  type=tunnel
  ikelifetime=8h
  keylife=1h
  phase2alg=aes128-sha1;modp1024
  ike=aes128-sha1;modp1024
  keyingtries=%forever
  keyexchange=ike
  leftsubnet=10.20.0.0/16
  rightsubnet=10.10.0.0/16
  dpddelay=10
  dpdtimeout=30
  dpdaction=restart_by_peer
Configure /etc/ipsec.d/aws.secrets, using this format:

<CUSTOMER_GATEWAY_IP_ADDRESS> <VIRTUAL_PRIVATE_GATEWAY_IP_ADDRESS>: PSK "<PRE_SHARED_KEY>"
For example:

50.100.25.6 75.80.65.12: PSK "34nkfwoe732ddf"
Restart the network service:

service network restart
Set the ipsec service to run automatically if the server restarts:

chkconfig ipsec on
Start the ipsec service:

service ipsec start
Check the status of the ipsec service:

service ipsec status

### Enable Route Propagation in Main Branch
Go to Route table main office and click on Route Propagation Tab
Click Edit Route Propagation and select Propagate check box.

###  Wait for Tunnel to be UP
Go Site-to-Site VPN Connection and Select Tunnel Details and wait for Status to be `UP`. It may take upto 5-10 min.

### Test Connectivity Across VPN
Connect via SSH to EC2-BranchOffice, and attempt to ping EC2-MainOffice:

ping <EC2-MainOffice_PRIVATE_IP_ADDRESS>
Connect via SSH to EC2-MainOffice, and attempt to ping EC2-BranchOffice:

ping <EC2-BranchOffice_PRIVATE_IP_ADDRESS>