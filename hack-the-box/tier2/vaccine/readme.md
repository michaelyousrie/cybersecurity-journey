# Vaccine

[Vaccine Machine](https://app.hackthebox.com/machines/Vaccine)

This machine is best watched on the YouTube video where we go through the full thought process in real time.

[YouTube Video](https://youtu.be/QmYbmXJdqF0)

## Questions

> Besides SSH and HTTP, what other service is hosted on this box?
>> FTP

> This service can be configured to allow login with any password for specific usernames. What is that username?
>> anonymous

> What is the name of the file downloaded over this service?
>> backup.zip

> What script comes with the John The Ripper toolkit and generates a hash from a password protected zip archive in a format to allow for cracking attempts?
>> zip2john

> What is the password for the admin user on the website?
>> qwerty789

> What option can be passed to sqlmap to try to get command execution via the sql injection?
>> --os-shell

> What program can the postgres user run as root using sudo?
>> vi

## Attack Chain Summary

This box demonstrates a realistic multi-stage attack chain where each vulnerability builds on the previous finding.

### Phase 1: Enumeration

- Scan the target with Nmap to discover open services.

        nmap -sV -sC ${MACHINE_IP}

- Three services are open: FTP (21), SSH (22), and HTTP (80).
- FTP allows anonymous login.

### Phase 2: FTP and File Extraction

- Connect to FTP with anonymous credentials.

        ftp ${MACHINE_IP}

- Log in with username `anonymous` and any password.
- Download the backup file found on the server.

        get backup.zip

- The zip file is password protected. We need to crack it.

### Phase 3: Password Cracking

- Use zip2john to extract the hash from the password-protected zip.

        zip2john backup.zip > hash.txt

- Crack the hash using John the Ripper.

        john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

- Extract the zip with the cracked password. Inside, find credentials for the web application admin user. The admin password is `qwerty789`.

### Phase 4: Web Application and SQL Injection

- Log into the web application at `http://${MACHINE_IP}` using the admin credentials.
- The application has a search functionality that is vulnerable to SQL injection.
- Use sqlmap with the `--os-shell` flag to get command execution through the SQL injection.

        sqlmap -u "http://${MACHINE_IP}/dashboard.php?search=test" --cookie="PHPSESSID=YOUR_SESSION_COOKIE" --os-shell

- This gives us a shell as the `postgres` user on the system.

### Phase 5: User Flag

- From the OS shell, stabilize your connection with a reverse shell if needed.
- Read the user flag from the postgres user's home directory.

### Phase 6: Privilege Escalation

- Check what the postgres user can run with sudo.

        sudo -l

- The postgres user can run `vi` as root.
- Exploit vi to escape to a root shell.

        sudo vi
        :!/bin/bash

- We are now root. Read the root flag.

        cat /root/root.txt

## Key Takeaways

- **Anonymous FTP is a goldmine for attackers.** Any files left on an anonymous FTP server are essentially public. Never store sensitive data like backups on anonymous FTP.

- **Password-protected zips are not secure storage.** Tools like zip2john and John the Ripper can crack zip passwords quickly, especially when common passwords are used. `qwerty789` falls in seconds against rockyou.txt.

- **Credential reuse across systems is devastating.** Credentials found in a backup file gave us admin access to the web application.

- **SQL injection can lead to OS-level access.** sqlmap's `--os-shell` flag demonstrates how a web vulnerability can escalate to full system access.

- **Sudo misconfigurations are common privilege escalation vectors.** Programs like vi, vim, less, more, and many others can be abused to escape to a shell when run with sudo. Always check GTFOBins for known sudo escapes.

## Tools Used

- Nmap
- FTP client
- zip2john / John the Ripper
- sqlmap
- vi (for privilege escalation)