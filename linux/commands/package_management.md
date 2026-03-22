### Package Management

##### Given below are default commands in Linux which are used for package management.

1. Refresh package list.
    ```
    #Refresh package list
    sudo apt update
    ```
2. Upgrade installed packages.
    ```
    #Upgrade installed packages.
    sudo apt upgrade
    ```
3. Upgrade with dependency changes.
    ```
    #Upgrade with dependency changes
    sudo apt full-upgrade
    ```
4. Install package
    ```
    #Install package.
    sudo apt install nginx
    ```
5. Remove package
    ```
    #Remove (keep config files)
    sudo apt remove nginx
    ```
6. Cleanup package
    ```
    #Remove unused dependencies.
    sudo apt autoremove
    
    #Clear cached packages.
    sudo apt clean
    ```