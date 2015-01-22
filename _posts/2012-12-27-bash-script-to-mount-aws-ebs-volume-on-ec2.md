---
title: Bash script to mount AWS EBS Volume on EC2
tags:
  - web development
  - deployment
  - aws
description: "Here I show the exact steps to write a script automating deployment of an EBS volume to an EC2 instance"
---

I got tired of SSHing into an Amazon EC2 instance every time I spun one up and writing the same code to mount an EBS volume.  *(I also had to search around to find out how to do this in the first place - hopefully this helps anyone else looking as well.)*

I started doing this with the `Net:SSH` Ruby library, as it can accept a number of commands in the same SSH session via a block, but I found the simplest way to do this was to write one script to handle the SSH connection to the server, and then pipe a second local script into the SSH session to accomplish the task.

## Script 1 ##

You would call this script like so: `/path/to/the/script/script1.sh amazonEC2 xx.xx.xx.xx`.  Make sure to `chmod +x` the script.

```bash
#!/usr/bin/env bash

if [ "$1" = "amazonEC2" ]; then
  USER="ubuntu"
  SERVER_ADDRESS=$2
else
  echo 'Invalid argument...'
  exit 1
fi

ssh $USER@$SERVER_ADDRESS 'bash -s' < script2.sh
```

## Script 2 ##

Ensure this script is in the same directory as the first.

```bash
#!/usr/bin/env bash

MOUNT_NAME="snaphost_data"

sudo mkfs.ext4 /dev/xvdf
sudo mkdir -m 000 /$MOUNT_NAME
echo "/dev/xvdf /$MOUNT_NAME auto noatime 0 0" | sudo tee -a /etc/fstab
sudo mount /dev/xvdf /$MOUNT_NAME
```

Set the `MOUNT_NAME` variable to whatever you would like the drive to be mounted as, and you are away to the races!
