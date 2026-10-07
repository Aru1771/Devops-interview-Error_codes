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
      1. Trivy isn't installed
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
