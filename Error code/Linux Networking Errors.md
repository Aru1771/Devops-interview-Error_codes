Linux Networking Errors
=======================


Lesson 1: DNS resolution failure
-----------------------------------

Your application is running on an EC2 instance. A user reports that the application cannot connect to its backend using this hostname:

    curl https://api.example.com


The command returns:

    curl: (6) Could not resolve host: api.example.com

Your application is running on an EC2 instance. A user reports that the application cannot connect to its backend using this hostname:
curl https://api.example.com


Command 1 — getent hosts <host.name>

    Returns an IP address if the hostname resolves through the system's configured name-resolution sources.
    If it returns nothing, resolution may have failed. Check the exit status and investigate further.

Command 2 — nslookup

     nslookup api.example.com


    - Queries DNS to investigate whether the hostname resolves.
    - Useful for identifying DNS server errors or missing DNS records.

Command 3 — cat /etc/resolv.conf

      cat /etc/resolv.conf


    - Shows the configured DNS resolver information.
    - Helps you check which DNS servers the machine is configured to use.

Command 4: you could use to test connectivity to TCP port 8080.

    sudo ss -lntp
    sudo ss -lntp | grep ':8080'

   
