### System Information Monitoring

##### Given below are default commands in Linux which are used for system information monitoring.
1. CPU and Process Monitoring
    ```
    #Monitor processes for a specific user only.
    top -u username

    #View the CPU usage for all active tasks.
    pidstat
    ```
2. Memory Monitoring
    ```
    #View RAM usage
    free -h

    #View memory
    vmstat -S M 1 5
    ```
3. Disk Usage and IO Monitoring
    ```
    #Check the disk space usage.
    df -h

    #Statistics since the system was last booted.
    iostat -xz 1 5
    ```
4. Network Monitoring
    ```
    #View active ports and services.
    netstat -tulnp

    #Monitor traffic wlan0
    nload wlan0
    ```
