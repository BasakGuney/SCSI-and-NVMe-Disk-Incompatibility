# SCSI and NVMe Disk Incompatibility

## Environment Information  
>  We are working with VMware Workstation Pro VM provider and we have a Rocky9 machine. The disk which has Rocky9 installed is type NVMe and we want to add an extra disk and extend the root filesystem.  

## Explanation
> When we add a hard disk with type SCSI on a VM that uses NVMe type as main disk, the VM counters some problem at booting and cannot start the machine. Possible cause of this is,  
When you tyry to combine a high-performance NVMe disk with a lower-performance disk like SCSI for the root filesystem, it can lead to problems if the setup isn't managed carefully, especially with respect to booting, disk failures, and performance. So it is better use same disk type(NVMe in our case) while extending root filesystem.

## Set-up
### 1. Adding Disk
```txt            
First Power Off the machine.
``` 

```txt            
Right click to your VM in VMware Workstation 
and click on "Settings".
``` 

```txt            
Find "Add" button and click it on the Hardware screen.
```
```txt            
Don't forget to choose disk type NVMe and set the memory size as you wish then you can continue with the defaults.
``` 
### 2. Expanding Filesystem in Command Line
```txt            
Power On your VM.
```
            
If you're encountering a problem at booting you can check [Booting Problem](#booting-problem) part.  
<br>

```bash
lsblk
```
Run the command above on your machine and look for the newly added NVMe.  
<br>
If you can't see the the disk in the list you can try the command below and run lsblk again:
```bash
echo "- - -" | sudo tee /sys/class/scsi_host/host*/scan
```
<br>

```bash
df -h
```
Run the command above to check your root filesyste's size. You can compare it after extending the filesystem.
<br>

```bash
vgs
```
Run the command above to check the volume group name which our main disk uses.(it's usually rl).

<br>

```bash
vgextend <vg_name> /dev/<added_disk_name>
```
You can extend the volume group with the newly added disk using the coomand above.  
<br>

```bash
lvdisplay
```
Run the command above and look for the logical volume path which root is mounted on.  
<br>

```bash
lvextend -l +100%FREE  <path-to-root-lv>
```
The command above will extend the logical volume with the all free space in the volume group where it is located.  
<br>
<br>

You can extend the filesystem according to your filesystem type( XFS, ext4 etc.)    

To check your filesystem type you can use the following command:
```bash
df -T
```

For XFS,
```bash
xfs_growfs /
```
The command above will extend the root filesystem.  
<br>
For ext4,
```bash
resize2fs /dev/<added_disk_name>
```
The command above will extend the root filesystem.  
<br>

```bash
df -h
```
Check the size of the root filesystem wether it is extended or not.

### Booting Problem
> If you're facing with the error operating system not found, it might chooses wrong disk to boot. You can solve this issue by changing the boot order from BIOS menu.

```txt
 1. Restart the VM.
```
```txt
 2. Press Esc before booting begins.
```
```txt
 3. Choose <Enter Setup> option.
```
```txt
 4. Go to Boot menu.
```
```txt
 5. Open Hard Drive by pressing enter on it in the Boot menu.
```
```txt
 6. Check the disk you want to boot and reorder disks by pressing +/- and move the desired disk to the top.
```
```txt
7. Then press F10 to Save and Exit.
```
 
