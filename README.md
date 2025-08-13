# vpn-client-to-site-aws

- Stage 1 - Create Directory Service (authentication for VPN users) aws
- Stage 2 - Certificates
- Stage 3 - Create VPN Endpoint
- Stage 4 - Configure VPN Endpoint & Associations
- Stage 5 - Download, install and test VPN Client
- Stage 6 - Cleanup

# Create a simple AD instance

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

# Creating Certificates

This past of the demo involves downloading easy-rsa and using this to create certificates which will be imported into ACM. If using this in production, you can create server and client certificates for mutual authentication. To keep things simple (the focus is on ClientVPN itself) we will only be using the server certificate.

# Linux/macOS

```
open a terminal
cd /tmp
git clone https://github.com/OpenVPN/easy-rsa.git
cd easy-rsa/easyrsa3
./easyrsa init-pki
./easyrsa build-ca nopass
ANIMALS4LIFEVPN
./easyrsa build-server-full corp.xvia.org nopass
aws acm import-certificate --certificate fileb://pki/issued/corp.xvia.org.crt --private-key fileb://pki/private/corp.xvia.org.key --certificate-chain fileb://pki/ca.crt --profile iamadmin-general
```
# Windows

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
