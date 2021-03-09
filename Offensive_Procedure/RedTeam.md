# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services

Nmap scan results for each machine reveal the below services and OS details:

```bash
nmap -sV -O -vv 192.168.1.110
#Scan revealed:
```
![Path](Diagrams/nmapscan1.jpg)
![Path](Diagrams/nmapscan2.jpg)


This scan identifies the services below as potential points of entry:
- Target 1
  - Port 22/tcp - ssh - Open
  - Port 80/tcp - http - Open
  - Port 111/tcp - rbcbind - Open
  - Port 139/tcp - netbios-ssn - Open
  - Port 445/tcp - netbios-ssn - Open

#### Port 22
An open port 22 can allow ssh logins. Keeping port 22 open usually welcomes attackers to brute force using a dictionary attack. 

#### Port 80
Port 80 allows for HTTP traffic. Attackers can also use brute force and dictionary attacks to gain access.

#### Port 111
Port 111 allows attackers to see what services are running. This can be used for reconnaissance activity. The RPC Service has a history of security vulnerabilities.

#### Port 139 & 445
These ports allow for network file sharing. Also allow the potential of a possible reverse shell via Metasploit. 


The following vulnerabilities were identified on each target:

### Target 1
#### WP Scan
- By knowing that Raven Security is running Wordpress I ran a wpscan to enumerate the version and its users. By gaining this information I could see if their version of word press is vulnerable to attacks as well as gain user information. 
- Raven Security is running Wordpress version 4.8.15 - this version is susceptible to numerous vulnerabilities:
https://www.cvedetails.com/vulnerability-list/vendor_id-2337/product_id-4096/ 
![Path](Diagrams/wordpress_version.png)
- By running Wordpress Raven Security also allowed me to view users. I found two users for Raven Security: Michael and Steven
![Path](Diagrams/wordpress_users.png)

#### Brute Force Attack:
- The first Brute Force attack I ran was to gain Michael's password. By running a simple Hydra command I was able to easily and successfully gain Michael's password.
hydra -l michael -P rockyou.txt -s 22 -f -vV 192.168.1.110 ssh
![Path](Diagrams/hydra_command.png)
![Path](Diagrams/michael-hydra-brute-password.jpg)

- The second Brute Force attack I ran was to gain Steven's password.  Although this came later on, I used John the Ripper to crack Steven's password hash. The password hash was gained when I managed to pull up the wordpress sql database.

![Path](Diagrams/steven_password.png)

#### Spawning a TTY Shell
- After gaining access back into the system as Steven I attempted to spawn a TTY Shell using a python command. By using this command I gained root access on the system:
```
sudo python -c 'import pty; pty.spawn("/bin/sh")'
```

#### Information Disclosure Issues:
- There can be many different types of Information Disclosure issues in web applications. These can be common mistakes that developers or make that can lead to the discovery of confidential and sensitive information. From Banner Grabbing to File Path Disclosure, attackers can utilize the web page itself to discover "hidden" information. The specific Information Disclosure Vulnerability used for this attack involved "Inappropriate Handling of Sensitive Data." For this I ran a simple, View Page Source, and started analyzing the data. Since I knew specifically what I was looking for I was able to run a find command on each page to gather my information.  

#### Directory Indexing:
Directory Indexing or Directory Browsing can lead to exposing directory listings on a site that a company may not want you to know about. By running a dirb command in Kali an attacker could find other hidden files.

![Path](Diagrams/dirb.png)


### Exploitation:

The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:
### Target 1
### Flag1.txt: Hash Value:
`flag1{b9bbcb33e11b80be759c4e844862482d}`


![Path](Diagrams/Flag1.jpg)

**Exploit Used:**
`Information Disclosure Issue`
  - By right clicking on the web page, I went to: ```View Page Source.```  From here I started to inspect the page and see if anything stood out. I did a search through the various folders and found flag1 in the service.html folder.

### Flag2.txt: Hash Value:
`flag2{fc3fd58dcdad9ab23faca6e9a36e581c}`

**Exploit Used:**
`Brute Force Attack`
  - By running a Brute Force Attack against Michael's username we managed to uncover his "secret" password. As noted above, Michael's password turned out to be his name: michael

    ```
    hydra -l michael -P rockyou.txt -s 22 -f -vV 192.168.1.110 ssh
    ```

  - By then doing a ssh into the IP we can successfully log in as Michael: 
    
    ```
    ssh michael@192.168.1.110 -p 22
    password: michael
    ```
  - Now that we are logged in as Michael we can begin the search for the Flag. Since the first flag was labeled with a .txt let's begin our search with that.
    ```
    locate flag2.txt
    ```
    - As seen in the screenshot - this directs us to the /var/www/ folder where Flag2 is hiding.

    ![Path](Diagrams/Flag2.png)

### Flag3.txt: Hash Value:
`flag3{afc01ab56b50591e7dccf93122770cd2}`

**Exploit Used:**
`Directory Browsing`
  - One role of a hacker is to seek out information that maybe a developer or sysadmin forgot to remove. That can involve searching through different directories and files to find clues that will lead to other "secret" information. For the case of Flag3 I did some digging into different directories and files to recover usernames and passwords which then allowed me to access other important databases. Here is how that worked:

  - Since I was already in the /var/www/ folder, logged in as Michael, I decided to navigate deeper to see what other information I could find. This lead me to the `html` folder and then deeper into the `wordpress` folder. 

  ![Path](Diagrams/wordpress_folder.png)

  - Navigating into the `wordpress` folder gave me a lot of options. One of those options was the `wp-config.php` file. I decided to do a `cat command` and see if there was anything useful in this file.

  - I came across some possible useful information and decided to take note of what I found:

  ![Path](Diagrams/mysql_info.png)
  
  - Looks like a possible username, password, and hostname for a mysql database.

  - With this information I proceeded to delve deeper to see what other secrets I could discover. By running the following command and using `R@v3nSecurity` as the password I was able to get into the mysql database:
  ```
  mysql -h localhost -u root -p
  ```
  ![Path](Diagrams/mysql_login.png)

  - Now logged into mysql I could continue my search.  I decided to run the following commands to see what information I could uncover:
  ```
  show databases;
  use wordpress;
  show tables;
  ```
![Path](Diagrams/mysql_database.png)

  - As seen in the screenshot this brought forth a good amount of information. I decided to dig into each folder to see if I could unravel anything new. By running the following command I entered the wp_posts table and immediately found something interesting:  
  ```
  select * from wp_posts;
  ```
![Path](Diagrams/wp_posts.png)

  - And there we have flag3 (It also uncovered flag4 but let's see if there is another way we could uncover that flag).

### Flag4.txt: Hash Value:
`flag4{715dea6c055b9fe3337544932f2941ce}`

**Exploit Used:**
`Spawning a TTY Shell` 
  - This took a little bit of time to find and conquer but in the end, it was totally worth it. By password cracking Steven's password hash I was able to log in under his username and ultimately gain root access. Any attacker's main objective!

  - Picking up where I left off I continued digging into the mysql table to see what other treasures I could find. This eventually lead me to the wp_users table and brought me to the password hashes of Michael and Steven.

  Commands:
  ```
  select * from wp_users;
  ~This resulted in a bit of a jumbled mess so I scaled the search down:
  select user_login, user_pass from wp_users;
  ~This narrowed it down to a nice table
  ```

  ![Path](Diagrams/mysql-chart-user-hash.jpg)

  
  - I quickly gathered those hashes, put them into a text file, and ran my John the Ripper to see if they could be cracked. As noted above, John the Ripper came back with a password of `pink84` for Steven.  Without hesitation I jumped back to it and did another ssh into the IP as Steven
  ```
  ssh steven@192.168.1.110 -p 22
  password: pink84
  ```
  -  I was now back into the system, logged in under Steven. However, I still had restricted access as to what I could be doing (much like with Michael). Therefore, I ran a python script to spawn a TTY shell and obtain root access. 

  Command:
  
  ```
  sudo python -c 'import pty; pty.spawn("/bin/sh")'
  ```

  - By obtaining root access I quickly found flag4:

  ![Path](Diagrams/python_script.png)
  
  ![Path](Diagrams/Flag4.jpg)








