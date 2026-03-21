### Scripting and Shell Management

##### Given below are default commands in Linux which are used for scripting and shell utlities.
1. Shell and Script Basics
    ```
    #!/bin/bash
    echo "Enter your name:"
    read name
    echo "Hello $name"
    ```
2. Variables and Environment
    ```
    #Setting the environment varibales.
    export PATH=$PATH:/new/path
    echo $PATH
    ```
3. Control Flow
    ```
    #A script with conditions.
    if [ $age -gt 18 ]; then
      echo "Adult"
    else
      echo "Minor"
    fi
    ```