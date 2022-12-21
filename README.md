# home-rpi-NFS
NFS server on raspberry pi k3s cluster

## Prereq Steps
### Raspberry Pi Setup
- Please follow this link for instructions on [How to install and setup K3s Cluster on raspberry pi](https://github.com/philgladman/home-rpi-k3s-cluster.git)
- This link will show you how to;
  - Prepare your raspberry pi for kubernetes
  - Spin up K3s
  - Install MetalLb
  - Install Nginx Ingress
- If you already have a Raspberry pi configured with K3s, MetalLb, and Nginx ingress, please move on to [Step 1.)](README.md#step-1---setup-external-drive-for-nas)


## Step 1.) - Setup external drive for NFS Server
- now that the Raspberry pi is up and running with K3s, lets prepare the external drive
- Plug in external drive to rpi.
- run `sudo fdisk -l` to find the drive, should be at the bottom, labeled something like`/dev/sda/`
- run `sudo fdisk /dev/sda` to partition the drive
- type in `d` and hit enter to delete, and then hit eneter to delete the default partition. Do this for each partition on the drive.
- Now that all partions have been deleted, lets create a new partition.
- hit `n` and enter to create a new partition
- hit enter for default for `primary` partition
- hit enter for default for `1 partition number`.
- hit enter for default for `First sector` size.
- hit enter for default for `Last sector` size.
- Partition 1 has been created of type `linux`, now hit `w` to write, this will save/create the partion and exit out of fdisk.
- Now make a filesystem on the newly created partition by running the following command `sudo mkfs -t ext4 /dev/sda1`

## Step 2.) - Mount volume for NFS Server
- create directory to mount drive to `sudo mkdir /NFS-vol`
- To make mount perisistent, edit /etc/fstab file with the following command `sudo vim /etc/fstab/` and add the following to the bottom of the existing mounts. `/dev/sda1 /NFS-vol ext4 defaults 0 2`
- mount the drive `sudo mount -a`

## Step 3.) - Label Master node
- label master node so the NFS Server container will only run on the master node where the external drive is, `kubectl label nodes $(hostname) disk=disk1`

## Step 4.) - Label Master node
- label master n
