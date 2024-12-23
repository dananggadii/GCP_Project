# Setup Windows Server on Google Cloud

### 1. Click the navigation menu in the top-left corner.
![image](https://github.com/user-attachments/assets/67d6a8ae-24e1-4f1e-aea1-0fcd58b54c5f)

### 2. Select Compute Engine > VM Instances.
![image](https://github.com/user-attachments/assets/143d3e5f-9d7f-48be-b88e-9e9f3837471f)

### 3. On the VM Instances page, click the "Create Instance" button.
![image](https://github.com/user-attachments/assets/99396153-6ebe-477f-9a86-856da4fd24d3)

### 4. Name the instance and set the region and zone.
![image](https://github.com/user-attachments/assets/806db4db-f359-4099-ba5b-295e1ada1ada)

### 5. Select the instance family type, then set the machine type.
![image](https://github.com/user-attachments/assets/48cf2062-7347-42aa-9759-42c7097c11ae)

### 6. Choose the boot disk by clicking "Change".
![image](https://github.com/user-attachments/assets/372b8bfa-4b15-4c8b-a9d0-c388a412646b)

### 7. Select the boot disk type, OS specifications, and disk size, then click "Select".
![image](https://github.com/user-attachments/assets/cb6803ff-d02f-4336-957a-cf30fba926e3)

### 8. Configure: Service Account > Access Scopes > check "Allow Firewall" > check "Install Ops Agent".
![image](https://github.com/user-attachments/assets/1e212854-0462-401d-bbe8-26bfe26e7ed5)

### 9. Set up the network interface.
![image](https://github.com/user-attachments/assets/af66bcae-43e2-4e64-9360-a708b8f7e4a6)

![image](https://github.com/user-attachments/assets/a42509fc-4df9-44c7-8458-80a685bc2226)

### 10. For Windows Server login, configure a password > click the Terminal icon.
![image](https://github.com/user-attachments/assets/4699c6f0-8761-4188-9453-106b5ff5fa65)

### 11. Run the following command, replacing [name_instance] and [username]:
```
gcloud compute reset-windows-password [name_instance] --zone us-central1-c --user [username]
```
![image](https://github.com/user-attachments/assets/4e83a2be-113a-45b1-a89b-6b66746e06fa)

### 12. Return to the instance page > click "RDP" to connect to the Windows Server.
![image](https://github.com/user-attachments/assets/353c14df-4892-4f2e-b0ff-17241273e657)

### 13. Download the RDP file.
![image](https://github.com/user-attachments/assets/1a136b34-fd90-4ea0-b6f1-34b078733f98)

### 14. Enter port 3389 (RDP port) after the IP > check "Allow" > click "Connect".
![image](https://github.com/user-attachments/assets/526133e4-7419-43c3-9027-44b54699c681)

