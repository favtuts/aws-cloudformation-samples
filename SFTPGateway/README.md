# SFTP Gateway Product Overview

SFTP Gateway is an all-in-one solution to provide and configure SFTP access to limitless and reliable cloud storage locations, like S3. It includes a web interface and REST API that simplifies user management, folder permissions and instance administration, whether you're supporting a single user or thousands.

## How it works:

SFTP Gateway is an SFTP server that provides real-time access to files stored on cloud storage. Users can move files to and from cloud platforms using an SFTP client like FileZilla or WinSCP. Behind the scenes, SFTP Gateway uses a software layer that translates the SFTP protocol to AWS SDK commands.

## Easy User Management:

SFTP Gateway comes with a web admin UI for managing users and folders. The new folder management feature lets you set up flexible file sharing scenarios. Also new in version 3 is the ability to configure per-user IP address restrictions.

## Security and Compliance:

SFTP Gateway supports both key-based and password authentication.

Data is encrypted in transit and at rest.

## Highlights
- Read and write files to S3 in real-time using any SFTP client.
- Manage users, credentials and folders with ease, using a simple web interface. Map folders to locations in S3 to create data separation or file sharing scenarios.
- Encrypt data at rest and in transit. Data is encrypted at rest on S3 via SSE-S3 or SSE-KMS. The EBS volume is encrypted by default via CloudFormation. Data is encrypted in transit, whether communicating with the user (SFTP), or with S3 (HTTPS).

# High Availibility Well-Architected

![SFTP-Gateway-HA](SFTP-Gateway-for-AWS-High-Availability-Architecture.png)

# Single Instance Architecture

![SFTP-Gateway-SI](SFTP-Gateway-for-AWS-Single-Instance-Architecture.png)

# CloudFormation Templates

[SFTP Gateway (High Availability-Existing Network)](SFTP-Gateway-High-Availability-Existing-Network.yml)

This CloudFormation template automatically deploys a highly available infrastructure of SFTP Gateway, provisioning the necessary AWS resources into a VPC and two subnets of your choice. A Network Load Balancer routes traffic to instances deployed in an Autoscaling Group that spans two Availability Zones. An IAM role grants the EC2 instances permission to create S3 buckets and upload objects. EFS syncs local state between instances. And a Security Group restricts inbound access to certain network ranges over specific ports. After the CloudFormation stack spins up, you can connect to the Network Load Balancer hostname (found in the CloudFormation Outputs tab) from your web browser to create and manage SFTP users.

[SFTP Gateway (Single Instance - Existing Network)](SFTP-Gateway-Single-Instance-Existing-Network.yml)

This CloudFormation template deploys a single instance of SFTP Gateway into an existing VPC and subnet of your choice. The template also provisions the following AWS resources: An IAM role grants the EC2 instance permission to create S3 buckets and upload objects. A Security Group restricts inbound access to certain network ranges over ports. And an Elastic IP reserves your IP address. After the CloudFormation stack spins up, you can connect to the public IP address (found in the CloudFormation Outputs tab) from your web browser to create and manage SFTP users.

[SFTP Gateway (Single Instance - New Network)](SFTP-Gateway-Single-Instance-New-Network.yml)

This CloudFormation template deploys a single instance of SFTP Gateway into a new VPC and subnet. The template also provisions the following AWS resources: (1) An IAM role grants the EC2 instance permission to create S3 buckets and upload objects. (2) An EC2 Security Group restricts inbound access to certain network ranges over ports. (3) And an Elastic IP reserves your IP address. After the CloudFormation stack spins up, you can connect to the public IP address (found in the CloudFormation Outputs tab) from your web browser to create and manage SFTP users.
