### Gather XML Metadata from the Lab Identity Provider (IdP)

https://<PUBLIC_IP_ADDRESS_OF_SHIBBOLETH_IDP>/idp/profile/Metadata/SAML

curl -k https://3.232.95.160/idp/profile/Metadata/SAML > saml2.xml  

Save it to a file named saml2.xml

### Create IAM Identity Provider

In the IAM browser tab, navigate to IAM > Identity providers.
Click Create Provider, and set the following values:
Provider Type: SAML
Provider Name: intranet-local-idp
Metadata Document: Click Choose File, and select the saml2.xml file
Click Next Step.
Click Create.

### Create IAM Role
Click Roles in the left-hand menu.
Click Create role.
Select SAML 2.0 Federation as the type of trusted entity.
Set the following values:
SAML provider: intranet-local-idp
Allow programmatic and AWS Management Console access: Select
Click Next: Permissions.
On the permissions policies page, search for (in the Filter policies box) and select ReadOnlyAccess. (It will be far down on the list.)
Click Next: Tags > Next: Review.
Set the following values:
Role name: intranet-local-readonly
Role description: Intranet local read only role
Click Create role.
Click the newly created intranet-local-readonly role.
Copy its role ARN, and paste it into a text document for later use.
arn:aws:iam::494480237640:role/intranet-local-readonly
Click the Trust relationships tab.
Under Trusted entities copy the listed IdP ARN, and paste it into a text document for later use.
arn:aws:iam::494480237640:saml-provider/intranet-local-idp

### Configure the Source IdP to Work with AWS

Log in to the Shibboleth IdP via SSH, replacing <PUBLIC_IP_ADDRESS_OF_SHIBBOLETH_IDP> with the value provided on the lab page:

ssh cloud_user@<PUBLIC_IP_ADDRESS_OF_SHIBBOLETH_IDP>
Move to the Shibboleth configuration directory:

cd shibconf
ls
Edit the configuration file:

sudo vim attribute-resolver.xml
Scroll down in the file until you find the AWS Resolver block.

Replace ROLE_ARN with the role ARN you copied earlier.
Replace IDP_ARN with the IdP ARN you copied earlier.
Save the file.
Move to the root directory:

cd ..
Run the script to restart tomcat:

sudo ./restart_tomcat.sh

 <resolver:AttributeDefinition id="awsRoles" xsi:type="ad:Mapped" sourceAttributeID="uid">
        <resolver:Dependency ref="myLDAP" />
        <resolver:AttributeEncoder xsi:type="enc:SAML2String" name="https://aws.amazon.com/SAML/Attributes/Role" friendlyName="awsRoles" />
        <ad:ValueMap>
            <ad:ReturnValue>
               arn:aws:iam::494480237640:role/intranet-local-readonly,arn:aws:iam::494480237640:saml-provider/intranet-local-idp
            </ad:ReturnValue>
            <ad:SourceValue>.*clouduser.*</ad:SourceValue>
        </ad:ValueMap>    </resolver:AttributeDefinition>

### Test SAML Federation from Shibboleth to AWS

Open a remote desktop client.
Click + New to configure a new connection.
Enter "Bastion Host" as the connection name.
For PC name, enter the public IP address of the bastion host provided on the lab page.
Enter "cloud_user" as the user name.
For password, enter the password for the bastion host provided on the lab page.
Open the connection.
Click Use Default Config.
Open a web browser from the bastion host.
Navigate to the login URL: https://idp.intranet.local/idp/profile/SAML2/Unsolicited/SSO?providerId=urn:amazon:webservices
The site uses a self-signed certificate, which is fine for this lab. Click Advanced, and then Click Proceed to idp.intranet.local (unsafe).
Enter "clouduser" as the name.
Enter "bettertogether" as the password. You should then be directed to the AWS Management Console.