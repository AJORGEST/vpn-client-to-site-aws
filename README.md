# vpn-client-to-site-aws

- Stage 1 - Create Directory Service (authentication for VPN users) aws
- Stage 2 - Certificates
- Stage 3 - Create VPN Endpoint
- Stage 4 - Configure VPN Endpoint & Associations
- Stage 5 - Download, install and test VPN Client
- Stage 6 - Cleanup

# 1.Create a simple AD instance

* Type Directory Service into the top searchbox and open the directory services console in a new tab.
* Acesse Or use directly open via https://console.aws.amazon.com/directoryservicev2/identity?region=us-east-1#!/directories
* **Click Set up Directory**
* Select `Simple AD` and click Next
* Choose `Small` (this is a demo, for larger deployments you might pick large, or chose alternative methods of authentication)
* Pick a Directory DNS Name, i'll use `corp.xvia.org`
* Pick a Directory `NetBIOS` name, i'll use CORP
* Choose an `Administrator password, it will need to be strong` enough to meet the complexity requirements.
* Enter the same password again in the Confirm Password box
* for Directory description - Optional put `Directory Service for AWS Client VPN Demo`
* Click Next
* Under VPC click the dropdown and pick `VPC-DEMO-PROD`
* Under Subnets ideally pick **PRIV-A and PRIV-B** (but if one of those isn't available, just pick 2 of the `PRIV subnets`)
* Click Next
* ***Click Create Directory**

# 2.Creating Certificates

This past of the demo involves downloading easy-rsa and using this to create certificates which will be imported into ACM. If using this in production, you can create server and client certificates for mutual authentication. To keep things simple (the focus is on ClientVPN itself) we will only be using the server certificate.

# Linux/macOS

```
open a terminal
cd /tmp
git clone https://github.com/OpenVPN/easy-rsa.git
cd easy-rsa/easyrsa3
./easyrsa init-pki
./easyrsa build-ca nopass
./easyrsa build-server-full corp.xvia.org nopass
aws acm import-certificate --certificate fileb://pki/issued/corp.xvia.org.crt --private-key fileb://pki/private/corp.xvia.org.key --certificate-chain fileb://pki/ca.crt --profile iamadmin-general
```
# Windows

```
Open the OpenVPN Community Downloads page, download the Windows installer for your version of Windows, and run the installer.

Open the EasyRSA releases page and download the ZIP file for your version of Windows. Extract the ZIP file and copy the EasyRSA folder to the \Program Files\OpenVPN folder.

Open the command prompt as an Administrator, navigate to the \Program Files\OpenVPN\EasyRSA directory, and run the following command to open the EasyRSA 3 shell.

EasyRSA-Start

./easyrsa init-pki

./easyrsa build-ca nopass

./easyrsa build-server-full server nopass

./easyrsa build-client-full client1.domain.tld nopass

exit

aws acm import-certificate --certificate fileb://pki/issued/server.crt --private-key fileb://pki/private/server.key --certificate-chain fileb://pki/ca.crt --profile iamadmin-general

Type ACM or Certificate Manager into the search box at the top of the screen then right click and open in a new tab.

Verify that your certificate exists in the us-east-1 region.
```

# 3.Create VPN Endpoint

- Type VPC in the services search box at the top of the screen, right click and open in a new tab.
- Under **Virtual Private Network (VPN)** on the menu on the left, locate and `click Create VPN Endpoint`
- `Click Create Client VPN Endpoint`
- For Name Tag enter `Demo Client VPN`
- Under Client IPv4 CIDR* enter `192.168.12.0/22`
- Click the Server certificate ARN* dropdown and select the **server certificate** you created in stage 2.
- Under Authentication Options `check Use user-based authentication`
- `Check Active Directory authentication`
- Under Directory ID* chose the directory you created in Stage 1 (e.g. corp.xvia.org)
- Under Connection Logging, Do you want to log the details on client connections?* check no
- for `DNS Server 1 IP address and DNS Server 2` IP address we need to enter the IP addresses of the directory service instance. Go back to the tab with the directory service       - console open, click the directory service instance ID , locate the DNS address area and copy one IP into each of the DNS Server IP boxes.
- `Check Enable split-tunnel`
- in the VPC ID dropdown select `VPC-DEMO`
- Ensure the Default SG is checked and the A4L DefaultSG
- `Check Enable self-service portal`
- Click Create Client VPN Endpoint
- Click Close

# 4.Associate ClientVPN Endpoint

* From the `Client VPN Endpoints` area of the VPC console, select the DEMO Client VPN endpoint.
* Click the Associations tab and click Associate
* Click the VPC* dropdown and click the `VPC-DEMO`
* Open in a new tab, the VPC, Subnets -> console `https://console.aws.amazon.com/vpc/home?region=us-east-1#subnets:`
* Locate the subnet ID for the 3 private subnets in the VPC DEMO
* Click the Choose a subnet to associate* dropdown and pick the first available PRIV subnet from the list `(PRIV-A, PRIV-B, PRIV-C)`
* Click Associate then click Close
* The VPN Endpoint will now be associated, you need to pause here and wait for the state of the VPN endpoint to change from Pending-associate to Available

# 5.Download the ClientVPN 

* From the Client VPN Endpoints area of the VPC console, select the `DEMO Client VPN endpoint.`
* Click Download Client Configuration, `Click Download and save the file.`
* Go to https://aws.amazon.com/vpn/client-vpn-download/ and download the client for your operating system
* Install the VPN Application
* Start the application
* Go to manage profiles
* add a profile
* load the profile you downloaded (client configuration) - use DEMO for Displayname

# 6.Connect

Connect to DEMO VPN
enter the Administrator username and password you chose in stage 1 for the Directory Service
once connected open a terminal and `ping DIRECTORY_SERVICE_IP_ADDRESS' (you can get this from the DS Console)
Notice it doesn't work ? once more step, and thats authorizations

# 7.Authorize

* From the Client VPN Console `https://console.aws.amazon.com/vpc/home?region=us-east-1#ClientVPNEndpoints:sort=status.code select the A4L VPN Endpoint`
* Click the Authorization tab and click Authorize Ingress
* For Destination network to enable enter `10.16.0.0/16`
* For Grant access to check `Allow access to all users`
* Click Add Authorization Rule
* open a terminal and `ping DIRECTORY_SERVICE_IP_ADDRESS'` (you can get this from the DS Console)
* notice how this works? you are now connected.

  

