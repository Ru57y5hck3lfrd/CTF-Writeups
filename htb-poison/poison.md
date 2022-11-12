# HTB: Poison

## Reconnaissance

Navigating to the web server running on port 80 you will find a temporary test 
site to "test scripts." You are able to run php scripts located on the server as
well as traverse directories to read files. 

![Landing page](screenshots/2022-11-04_15-33.png)

`listfiles.php` contained an interesting file name `pwdbackup.txt`. Which you
are able to read using the sites functionality. This file contained base64
encoded data. 

![Contents of listfiles.php](screenshots/2022-11-04_15-33_1.png)

![Contents of pwdbackup.txt](screenshots/2022-11-04_15-34.png)

Decoded the output of `pwdbackup.txt` with cyberchef. Repeatedly decoding base64
until pasword was revealed. I then read the contents of `/etc/passwd` to get
the users with login shells on the system. 

![Decoded pwdbackup.txt](screenshots/2022-11-04_15-35.png)

![LFI /etc/passwd for usernames](screenshots/2022-11-04_15-36.png)

## Initial Access

You are able to log in as user _charix_ with the decoded password.

![Proof of low-privilege command execution](screenshots/2022-11-04_15-37.png)

## Privilege Escalation

Listing the open ports you will find that 2 common vnc ports are listening 
locally. 

![Listening vnc ports](screenshots/2022-11-04_15-44.png)

In charix's home folder there is a file `secret.zip`. Download it using `scp`
and unzip as seen below. 

![Downloading secret.zip](screenshots/2022-11-04_16-02.png)

![Unzip with charix password](screenshots/2022-11-04_16-03.png)

Use ssh for local port forwarding. 

![Port forwarding vnc](screenshots/2022-11-04_16-12.png)

Contents of `secret.zip` contained the file `secret`. I was not sure what this
file was but searching the `vncviewer` man page I found a possible use for it.

![vnc -passwd flag](screenshots/2022-11-04_16-06.png)

Use `vncviewer` to log in as root using the secret file.

![Log in to vnc on forwarded port](screenshots/2022-11-04_16-04.png)

![Proof of command execution as root](screenshots/root_proof.png)
