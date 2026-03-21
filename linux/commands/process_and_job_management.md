### Process and Job Management

##### Given below are default commands in Linux which are used for process and job manageemnt.
1. Process Management
    - View Process
        ```
        #View all running processes.
        ps aux
        ```
    - Find a specific process.
        ```
        #Filter a specific process.
        ps aux | grep nginx
        ```
    - Process Identification
        ```
        #Identify a process.
        pidof nginx
        ```
    - Kill / Terminate Processes
        ```
        #Terminating the process 1234.
        kill 1234
        ```
2. Job Management
    - View all jobs
        ```
        #List all the jobs.
        jobs
        ```
    - Kill a running job.
        ```
        #Terminate a job.
        kill %1
        ```
        