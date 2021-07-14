
*FOREWORD: We all get stuck which is why write-ups exist however, I would suggest you try your best to come up with the correct answer before seeking the answer. Most problems are meant to be solved but a process must be followed which is why I explain the various steps/commands used in order to arrive at a solution. Do your best ! Enjoy the process !*

![image](https://user-images.githubusercontent.com/61631671/125622280-29010d8c-92e5-44c3-b1eb-1bb5cfa0d713.png)

[This Room](https://tryhackme.com/room/juicydetails) focuses on using basic research and reconnaissance skills to analyze logs of a breached system and the methods the attacker used to breach it. 

## [ Task 1 - Introduction ] ##

*There are three .log files that need to be downloaded in order to complete this room auth.log, vsftpd.log and access.log. Once they are downloaded and unzipped we can begin !*

![image](https://user-images.githubusercontent.com/61631671/125633293-fb256c03-5d84-435c-a585-567d701a565f.png)

### Are you ready? ###

`I am ready!`

## [ Task 2 - Reconnaissance ] ##

The objective of this task is to figure out which tools the attacker used to access various endpoints on the target system.

### What tools did the attacker use? (Order by the occurrence in the log) ###

In order to locate the tools used, I analyzed access.log using `cat access.log`. These are the results: 

![image](https://user-images.githubusercontent.com/61631671/125636324-cccbc357-9617-47e2-9508-1c71e07647e3.png)

![image](https://user-images.githubusercontent.com/61631671/125636492-38a72a75-2105-4bb4-b084-213b51ce37d1.png)

![image](https://user-images.githubusercontent.com/61631671/125636728-6fc7115d-abed-4770-9c0c-10a59536ccbe.png)

![image](https://user-images.githubusercontent.com/61631671/125636996-46f19e27-25de-4957-8495-dc72b55b7d7c.png)

![image](https://user-images.githubusercontent.com/61631671/125636856-cf26a724-3620-4f9c-b9c7-fd8fc56ae206.png)

*The tools used were: `nmap, hydra, sqlmap, curl, and feroxbuster`*

### What endpoint was vulnerable to a brute-force attack? ###

![image](https://user-images.githubusercontent.com/61631671/125639664-24997901-7f2b-4731-8f78-720f4933f6c4.png)


*Using information gathered from access.log the attacker used Hydra to exploit the `/rest/user/login` endpoint.*

### What endpoint was vulnerable to SQL injection? ###

![image](https://user-images.githubusercontent.com/61631671/125640412-9489158b-df33-4b3f-8350-d0f71285157b.png)

*Using information gathered from access.log SQLInjection was used to exploit the `/rest/products/search` endpoint.*

### What parameter was used for the SQL injection? ###

![image](https://user-images.githubusercontent.com/61631671/125640901-d7ed69fe-e398-4b9c-9db7-aec4a67e605e.png)

*`q`*

### What endpoint did the attacker try to use to retrieve files? (Include the /) ###

![image](https://user-images.githubusercontent.com/61631671/125641428-0753ce6b-af68-4dc1-befd-2bb1bf42cd02.png)

For this I had to analyze the contents of vsftpd.log `cat vsftpd.log`. 

*Almost immediately I noticed the attacker used `FTP` to try to access the files*

# References #

[Nmap](https://nmap.org/)

[Hydra](https://tools.kali.org/password-attacks/hydra)

[Sqlmap](https://sqlmap.org/)

[Curl](https://www.geeksforgeeks.org/curl-command-in-linux-with-examples/)

[Feroxbuster](https://www.hackingloops.com/feroxbuster-forced-browsing/) 


# [ Task 3 - Stolen Data ] #
 

The objective of this task is to analyze the provided log files to understand the steps the attacker took to exlpoit the server.


### What section of the website did the attacker use to scrape user email addresses? ###

Using `access.log | less` I came across the following information: 

![image](https://user-images.githubusercontent.com/61631671/125644073-4813aedf-947a-4b2c-b012-b5ba57afb6b1.png)

*The attacker used the `product reviews` section to scape the adresses of targets.*

### Was their brute-force attack successful? If so, what is the timestamp of the successful login? (Yay/Nay, 11/Apr/2021:09:xx:xx +0000) ###

I provided a link below for further reading on status codes. *The status code for *OK* or *Successful* is 200.* To find whether or not the attack was successful I searched access.log using `cat access.log | grep 200`.

![image](https://user-images.githubusercontent.com/61631671/125646421-7a03340b-f910-4082-aa28-7c68b2981e81.png)

*The timestamp is ` [11/Apr/2021:09:16:31 +0000]`*

### What user information was the attacker able to retrieve from the endpoint vulnerable to SQL injection? ###

![image](https://user-images.githubusercontent.com/61631671/125648245-85a6d81c-0c9e-4c54-99fd-c0fe3b16244f.png)


Using the information gathered from the previous task I know that the endpoint the attacker used was */rest/products/search*. With that I analyzed the log file to find that the attacker gathered information on the users `email` and `password`. 

### What files did they try to download from the vulnerable endpoint? (endpoint from the previous task, question #5) ###

Using the information gathered from the previous question the attacker successfully downloaded  backup files (.bk) form the server.

![image](https://user-images.githubusercontent.com/61631671/125650331-adfa64b7-5a42-4811-9d15-c51a2b710882.png)

*The files are `coupons_2013.md.bak` and `www-data.bak`*

### What service and account name were used to retrieve files from the previous question? (service, username) ###

![image](https://user-images.githubusercontent.com/61631671/125651899-b5578e86-054c-4c6d-8c2b-c98408182410.png)

*Using data obtained from *vsftpd.log* the attacker used  `FTP` and `anonymous` to retrieve the backup files.*

### What service and username were used to gain shell access to the server? (service, username) ###

For this question I analyzed the contents of *access.log* to find the attacker failed many times to gain access.

![image](https://user-images.githubusercontent.com/61631671/125653555-7fad1c88-7d91-4560-8d3e-f5f0b9edfb92.png)


*After multiple attempts the attacker gained access to the server using `ssh` and `www-data` for the username*


# References # 

[Status Codes](https://umbraco.com/knowledge-base/http-status-codes/)


And we are done ! I hope you enjoyed my write-up for this room !















