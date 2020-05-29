# **Raven-1-CTF-Walkthrough**

## **Background**
As I learn about the basics of penetration testing and offensive security, I thought I'd post a walkthrough of the Raven 1 CTF challenge, which was posted to VulnHub by William McCann. Keep in mind this is just one of many routes you could take to find a flag.
- VulnHub is a popular site for security enthusiasts and researchers to practice hacking "boxes" in a legal environment.
- These kinds of boxes are great resources for the OSCP and pentesting in general. Working through them will give you great exposure to concepts like brute forcing, privilege escalation, etc. 
- Keep in mind that these kinds of tools and methodolgies should be practiced in a safe, controlled, and legal environment, and should never be used for malicious purposes.

## **Goal**
The goal of most any CTF is to obtain root access of the target machine and capture/read the "flag" files.

## **Tools**
- **Kali Linux** (including tools like Dirbuster, Wpscan, Netdiscover, Nmap, Hydra, John the Ripper, and more)
- **Raven** - intentionally vulnerable box that can be found on VulnHub
- **Terminal**
![image](https://user-images.githubusercontent.com/55573209/79625597-a0b55980-80ef-11ea-96cf-b80c6388de12.png)

## **Process**
1. **Host Discovery and Network Scanning**
- Before starting, make sure both your Kali and Raven boxes are on the same internal network. Go to `Devices>Network>Network Settings` and set both boxes to the same Host-only Adapter, which will be vboxnet0. On the Kali box, set Adapter 2 to NAT, which will allow you to still connect to the internet during the lab.
![network settings](https://user-images.githubusercontent.com/55573209/79625744-ac555000-80f0-11ea-9ccb-dd13123b478c.png)
- In Kali, open a terminal. Run `ifconfig` to see your IP address and interface. Here, we are 192.168.56.6 - our subnet is 192.168.56, and we’re on “6”.
![1](https://user-images.githubusercontent.com/55573209/79625761-dc9cee80-80f0-11ea-9251-10d8a4ea2e7e.png)
- Run `netdiscover -i eth0 -r 192.168.56.0/24` (which uses ARP requests to find hosts) on the Kali box. The “i” is for your interface, and the “r” indicates the range you’re scanning over. ![2](https://user-images.githubusercontent.com/55573209/79625770-f0e0eb80-80f0-11ea-9297-b66abfac7aa7.png)
- Now we can run a ping scan of one of the IP addresses using `nmap -sn [IP]`. We see confirmation that the address is up, so we want to add it to our hosts file for easy access by doing `nano /etc/hosts`. 
![3](https://user-images.githubusercontent.com/55573209/79625782-06561580-80f1-11ea-90f6-1576b3c12ecf.png)

![4](https://user-images.githubusercontent.com/55573209/79625784-0a823300-80f1-11ea-9107-48bce0ed7ae9.png)
- Now let's run `nmap` command with the -sC -sV arguments added, which enable default script scanning and version scanning. Run `nmap -sC -sV 192.168.56.7` and we get some interesting info:
![5](https://user-images.githubusercontent.com/55573209/79625793-1a9a1280-80f1-11ea-9634-6922e40ec519.png)
- The scan shows that port 22 is open (can’t do much unless we know usernames and brute force with password lists) 111 is open (not a common port to exploit), and 80 is open and appears to be running a website, which is of interest to us. 
- 1. Let’s visit that port in the Iceweasel browser. 192.168.56.7 in the search bar takes us to a pretty static, not too interesting site:
![6](https://user-images.githubusercontent.com/55573209/79625948-289c6300-80f2-11ea-900d-2980e456cf73.png)

2. **Website URL Enumeration with Dirbuster** 
- However, now that we have a website to work with, we can use `dirbuster`, which brute forces website URLs. Run the command in your terminal, and in the prompt that follows, enter `http://[IP of Raven]` in the “Target URL” section, and then click “Browse”. Go to the `/` directory, then go to `usr>share>wordlists>dirbuster>directory-list-2.3-medium.txt` in order to select a wordlist to use in our upcoming Brute Force attack. Click “Select List” and you’re taken back to the main Dirbuster prompt. Uncheck “Be Recursive” to speed up the process, then hit “start”.
![7](https://user-images.githubusercontent.com/55573209/79626020-8a5ccd00-80f2-11ea-8150-88b2e0c09447.png)
![8](https://user-images.githubusercontent.com/55573209/79626042-b9733e80-80f2-11ea-812c-968ad2288f23.png)
- Now Dirbuster will start brute forcing URLs, and any 200 codes mean that Dirbuster is finding different URLs. `Ctrl+C` after a few seconds (the program will go on almost indefinitely if you don’t). Browse through your results. We see two interesting ones - `/manual/` and `/wordpress`. 
![9](https://user-images.githubusercontent.com/55573209/79626072-f63f3580-80f2-11ea-9ef4-45c747b90e48.png)
![10](https://user-images.githubusercontent.com/55573209/79626077-fdfeda00-80f2-11ea-83b3-4959360ad22a.png)
- Let’s look further into these. Visit the Iceweasel web browser we still have open, and after the IP address in the search bar, type `/manual`, which just pulls up documentation for Apache. Now, try `/wordpress` instead, and you'll find yourself here:
![11](https://user-images.githubusercontent.com/55573209/79626113-3dc5c180-80f3-11ea-888d-49faca5d884f.png)
![12](https://user-images.githubusercontent.com/55573209/79626136-6d74c980-80f3-11ea-8696-ffc879c4c56a.png)

3. **Scanning/Exploiting WordPress for Login Credentials**
- Now we have a Wordpress site to work with. Quick background on Wordpress:
- 1. WordPress is an open-source PHP framework for building websites that powers 33% of websites on the Internet, making it a popular target for attackers.
- 2. Keeping WordPress and various plugins updated is a logistical challenge that often leaves websites vulnerable to attacks, which is why we'll be attempting to scan it for vulnerabilities, enumerate users, and brute force login credentials.
- Back in the terminal, let’s run `wpscan —url 192.168.52.7/wordpress -e` (“-e” for enumerate). Wpscan is a tool that scans remote WordPress installations to find security issues. Here's our output:
![13](https://user-images.githubusercontent.com/55573209/79626175-ab71ed80-80f3-11ea-8ea5-853cfeee80b4.png)
![14](https://user-images.githubusercontent.com/55573209/79626177-b3319200-80f3-11ea-84eb-c9a6b6d9c0d6.png)
![15](https://user-images.githubusercontent.com/55573209/79626206-f0961f80-80f3-11ea-9543-dd1bbb3a2b67.png)

- Notice that at the end of our output, there's a small table of users that’s been enumerated, containing the users "Michael" and "Steven". Our next step is to add these two into a wordlist/notes page. `cd` to `~/Desktop` and run `nano users.txt`. Add the users and save the file as `users.txt`. 
![15.1](https://user-images.githubusercontent.com/55573209/79626248-54b8e380-80f4-11ea-9ac6-b5eaecf07567.png)

- Now we have two usernames, but no passwords. From here, we can try to Brute force the Wordpress login to crack passwords. 
- First, we need to unzip a brute force wordlist file, which is built into Kali Linux. Use `cd /usr/share/wordlists`, to get into the directory, run `ls -l`  to see the contents, and run `gzip -d rocky.txt.gz` (d for deflate or decompress) to unzip:
![16](https://user-images.githubusercontent.com/55573209/79626259-68644a00-80f4-11ea-882e-4a47dec9e463.png)
- Now that we have the wordlist file, we can use **Hydra**, a brute force password hacking tool, against the first user, Michael. Run `hydra -l michael -P /usr/share/wordlists/rockyou.txt 192.168.56.7 ssh -t 4` and observe the first password being cracked! `michael:michael`
- 1. (Notes: the lowercase l for login indicates 1 user, uppercase L is for list (file) of users. uppercase P is for list of passwords (which we just unzipped!))
![17](https://user-images.githubusercontent.com/55573209/79626265-6f8b5800-80f4-11ea-8b63-a3dd051a0ae2.png)

4. **Compromising the target's SSH server to gain deeper info**

- Now that we have login credentials that work, we can use SSH to compromise one of the users that was enumerated from our WordPress scan. When we ran our nmap scan, we saw that port 22 was open, which makes this a good bet. Run `ssh michael@192.168.56.7`, enter the password (which we just discovered is "michael"), and you’ve compromised this user! Now you're logged in as "Michael@Raven".
![18](https://user-images.githubusercontent.com/55573209/79626270-774afc80-80f4-11ea-803d-12e1a7ab92d2.png)

- Now, we're hunting for something that gives us deeper info about this WordPress site. Examine the `/var/www/html` directory. `cd` into it and run `ls -l` and we see a bunch of files. `Wordpress` tends to have files that connect to databases, so let’s `cd` into there. 
![19](https://user-images.githubusercontent.com/55573209/79626275-7ca84700-80f4-11ea-8bb8-ec86ebdbdc9d.png)

- `List` the contents out again, and notice the “wp-config.php” file, our goldmine. This is the configuration file that Wordpress uses to connect to the database, filled with passwords and more. The user that can run it is www-data. If you could hack into the website, you could also view this file without having to use SSH. 
![20](https://user-images.githubusercontent.com/55573209/79626281-8467eb80-80f4-11ea-8774-32cfe700c979.png)

- If we `cat` the file, we notice some lines with SQL language that say something along the lines of `define(‘DB_PASSWORD, ‘R@v3nSecurity'` etc. It looks like this file is showing us the actual username and password we need to perform a major breach. Open a leafpad file and copy paste the info inside of it and keep it in the background for later. 
![21](https://user-images.githubusercontent.com/55573209/79626285-8cc02680-80f4-11ea-8367-8025defdba69.png)
![22](https://user-images.githubusercontent.com/55573209/79626484-18868280-80f6-11ea-8309-db8736d3f7ef.png)

5. **Pillaging MySQL**
- Now that we have the username and password, let’s login to mysql using `mysql -u root -p` (the “u” is for the username and password we found in the `wp-config.php` file, which was “root” and “R@v3nSecurity”, respectively). 
![23](https://user-images.githubusercontent.com/55573209/79626679-8da68780-80f7-11ea-9852-bbeab5fabf85.png)

- When you're in, run `show databases;` - we see “Wordpress”, which is of interest, so we query `show tables;` and see this: 
![24](https://user-images.githubusercontent.com/55573209/79626684-9ac37680-80f7-11ea-9b06-741ca90099fc.png)
![25](https://user-images.githubusercontent.com/55573209/79626686-a616a200-80f7-11ea-9330-7634f7970ef0.png)

- The “wp_users” entry at the bottom of the `Tables_in_wordpress` table is interesting, so query `select * from wp_users;` and notice that the entire users table has been dumped in front of us. We've already compromised Michael, but we haven't tackled Steven yet. Next to his username, you'll see a password hash. Copy it into an empty leafpad document called `hashes.txt` and save it to your `Desktop`.
![25](https://user-images.githubusercontent.com/55573209/79626686-a616a200-80f7-11ea-9330-7634f7970ef0.png)
![25.1](https://user-images.githubusercontent.com/55573209/79626734-01489480-80f8-11ea-8c82-b1ff3d15c2a2.png)

- Exit the database, open a new terminal, and brute force the hash with a new tool - **John the Ripper**. Run `john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt`, and then run `john -—show hashes.txt` and you’ll get a password, `pink84`! 
![26](https://user-images.githubusercontent.com/55573209/79626774-3b199b00-80f8-11ea-9279-1ec3ef8721a0.png)

6. **Escalating Privileges and Spawning a Shell**

- Go back to your SSH shell, where you’re logged in as Michael. To switch users, run `su steven` or `ssh steven@192.168.56.7` and enter the password we just discovered for Steven (`pink84`). 
- 1. You can see what things you can run with sudo by using `sudo -l`, which shows the things a system admin has given to the user. 
- 2. Now that we’re in, it's time for one of the most exciting steps. We see that we can run Python with root privileges on this box, so we'll be using a famous python script that imports a library called pty, and in that library, we can spawn a shell on the system.   
- 3. This script is great for boot to root boxes and pentesting and CTFs in general, so make a note of it. Moment of truth. Run `sudo python -c “import pty; pty.spawn(‘/bin/bash’)”` and you'll see the magical words: `root@Raven`. Run a few commands to get info, like `id` and `whoami`, and finally `cd` into `root` and `ls` to see your first flag!!!
![27](https://user-images.githubusercontent.com/55573209/79626794-68fedf80-80f8-11ea-99a3-e95c2f3ea580.png)

- Cat the flag file, and you see:
![28](https://user-images.githubusercontent.com/55573209/79626893-9a2bdf80-80f9-11ea-9ca8-2c4a8e0c6331.png)

**Congrats!** You've found your first flag.

## **Conclusion**
1. There are plenty of walkthroughs for this box online, and many different ways to tackle this box. This method focused more on the Network angle, which involved brute forcing logins with Hydra/SSH, pillaging MySQL to gain and dump important info, and spawning a root shell using Python.
2. The Web angle, which I haven't tried myself yet, involves generating a sitemap of the HTTP server with Burpsuite and Spider, finding flags in the website's HTML, performing URL enumeration against the HTTP server with `wfuzz`, performing user enumeration against the Wordpress blog and brute-forcing the passwords of discovered users with `wpscan` (which we involved in our Network angle), and finding flags in the WordPress administrator panel.
3. Understanding both website enumeration and network enumeration, in addition to brute-forcing and information pillaging strategies, are some of the great overarching takeaways from this project.

![image](https://user-images.githubusercontent.com/55573209/83311289-60f48e00-a1d4-11ea-9ebc-72f585f03b84.png)
