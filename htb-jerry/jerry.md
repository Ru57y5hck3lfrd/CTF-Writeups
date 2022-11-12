# HTB: Jerry

## Reconnaissance

Nmap scan found that a Tomcat instance was running on port 8080. 

![Nmap output of Tomcat server on port 8080](screenshots/2022-10-29_18-33.png)

![Tomcat server landing page.](screenshots/2022-10-29_18-34.png)

## Initial Access

Default credentials (tomcat:s3cret) were used to log in to the management panel. 

![Logging in to management interface.](screenshots/2022-10-29_18-35.png)

![Tomcat management interface.](screenshots/2022-10-29_18-36.png)

Craft a malicious WAR file with msfvenom and upload it to "WAR file to deploy."

![Crafting malicious WAR file.](screenshots/2022-10-29_18-40.png)

![Uploading malicious WAR file.](screenshots/2022-10-29_18-41.png)

Start netcat listener, navigate to the directory path named after 
uploaded malicious WAR file, and receive connection back from payload. 

![Executing malicious WAR file.](screenshots/2022-10-29_18-42.png)

![Recieving call-back to listener.](screenshots/2022-10-29_18-42_1.png)

No escalation of privileges required as payload was executed as _NT
Authority\SYSTEM._

![Proof of execution as System level user.](screenshots/2022-10-29_18-48.png)
