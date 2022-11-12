# HTB: Knife

Apache server on this machine was running a vulnerable version of PHP which
allowed for remote command execution. Privileges were then escalated via a
misconfigured sudo permission. 

## Reconnaissance

Nmap scan found that Apache server running on port 80 had PHP version 8.1.0-dev
installed. Researching this version I found that a threat actor had pushed 2
malicious commits to the PHP source code which contained a backdoor. 
Reference this 
[blog post](https://flast101.github.io/php-8.1.0-dev-backdoor-rce/)
by [flast101](https://github.com/flast101) for further detail.

![Nmap output of port 80](screenshots/2022-10-30_14-02.png)

## Initial Access

Searching exploit-db for this version of PHP I found `php/webapps/4933.py`.
After inspection of the [source code](https://www.exploit-db.com/exploits/49933)
found nothing suspicious, I ran it, aquiring non-interactive command execution 
as the user "james."

![Mirroring the exploit with searchsploit](screenshots/2022-10-30_14-13.png)

![Running php exploit.](screenshots/2022-10-30_14-15.png)

To gain an interactive shell I utilized a "nc mkfifo" reverse shell, caught it 
with a netcat listener, and stabilized it with python.

![Mkfifo reverse shell.](screenshots/2022-10-30_14-15_1.png)

![Catching reverse shell.](screenshots/2022-10-30_14-16.png)

![Using python to stabilize shell.](screenshots/2022-10-30_14-16_1.png)

![Proof of user level command execution.](screenshots/user_proof.png)

## Privilege Escalation

Executing `sudo -l` I found that this user could run the `knife` binary with 
sudo - without a password. Searching [GTFOBins](https://gtfobins.github.io/) 
I found a viable method to abuse this binary to escalate privileges as seen
below.

![sudo -l output](screenshots/2022-10-30_14-17.png)

![Proof of root level command execution.](screenshots/root_proof.png)
