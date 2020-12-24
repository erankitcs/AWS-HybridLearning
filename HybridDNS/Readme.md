### Configuring Hybrid DNS with AWS

You are a network engineer who has been tasked with configuring DNS resolution between an on-premises network and an AWS VPC. The best practice method for achieving this is with Route 53 Resolver inbound and outbound endpoints, which provide a highly available, fault-tolerant DNS name resolution between VPCs and external networks.

We will need to create and configure the Resolver endpoints in the AWS Management Console, as well as a rule to indicate where outbound DNS queries should be forwarded. We will also need to add a forwarder zone for our VPC network in the provided BIND DNS server.

There are two intranet endpoints, one for each endpoint:

inet.intranet.onprem
inet.intranet.vpc
When the lab is launched, it should be possible to curl each endpoint from the bastion in the respective networks, while cross-network name resolution will fail.

Log in to the live AWS environment using the credentials provided. Make sure you're in the N. Virginia (us-east-1) region throughout the lab.

### Create Route 53 Resolver Endpoints: Part 1 — Inbound Endpoint
In the AWS Management Console, navigate to Route 53 Resolver.
Click Configure Endpoints.
Make sure you are in the N. Virginia (us-east-1) region.
Leave Inbound and outbound selected.
Click Next.
#### Create the Inbound Endpoint
Enter "DNSLabInbound" as the endpoint name.
Select the VPC ending with AWSVPC.
In a new browser tab, navigate to VPC > Security Groups.
Look for a security group named EndpointSecurityGroup, and note the last four digits of its group ID.
In the Route 53 Resolver Endpoints browser tab, select the security group ending in the last four digits you just noted.
#### Configure IP Address 1
In the VPC browser tab, click Subnets in the left-hand menu.
Look for the subnet named AWSPubSubnet1, and note its Availability Zone.
In the Route 53 Resolver Endpoints browser tab, select the Availability Zone for the subnet we just identified.
Select the only available subnet for that Availability Zone.
Ensure Use an IP address that is selected automatically is selected.
#### Configure IP Address 2
In the VPC subnets browser tab, look for the subnet named AWSPubSubnet2, and note its Availability Zone.
In the Route 53 Resolver Endpoints browser tab, select the Availability Zone for the subnet we just identified.
Select the only available subnet for that Availability Zone.
Ensure Use an IP address that is selected automatically is selected.
Click Next.

### Create Route 53 Resolver Endpoints: Part 2 — Outbound Endpoint
#### Create the Inbound Endpoint
Enter "DNSLabOutbound" as the endpoint name.
Select the VPC ending with AWSVPC.
In another browser tab, navigate to VPC > Security Groups.
Look for a security group with Use me for endpoints in the description, and note the last four digits of its group ID.
In the Route 53 Resolver Endpoints browser tab, select the security group ending in the last four digits you just noted.
#### Configure IP Address 1
In the VPC browser tab, click Subnets in the left-hand menu.
Look for the subnet named AWSPubSubnet1, and note its Availability Zone.
In the Route 53 Resolver Endpoints browser tab, select the Availability Zone for the subnet we just identified.
Select the only available subnet for that Availability Zone.
Ensure Use an IP address that is selected automatically is selected.
#### Configure IP Address 2
In the VPC subnets browser tab, look for the subnet named AWSPubSubnet2, and note its Availability Zone.
In the Route 53 Resolver Endpoints browser tab, select the Availability Zone for the subnet we just identified.
Select the only available subnet for that Availability Zone.
Ensure Use an IP address that is selected automatically is selected.
Click Next.
#### Create Rule for Outbound Traffic
Enter "ToBind" as the name for the rule for outbound traffic.
Ensure Forward is selected as the rule type.
Enter intranet.onprem as the DNS name.
Select the VPC ending with AWSVPC as the name.
Enter the Private IP address of BIND server provided in your lab instructions.
Click Next and review the settings.
Click Submit.

### Configure the BIND Server
Navigate to Route 53 > Inbound Endpoints.
Click the hyperlink in the ID column for the DNSLabINbound endpoint.
Copy the two IP addresses for the inbound endpoint, as we will use this later in the lab.
Open the terminal you used to log in to the BIND server.
Enter the command:

sudo vim /etc/named/named.conf.local
If prompted, enter the password.

At the bottom of the file, enter the following zone directive. Be sure to replace <IP1> and <IP2> with the IP addresses for the inbound endpoint we noted above. Also be sure that each IP address has a semicolon (;) after it:

zone "vpc" {
    type forward;
    forward only;
    forwarders  { <IP1>; <IP2>; };
};
Save and exit the file.

Run the following command to restart the BIND server:

sudo service named restart
If prompted, enter the password provided for the BIND server.

### Evaluate Hybrid DNS Resolution Using Route 53 Resolver Endpoints

In the Route 53 browser tab, click Inbound Endpoints in the left-hand menu.
Verify the status says Operational.
Click Outbound Endpoints in the left-hand menu.
Verify the status says Operational.
Wait until both endpoints are Operational before continuing the lab.
Evaluate DNS Resolution from the On-Premises Bastion Host
In the terminal window connected to the on-premises bastion host, run the following command:

curl inet.intranet.onprem
You should receive a message stating: This is an On Premises intranet.

Run the following command:

curl inet.intranet.vpc
You should receive a message stating: This is the VPC intranet.

Evaluate DNS Resolution from the VPC Bastion Host
In the terminal window connected to the VPC bastion host, run the following command:

curl inet.intranet.vpc
You should receive a message stating: This is the VPC intranet.

Run the following command:

curl inet.intranet.onprem
You should receive a message stating: This is the On Premises intranet.