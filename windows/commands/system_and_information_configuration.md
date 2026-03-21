### System and Information Configuration Commands

##### Commands given below are used for system and information configuration commands.
- Basic System Information
    1. View system configuration.
        ```
        #Displays detailed system configuration (OS, RAM, CPU, uptime, patches).
        systeminfor
        ```
    2. View hostname
        ```
        #Display the computer name
        hostname
        ```
    3. View OS infor
        ```
        #View windows information.
        ver
        ```
- Hardware Information
    1. View CPU and BIOS    
        ```
        #List down CPU and BIOS.
        wmic cpu get name
        ```
    2. View Memory
        ```
        #View Memory
        wmic MEMORYCHIP get BankLabel, Capacity, Speed
        ```
    3. View Disk / Storage
        ```
        #List down diskdrive.
        wmic diskdrive get model,size
        ```
- Network Information
    1. View Network Details    
        ```
        #View ip configuration.
        ipconfig /all
        ```
    2. View Mac Address
        ```
        #View mac address information.
        getmac
        ```
    3. View Active connections and ports.
        ```
        #View active connections and ports.
        netstat -ano
        ```
- Installed Softwares and Drivers
    1. View Software Installed     
        ```
        #View software information
        wmic product get name,version
        ```
    2. View Drivers
        ```
        #View drive information.
        driverquery
        ```

