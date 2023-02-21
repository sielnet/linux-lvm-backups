## Linux LVM backup script

### Main goals of this script:
* Provides better alternative than using disk cloning tools (e.g. Clonezilla) as a bitwise backup strategy
* Creates full image of a disk partition using LVM snapshot and dd
* Doesn't interfere with nor interrupt normal operation of the system and can be run in the background, thanks to the nature of LVM
* Backups are compressed with pigz, utilizing multiple CPU cores
* Can be scheduled to run unattended using cron etc.

![script_running](misc/backup.png)

### Requirements:
* Disk partitions must be provisioned using LVM
* Volume group (VG) must have enough unallocated free space for the snapshot. Script will attempt to use around 5% of the partition size. If the logical volume (LV) being backed up is 60GB in size, then 3GB of free space is needed for the snapshot. As best practice, the backups should be taken during periods of low activity on the system in order to ensure that changes made to the file system during backup process can't outgrow snapshot size and therefore render it (and the backup image) unusable.
* Required software: `pigz`, `openssl`, `pv`

### Setup:
0. Become root / su:
```
sudo -i
```
1. Ensure required software is installed:
```
apt install pigz openssl pv
```
2. Download the script to /usr/local/bin on your system:
```
curl https://raw.githubusercontent.com/sielnet/linux-lvm-backups/main/auto_backup.sh -o /usr/local/bin/auto_backup.sh
```
3. Make it executable:
```
chmod +x /usr/local/bin/auto_backup.sh
```
4. Edit the script and define variables:
```
### DEFINE THESE VARIABLES:

# Volume group. Run "sudo lvs" to check:
VG="vgbox"

# Logical volume inside volume group you want to backup. Run "sudo lvs" to check:
LV="root"

# Output folder (include slash at the end). Ideally it should be on a separate disk and (ofc) MUST NOT be on the logical volume you want to backup:
OUTPUT_FOLDER="/mnt/backups/"

### END OF USER VARIABLES

5. Script can be run at this point or scheduled using cron.

### How to restore?
You can list available Logical Volumes with `sudo lvdisplay` command. Let's assume your target partition is /dev/vg_test/temp. Volume Group in this case is named `vg_test`, Logical Volume is `temp`. To restore the image:

0. Become root / su first:
```
sudo -i
```
1. Run this to restore:
```
cat 'PATH_TO_YOUR_IMAGE' | gzip -dk | dd of=/dev/vg_test/temp bs=50M
```
This will wipe all the existing data on that partition! Target partition must be the same size or bigger than the source one.
