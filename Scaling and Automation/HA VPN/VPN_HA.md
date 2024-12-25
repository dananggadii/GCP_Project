# Configuring Google Cloud HA VPN

![alt text](image.png)

### 1. Set up a Global VPC environment

1. In Cloud Shell, create a VPC network called vpc-demo:

```bash
gcloud compute networks create vpc-demo --subnet-mode custom
```

The output should look similar to this:

2. In Cloud Shell, create subnet `vpc-demo-subnet1` in the region `REGION 1`:

```bash
gcloud compute networks subnets create vpc-demo-subnet1 \
--network vpc-demo --range 10.1.1.0/24 --region "REGION"
```

3. Create subnet `vpc-demo-subnet2` in the region `REGION 2`:

```bash
gcloud compute networks subnets create vpc-demo-subnet2 \
--network vpc-demo --range 10.2.1.0/24 --region REGION 2
```

4. Create a firewall rule to allow all custom traffic within the network:

```bash
gcloud compute firewall-rules create vpc-demo-allow-custom \
  --network vpc-demo \
  --allow tcp:0-65535,udp:0-65535,icmp \
  --source-ranges 10.0.0.0/8
```

The output should look similar to this:

5. Create a firewall rule to allow SSH, ICMP traffic from anywhere:

```bash
gcloud compute firewall-rules create vpc-demo-allow-ssh-icmp \
    --network vpc-demo \
    --allow tcp:22,icmp
```

6. Create a VM instance `vpc-demo-instance1` in zone `ZONE 1`:

```bash
gcloud compute instances create vpc-demo-instance1 --machine-type=e2-medium --zone "ZONE" --subnet vpc-demo-subnet1
```

The output should look similar to this:

7. Create a VM instance `vpc-demo-instance`2 in zone `ZONE 2`:

```bash
gcloud compute instances create vpc-demo-instance2 --machine-type=e2-medium --zone ZONE2 --subnet vpc-demo-subnet2
```

### 2. Set up a simulated on-premises environment

1. In Cloud Shell, create a VPC network called `on-prem`:

```bash
gcloud compute networks create on-prem --subnet-mode custom
```

The output should look similar to this:

2. Create a subnet called `on-prem-subnet1`:

```bash
gcloud compute networks subnets create on-prem-subnet1 \
--network on-prem --range 192.168.1.0/24 --region "REGION"
```

3. Create a firewall rule to allow all custom traffic within the network:

```bash
gcloud compute firewall-rules create on-prem-allow-custom \
  --network on-prem \
  --allow tcp:0-65535,udp:0-65535,icmp \
  --source-ranges 192.168.0.0/16
```

4. Create a firewall rule to allow SSH, RDP, HTTP, and ICMP traffic to the instances:

```bash
gcloud compute firewall-rules create on-prem-allow-ssh-icmp \
    --network on-prem \
    --allow tcp:22,icmp
```

5. Create an instance called `on-prem-instance1` in the region `REGION 1`.

> Note: In the below command replace with a zone in `REGION 1` but different from the one used to create the `vpc-demo-instance1 in the vpc-demo-subnet1`.

```bash
gcloud compute instances create on-prem-instance1 --machine-type=e2-medium --zone zone_name --subnet on-prem-subnet1
```

### 3. Set up an HA VPN gateway

1. In Cloud Shell, create an HA VPN in the vpc-demo network:

```bash
gcloud compute vpn-gateways create vpc-demo-vpn-gw1 --network vpc-demo --region "REGION"
```

The output should look similar to this:

2. Create an HA VPN in the on-prem network:

```bash
gcloud compute vpn-gateways create on-prem-vpn-gw1 --network on-prem --region "REGION"
```

3. View details of the vpc-demo-vpn-gw1 gateway to verify its settings:

```bash
gcloud compute vpn-gateways describe vpc-demo-vpn-gw1 --region "REGION"
```

The output should look similar to this:

4. View details of the on-prem-vpn-gw1 vpn-gateway to verify its settings:

```bash
gcloud compute vpn-gateways describe on-prem-vpn-gw1 --region "Region"
```

The output should look similar to this:

Create cloud routers

1. Create a cloud router in the vpc-demo network:

```bash
gcloud compute routers create vpc-demo-router1 \
    --region "REGION" \
    --network vpc-demo \
    --asn 65001
```

The output should look similar to this:

2. Create a cloud router in the on-prem network:

```bash
gcloud compute routers create on-prem-router1 \
    --region "REGION" \
    --network on-prem \
    --asn 65002
```

### 4. Create two VPN tunnels

In this task you create VPN tunnels between the two gateways. For HA VPN setup, you add two tunnels from each gateway to the remote setup. You create a tunnel on `interface0` and connect to `interface0` on the remote gateway. Next, you create another tunnel on `interface1` and connect to `interface1` on the remote gateway.4

When you run HA VPN tunnels between two Google Cloud VPCs, you need to make sure that the tunnel on `interface0` is connected to `interface0` on the remote VPN gateway. Similarly, the tunnel on `interface1` must be connected to `interface1` on the remote VPN gateway.

> Note: In your own environment, if you run HA VPN to a remote VPN gateway on-premises for a customer, you can connect in one of the following ways:
>
> - Two on-premises VPN gateway devices: Each of the tunnels from each interface on the Cloud VPN gateway must be connected to its own peer gateway.
> - A single on-premises VPN gateway device with two interfaces: Each of the tunnels from each interface on the Cloud VPN gateway must be connected to its own interface on the peer gateway.
> - A single on-premises VPN gateway device with a single interface: Both of the tunnels from each interface on the Cloud VPN gateway must be connected to the same interface on the peer gateway.

In this lab you are simulating an on-premises setup with both VPN gateways in Google Cloud. You ensure that interface0 of one gateway connects to interface0 of the other and interface1 connects to interface1 of the remote gateway.

1. Create the first VPN tunnel in the vpc-demo network:

```bash
gcloud compute vpn-tunnels create vpc-demo-tunnel0 \
    --peer-gcp-gateway on-prem-vpn-gw1 \
    --region "REGION" \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router vpc-demo-router1 \
    --vpn-gateway vpc-demo-vpn-gw1 \
    --interface 0
```

The output should look similar to this:

2. Create the second VPN tunnel in the vpc-demo network:

```bash
gcloud compute vpn-tunnels create vpc-demo-tunnel1 \
    --peer-gcp-gateway on-prem-vpn-gw1 \
    --region "REGION" \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router vpc-demo-router1 \
    --vpn-gateway vpc-demo-vpn-gw1 \
    --interface 1
```

3. Create the first VPN tunnel in the on-prem network:

```bash
gcloud compute vpn-tunnels create on-prem-tunnel0 \
    --peer-gcp-gateway vpc-demo-vpn-gw1 \
    --region "REGION" \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router on-prem-router1 \
    --vpn-gateway on-prem-vpn-gw1 \
    --interface 0
```

4. Create the second VPN tunnel in the on-prem network:

```bash
gcloud compute vpn-tunnels create on-prem-tunnel1 \
    --peer-gcp-gateway vpc-demo-vpn-gw1 \
    --region "REGION" \
    --ike-version 2 \
    --shared-secret [SHARED_SECRET] \
    --router on-prem-router1 \
    --vpn-gateway on-prem-vpn-gw1 \
    --interface 1
```

### 5. Create Border Gateway Protocol (BGP) peering for each tunnel

In this task you configure BGP peering for each VPN tunnel between vpc-demo and VPC on-prem. HA VPN requires dynamic routing to enable 99.99% availability.

1. Create the router interface for tunnel0 in network vpc-demo:

```bash
Create the router interface for tunnel0 in network vpc-demo:
```

The output should look similar to this:

2. Create the BGP peer for tunnel0 in network vpc-demo:

```bash
gcloud compute routers add-bgp-peer vpc-demo-router1 \
    --peer-name bgp-on-prem-tunnel0 \
    --interface if-tunnel0-to-on-prem \
    --peer-ip-address 169.254.0.2 \
    --peer-asn 65002 \
    --region "REGION"
```

The output should look similar to this:

3. Create a router interface for tunnel1 in network vpc-demo:

```bash
gcloud compute routers add-interface vpc-demo-router1 \
    --interface-name if-tunnel1-to-on-prem \
    --ip-address 169.254.1.1 \
    --mask-length 30 \
    --vpn-tunnel vpc-demo-tunnel1 \
    --region "REGION"
```

4. Create the BGP peer for tunnel1 in network vpc-demo:

```bash
gcloud compute routers add-bgp-peer vpc-demo-router1 \
    --peer-name bgp-on-prem-tunnel1 \
    --interface if-tunnel1-to-on-prem \
    --peer-ip-address 169.254.1.2 \
    --peer-asn 65002 \
    --region "REGION"
```

5. Create a router interface for tunnel0 in network on-prem:

```bash
gcloud compute routers add-interface on-prem-router1 \
    --interface-name if-tunnel0-to-vpc-demo \
    --ip-address 169.254.0.2 \
    --mask-length 30 \
    --vpn-tunnel on-prem-tunnel0 \
    --region "REGION"
```

6. Create the BGP peer for tunnel0 in network on-prem:

```bash
gcloud compute routers add-bgp-peer on-prem-router1 \
    --peer-name bgp-vpc-demo-tunnel0 \
    --interface if-tunnel0-to-vpc-demo \
    --peer-ip-address 169.254.0.1 \
    --peer-asn 65001 \
    --region "REGION"
```

7. Create a router interface for tunnel1 in network on-prem:

```bash
gcloud compute routers add-interface  on-prem-router1 \
    --interface-name if-tunnel1-to-vpc-demo \
    --ip-address 169.254.1.2 \
    --mask-length 30 \
    --vpn-tunnel on-prem-tunnel1 \
    --region "REGION"
```

8. Create the BGP peer for tunnel1 in network on-prem:

```bash
gcloud compute routers add-bgp-peer  on-prem-router1 \
    --peer-name bgp-vpc-demo-tunnel1 \
    --interface if-tunnel1-to-vpc-demo \
    --peer-ip-address 169.254.1.1 \
    --peer-asn 65001 \
    --region "REGION"
```

### 6. Verify router configurations

1. View details of Cloud Router vpc-demo-router1 to verify its settings:

```bash
gcloud compute routers describe vpc-demo-router1 \
    --region "REGION"
```

The output should look similar to this:

2. View details of Cloud Router on-prem-router1 to verify its settings:

```bash
gcloud compute routers describe on-prem-router1 \
    --region "REGION"
```

The output should look similar to this:

Configure firewall rules to allow traffic from the remote VPC\

1. Allow traffic from network VPC on-prem to vpc-demo:

```bash
gcloud compute firewall-rules create vpc-demo-allow-subnets-from-on-prem \
    --network vpc-demo \
    --allow tcp,udp,icmp \
    --source-ranges 192.168.1.0/24
```

The output should look similar to this:

2. Allow traffic from vpc-demo to network VPC on-prem:

```bash
gcloud compute firewall-rules create on-prem-allow-subnets-from-vpc-demo \
    --network on-prem \
    --allow tcp,udp,icmp \
    --source-ranges 10.1.1.0/24,10.2.1.0/24
```

Verify the status of the tunnels

1. List the VPN tunnels you just created:

```bash
gcloud compute vpn-tunnels list
```

There should be four VPN tunnels (two tunnels for each VPN gateway). The output should look similar to this:

2. Verify that vpc-demo-tunnel0 tunnel is up:

```bash
gcloud compute vpn-tunnels describe vpc-demo-tunnel0 \
      --region "REGION"
```

The tunnel output should show detailed status as Tunnel is up and running.

3. Verify that vpc-demo-tunnel1 tunnel is up:

```bash
gcloud compute vpn-tunnels describe vpc-demo-tunnel1 \
      --region "REGION"
```

The tunnel output should show detailed status as Tunnel is up and running.

4. Verify that on-prem-tunnel0 tunnel is up:

```bash
gcloud compute vpn-tunnels describe on-prem-tunnel0 \
      --region "REGION"
```

The tunnel output should show detailed status as Tunnel is up and running.

5. Verify that on-prem-tunnel1 tunnel is up:

```bash
gcloud compute vpn-tunnels describe on-prem-tunnel1 \
      --region "REGION"
```

The tunnel output should show detailed status as Tunnel is up and running.

1. Verify private connectivity over VPN
   Navigate to Compute engine and note the zone in which the on-prem-instance1 was created.

2. Open a new Cloud Shell tab and type the following to connect via SSH to the instance on-prem-instance1: Replace <zone_name> with the zone in which the on-prem-instance1 was created.

```bash
gcloud compute ssh on-prem-instance1 --zone zone_name
```

3. Type "y" to confirm that you want to continue.

4. Press Enter twice to skip creating a password.

5. From the instance on-prem-instance1 in network on-prem, to reach instances in network vpc-demo, ping 10.1.1.2:

```bash
ping -c 4 10.1.1.2
```

Pings are successful. The output should look similar to this:

Global routing with VPN

1. Open a new Cloud Shell tab and update the bgp-routing mode from vpc-demo to GLOBAL:

```bash
gcloud compute networks update vpc-demo --bgp-routing-mode GLOBAL
```

2. Verify the change:

```bash
gcloud compute networks describe vpc-demo
```

The output should look similar to this:

3. From the Cloud Shell tab that is currently connected to the instance in network on-prem via ssh, ping the instance vpc-demo-instance2 in region `REGION 2`:

```bash
ping -c 2 10.2.1.2
```

Pings are successful. The output should look similar to this:

### 7. Verify and test the configuration of HA VPN tunnels

1. Open a new Cloud Shell tab.

2. Bring tunnel0 in network vpc-demo down:

3. gcloud compute vpn-tunnels delete vpc-demo-tunnel0 --region "REGION"

Respond "y" when asked to verify the deletion. The respective tunnel0 in network on-prem will go down.

3. Verify that the tunnel is down:

```bash
gcloud compute vpn-tunnels describe on-prem-tunnel0  --region "REGION"
```

The detailed status should show as Handshake_with_peer_broken.

4. Switch to the previous Cloud Shell tab that has the open ssh session running, and verify the pings between the instances in network vpc-demo and network on-prem:

```bash
ping -c 3 10.1.1.2
```

The output should look similar to this:

Pings are still successful because the traffic is now sent over the second tunnel. You have successfully configured HA VPN tunnels.

### 8. Clean up

1. Delete VPN tunnels

From Cloud Shell, type the following commands to delete the remaining tunnels. Type "y" to confirm each action when asked:

```bash
gcloud compute vpn-tunnels delete on-prem-tunnel0  --region "REGION"
```

```bash
gcloud compute vpn-tunnels delete vpc-demo-tunnel1  --region "REGION"
```

```bash
gcloud compute vpn-tunnels delete on-prem-tunnel1  --region "REGION"
```

2. Remove BGP peering

Type the following commands from each BGP peer to remove peering:

```bash
gcloud compute routers remove-bgp-peer vpc-demo-router1 --peer-name bgp-on-prem-tunnel0 --region "REGION"
```

```bash
gcloud compute routers remove-bgp-peer vpc-demo-router1 --peer-name bgp-on-prem-tunnel1 --region "REGION"
```

```bash
gcloud compute routers remove-bgp-peer on-prem-router1 --peer-name bgp-vpc-demo-tunnel0 --region "REGION"
```

```bash
gcloud compute routers remove-bgp-peer on-prem-router1 --peer-name bgp-vpc-demo-tunnel1 --region "REGION"
```

3. Delete cloud routers

Type each command to delete the routers. Type "y" to confirm each action when asked:

```bash
gcloud compute  routers delete on-prem-router1 --region "REGION"
```

```bash
gcloud compute  routers delete vpc-demo-router1 --region "REGION"
```

4. Delete VPN gateways

Type each command to delete the VPN gateways. Type "y" to confirm each action when asked:

```bash
gcloud compute vpn-gateways delete vpc-demo-vpn-gw1 --region "REGION"
```

```bash
gcloud compute vpn-gateways delete on-prem-vpn-gw1 --region "REGION"
```

5. Delete instances

Type the following commands to delete each instance. Type "y" to confirm each action when asked:

```bash
gcloud compute instances delete vpc-demo-instance1 --zone "ZONE"
```

```bash
gcloud compute instances delete vpc-demo-instance2 --zone "ZONE"
```

```bash
gcloud compute instances delete on-prem-instance1 --zone zone_name
```

> NOTE: Replace with the zone in which the on-prem-instance1 was created.

6. Delete firewall rules

Type the following to delete the firewall rules. Type "y" to confirm each action when asked:

```bash
gcloud compute firewall-rules delete vpc-demo-allow-custom
```

```bash
gcloud compute firewall-rules delete on-prem-allow-subnets-from-vpc-demo
```

```bash
gcloud compute firewall-rules delete on-prem-allow-ssh-icmp
```

```bash
gcloud compute firewall-rules delete on-prem-allow-custom
```

```bash
gcloud compute firewall-rules delete vpc-demo-allow-subnets-from-on-prem
```

```bash
gcloud compute firewall-rules delete vpc-demo-allow-ssh-icmp
```

7. Delete subnets

Type the following to delete the subnets. Type "y" to confirm each action when asked:

```bash
gcloud compute networks subnets delete vpc-demo-subnet1 --region "REGION"
```

```bash
gcloud compute networks subnets delete vpc-demo-subnet2 --region REGION 2
```

```bash
gcloud compute networks subnets delete on-prem-subnet1 --region "REGION"
```

8. Delete VPC

Type these commands to delete the VPCs. Type "y" to confirm each action when asked:

```bash
gcloud compute networks delete vpc-demo
```

```bash
gcloud compute networks delete on-prem
```
