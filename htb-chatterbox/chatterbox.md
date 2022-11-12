# HTB: Chatterbox

## Reconnaissance

A scan of all tcp ports will find that the service _Achat_ is running on ports
9255 and 9256. 

![Ports 9255 & 9256 Nmap](screenshots/2022-11-07_12-00.png)

## Initial Access

A buffer overflow vulnerability is present in some versions of Achat. Searching
exploit-db I found [this exploit](https://www.exploit-db.com/exploits/36025).
In the comments of the source code you will a template for msfvenom to generate
your own shellcode. 

![Command for shellcode from exploit](screenshots/2022-11-07_12-01.png)

Generate shellcode using the `windows/shell_reverse_tcp` non-staged payload. 

![Generating reverse shell shellcode](screenshots/2022-11-07_12-02.png)

Replace the shellcode in the exploit with the output of msfvenom and modify the
`server_address` variable to point to the victim machine.

![Modifying server address in exploit](screenshots/2022-11-07_12-26.png)

Start a netcat listener and run exploit using `python2`. You should now have an
interactive shell as the user _alfred_. 

![Catching reverse shell](screenshots/2022-11-07_12-27.png)

![Proof of command execution as low-privilege user](screenshots/user_proof.png)

## Privilege Escalation

There is a few options you can take here. Alfred's credentials are stored in
plaintext in the registry key `HKLM\SOFTWARE\Microsoft\Windows
NT\CurrentVersion\WinLogon`, and this password is _reused for the Admin user._
You could potentially login as Administrator with PsExec, but not without
forwarding port 139, as it is not exposed externally. However, `root.txt` is
owned by our current user, and you could just grant yourself access to it to
read the contents. As seen below. 

![Alfred's credentials stored in registry](screenshots/2022-11-07_12-40.png)

![Reading root.txt as Alfred](screenshots/root_proof.png)
