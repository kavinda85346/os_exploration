### Adavanced Tools

##### There are default commands in Linux which are used to launch tools.

##### We can classify these commands into 10 categories and they are listed below.
1. System Monitoring and Performance
2. Process Management
3. File and Text Processing
4. Disk and File System Management
5. Networking
6. File Searching and Investigation
7. Permissions and Ownership
8. System Information
9. Powerful Pipe and Redirection Features  

##### Commands that are belong to each of these categories are listed below along with their usage.
1. System Monitoring and Performance
```
#View Real-time process.
top

#View System Load Averages.
uptime

#View memory usage
free -h

#View virtual memory Statistics.
vmstat

#View CPU and disk I/O Statistics.
iostat

#View CPU Usage per Processor.
mpstat
```
2. Process Management
```
#View all running processes.
ps aux

#Start process with priority.
nice

#Find the processes related to kubelite.
ps aux | grep kubelite
```
3. File and Text Processing.
```
#Count the lines of hello.sh file.
wc hello.sh

#Compare the difference between hello.sh and super.sh files.
diff hello.sh super.sh

#Find the word that includes "thing".
cat hello.sh | grep thing
```
4. Disk and File System Management
```
#View the disk usage partitions.
df -h

#View the size of the directory.
du -sh try

#List blockdevices
lsblk
```
5. Networking
```
#Testing end to end connectivity
ping 127.0.0.1

#View the route that a packet has from point a to point b.
traceroute 8.8.8.8

#View open sockets
ss
```
6. File Searching and Investigation
```
#Find files with .sh extension.
find / name "*.sh"

#Locate hello.sh file.
locate hello.sh
```
7. Permissions and Ownership
```
#Changing the file ownership alng with all file permissions.
sudo chown 7 try

#Allocating read only file permissions to the user.
chmod u+r hello.sh
```
8. System Information
```
#View the current user.
whoami

#View the current directory.
pwd

#View cpu details.
lscpu

#View PCI Hardware Details
lspci
```
9. Powerful Pipe and Redirection Features
```
Append the hello.sh by including the text bonjour
echo bonjour >> hello.sh
```

