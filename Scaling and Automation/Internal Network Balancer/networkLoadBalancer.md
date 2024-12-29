# Configure an Internal Network Load Balancer

![alt text](image.png)

### 1. Configure internal traffic and health check firewall rules

Configure firewall rules to allow internal traffic connectivity from sources in the 10.10.0.0/16 range. This rule allows incoming traffic from any client located in the subnet.

Health checks determine which instances of a load balancer can receive new connections. For Application Load Balancing (HTTP), the health check probes to your load-balanced instances come from addresses in the ranges 130.211.0.0/22 and 35.191.0.0/16. Your firewall rules must allow these connections.

#### Explore the my-internal-app network

The network my-internal-app with subnet-a and subnet-b and firewall rules for RDP, SSH, and ICMP traffic have been configured for you.

- In the Cloud Console, on the Navigation menu, click VPC network > VPC networks.
  Notice the my-internal-app network with its subnets: subnet-a and subnet-b.

- Each Google Cloud project starts with the default network. In addition, the my-internal-app network has been created for you as part of your network diagram.

- You will create the managed instance groups in subnet-a and subnet-b. Both subnets are in the `Region` region because an internal Network Load Balancer is a regional service. The managed instance groups will be in different zones, making your service immune to zonal failures.

#### Create the firewall rule to allow traffic from any sources in the 10.10.0.0/16 range

Create a firewall rule to allow traffic in the 10.10.0.0/16 subnet.

1. On the Navigation menu, click VPC network > Firewall.
   Notice the app-allow-icmp and app-allow-ssh-rdp firewall rules. These firewall rules have been created for you.

2. Click Create Firewall Rule.

![image](https://github.com/user-attachments/assets/2697fd13-4cbe-4ea4-8f84-ad4e5679ac47)

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

![image](https://github.com/user-attachments/assets/64a3f843-49e9-4543-bfa0-bda0578f019b)

![image](https://github.com/user-attachments/assets/e591435b-8d29-4248-8bce-30439df6d05a)

4. Click Create.

#### Create the health check rule

Create a firewall rule to allow health checks.

1. On the Navigation menu (Navigation menu icon), click VPC network > Firewall.

2. Click Create Firewall Rule.

![image](https://github.com/user-attachments/assets/62e18bc2-0bea-48a3-9ab5-769000356118)

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

![image](https://github.com/user-attachments/assets/d7fdb428-8d55-4aaa-aa07-d3884d6115c4)

![image](https://github.com/user-attachments/assets/276fe2d4-f311-4430-9cc0-1b69deba1cd5)

5. Click Create.

### 2. Create a NAT configuration using Cloud Router

#### Create the Cloud Router instance

1. On the Google Cloud console title bar, type Network services in the Search field, then click Network services in the Products & Page section.

2. On the Network service page, click Pin next to Network services.

![image](https://github.com/user-attachments/assets/98a0ebde-b3c1-4bc9-8411-109ea68d5b82)

3. Click Cloud NAT.

4. Click Get started to configure a NAT gateway.

![image](https://github.com/user-attachments/assets/dee77ab3-e01a-4ce9-8d3e-4622861fc3a6)

5. Specify the following, and leave the remaining settings as their defaults.

| Property     | Value (type value or select option as specified) |
| ------------ | ------------------------------------------------ |
| Gateway name | nat-config                                       |
| Network      | my-internal-app                                  |
| Region       | `Region`                                         |

![image](https://github.com/user-attachments/assets/523b6fd0-9588-4dbe-9650-ad35e152d1e8)

6. Click Cloud Router, and select Create new router.

![image](https://github.com/user-attachments/assets/4497ddf7-4785-49fe-ae40-404e1e987243)

7. For Name, type nat-router-`Region`.

8. Click Create.

![image](https://github.com/user-attachments/assets/5427e591-f462-40af-99a0-4bf5ceeae65e)

9. In Create Cloud NAT gateway, click Create.

### 3. Configure instance templates and create instance groups

1. On the Navigation menu, click Compute Engine > VM instances.
   Notice the instances that start with instance-group-1 and instance-group-2.

2. Select the SSH button next to instance-group-1 to SSH into the VM.

3. If prompted allow SSH-in-browser to connect to VMs, click Authorize.

4. Run the following command to re-run the instance's startup script:

```bash
sudo google_metadata_script_runner startup
```

![image](https://github.com/user-attachments/assets/c18f7b16-56c8-4755-a3b0-edc37e9b0a74)

5. Repeat the previous steps for instance-group-2.

![image](https://github.com/user-attachments/assets/fa0f2476-a50f-448f-8c90-afca50eef2b3)

6. Wait for both startup scripts to finish executing, then close the SSH terminal to each VM. The output of the startup script should state the following:

#### Verify the backends

Verify that VM instances are being created in both subnets and create a utility VM to access the backends' HTTP sites.

1. On the Navigation menu, click Compute Engine > VM instances.
   Notice the instances that start with instance-group-1 and instance-group-2.
   These instances are in separate zones, and their internal IP addresses are part of the subnet-a and subnet-b CIDR blocks.

2. Click Create Instance.

3. For the **Machine configuration**page, specify the following, and leave the remaining settings as their defaults:

| Property     | Value                           |
| ------------ | ------------------------------- |
| Name         | utility-vm                      |
| Region       | `Region`                        |
| Zone         | `Zone 3`                        |
| Series       | E2                              |
| Machine type | e2-medium (2 vCPU, 4 GB memory) |

![image](https://github.com/user-attachments/assets/71ce121e-6b1b-480f-a92a-7e9c528868b8)

![image](https://github.com/user-attachments/assets/9fbf3e01-8de7-4a07-a1fc-26b355d05506)

4. Click OS and storage.

5. If the Image shown is not Debian GNU/Linux 12 (bookworm), click Change and select Debian GNU/Linux 12 (bookworm), and then click Select.

![image](https://github.com/user-attachments/assets/84b76995-3252-4acf-a4c9-abff96b713b0)

6. Click Networking.

7. For Network interfaces, click the dropdown to edit the network interface.

8. Specify the following, and leave the remaining settings as their defaults.

| Property                      | Value              |
| ----------------------------- | ------------------ |
| Network                       | my-internal-app    |
| Subnetwork                    | subnet-a           |
| Primary internal IPv4 address | Ephemeral (Custom) |
| Custom ephemeral IP address   | 10.10.20.50        |
| External IPv4 address         | None               |

9. Click Done.

![image](https://github.com/user-attachments/assets/47679585-7585-4331-b070-ad68d0de0638)

10. Click Create.

11. Note that the internal IP addresses for the backends are 10.10.20.2 and 10.10.30.2.

> Note: If these IP addresses are different, replace them in the two curl commands below.

12. For utility-vm, click SSH to launch a terminal and connect.

13. If prompted allow SSH-in-browser to connect to VMs, click Authorize.

14. To verify the welcome page for instance-group-1-xxxx, run the following command:

```bash
curl 10.10.20.2
```

![image](https://github.com/user-attachments/assets/12bf94a5-72d7-4b2e-ac22-12c21334c5d7)

The output should look like this: 


15. To verify the welcome page for instance-group-2-xxxx, run the following command:

```bash
curl 10.10.30.2
```

![image](https://github.com/user-attachments/assets/b05fd300-6db6-4bc5-adba-3e3383559e2c)

The output should look like this.

16. Close the SSH terminal to utility-vm:

```bash
exit
```

### 4. Configure the internal Network Load Balancer

Configure the internal Network Load Balancer to balance traffic between the two backends (`instance-group-1 in Zone 1` and `instance-group-2 in Zone 2`), as illustrated in the network diagram.

![alt text](image-1.png)

#### Start the configuration

1. In the Cloud Console, on the Navigation menu, click Network services > Load balancing.

2. Click Create load balancer.

![image](https://github.com/user-attachments/assets/f424eafc-b3af-4af8-89eb-049a8f25a7d6)

3. For Type of load balancer, select Network Load Balancer (TCP/UDP/SSL), click Next.

![image](https://github.com/user-attachments/assets/6ce154c8-4816-4a08-935d-25a78bdaccac)

4. For Proxy or passthrough, select Passthrough load balancer and click Next.

![image](https://github.com/user-attachments/assets/3bb1f254-5d88-42fb-8896-9d66fd8401a5)

5. For Public facing or internal, select Internal and click Next.

![image](https://github.com/user-attachments/assets/502bf439-1a17-46f1-9111-70b8b41de3c1)

6. For Create load balance, click Configure.

![image](https://github.com/user-attachments/assets/f57471fd-fa01-4be0-97aa-b039593654e6)

7. For Load balancer name, type my-ilb.

8. For Region, type `Region`.

9. For Network, select my-internal-app from the dropdown.

![image](https://github.com/user-attachments/assets/0a966f63-a347-45c2-b463-0ac0edc124f1)

#### Configure the regional backend service

The backend service monitors instance groups and prevents them from exceeding configured usage.

1. Click Backend configuration.

2. Specify the following, and leave the remaining settings as their defaults.

| Property       | Value                       |
| -------------- | --------------------------- |
| Instance group | instance-group-1 (`Zone 1`) |

3. Click Done.

![image](https://github.com/user-attachments/assets/6e163e9f-bc9c-4493-9098-661c09a43f62)

![image](https://github.com/user-attachments/assets/1f6dad58-a1ca-4f7c-98d7-31101a81996a)

4. Click Add a backend.

5. For Instance group, select instance-group-2 (`Zone 2`).

6. Click Done.

![image](https://github.com/user-attachments/assets/30b33519-bdf3-4f50-b0b8-64efb695dc28)

7. For Health Check, select Create a health check.

8. Specify the following, and leave the remaining settings as their defaults.

| Property            | Value               |
| ------------------- | ------------------- |
| Name                | my-ilb-health-check |
| Protocol            | TCP                 |
| Port                | 80                  |
| Check interval      | 10 sec              |
| Timeout             | 5 sec               |
| Healthy threshold   | 2                   |
| Unhealthy threshold | 3                   |

![image](https://github.com/user-attachments/assets/a80c4508-013f-4e75-b87b-f7ecaa1eff0b)

![image](https://github.com/user-attachments/assets/af073112-d71f-4182-af0f-4ed53004847e)

> Note: Health checks determine which instances can receive new connections. This HTTP health check polls instances every 10 seconds, waits up to 5 seconds for a response, and treats 2 successful or 3 failed attempts as healthy threshold or unhealthy threshold, respectively.

9. Click Save.

10. Verify that there is a blue checkmark next to Backend configuration in the Cloud Console. If there isn't, double-check that you have completed all the steps above.

#### Configure the frontend

The frontend forwards traffic to the backend.

1. Click Frontend configuration.

2. Specify the following, and leave the remaining settings as their defaults.

| Property                         | Value             |
| -------------------------------- | ----------------- |
| Subnetwork                       | subnet-b          |
| Internal IP purpose > IP address | Create IP address |

![image](https://github.com/user-attachments/assets/e4dcbfd7-464b-4fe9-84d2-4ffad4351036)

3. Specify the following, and leave the remaining settings as their defaults.

| Property          | Value         |
| ----------------- | ------------- |
| Name              | my-ilb-ip     |
| Static IP address | Let me choose |
| Custom IP address | 10.10.30.5    |

4. Click Reserve.

![image](https://github.com/user-attachments/assets/559d0f59-6b3e-4a56-943b-5d6582e9f20c)

5. Under Ports, for Port number, type 80.

6. Click Done.

![image](https://github.com/user-attachments/assets/104915c2-d162-4040-9527-341e477b0d60)

#### Review and create the internal Network Load Lalancer

1. Click Review and finalize.

2. Review the Backend and Frontend.

3. Click Create.

![image](https://github.com/user-attachments/assets/0863728a-b742-46c1-acea-924c6ebbb66e)

### 5. Test the internal Network Load Balancer

Verify that the my-ilb IP address forwards traffic to instance-group-1 in `Zone 1` and instance-group-2 in `Zone 2`.

#### Access the internal Network Load Balancer

1. On the Navigation menu, click Compute Engine > VM instances.

2. For utility-vm, click SSH to launch a terminal and connect.

3. If prompted allow SSH-in-browser to connect to VMs, click Authorize.

4. To verify that the internal Network Load Balancer forwards traffic, run the following command:

```bash
curl 10.10.30.5
```

5. Run the same command a couple of times:

```bash
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
curl 10.10.30.5
```

You should be able to see responses from instance-group-1 in `Zone 1` and instance-group-2 in `Zone 2`. If not, run the command again.
