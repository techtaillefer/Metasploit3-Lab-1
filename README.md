# Metasploit3-Lab-1

## Metaspoilt setup
First let's check the kali machine I have already installed and launch  `msfconsole`
<img width="1535" height="1177" alt="image" src="https://github.com/user-attachments/assets/87ac6750-482b-46cc-a90f-f4852f57f8c4" />
<img width="1327" height="598" alt="image" src="https://github.com/user-attachments/assets/cb60f99c-99f9-41f6-98eb-81044c02aa1e" />

## Metaspoiltable Setup

Here we can see vagrant is running them via `vagrant status`.
<img width="757" height="136" alt="image" src="https://github.com/user-attachments/assets/d72eafaa-05b5-49cc-8b25-d90fe03b99ec" />


My setup in VMware. I will be using kali and the ubuntu machine (ub1404).

<img width="416" height="151" alt="image" src="https://github.com/user-attachments/assets/b2a30808-f9dd-406d-9fde-33b115d6c73d" />

I checked the ips of each one. 
**kali:** 192.168.239.128
**ubuntu:** 192.168.239.129
**windows:** 192.168.239.130

But again I will just be using kali (attacker) and ubuntu (victim), not windows.

## Showing two machines communicating using ping

Here, is when I did `ip a` and found the ip address the ubuntu machine had for this network:
<img width="828" height="141" alt="image" src="https://github.com/user-attachments/assets/c1b54e76-8d95-453c-b4d3-c26b6e3ca4a1" />

And to test connectivity I `ping`ed it from my kali machine, which was a success.
<img width="1415" height="571" alt="image" src="https://github.com/user-attachments/assets/aa80bf29-0bdf-476b-ae27-6567e69f85c0" />

## Reconnaissance using Nmap

Now I used `nmap` for enumeration and recon, for service versions and scripts.
<img width="1738" height="155" alt="image" src="https://github.com/user-attachments/assets/c85f0b70-2d5e-41d6-bcb7-c2da25f59a5b" />

Here is what I found:

This scan shows multiple open ports on the target machine, including FTP (21), SSH (22), HTTP (80), and SMB (445), along with their service versions. These results help identify potential entry points and known vulnerable services to target.
<img width="1973" height="975" alt="image" src="https://github.com/user-attachments/assets/8df5e99e-e16b-4432-ae09-190db41004d8" />

Additional services such as MySQL (3306), Jetty web server (8080), and CUPS (631) are detected, expanding the attack surface. The presence of multiple outdated services definitely suggests several possible exploitation paths.
<img width="1961" height="793" alt="image" src="https://github.com/user-attachments/assets/7c9596db-923d-4ab3-b569-22a39120b352" />

Now the `-sC` portion comes in and the scan reveals SMB configuration details, including disabled message signing and guest account access. This indicates potential weaknesses in authentication and makes SMB a possibly strong candidate for exploitation if other simpler methods don't work out.
<img width="1461" height="992" alt="image" src="https://github.com/user-attachments/assets/3163e0f8-37a7-47f2-b75b-60717bbcccc2" />

## Exploit #1 - Reverse Shell via SSH

First I needed to build the malicious payload which is an ELF.
<img width="1967" height="477" alt="image" src="https://github.com/user-attachments/assets/99846640-4e40-4c21-935d-2cd1f5f487ad" />

Then I used `scp` or secure copy to copy it into the victim machine. This exploits the ssh port (22) that is open on the metasploitable ubuntu machine.
<img width="1988" height="567" alt="image" src="https://github.com/user-attachments/assets/9adfe201-5edf-4a62-86c0-7dcf432495dd" />

Next on the msfconsole, we set up a listener for our attacker machine. We use port 4444 since it is the default one typical for the metasploit framework in the first place.
<img width="1583" height="441" alt="image" src="https://github.com/user-attachments/assets/39377c55-0970-4a7f-ab6d-f537efe918a5" />


I then ssh into the victim and ran the payload. We can see how it is hanging below. Which means our meterpreter should have the session ready.
<img width="1351" height="535" alt="image" src="https://github.com/user-attachments/assets/a2c57a86-f92e-4e84-9cd2-050e42a87fc5" />

And yes it does as seen below. I got a meterpreter session back here, and the reverse shell set up was successful. I tried `whoami` at first but that didn't work. So I looked into the `help` command for anything else I can use to very I was inside the victim machine.
<img width="1982" height="787" alt="image" src="https://github.com/user-attachments/assets/26622758-e942-4812-bb1c-7862393d2335" />


Thus, I used `sysinfo` and, alas, it shows our system to be Ubuntu, signaling success in infiltration.
<img width="1435" height="343" alt="image" src="https://github.com/user-attachments/assets/34732281-ba4e-4b20-9cd4-4593d38a3b86" />

**Summary**:
From the Kali attacker machine, SSH access was gained to the Metasploitable3 Ubuntu target (192.168.239.129) using the default credentials `vagrant/vagrant`. A malicious ELF payload generated with `msfvenom` was uploaded and executed on the victim machine, causing it to connect back to the Metasploit listener and open a meterpreter session on the attacker's machine.

## Exploit #2 - MySQL Unauthenticated Login (Port 3306)

First, let's recall that nmap scan result I had done. It had identified MySQL running on port 3306 of the target machine (192.168.239.129), flagged as "unauthorized." This open database port will be the attack vector for our 2nd exploit here, as it indicated the MySQL service was exposed with no access controls.
<img width="1985" height="765" alt="image" src="https://github.com/user-attachments/assets/72fda6d6-ac25-4433-8863-e6e0f378a70e" />


Metasploit's `auxiliary/scanner/mysql/mysql_login` module was configured with the target IP, root username, and a blank password to test for unauthenticated access. The scan completed successfully, confirming that one credential attempt was successful against the MySQL service on port 3306.
<img width="1985" height="765" alt="image" src="https://github.com/user-attachments/assets/07bd0697-a36c-4834-8cd8-fb8ac4cc6787" />

The `mysql_login` auxiliary module confirmed that the root account on the target MySQL service had no password set, a critical misconfiguration. Metasploit reported that a MySQL session could be opened using the discovered credentials with `CreateSession` set to true. A true bruteforce success!
<img width="1985" height="765" alt="image" src="https://github.com/user-attachments/assets/8f67f1a5-524e-4b55-9389-6f724e704185" />

After opening the MySQL session with `sessions -i 1`, I had initially thought it gave me access to the sql database server itself, but then soon realized after interaction with the target it returned a full command shell running as `www-data` on the victim machine or `metasploitable3-ub1404` as seen below. Commands `whoami`, `hostname`, `id`, and `ls` confirmed successful unauthorized access to the victim machine's web server directory. 
<img width="1517" height="644" alt="image" src="https://github.com/user-attachments/assets/dd5729f6-326d-4f6b-94b7-d54201bb8f64" />

**Summary:** Using Metasploit's `mysql_login` auxiliary module, the target MySQL service at 192.168.239.129 was found to have no root password configured, allowing unauthenticated access. A shell session was opened as `www-data` on `metasploitable3-ub1404`, with full access to the web server directory at `/var/www/html`, confirming successful exploitation of the misconfigured database service.
