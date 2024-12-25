# Configure an Internal Network Load Balancer

![alt text](image.png)

### 1. Configure internal traffic and health check firewall rules

Configure firewall rules to allow internal traffic connectivity from sources in the 10.10.0.0/16 range. This rule allows incoming traffic from any client located in the subnet.

Health checks determine which instances of a load balancer can receive new connections. For Application Load Balancing (HTTP), the health check probes to your load-balanced instances come from addresses in the ranges 130.211.0.0/22 and 35.191.0.0/16. Your firewall rules must allow these connections.

#### Explore the my-internal-app network

The network my-internal-app with subnet-a and subnet-b and firewall rules for RDP, SSH, and ICMP traffic have been configured for you.

- In the Cloud Console, on the Navigation menu (Navigation menu icon), click VPC network > VPC networks.
  Notice the my-internal-app network with its subnets: subnet-a and subnet-b.

- Each Google Cloud project starts with the default network. In addition, the my-internal-app network has been created for you as part of your network diagram.

- You will create the managed instance groups in subnet-a and subnet-b. Both subnets are in the `Region` region because an internal Network Load Balancer is a regional service. The managed instance groups will be in different zones, making your service immune to zonal failures.

#### Create the firewall rule to allow traffic from any sources in the 10.10.0.0/16 range

Create a firewall rule to allow traffic in the 10.10.0.0/16 subnet.

1. On the Navigation menu (Navigation menu icon), click VPC network > Firewall.
   Notice the app-allow-icmp and app-allow-ssh-rdp firewall rules. These firewall rules have been created for you.

2. Click Create Firewall Rule.

3. Specify the following, and leave the remaining settings as their defaults.

| Property            | Value (type value or select option as specified) |
| ------------------- | ------------------------------------------------ |
| Name                | fw-allow-lb-access                               |
| Network             | my-internal-app                                  |
| Targets             | Specified target tags                            |
| Target tags         | backend-service                                  |
| Source filter       | IPv4 ranges                                      |
| Source IPv4 ranges  | 10.10.0.0/16                                     |
| Protocols and ports | Allow all                                        |

> Note: Make sure to include the /16 in the Source IPv4 ranges.

4. Click Create.

#### Create the health check rule

Create a firewall rule to allow health checks.

1. On the Navigation menu (Navigation menu icon), click VPC network > Firewall.

2. Click Create Firewall Rule.

3. Specify the following, and leave the remaining settings as their defaults.

| Property            | Value (type value or select option as specified) |
| ------------------- | ------------------------------------------------ |
| Name                | fw-allow-health-checks                           |
| Network             | my-internal-app                                  |
| Targets             | Specified target tags                            |
| Target tags         | backend-service                                  |
| Source filter       | IPv4 Ranges                                      |
| Source IPv4 ranges  | 130.211.0.0/22 and 35.191.0.0/16                 |
| Protocols and ports | Specified protocols and ports                    |

> Note: Make sure to include the /22 and /16 in the Source IPv4 ranges.

4. For tcp, check the checkbox and specify port 80.

5. Click Create.

### 2. Create a NAT configuration using Cloud Router
