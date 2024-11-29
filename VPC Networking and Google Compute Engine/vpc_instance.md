# VPC Networking and Google Compute Engine

### 1. Create a VPC network and VM instances

Create an auto mode VPC network with Firewall rules

1. On the Navigation menu (Navigation menu icon), click VPC network > VPC networks.

2. Click Create VPC network.

3. For Name, type mynetwork.

4. For Subnet creation mode, click Automatic. Auto mode networks create subnets in each region automatically.

5. For Firewall, select all available rules.
   These are the same standard firewall rules that the default network had. The deny-all-ingress and allow-all-egress rules are also displayed, but you cannot check or uncheck them because they are implied. These two rules have a lower Priority (higher integers indicate lower priorities) so that the allow ICMP, custom, RDP and SSH rules are considered first.

6. Click Create.
   When the new network is ready, notice that a subnet was created for each region.

7. Explore the IP address range for the subnets in Region 1 and Region 2.

> **Note**: If you ever delete the default network, you can quickly re-create it by creating an auto mode network as you just did. After recreating the network, allow-internal changes to allow-custom firewall rule.

Create a VM instance in Region

1. On the Navigation menu (Navigation menu icon), click Compute Engine > VM instances.

2. Click Create instance.

3. Specify the following, and leave the remaining settings as their defaults:

> | Property     | Value                          |
> | ------------ | ------------------------------ |
> | Name         | mynet-us-vm                    |
> | Region       | name_region                    |
> | Zone         | name_zone                      |
> | Series       | E2                             |
> | Machine type | e2-micro (2 vCPU, 1 GB memory) |

4. Click Create
