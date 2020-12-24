### Configure AWS Transit Gateway
Navigate to VPC > Transit Gateways.
Click Create Transit Gateway.
For Name tag and Description, enter "labtransitgw".
For Amazon side ASN, enter "65065".
Leave DNS and ECMP support enabled.
Leave the default route table association and propagation.
Click Create Transit Gateway. It may take up to five minutes to enter an available state.
Attach Three VPCs to Transit Gateway and Create the Appropriate VPC Routes
In the left-hand menu, select Transit Gateway Attachments.

### VPC1
Click Create Transit Gateway Attachment, and set the following values:
Transit Gateway ID: labtransitgw
Attachment type: VPC
Attachment name tag: VPC1
VPC ID: VPC1
Subnet ID: PublicSubnet1
Click Create attachment > Close.
### VPC2
Click Create Transit Gateway Attachment, and set the following values:
Transit Gateway ID: labtransitgw
Attachment type: VPC
Attachment name tag: VPC2
VPC ID: VPC2
Subnet ID: PublicSubnet2
Click Create attachment > Close.
### VPC3
Click Create Transit Gateway Attachment, and set the following values:
Transit Gateway ID: labtransitgw
Attachment type: VPC
Attachment name tag: VPC3
VPC ID: VPC3
Subnet ID: PublicSubnet3
Click Create attachment > Close.
Give it a few minutes for all three transit gateway attachments to finish being created.
### Create the Appropriate Routes on the VPCs
In the left-hand menu, select Route Tables.

#### Public1-RT (with Routes to VPC2 and VPC3)
Select Public1-RT.
Click the Routes tab.
Click Edit routes.
Click Add route.
Set Destination as "10.2.0.0/16".
Set Target as Transit Gateway, and select labtransitgw.
Click Add route.
Set Destination as "10.3.0.0/16".
Set Target as Transit Gateway, and select labtransitgw.
Click Save routes.
#### Public2-RT (with Routes to VPC1 and VPC3)
Select Public2-RT.
Click the Routes tab.
Click Edit routes.
Click Add route.
Set Destination as "10.1.0.0/16".
Set Target as Transit Gateway, and select labtransitgw.
Click Add route.
Set Destination as "10.3.0.0/16".
Set Target as Transit Gateway, and select labtransitgw.
Click Save routes.
#### Public3-RT (with Routes to VPC1 and VPC2)
Select Public3-RT.
Click the Routes tab.
Click Edit routes.
Click Add route.
Set Destination as "10.1.0.0/16".
Set Target as Transit Gateway, and select labtransitgw.
Click Add route.
Set Destination as "10.2.0.0/16".
Set Target as Transit Gateway, and select labtransitgw.
Click Save routes.
### Validate Connectivity from Terminal to all VPCs
Note: All EC2 instance credentials and IP addresses referenced in this section are provided on the lab page.

Copy the public IP of EC2 INSTANCE1 listed on the lab page.

Open Terminal.

Log in to the instance via SSH:

ssh cloud_user@<INSTANCE1_PUBLIC_IP>
Answer yes, and then enter the password.

Ping the public IP address of EC2 INSTANCE2:

ping <INSTANCE2_PUBLIC_IP>
We should get a reply.

Exit the ping by pressing Ctrl+C.

Ping the private IP address for EC2 INSTANCE3:

ping <INSTANCE3_PRIVATE_IP>
It won't work this time.

Exit the ping by pressing Ctrl+C.

### Troubleshoot
Edit Inbound and Outbound Rules
In the AWS Management Console, navigate to EC2 > Instances.
Select Instance3.
In the Description section on the page, note that it's in the private subnet, not the public subnet we associated it with earlier.
Navigate to VPC > Network ACLs.
Select Private3-NACL.
Click the Inbound Rules tab.
Click Edit inbound rules.
Give it a Rule # of "100", and leave the other defaults.
Click Add Rule.
Give it a Rule # of "200".
For Type, choose All ICMP - IPv4.
Click Save.
Click the Outbound Rules tab.
Click Edit outbound rules.
Click Add Rule.
Give it a Rule # of "100", and leave the other defaults.
Click Add Rule.
Give it a Rule # of "200".
For Type, choose All ICMP - IPv4.
Click Save.
In the terminal, try to ping the private IP address for INSTANCE3 again:

ping <INSTANCE3_PRIVATE_IP>
It still won't work.

Exit the ping by pressing Ctrl+C.

### Modify Transit Gateway Attachment
In the AWS console, in the left-hand menu, select Transit Gateway Attachments.
Select VPC3.
Select Actions > Delete.
In the dialog, click Delete.
Give it a few minutes to finish being deleted. (You won't be able to create a new VPC3 transit gateway attachment until it's deleted, since they have the same name.)
Click Create Transit Gateway Attachment, and set the following values:
Transit Gateway ID: labtransitgw
Attachment type: VPC
Attachment name tag: VPC3
VPC ID: VPC3
Subnet ID: PrivateSubnet3
Click Create attachment > Close.
Give it a few minutes to finish being created.
Modify Routes
In the left-hand menu, select Route Tables.
Select Private3-RT.
Click the Routes tab.
Click Edit routes.
Click Add route.
Set Destination as "10.1.0.0/16".
Set Target as Transit Gateway, and select labtransitgw.
Click Add route.
Set Destination as "10.2.0.0/16".
Set Target as Transit Gateway, and select labtransitgw.
Click Save routes.
In the terminal, try to ping the private IP address for INSTANCE3 again:

ping <INSTANCE3_PRIVATE_IP>
This time, we should get a reply.

Exit the ping by pressing Ctrl+C.

Ping the private IP address of INSTANCE2:

ping <INSTANCE2_PRIVATE_IP>
We should get a reply.

Exit the ping by pressing Ctrl+C.