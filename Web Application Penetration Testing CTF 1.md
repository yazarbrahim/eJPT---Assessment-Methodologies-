# Lab Environment

In this lab environment, you will be provided with GUI access to a Kali Linux machine. The target website is accessible at http://target.ine.local.

**Objective**: Identify web application vulnerabilities in the target website and capture all the flags hidden within the environment.

**Useful wordlists:**

```
/usr/share/wordlists/dirb/common.txt 
/usr/share/seclists/Usernames/top-usernames-shortlist.txt 
/root/Desktop/wordlists/100-common-passwords.txt
```

**Flag 1: Sometimes, important files are hidden in plain sight. Check the root ('/') directory for a file named 'flag.txt' that might hold the key to the first flag.**

![ec34f352993fd187509a95487e426e39.png](resources/ec34f352993fd187509a95487e426e39.png)

click View File

![cc41c5587e5110d9590a71bb2f483c05.png](resources/cc41c5587e5110d9590a71bb2f483c05.png)

Change the directory and /flag.txt

The URL `http://target.ine.local/view_file?file=/file1.txt` suggests that the web application is fetching and displaying the contents of the file specified in the `file` parameter.

To see the flag, you need to change the parameter value to file=/flag.txt because:

1.  - If /file1.txt is accessible, and /flag.txt exists in the same directory (or is accessible via the same mechanism), changing the filename in the parameter could allow you to retrieve it

![4bfc005ea81ca78e899cda124fb7c08c.png](resources/4bfc005ea81ca78e899cda124fb7c08c.png)

![6525c9bde3dc76fdda2d77e2f1be69a4.png](resources/6525c9bde3dc76fdda2d77e2f1be69a4.png)

Flag Ttry to find login page and ip

cat /etc/hosts

arp -a

dig target.ine.local

nslookup target.ine.local

ping target.ine.local

and

![2cd060256c03ff415fe8be2a8ef1fe17.png](resources/2cd060256c03ff415fe8be2a8ef1fe17.png)

and do the

**Flag 2: Explore the structure of the server's directories. Enumeration might reveal hidden treasures.**

find the dircetories.

![c267e527da7af1762c495b3d21300fc5.png](resources/c267e527da7af1762c495b3d21300fc5.png)

![dbc8af5e1ddb8e6247501b499166a7ab.png](resources/dbc8af5e1ddb8e6247501b499166a7ab.png)

697d550c2eb5437ca64351a6fa4ba618

First go over all the directories if we can find some information http://target.ine.local/secured/ - http://target.ine.local/secured/loginb.....

and on http://target.ine.local/secured/ directory there is a flag.txt and it is a interesting lets try to open

![2e446ea4a23f36046bd9700651ddc0b3.png](resources/2e446ea4a23f36046bd9700651ddc0b3.png)

http://target.ine.local/secured/flag.txt

![28951171143fab192c090698cfed545f.png](resources/28951171143fab192c090698cfed545f.png)

### **1\. Manual Browsing**

Try to manually access the file by navigating to the root directory of the website:

- If the website's URL is `http://example.com`, enter `http://example.com/flag.txt` in the browser's address bar.
- Sometimes, files like `flag.txt` are publicly accessible but not linked anywhere.

**Flag 3: The login form seems a bit weak. Trying out different combinations might just reveal the next flag.**

6081137d318c4858b98418f07d002edb

Do the hydra and try to find other password.

![839ebd12f7b2ec9eccefc7125ce7afe6.png](resources/839ebd12f7b2ec9eccefc7125ce7afe6.png)

![255b5bd2ecfcdabe6268f911f58dbec7.png](resources/255b5bd2ecfcdabe6268f911f58dbec7.png)

![ff79f9a9896a4f3eb7211110af1b9f85.png](resources/ff79f9a9896a4f3eb7211110af1b9f85.png)


hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P '/root/Desktop/wordlists/100-common-passwords.txt' 192.39.34.3 http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid username or password"

![073ef8cfb8603c9fd43a0a597b9ff43d.png](resources/073ef8cfb8603c9fd43a0a597b9ff43d.png)

![51e4e2f7cae5c5a27f2f4c786a7b3211.png](resources/51e4e2f7cae5c5a27f2f4c786a7b3211.png)


![f39f6cd271731f25f7f50c93d4461bfc.png](resources/f39f6cd271731f25f7f50c93d4461bfc.png)

**Flag 4: The login form behaves oddly with unexpected inputs. Think of injection techniques to access the 'admin' account and find the flag.**

### 3\. **SQL Injection**

Open login page and test for SQL injection vulnerabilities in both username and password fields:

- - `admin' OR '1'='1`  
        \- `' OR '1'='1' --`  
        \- `admin' --`
- Look for changes in the login behavior, such as bypassing authentication.

![d9713c1f709f4540b3aca30cd0b908d4.png](resources/d9713c1f709f4540b3aca30cd0b908d4.png)

![b2819ea5f569bb417b6607e4e9196f0f.png](resources/b2819ea5f569bb417b6607e4e9196f0f.png)
