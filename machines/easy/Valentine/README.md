# Scanning and Enumeration
![nmap](screenshots/nmap.png)

The initial Nmap scan reveals three open ports:

- 22 (SSH) - SSH
- 80 (HTTP) - Web Server
- 443 (HTTPS) - Web Server 

## Web Server (HTTP)

Navigating to the web server on port 80, we are greeted with an image:
![http-homepage](screenshots/HTTP.png)

## Web Server (HTTPS)

Navigating to the web server on port 443, we see the same image.
![https-homepage](screenshots/HTTPS.png)

The image gives us a subtle hint. There might be something related to **Hearthbleed (CVE-2014-0160)**.

## Scanning for Heartbleed

To confirm, we use the `scanner/ssl/openssl-heartbleed` module in Metasploit.

![heartbleed-msfconsole](screenshots/msf-heartbleed.png)

We confirm that the web server is indeed vulnerable to Hearthbleed.

## Bruteforcing Directory

Next, we use `Gobuster` to brute-force directories and find the following:
![gobuster](screenshots/gobuster.png)

We locate a potentially interesting directory named `dev`:

![dev-directory](screenshots/dev_directory.png)

Inside this directory, we find a file called `hype_key`, which piques our interest.

# The hype-key

![hype-key](screenshots/hype-key.png)

The `hype_key` file appears to be a hex dump/

![hype-key-ascii](screenshots/hype-key-ascii.png)

Adter decoding the hex into ASCII, we find an encrypted private key. However, we still need to decrypt it.

## Decypting the Private key

At this point, we look for a way to obtain the decryption password. With the Heartbleed vulnerability, we can access the server's memory and search for anything that might give us the password.

![heartbleed-leak1](screenshots/heartbleed-leak1.png)
![heartbleed-leak2](screenshots/hearbleed-leak2.png)

Inside the leak, we find the string:
`$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==`

This is a **base64** encoded string, which convert to ASCII reveal the password:

`heartbleedbelievethehype`

![base64-password](screenshots/decrypted-base64-password.png)

![decrypted-private-key](screenshots/private-key.png)

We now have the decrypted private key.

# Accessing the server using SSH

With the private key, we can now access the server via SSH:

![ssh](screenshots/ssh-login.png)

After logging in, we can retrieve the user flag by running:

`cat flag.txt`

# Privilege Escalation

![ps-aux](screenshots/ps-aux.png)

Upon examining the process list with `ps aux`, we notice that **tmux** is running under the **root** user.

Once inside the session, we gain root access:
![tmux-command](screenshots/tmux.png)

![tmux-root](screenshots/tmux-root.png)

Now that we have root access, we can retrieve  the root flag by running:

`cat /root/root.txt`

# Lesson Learned

- Update software
- Do not expose key file to the public
- Memory leaks can be use to expose password that are stored in memory
- Keep an eye on processes running with elevated privileges
