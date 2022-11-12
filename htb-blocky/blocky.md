# HTB: Blocky

## Reconaissance

The web server redirects to `blocky.htb`. Add this to your `/etc/hosts/` file 
and navigate to `http://blocky.htb` to find a wordpress site.

![Landing page](screenshots/2022-10-31_13-51.png)

Using gobuster to bruteforce directories `/plugins/` was found. This directory
contained 2 `.jar` files which could be downloaded.

![Directory bruteforce](screenshots/2022-10-31_15-59.png)

![Contents of /plugins/](screenshots/2022-10-31_15-59_1.png)

Jar files can be decompiled using `jd-gui` which can be installed using `apt`. A
password for the root sql user was found in `BlockyCore.jar`. 

![Credentials found.](screenshots/2022-10-31_16-13.png)

## Initial Access

I was unable to log in via ssh as root with this password. However, I was able
to log in as the user _notch_ with these credentials who's username I found on 
a blog post on the site. 

![Finding username](screenshots/2022-10-31_16-11.png)

![SSH as notch](screenshots/2022-10-31_16-14.png)

![Proof of user level command execution](screenshots/user_proof.png)

## Privilege Escalation

User _notch_ was able to run any commands with sudo. I abused this to spawn a
shell as root.

![Proof of root level command execution](screenshots/root_proof.png)
