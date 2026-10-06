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


