### User and Security Commands

##### Commands given below are used for user and security commands.
- User Management
    1. View user accounts
        ```
        #View user accounts.
        net user
        ```
    2. Disable an active user
        ```
        #Deativate an active user.
        net user username /active:no
        ```
- Localgroup Management
    1. View local groups
        ```
        #View local groups.
        net localgroup
        ```
    2. Disable a localgroup
        ```
        #Remove a localgroup.
        localgroup groupname temporary guests /delete
        ```
- File and Folder Permission Management
    1. View all permissions assigned to file.
        ```
        #List permissions assigned to file delta
        icacls delta
        ```
    2. Remove file permissions assigned to a user.
        ```
        icacls delta /remove user
        ```
- Firewall and Defender
    1. Show firewall status.
        ```
        #View the overoll status.
        netsh advfirewall show allprofiles
        ```
    2. Check Windows Defender status.
        ```
        #Check wither Windows Defender status active.
        sc query WinDefend
        ```

    