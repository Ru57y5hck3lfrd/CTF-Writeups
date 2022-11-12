# HTB: Optimum

## Reconnaissance

Vulnerable version of Rejetto HFS http server running on port 80. 

![Rejetto HFS 2.3](screenshots/2022-10-31_12-09.png)

![HFS landing page](screenshots/2022-10-31_12-09_1.png)

## Initial Access

Searching for a public exploit I found
[this](https://www.exploit-db.com/exploits/39161) on exploit-db which did not
work. I did find MetaSploit module `exploit/windows/http/rejetto_hfs_exec`. 

![Msf module](screenshots/2022-10-31_12-34.png)

Load the module, set the required options, and run. Here I am using a reverse 
https payload so that the shell traffic is encrypted using a non-suspicious
port/protocol. 

![Setting msf hfs module options](screenshots/2022-10-31_12-38.png)

![Proof of user level command execution](screenshots/user_proof.png)

## Privilege Escalation

Background the meterpreter session. Load the module
`multi/recon/local_exploit_suggester`, set the session option, and run.

![Msf Local Exploit Suggester](screenshots/2022-10-31_12-35.png)

It found that the system may be vulnerable to MS16-032 (CVE-2016-099), a
vulnerability in the _Secondary Logon_ service. 

![Found privilege escalation vector](screenshots/2022-10-31_12-36.png)

Load the module, set the appropriate options, and run to get shell as NT
Authority\System.

![Running ms16-032 module](screenshots/2022-10-31_12-59.png)

![Proof of system level command execution](screenshots/root_proof.png)
