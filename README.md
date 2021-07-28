# Installing Ubuntu Server 20.04.02 LTS

## Update operating system

After installing the operating system, make sure it's updated.

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

## Install NFS

$ sudo apt install nfs-kernel-server

Once the installation is completed, the NFS services will start automatically.

Check what NFS versions that are enabled:

```
$ sudo cat /proc/fs/nfsd/versions
```

# Prepare the filesystem for the NFS volume

List the disks on the server:

```
$ sudo lshw -class disk -short (or lsblk -o KNAME,TYPE,SIZE,MODEL)
```

The output should look something like this:
```
H/W path      Device       Class      Description
=================================================
/0/1/0.0.0    /dev/cdrom   disk       Virtual CD/ROM
/0/2/0.0.0    /dev/sda     disk       136GB Virtual Disk
/0/3/0.0.0    /dev/sdb     disk       136GB Virtual Disk
```

In this case the /dev/sda is refering to the operating system disk and /dev/sdb is the data disk.

Warning! Make sure to understand what disks you have mounted to make sure you are not deleting data unintentionally.

## Format the data disk

This assumes that the data disk is blank.

Create a new partition by issuing the following command:
```
$ sudo fdisk /dev/sdb
```

Enter 'n' at the fdisk command prompt to add a new partition. Select defaults for all the inputs and remember to select "w" when returning to the fdisk command prompt in order to write table to disk and exit.

Format the partition as XFS:

```
sudo mkfs.xfs -L nfs /dev/sdb1
```

## Mount the new filesystem

Create the mountpoint

```
$ sudo mkdir /mnt/nfs
```

Mount the new filesystem

```
$ sudo mount -t xfs /dev/sdb1 /mnt/nfs
```

## Make sure new filesystem is mounted at boot

Get UUID for newly created file system

```
$ sudo blkid /dev/sdb1
```

Add this to "/etc/fstab":

```
UUID=<UUID> /mnt/nfs xfs defaults 1 1
```

## Export the filesystems

```
sudo nano /etc/exports

```

Add the following lines:
```
/mnt/nfs/<subdirectory-to-export>         172.20.0.0/16(rw,sync,no_subtree_check,no_root_squash)
```

Export the shares:
```
$ sudo exportfs -ar
```

View the exports:
```
$ sudo exportfs -v
```

## Install NFS on client

```
$ sudo apt update
$ sudo apt install nfs-common
```