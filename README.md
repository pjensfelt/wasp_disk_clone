# wasp_disk_clone

The scripts in here are used to make copies of the WASP external drive with Linux installed. They come in two version, one for PC and one for Mac. Mac requires that the EFI partiion is hfs+.

## Prepare the disk to be cloned

First ensure which disk to clone
sudo fdisk -l

The disk is partitioned using GPT.

### Mac disk

### PC disk
```
Disk /dev/sdb: 465,8 GiB, 500107861504 bytes, 976773167 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 33553920 bytes
Disklabel type: gpt
Disk identifier: 3A96D237-422F-4B63-8699-4FBD298BE1FC

Device         Start       End   Sectors  Size Type
/dev/sdb1       2048    677887    675840  330M EFI System
/dev/sdb2     677888 420108287 419430400  200G Linux filesystem
/dev/sdb3  420108288 453662719  33554432   16G Linux swap
/dev/sdb4  453662720 873093119 419430400  200G Linux filesystem
/dev/sdb5  873093120 956979199  83886080   40G Microsoft basic data
```

### 
Shrink the / partition to a almost minimum so that we hve the smallest possible amount to copy. With Ubuntu 16.04 and all the software installed for Module 1 of the Autonomous Systems course, this is roughly 13GB. I shrunk / to 16GB.

Assume below that it is /dev/sdb

sudo sgdisk --backup=backup_WASP_5part_16GB_PC.sgdisk /dev/sdb  
Generates lots of warnings

sudo umount /dev/sdb1
sudo umount /dev/sdb2
sudo umount /dev/sdb4
sudo umount /dev/sdb5


### Store the UUIDs
sudo blkid /dev/sdb > uuid-sdb.txt
sudo blkid /dev/sdb1 > uuid-sdb1.txt
sudo blkid /dev/sdb2 > uuid-sdb2.txt
sudo blkid /dev/sdb3 > uuid-sdb3.txt
sudo blkid /dev/sdb4 > uuid-sdb4.txt
sudo blkid /dev/sdb5 > uuid-sdb5.txt

### Copy the partition data (that we need to keep)
sudo dd if=/dev/sdb1 of=sdb1_PC.img bs=4096      # roughly 3s
sudo dd if=/dev/sdb2 of=sdb2_PC.img bs=4096      # roughly 120s

# We also need to do this for a disk with Mac formatting, ie a hfs+ partition instead of fat32 for sdb1
# We need both sdb1 and the partition table

## Make a new disk with script

Run the script

## Make a new disk by hand

Go to the directory where the cloned partitions are (also where this file most likely is)

# First ensure which disk to clone into
sudo fdisk -l

Assume below that it is /dev/sdb


# Make sure the partitions are unmounted
sudo umount /dev/sdb1
sudo umount /dev/sdb2
sudo umount /dev/sdb4
sudo umount /dev/sdb5

# Restore the GPT partition table
sudo sgdisk -g --load-backup=backup_WASP_5part_16GB_PC.sgdisk /dev/sdb  
# You might need to use option -g if it is not already a GPT
# You may need to plug out and in again for it to understand that it has been changed

sudo umount /dev/sdb1
sudo umount /dev/sdb2
sudo umount /dev/sdb4
sudo umount /dev/sdb5

# Restore the data to the disks
sudo mkswap -U `cat uuid-sdb3.txt | sed -n '/sdb3/s/.*UUID=\"\([^\"]*\)\".*/\1/p'` /dev/sdb3
sudo mkfs.exfat -n EXFAT /dev/sdb4
sudo mkdosfs -F 32 -n FAT32 -I /dev/sdb5
sudo dd if=sdb1_PC.img of=/dev/sdb1 bs=4096		# Should take about 5s
sudo dd if=sdb2_PC.img of=/dev/sdb2 bs=4096		# Should take about 180s

# Grow the / partition to fill the whole space
# Warning this assumes a very specific geometry of the disk!!!!!
sudo parted /dev/sdb resizepart 2 420108287s
sleep 2
sudo e2fsck -f /dev/sdb2 
sleep 2
sudo resize2fs /dev/sdb2 



