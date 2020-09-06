# Lab 2 write-up

## Working with Virtual Machines

### Task 1 : Get access to Google Cloud.
 From command line(Windows), BASH(linux), run  the following command to initialize gcloud and  press Y to login when prompted. 

 `gcloud init`

### Task 2 : Create the VM using the Cloud Console. (command line version)
This command creates a VM called mc-server in us-central1 region us-central1-a zone with default service account with scopes:
- service control enabled
- service management
- stack driver logging write only
- stack driver monitoring write only
- stack trace and
- cloud storage read and write

  with tag minecrast-server, image debian 9 stretch 2020 edition, boot disk space 10GB SSD

`gcloud compute instances create mc-server --zone=us-central1-a --machine-type=n1-standard-1 --service-account=704908768855-compute@developer.gserviceaccount.com --scopes=service-control,service-management,logging-write,monitoring-write,trace,storage-rw  --tags=minecraft-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-ssd --boot-disk-device-name=mc-server-ip --create-disk=mode=rw,size=50,type=projects/qwiklabs-gcp-00-23d9e9d45b01/zones/us-central1-a/diskTypes/pd-ssd,name=minecraft-disk,device-name=minecraft-disk --reservation-affinity=any
`

### Task 3 : Prepare the data disk
- Command creates a directory that serves as the mount point for the data disk
`sudo mkdir -p /home/minecraft`

- Command formats the disk to ext4 file system
`sudo mkfs.ext4 -F -E lazy_itable_init=0,\
lazy_journal_init=0,discard \
/dev/disk/by-id/google-minecraft-disk`

- Command mounts the disk
`sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft`

### Task 4: Install and run the application
- In the SSH terminal for mc-server, this command updates the Debian repositories on the VM
`sudo apt-get update`

- Commands installs the headless JRE, navigate to mount directory and install wget followed by minecraft server
`sudo apt-get install -y default-jre-headless
cd /home/minecraft
sudo apt-get install wget
sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar`

- command initializes minecraft server
`sudo java -Xmx1024M -Xms1024M -jar server.jar nogui`

#### Note: The Minecraft server won't run unless you accept the terms of the End User Licensing Agreement (EULA).

- this command `sudo nano eula.txt` open eula file 
Manually change `eula=false` to `eula=true`

- To create a virtual terminal screen to start minecrast server, this command installs screen following by it's initialization
`sudo apt-get install -y screen
sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui`

- Command detaches the screen and exits ssh
`sudo screen -r mcs
exit`


- Command creates a firewall rule called minecraft-rule which allows ingress tcp traffic on port 25565 with tag minecraft-server with source ranges 0.0.0.0/0
`gcloud compute --project=qwiklabs-gcp-00-23d9e9d45b01 firewall-rules create minecraft-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:25565 --source-ranges=0.0.0.0/0 --target-tags=minecraft-server`


### Task 5: Schedule regular backups

- In mc-server ssh terminl, command creates env variable, a bucket, navigates to home directory and  creates a script
`export YOUR_BUCKET_NAME=my-bucket
gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup
cd /home/minecraft
sudo nano /home/minecraft/backup.sh`

This is the content of the script
```
    #!/bin/bash
    screen -r mcs -X stuff '/save-all\n/save-off\n'
    /usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
    screen -r mcs -X stuff '/save-on\n'
```

- Command makes the script executable and runs it

`sudo chmod 755 /home/minecraft/backup.sh`

`. /home/minecraft/backup.sh`

- Open cron table and append the instruction to run backups every 4 hours

`sudo crontab -e`
Instruction to paste in file: `0 */4 * * * /home/minecraft/backup.sh`

- Command stops screen service
`sudo screen -r -X stuff '/stop\n'`

- Command adds startup and shutdown scripts to VM
`gcloud compute instances add-metadata mc-server --metadata=startup-script-url=https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh,shutdown-script-url=https://storage.googleapis.com/cloud-training/archinfra/mcserver/shutdown.sh`


#### Completion of the lab produced the following result. 
![Image of lab 2](screenshots/working-with-virtual-machines.png)