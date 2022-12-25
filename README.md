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
- If you already have a Raspberry pi configured with K3s, MetalLb, and Nginx ingress, please move on to [Step 1.)](README.md#step-1---setup-external-drive-for-nfs-server)


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
- create directory to mount drive to `sudo mkdir /nfs-vol`
- To make mount perisistent, edit /etc/fstab file with the following command `sudo vim /etc/fstab/` and add the following to the bottom of the existing mounts. `/dev/sda1 /NFS-vol ext4 defaults 0 2`
- mount the drive `sudo mount -a`
- create a directory in the new mount for our test nginx pod that will us later, `sudo mkdir /nfs-vol/nginx-vol`

## Step 3.) - Label Master node
- label master node so the NFS Server container will only run on the master node where the external drive is, `kubectl label nodes $(hostname) disk=disk1`

## Step 4.) - Install NFS Driver & NFS Server
- Install the NFS Driver with `curl -skSL https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/v4.1.0/deploy/install-driver.sh | bash -s v4.1.0 --`
- if you created a different name for your NFS Volume, use this command to change the name from `/nfs-vol` to your custom name, `sed -i "s|/nfs-vol|<your-custom-name>|g" nfs-driver/kube-nfs-server.yaml`
- Create the nfs namespace and the nfs-server `kubectl apply -k nfs-driver/.`

## Step 5.) - Test NFS Server
- Now that our NFS server is up and running, lets test it out with an nginx pod
- Create the test pod `kubectl apply -f nfs-driver/test-nfs.yaml`
- Once the nginx pod is up and running, run `kubectl exec nginx-nfs-example -- bash -c "findmnt /var/www -o TARGET,SOURCE,FSTYPE"`
- Output should look like below
```bash
TARGET   SOURCE                   FSTYPE
/var/www 10.43.184.230:/nginx-vol nfs4
```
- We can further test by executing into the pod, and creating a test file in the `/var/www` folder which is mounted to the node at `/nfs-vol/nginx-vol`. Do this by running this command `kubectl exec nginx-nfs-example -- bash -c "echo 'this is a test' > /var/www/test.txt"`.
- Now we can check the `/nfs-vol/nginx-vol` on the master node to see if our test volume is there, `cat /nfs-vol/nginx-vol/test.txt`
