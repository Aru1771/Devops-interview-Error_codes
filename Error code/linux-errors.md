Linux Exit Codes & Error Troubleshooting
----------------------------------------------

Most important exit codes for DevOps
----------------------------------------

    | Exit code | Meaning | DevOps relevance |
    |---       :|   ---|---|
    | `0` | Success | Command completed |
    | `1` | General error | Application/script failure |
    | `2` | Incorrect usage / file or directory issue | Shell scripts, Linux commands |
    | `126` | Command found but cannot execute | Permissions |
    | `127` | Command not found | PATH/package/configuration |
    | `128` | Invalid exit argument | Scripts/processes |
    | `137` | Killed by `SIGKILL` | **Very important: OOM/container memory** |
    | `139` | Segmentation fault | Application crash |
    | `143` | Killed by `SIGTERM` | Graceful container/application termination |


1 ---> SUCCESS

Exit code 1 --> General failure --> DevOps point of view--> The application/process failed.

      You may see:
      Jenkins build failed
      Docker container exited with code 1
      Kubernetes container terminated with exit code 1

Exit code 2 — Command/file problem --> is commonly associated with incorrect command usage or missing files in utilities such as ls.

    Is the file missing?
    Wrong directory?
    Wrong filename?
    Wrong workspace?


Exit code 126 — Permission problem --> DevOps troubleshooting thought process

    126
     ↓
    Command/script exists
     ↓
    But cannot execute
     ↓
    Check permissions
     ↓
    ls -l
     ↓
    chmod / ownership / filesystem restrictions

Exit code 127 — Command not found --> The shell couldn't find the command.


      Suppose Jenkins executes:
      trivy image myapp:latest
      
      and gets:
      trivy: command not found
      
      Possible causes:
      1. Trivy isn't installed  CMDS: which trivy, command -v trivy, trivy --version
      2. Trivy isn't in PATH
      3. Wrong Jenkins agent
      4. Tool installation failed
      5. Different container/agent than expected

Exit code 137 — VERY IMPORTANT 🚨 --> This is one of the most important errors for a Kubernetes/DevOps engineer.

     process was killed with SIGKILL
     137 means SIGKILL. Not always OMM kill

     SIGKILL = 9
    Therefore:
    128 + 9 = 137


     Kubernetes example
    You run:
    kubectl describe pod my-app
    
    and see:
    Last State:     Terminated
    Reason:         OOMKilled
    Exit Code:      137
    OOM killing is a very common reason for seeing 137 in containers, but the root cause still needs confirmation.


Exit code 143 — SIGTERM  --> A properly behaved application should handle SIGTERM and perform graceful shutdown.


    IGTERM = 15
    Therefore:
    128 + 15 = 143

    DevOps example
    
    During deployment:
    
    Old Pod
       ↓
    SIGTERM
       ↓
    Graceful shutdown
       ↓
    New Pod starts
    
    This is normal behavior.
    So 143 doesn't necessarily mean something is broken.


10. Exit code 139 — Segmentation fault -->This usually indicates a program accessed invalid memory.

         139 = 128 + 11



Your DevOps troubleshooting mindset
-------------------------------------

    ERROR
     ↓
    What exactly failed?
     ↓
    Check logs
     ↓
    Check exit code
     ↓
    Check recent deployment/config change
     ↓
    Check dependencies
     ↓
    Check resource usage
     ↓
    Identify root cause
     ↓
    Fix
     ↓
    Prevent recurrence


Example:

    Pod CrashLoopBackOff
           ↓
    kubectl describe pod
           ↓
    Exit Code: 137
           ↓
    Reason: OOMKilled
           ↓
    Check memory usage
           ↓
    Check container memory limit
           ↓
    Check JVM Xmx
           ↓
    Determine actual cause


🚀 Day 2 — Linux Production Errors
-----------------------------------

We'll cover:

    1. Disk space full
    2. Inode exhaustion
    3. High CPU
    4. High memory
    5. Process problems
    6. Permission/ownership problems
    7. How to approach a Linux incident

Disk Space Full — Very Common 🚨
---------------------------------

Imagine your application suddenly stops writing logs.
You check:

    df -h

and see:

    Filesystem      Size  Used Avail Use%
    /dev/xvda1       30G   30G     0  100% /
So applications may fail to:

    - write logs
    - create temporary files
    - create new files
    - write application data
    - start properly

You might see errors such as:

    No space left on device

First troubleshooting step
--------------------------

If you see:
No space left on device

run:

    df -h

df tells you: Which filesystem is full?

Find the large directory

Suppose / is full.

Run:

sudo du -sh /*

You may find:

    2G     /home
    1G     /opt
    20G    /var
    3G     /usr

Now investigate /var.

Then you need to find what is consuming the space.


Now investigate /var.

    sudo du -sh /var/*

Maybe:

    18G    /var/log
    1G     /var/lib

Now you know:

    /var/log → 18G

Then investigate further:

    sudo du -sh /var/log/*

You might discover:

    15G    /var/log/application.log
    2G     /var/log/messages
    1G     /var/log/secure

Inode exhaustion
--------------------  

df -i
   ↓
How many FILE ENTRIES/inodes are used?

Even if those files don't consume much total storage, they consume inodes.

Why?

You may have millions of tiny files.


High CPU:
---------

Now imagine users report:

    "Application is very slow."

use : top
What do you do?

    Don't immediately restart the server.

    Infinite loop
    High traffic
    Expensive database queries
    Excessive garbage collection
    Thread problems
    Application bug
    Large batch processing

High Memory:
------------

run: free -h and run: top to find which process is useing


You might find:

    java    5.8G

Then investigate the application.

For Java:
    
    -Xms
    -Xmx
    heap
    GC
    threads
    metaspace
    off-heap/native memo

Process problems:
-----------------

Check:

    ps -ef | grep java

or:

    systemctl status myapp

You might see:

    Active: failed

Then:

    journalctl -u myapp

This can show why the service failed.

Possible causes:

    Configuration error
    Port already in use
    Missing environment variable
    Missing file
    Permission issue
    Dependency unavailable
    Out of memory
    Application crash
    Suppose your application isn't running.


🧠 Production troubleshooting flow
------------------------------------

For Linux incidents, remember this basic flow:

    User reports application problem
                ↓
    Check server health
                ↓
    CPU
    Memory
    Disk
    Network
    Processes
                ↓
    Identify abnormal resource
                ↓
    Find process/application
                ↓
    Check logs
                ↓
    Find root cause
                ↓
    Fix
                ↓
    Prevent recurrence
