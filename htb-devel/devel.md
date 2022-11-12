# HTB: Devel

## Reconnaissance

FTP server allowed for anonymous login and unrestricted file uploads via the
`put` command. The root of the ftp server was the root directory for the IIS web
server running on port 80.

![Root of FTP server](screenshots/2022-11-02_14-46.png)

![IIS confirmed](screenshots/2022-11-02_14-45.png)


## Initial Access

Generate a meterpreter payload with `msfvenom`, start a MetaSploit
`multi\handler` listener, upload payload via ftp, and then navigate to the file
on the web server with your browser to trigger payload. This will grant you a
shell as the low-privileged user _iis apppool\web_. 

![Generating meterpreter payload](screenshots/2022-11-02_16-46.png)

![Setting multi handler options](screenshots/2022-11-02_16-48.png)

![Proof of low privilege command execution](screenshots/user_proof.png)

## Privilege Escalation

Metasploit's exploit suggester module found that the system was vulnerable to
[MS10-015](https://learn.microsoft.com/en-us/security-updates/SecurityBulletins/2010/ms10-015)
([CVE-2010-0232](https://www.cve.org/CVERecord?id=CVE-2010-0232)). 
This exploits a now patched vulnerability in the Windows kernel. 

Background the current session and load the module
`windows/local/ms10_015_kitrap0d`, set the session and other options as needed,
and run to gain shell as _NT Authority/SYSTEM_.

![Using suggested exploit](screenshots/2022-11-02_16-54.png)

![Shell as SYSTEM](screenshots/2022-11-02_16-55.png)

![Proof of system level command execution](screenshots/root_proof.png)
