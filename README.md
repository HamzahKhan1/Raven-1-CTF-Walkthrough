# **Raven-1-CTF-Walkthrough

## **Background**
As I learn about the basics of penetration testing and offensive security, I thought I'd post a basic walkthrough of the Raven 1 CTF challenge, which was posted to VulnHub by William McCann.
- VulnHub is a popular site for security enthusiasts and researchers to practice hacking "boxes" in a legal environment.
- These kinds of boxes are really good resources for the OSCP and pentesting in general. Working through even the beginner ones will give you great exposure to concepts like brute forcing, privilege escalation, etc. 
- Keep in mind that these kinds of tools and methodolgies should be practiced in a safe, controlled, and legal environment, and should never be used for malicious purposes.

## **Goal**
The goal of most any CTF is to obtain root access of the target machine and capture/read the "flag" files.

## **Tools**
- **Kali Linux** (including tools like Dirbuster, Wpscan, Netdiscover, Nmap, Hydra, John the Ripper, and more)
- **vulnerable Raven box** - can be found on VulnHub
- **Terminal**

## **Process**
1. **Host Discovery and Network Scanning**
- Before starting, make sure both your Kali and Raven boxes are on the same internal network. 
- (pic)
- In Kali, open a terminal. Run `ifconfig` to see your IP address and interface. Here, we are 192.168.56.6 - our subnet is 192.168.56, and we’re on “6”. (picture)
- Run `netdiscover -i eth0 -r 192.168.56.0/24`, which uses ARP requests to find hosts, on the Kali box. The “i” is for your interface, and the “r” indicates the range you’re scanning over. (picture)
- 4. Now we can run a ping scan of one of the IP addresses using `nmap -sn [IP]`. We see confirmation that the address is up, so we want to add it to our hosts file for easy access by doing `nano /etc/hosts`. (picture)
- Now let's run `nmap` command with the -sC -sV arguments added, which ( … ). Run `nmap -sC -sV 192.168.56.7` and we get a whole host of interesting info that includes: (pic)



## **Conclusion**


