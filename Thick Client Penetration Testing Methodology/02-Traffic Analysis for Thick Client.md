# Traffic Analysis

---

## Intercepting HTTP Traffic:

- Burp-Suite is used to analyse HTTP traffic.

## Intercepting TCP Traffic:

- Wireshark
- Echo Mirage (Very old and not maintained)
- MITM-RELAY + Burp Suite

---

# Tools

---

- Echo-Mirage: https://drive.google.com/open?id=1JE70HH-CNd_VIl190sheL72w3P5dYK58
- MITM-Relay: https://github.com/jrmdev/mitm_relay
- Burp-Suite: https://portswigger.net/burp/releases/professional-community-2024-1-1-4
- Python : https://www.python.org/downloads/
- PIP File : https://bootstrap.pypa.io/get-pip.py

---

- To make MITM-Relay working we need to install PIP and requests module in our host using the cmd:

```python
python.exe <path to get-pip.py file> 

#then

python -m pip install requests 
```

- Now we can check by running the MITM-Replay if it’s working.

```python
python.exe <path to mitm_relay.py file>
```

![alt text](data/image-12.png)

**Great job all done 👍.**

---

# Wireshark - Capturing Traffic

---

<aside>
💡 In this module we will capture and analyze FTP traffic from DVTA app.

</aside>

- Open Wireshark and FileZilla make sure the FileZilla is connected and running.

![alt text](data/image-13.png)

- Open DVTA app and login using the admin creds:
    
    `admin:admin123`
    
![alt text](data/image-14.png)


- So, now clear the traffic in Wireshark and now when we click on “Backup Data to FTP Server” we can see traffic is going through the Wireshark.

![alt text](data/image-15.png)

![alt text](data/image-16.png)

- To see only FTP traffic, apply the display filter as ftp (*or we can set it according to IP addresses in real testing*) and hit enter.

![alt text](data/image-17.png)

- Now, we can see the traffic only by the FTP.
- On analyzing the traffic, we found the user as “dvta” and password as “p@ssw0rd”.

![alt text](data/image-18.png)

![alt text](data/image-19.png)

- From the above captured credentials, let’s try to login into the FTP from browser or from the file explorer.

![alt text](data/image-20.png)

- Enter the username and password we captured.

![alt text](data/image-22.png)

- We successfully logged in, the new window opens showing us the available files in FTP server.

![alt text](data/image-21.png)

---

# Echo Mirage

---

- Open **Echo Mirage** and **DVTA** application and login with creds.

![alt text](data/image-23.png)

- In Echo Mirage go to **Options > Configuration** and select these rules in below image.

![alt text](data/image-24.png)

- Next go to **Rules > New > Direction = Any > Port = 21 > Intercept.**

![alt text](data/image-25.png)

- So we have got our rules which are intercepting the traffic of inbound and outbound traffic on port 21.

![alt text](data/image-26.png)

- Now go to **Process > Inject > Select DVTA.exe.**

![alt text](data/image-27.png)

- So now when any traffic goes in and out from DVTA we get traffic logs in Echo Mirage.
- For example when we hit Backup Data button we get traffic intercepted in Echo Mirage.

![alt text](data/image-28.png)

- Keep click OK for next traffic.
- Example: Now we have username and Password intercepted in Echo Mirage.

![alt text](data/image-29.png)

![alt text](data/image-30.png)

- Same we can also change rules according to our needs: for example If we want to select only inbound or outbound traffic we can do that also.
- Same we can change the traffic information also that is passing through the echo mirage,
- Like in image below we have changed the username to dvtb from dvta.

![alt text](data/image-31.png)

- And we got the incorrect login because user dvtb doesn’t exist.

![alt text](data/image-32.png)

---

# MITM Relay + Burp Suite

---

```vhdl
DVTA ------> MITM Realy --------> Burp Suite --------> Mitm Realy ----------> FTP Server
```

**DVTA:** An app that is communicating on server

**MITM Relay:** Wraps TCP packets to HTTP to view in Burp Suite. Listen or port on which the app is communicating on (in this case port 21)

**BURP Suite:** To edit the received HTTP packets back to MITM Relay which converts again HTTP packets to TCP packets.

**FTP Server:** The traffic is then passed to FTP Server. ( I configured the server for port 2111).

- Now open DVTA app and configure the server matches to our local machine (on which the app is installed).

![alt text](data/image-33.png)

- Use the command below for `mitm_relay.`

```visual-basic
mitm_relay.py -l 0.0.0.0 -r tcp:21:192.168.121.130:2111 -p 127.0.0.1:8080
```

- **`0.0.0.0`**: Specifies the *local address* on which the relay server will listen. “*0.0.0.0”* means it will listen on all available IP addresses/interfaces of the host machine.
- **`r TCP:21:192.168.1.104:2111`**: Defines a relay rule.
    - **`TCP`**: Indicates the protocol used for the relay (TCP in this case).
    - **`21`**: The source port on the local machine where the relay listens for incoming connections.
    - **`192.168.1.104`**: The *destination IP* address where the traffic is forwarded to.
    - **`2111`**: The *destination port* at the specified IP address where the traffic is sent.
- **`p 127.0.0.1:8080`**: Specifies the *proxy server* setting.

![alt text](data/image-34.png)

- Next login to DVTA application, and in burp Switch the `intercept`  on.
- So when we click on Backup Data button, we got our traffic relayed on Burp Suite.

![alt text](data/image-35.png)

- So we got all the traffic to burp-suite including username and password.

![alt text](data/image-36.png)

---

