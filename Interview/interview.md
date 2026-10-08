1. my application uploading large files to s3. suddenly upload time increases from 20 sec to several min. EC2 cpu and mem is normal. How would you troubleshoot it ?

        A) 1. Measure where the upload latency is happening.
           2. we have to check the network b/w the EC2 and S3
           3. verify whether private traffic is going through NAT Gateway or an S3 VPC Endpoint
           4. check whether file size increased.
           5. verify multiple uploads configurations for large files in the app code.
           6. compare the upload from a know-good instance or subnet
           7. Check whether the slowdown happens for all files or only large files.
           8. check VPC Flow Logs if required
           9. check packet drops/errors

2. we have changed the configmap but my app is still using the old values ?

        A) 1. check how config maps is consumed.
              if configmaps is exposed as environment variables, existing pods will not automatically get the new values.
              then we have to perform:
              CMD: kubectl describe pod <pod_name>
              you many need to restart/roll out the deployment.
          
           2. if mounted as a volume.
              ConfigMap volume files can be updated by Kubernetes, but the application must reread the file.
              If the app only loads configuration at startup, it may continue using the old value.
          
          3. Verify the actual ConfigMap
             kubectl get configmap <name> -o yaml
             Ensure the update reached the expected ConfigMap.
         
          4. Check Pod configuration
             Look at the Deployment spec.
             Confirm the correct ConfigMap name/key is referenced.

          "First I’d check how the ConfigMap is consumed. If it’s injected as environment variables, existing Pods won’t receive the updated value, so a rollout             may be required. If it’s mounted as a volume, Kubernetes can update the file, but I’d verify whether the application rereads that file or only loads               configuration at startup. I’d also verify the actual ConfigMap and Deployment configuration."

        if my app is not capable to reread the file then we have to restart the app
