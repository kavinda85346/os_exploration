### Disk and File Systems

##### Given below are default commands in Linux which are used for disk and file systems.
1. Check Disk and Filesystem Information
    - Disk Usage
        ```
        #View mounted disks and free space (human-readable).
        df -h
        ```
    - Directory size
        ```
        #View the total size of a directory.
        du -sh /path
        ```
2. List and Inspect Disks and Partitions
    - View disks
        ```
        #Tree view of disks & partitions
        ls
        ```
    - Detailed disk info
        ```
        #Lists all disks & partitions
        sudo fdisk -l
        ```
3. Mounting and Unmounting
    - Mount a filesystem
        ```
        #Mount a filesystems.
        sudo mount /dev/sdb1 /mnt
        ```
    - Unmount
        ```
        #Unmount a filesystems.
        sudo umount /mnt
        ```
4. Create and Manage Filesystems
    - Create filesystem.
        ```
        #Create ext4 filesystem
        mkfs.ext4 /dev/sdb1
        ```
    - Label a filesystem.
        ```
        #ext filesystems
        e2label /dev/sdb1 mydisk
        ```