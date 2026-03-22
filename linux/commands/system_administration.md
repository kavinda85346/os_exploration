### System Administration

##### Given below are default commands in Linux which are used for system administration.
1. System Information and Monitoring
    ```
    #Check the disk space usage
    df -h

    #View CPU information
    lscpu
    ```
2. User and Group Management
    ```
    #Create a user.
    sudo useradd -m -s /bin/bash alex

    #Delete a user.
    sudo userdel -r alex
    ```
3. Permissions and Ownership
    ```
    #Assigning the execute permission to user.
    chmod u+x script.sh

    #Assigning file permissions to ownership.
    sudo chown jeff report.txt
    ``` 
4. Process and Job Management
    ```
    #List all possess.
    ps aux

    #List all the backgrounds.
    jobs
    ```
 