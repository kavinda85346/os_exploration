### Process Services and Task Manageemnt Commands

##### Commands given below are used for process services and task manageemnt commands.
- View Running Processes
1. Show all running processes.
```
#List all the running processes.
tasklist
```
2. Get more detailed information about running processes.
```
#Get more detailed infromation about the running processes.
wmic process list brief
Get-Process    #PowerShell command to view all running process.
```
- Kill / Stop Processes
1. Terminate a running procress.
```
#Killing the running process under pid 1234
taskkill /pid 1234
Stop-Process -Id 1234    #PowerShell command to kill a running process.
```
- Start Processes / Programs
1. Launch a program
```
#Start a notepad.
start notepad
Start-Process notepad
```
- System Performance Monitoring
1. Launch task manager.
```
#Start task manager.
taskmgr
```
2. Launch resource monitor.
```
#Start resource monitor.
resmon
```
3. Launch performance monitor.
```
#Start performance monitor.
perfmon
```
- Background Tasks / Services
1. List the status and configuration of System Services and Drivers.
```
#View the status and configuration of system services and drivers.
sc query
net start
Get-Service    #View the status and configuration of system services and drivers.
```
- Schdule Task Management
1. List down schduled task
```
#View all schduled tasks.
schtasks /query
```
2. Create a schduled task
```
#Create a task to launch the notepad at 12:25 am.
schtasks /create /tn "MyTask" /tr notepad.exe /sc once /st 12:00
```

