# PG: Wombo

There are a few rabbit holes on this box, but once you find the intended path it
is straight forward to exploit.

## Reconnaissance

An nmap scan of all ports will reveal that Redis version 5.0.9 is running on 
port 6379. Searching exploit-db you will find a metasploit module for this 
version. 

![Redis port open](screenshots/2022-11-16_12-09.png)

![Searchsploit results](screenshots/2022-11-16_12-16.png)

## Initial Access

Load the module `linux/redis_replication_cmd_exec`, set the required options and
run it to get a shell as root. 

![Setting module options](screenshots/2022-11-16_12-13.png)

![Proof of shell as root user](screenshots/root_proof.png)
