# VPC Networking

![alt text](image.png)

### 1. Create VPC Using Cloud Console

1. In the Cloud Console, on the Navigation menu (Navigation menu icon), click VPC network > VPC networks.

2. Click Create VPC Network.

3. For Name, type managementnet

4. For Subnet creation mode, click Custom.

5. Specify the following, and leave the remaining settings as their defaults:

| Property   | Value               |
| ---------- | ------------------- |
| Name       | managementsubnet-us |
| Region     | name_region         |
| IPv4 range | 10.240.0.0/20       |
