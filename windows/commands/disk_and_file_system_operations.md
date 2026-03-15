### Disk and File System Operations Commands

##### Commands given below are used for disk magaement.
1. View disk partitions.
```
#List down the disk volumes in the pc.
disk
```
2. View down the volumes.
```
#Check the volumes in the pc.
volume
```
3. Launch the disk partition tool.
```
#Activate the disk partition tool
diskpart
```
##### Commands given below are used for file system check and repair.
1. View disk errors.
```
#View disk errors in pc.
chkdsk
```
2. Fix disk errors
```
#Fix the disk errors.
chkdsk /f
```
##### Commands given below are used for file and directory management.
1. View directories
```
#Check directorires in a file/disk...
dir
```
2. Move from one directory to another direcotory.
```
#Change the the directory.
cd C:\Users\Dew\Downloads
```
3. Display the folder structure.
```
#View the folder structure in a current directory.
tree
```
4. Replicate file in one location to another desired location.
```
#Copy a file/folder from one to another location
move C:\folder1\report.txt C:\folder2
```
##### Commands given below are used to view disk usage and information.
1. View the size of a specific folder.
```
#Check size of a specific folder.
diskusage C:\Users\Documents
```
2. View the volume a drive.
```
#Check the volume a drive.
volume c
```
3. Rename a drive
```
#Change the label of a drive.
label d e
```
4. View disk space information
```
#Check disk space information of a pc.
 wmic logicaldisk get size,freespace,caption
```
##### Commands given below are used for file system conversion management.
1. Convert from fat to ntfs
```
#FAT to NTFS file conversion.
convert C: /fs:ntfs
```
2. Quickly format a drive.
```
#Format drive within a shorter time period.
format e: /q
```
##### Commands given below are used for file system permission management.
1. Removing uer access for file
```
#Prevent a user accessing the projects folder.
icacls "D:\Projects" /remove JohnDoe
```
2. Quickly format a drive.
```
#Assign the full file ownership to the user.
takeown /f "C:\Folder\Example.txt"
```