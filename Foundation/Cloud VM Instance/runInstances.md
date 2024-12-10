# Working with Virtual Machines

### 1. Create the VM

1. In the Cloud Console, on the Navigation menu (Navigation menu), click Compute Engine > VM instances.

2. Click Create Instance.

3. Specify the following and leave the remaining settings as their defaults:

| Property                                | Value                               |
| --------------------------------------- | ----------------------------------- |
| Name                                    | Type a name for your VM             |
| Region                                  | region_name                         |
| Zone                                    | zone_name                           |
| Series                                  | E2                                  |
| Machine type                            | e2-standard-2(2 vCPUs, 8 GB memory) |
| Boot disk                               | Debian GNU/Linux 12 (bookworm)      |
| Boot disk type                          | SSD persistent disk                 |
| Identity and API access > Access scopes | Set access for each API             |
| Storage                                 | Read Write                          |

4. Click Advanced options.

5. Click Disks and backups. You will add a disk to be used for game storage.

6. Click Add new disk.

7. Specify the following and leave the remaining settings as their defaults:

| Property         | Value                         |
| ---------------- | ----------------------------- |
| Name             | Type a name for your volume   |
| Disk type        | SSD Persistent Disk           |
| Disk Source type | Blank disk                    |
| Size (GB)        | 50                            |
| Encryption       | Google-managed encryption key |

8. Click Save. This creates the disk and automatically attaches it to the VM when the VM is created.

9. Click Networking.

10. Specify the following and leave the remaining settings as their defaults:

| Property              | Value                               |
| --------------------- | ----------------------------------- |
| Network tags          | minecraft-server                    |
| Network interfaces    | Click default to edit the interface |
| External IPv4 address | Reserve Static External IP Address  |
| Name                  | type a name for EIP                 |

11. Click Reserve.

12. Click Done.

13. Click Create.

### 2. Prepare the data disk

1. For `vm_name`, click SSH to open a terminal and connect.

2. To create a directory that serves as the mount point for the data disk, run the following command:

```bash
sudo mkdir -p /home/minecraft
```

3. To format the disk, run the following command:

```bash
sudo mkfs.ext4 -F -E lazy_itable_init=0,\
lazy_journal_init=0,discard \
/dev/disk/by-id/google-minecraft-disk
```

4. To mount the disk, run the following command:

```bash
sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft
```

### 3. Install and run the application

1. In the SSH terminal for mc-server, to update the Debian repositories on the VM, run the following command:

```bash
sudo apt-get update
```

2. After the repositories are updated, to install the headless JRE, run the following command:

```bash
sudo apt-get install -y default-jre-headless
```

3. To navigate to the directory where the persistent disk is mounted, run the following command:

```bash
cd /home/minecraft
```

4. To install wget, run the following command:

```bash
sudo apt-get install wget
```

5. If prompted to continue, type Y.

6. To download the current Minecraft server JAR file (1.11.2 JAR), run the following command:

```bash
sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
```

Initialize the Minecraft server

1. To initialize the Minecraft server, run the following command:

```bash
sudo java -Xmx1024M -Xms1024M -jar server.jar nogui
```

2. To see the files that were created in the first initialization of the Minecraft server, run the following command:

```bash
sudo ls -l
```

3. To edit the EULA, run the following command:

```bash
sudo nano eula.txt
```

4. Change the last line of the file from eula=false to eula=true.

5. Press Ctrl+O, ENTER to save the file and then press Ctrl+X to exit nano.

Create a virtual terminal screen to start the Minecraft server

1. To install screen, run the following command:

```bash
sudo apt-get install -y screen
```

2. To start your Minecraft server in a screen virtual terminal, run the following command (using the -S flag to name your terminal mcs):

```bash
sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui
```

Detach from the screen and close your SSH session

1. To detach the screen terminal, press Ctrl+A, Ctrl+D. The terminal continues to run in the background. To reattach the terminal, run the following command:

```bash
sudo screen -r mcs
```

2. If necessary, exit the screen terminal by pressing Ctrl+A, Ctrl+D.

3. To exit the SSH terminal, run the following command:

```bash
exit
```

### 4. Allow client traffic

Create a firewall rule

1. In the Cloud Console, on the Navigation menu (Navigation menu), click VPC network > Firewall.

2. Click Create firewall rule.

3. Specify the following and leave the remaining settings as their defaults:

| Property            | Value                         |
| ------------------- | ----------------------------- |
| Name                | type firewall name            |
| Network             | vpc_name                      |
| Target              | Specified target tags         |
| Target tag          | vm_name                       |
| Source filter       | IPv4 Ranges                   |
| Source IPv4 ranges  | 0.0.0.0/0                     |
| Protocols and ports | Specified protocols and ports |

4. For tcp, specify port 25565.

5. Click Create. Users can now access your server from their Minecraft clients.

Verify server availability

1. Navigate to VPC network.

2. In the left pane, click IP addresses.

3. Locate and copy the External IP address for the mc-server VM.

4. Use [Minecraft Server Status](https://mcsrvstat.us/) to test your Minecraft server.

### 5. Schedule regular backups

Create a Cloud Storage bucket

1. On the Navigation menu, click Compute Engine > VM instances.

2. For mc-server, click SSH.

3. Create a globally unique bucket name, and store it in the environment variable YOUR_BUCKET_NAME. To make it unique, you can use your Project ID. Run the following command:

```bash
export YOUR_BUCKET_NAME=<Enter your bucket name here>
```

4. Verify it with echo:

```bash
echo $YOUR_BUCKET_NAME
```

5. To create the bucket using the gcloud storage tool, part of the Cloud SDK, run the following command:

```bash
gcloud storage buckets create gs://$YOUR_BUCKET_NAME-minecraft-backup
```

> Note: To make this environment variable permanent, you can add it to the root's .profile by running this command:
>
> ```bash
> echo YOUR_BUCKET_NAME=$YOUR_BUCKET_NAME >> ~/.profile
> ```

Create a backup script

1. In the mc-server SSH terminal, navigate to your home directory:

```bash
cd /home/minecraft
```

2. To create the script, run the following command:

```bash
sudo nano /home/minecraft/backup.sh
```

3. Copy and paste the following script into the file:

```bash
#!/bin/bash
screen -r mcs -X stuff '/save-all\n/save-off\n'
/usr/bin/gcloud storage cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
screen -r mcs -X stuff '/save-on\n'
```

4. Press Ctrl+O, ENTER to save the file, and press Ctrl+X to exit nano.

5. To make the script executable, run the following command:

```bash
sudo chmod 755 /home/minecraft/backup.sh
```

Test the backup script and schedule a cron job

1. In the mc-server SSH terminal, run the backup script:

```bash
. /home/minecraft/backup.sh
```

2. After the script finishes, return to the Cloud Console.

3. To verify that the backup file was written, on the Navigation menu ( Navigation menu icon), click Cloud Storage > Buckets.

4. Click on the backup bucket name. You should see a folder with a date-time stamp name. Now that you've verified that the backups are working, you can schedule a cron job to automate the task.

5. In the mc-server SSH terminal, open the cron table for editing:

```bash
sudo crontab -e
```

6. When you are prompted to select an editor, type the number corresponding to nano, and press ENTER.

7. At the bottom of the cron table, paste the following line:

```bash
0 */4 * * * /home/minecraft/backup.sh
```

> Note: That line instructs cron to run backups every 4 hours.

8. Press Ctrl+O, ENTER to save the cron table, and press Ctrl+X to exit nano.

### 6. Server maintenance

Connect via SSH to the server, stop it and shut down the VM

1. In the mc-server SSH terminal, run the following command:

```bash
sudo screen -r -X stuff '/stop\n'
```

2. In the Cloud Console, on the Navigation menu ( Navigation menu icon), click Compute Engine > VM instances.

3. Select mc-server.

4. Click Stop.

5. In the confirmation dialog, click Stop to confirm. You will be logged out of your SSH session.

Automate server maintenance with startup and shutdown scripts

1. Click `vm_name`.

2. Click Edit.

3. For Metadata, click + Add Item and specify the following:

| Property            | Value                                                                        |
| ------------------- | ---------------------------------------------------------------------------- |
| startup-script-url  | https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh  |
| shutdown-script-url | https://storage.googleapis.com/cloud-training/archinfra/mcserver/shutdown.sh |

4. Click Save.
