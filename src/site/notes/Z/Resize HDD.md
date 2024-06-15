---
{"dg-publish":true,"dg-path":"HowTo/resize-hdd","permalink":"/how-to/resize-hdd/","tags":["📝/🌲"],"noteIcon":"tree","created":"2023-02-16T14:28","updated":"2024-06-15T22:36"}
---

## Show disc type

```shell
# Show filesystem types
sudo df -hT

# Show partition tables
sudo lsblk
```
## ext4 without lvm
```shell
# Requirements
sudo apt install scsitools
sudo rescan-scsi-bus --forcerescan

# Resize disk
sudo cfdisk /dev/{{ disk }}

# Resize partition
sudo resize2fs /dev/sdb1
```

## ext4 with lvm
```shell
# Requirements
sudo apt install scsitools
sudo rescan-scsi-bus --forcerescan

# Enter cfdisk and choose partition to enlarge or create new partition
sudo cfdisk /dev/sdb

# Make lvm aware of the changed partitions / partition size:
# Shows current state
sudo pvdisplay

# Resize physical volume
sudo pvresize /dev/sdb3

# Resize the lvm
sudo lvextend -l +``100``%FREE /dev/mapper/deagsxxxxx--vg-root 

# Resize logical volume
sudo resize2fs /dev/mapper/deagsxxxxx--vg-root 

# Show file system state
sudo df -h 
```

## xfs
```shell
# Requirements
sudo apt install scsitools
sudo rescan-scsi-bus --forcerescan

# Resize disk
sudo cfdisk /dev/sdb

# Resize xfs file system
sudo xfs_growfs /dev/sdb1
```


## Known errors

### GPT PMBR size mismatch...
```shell
# Fix signature with
sudo parted -l
```