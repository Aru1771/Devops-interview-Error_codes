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

          "First I’d check how the ConfigMap is consumed. If it’s injected as environment variables, existing Pods won’t receive the updated value, so a rollout may be required. If it’s mounted as a volume, Kubernetes can update the file, but I’d verify whether the application rereads that file or only loads configuration at startup. I’d also verify the actual ConfigMap and Deployment configuration."

        if my app is not capable to reread the file then we have to restart the app


3. What is kube-proxy ?

        • The Controller, Not a Proxy: Despite its name, kube-proxy is a data-plane network controller running on every node—traffic does not actually flow through it.
        • The Rule Writer: It watches the API server for creation, deletion of servers and changes of Services endpoints and their corresponding backend PodEndpoints.
        • The Kernel Programmer: It translates those high-level Kubernetes objects into low-level routing instructions by writing rules directly into the host node's iptables or IPVS (IP virtual services)engine ahead of time.

4. what is core-dns ?
   
        • The Component Type: CoreDNS is a cluster add-on that runs as standard workloads in the data plane (typically within the kube-system namespace), not on             the control plane nodes.
        • The Watcher: It constantly watches the kube-apiserver for the creation, modification, or deletion of Services.
        • The Record Creator: Whenever a new Service is born, CoreDNS automatically generates a standard DNS A Record mapping the human-readable FQDN (<service>.<namespace>.svc.cluster.local) to the newly assigned virtual ClusterIP.

5. who will assign clusterIP's, NodePort to services ?

        • The Core Authority: The kube-apiserver is entirely responsible for allocating and managing Service IPs.
        • The Mechanism: When you submit a Service manifest, the API server dynamically picks an available IP address from a pre-configured pool defined by the cluster’s --service-cluster-ip-range flag.
        • The Distinction: The CNI plugin (like Calico or Cilium) has nothing to do with this process; the CNI only manages and assigns ephemeral Pod IPs from the            pod CIDR block.
        • The Broadcast: Once the API server assigns the virtual ClusterIP, it saves the state to etcd and broadcasts the update across the cluster.

6. What is the flow of request from one service to another service ?

        When Pod A wants to communicate with Pod B using a Service name, the execution flows precisely like this:
        • Step 1: DNS Resolution — Pod A sends a DNS query to the CoreDNS pods. CoreDNS looks up its internal phonebook and responds to Pod A with the virtual ClusterIP.
        • Step 2: Hitting the Host — Pod A constructs a network packet with the destination set to that virtual ClusterIP. The packet leaves the container and hits the host node's Linux kernel.
        • Step 3: Rule Matching & Load Balancing — The Linux kernel intercepts the packet. It checks its iptables or IPVS rules (programmed by kube-proxy),evaluates healthy backend endpoints, and picks a destination Pod IP (using a built-in random probability algorithm).
        • Step 4: Destination NAT (DNAT) — The kernel modifies the packet header mid-flight, swapping the virtual ClusterIP out for the real, physical target Pod  IP.
        • Step 5: Delivery — The CNI network fabric routes the rewritten packet across the cluster network directly to the destination Pod.
        
        
7. Your production application uses AWS Secrets Manager for database credentials.The database password was rotated successfully, but now the application is failing with authentication errors. Why is this happening and how would you troubleshoot it?

       I would not immediately change the database password again. Instead, I’d troubleshoot step by step:
       Check application logs

        1. Look for errors like: Authentication failed, Invalid username/password, Access denied, Connection refused.
        
        2. Check the secret in Secrets Manager
        
        Verify the new password is stored correctly.
        
        Confirm the application is reading the correct secret and version.

        3. Check IAM permissions

        Ensure the application’s IAM role has secretsmanager:GetSecretValue.
        
        If the secret is encrypted with KMS, verify the role has the required KMS permissions.

       4. Check how the application loads the secret

       Some apps only read the secret at startup.
        
       Even though Secrets Manager has the new password, the running app may still be using the old password.

       5. Check deployment/restart strategy

        If the app doesn’t refresh secrets dynamically, you may need to restart or redeploy workloads so they load the new credentials.
        
        Do this in a controlled way, not blindly restarting everything.

   
           
