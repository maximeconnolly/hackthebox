# Scanning and Enumeration
![nmap](screenshots/nmap-scan.png)

The initial Nmap scan reveals two open ports:

- 22 (SSH) - SSH
- 80 (HTTP) - Web Server

## Web Server

Navigating to the web server on port 80, we're greeted with a simple "Hello World!" message.

![hello-world](screenshots/Web-Hello-World.png)

Inspecting the page source code reveals a reference to a `/nibbleblog` directory:

![source-code](screenshots/Hello-World-Source-Code.png)

Accessing this directory brings us to the Nibbleblog interface

![nibbleblog](screenshots/nibbleblog.png)

At the bottom of the page, we see the site is powered by **Nibbleblog**, a lightweight CMS written in PHP that is no longer actively maintained.

### Credential Enumeration

Exploring the `/content` directory, we uncover a file containing a **user blacklist**

![nibbleblog-user](screenshots/User-Blacklist.png)

from this, we learn that the username `admin` is valid.

### Getting the admin password

Since this is an easy box, we tried a few common passwords manually. After a few attempts, we successfully logged in using:

- Username: `admin`
- Password: `nibbles`


# Explotation

A quick search for know vulnerabilities in Nibbleblog leads us to a Metasploit module:

https://www.rapid7.com/db/modules/exploit/multi/http/nibbleblog_file_upload/

This module exploits an **arbitary file upload vulnerability**, allowing an attacker to execute remote code on the server.

![metasploit-module](screenshots/metasploit-1.png)

After setting the appropriate parameters. We launch the exploit and succesfully obtain a Meterpreter session.

![metasploit-session](screenshots/metasploit-session.png)

With a foothold on the target machine, we can now begin post-exploitation and privilege escalation.

# Privilege Escalation

We begin post-exploitation by checking for `sudo` permission using `sudo -l`:

![sudo-l](screenshots/sudo-l.png)

We discover that the current user has permissions to run the following script as root without a password: `/home/nibbler/personal/stuff/monitor.sh`

Since we have write access to this file, we can replace its contents with a simple reverse shell. After modifying the script, we execute it using `sudo`, which gives us `root access`:

![root-shell](screenshots/root-user.png)

We now have full control over the system.

# Lessons Learned

- Weak credentials are dangerous
- Legacy software is a Common Attack Vector
- Sudo Misconfiguration Can Lead to Root