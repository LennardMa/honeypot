![image](https://user-images.githubusercontent.com/74513606/166928526-f3af1e6f-48bd-4638-8901-e98df0ef6c5c.png)

This is my documentation for creating a simple honeypot with an exposed Azure VM, 
logging the failed RDP login tries and analyzing the data with the help of Microsoft Sentinel SIEM.

# Setup

![image](https://user-images.githubusercontent.com/74513606/166930907-83b67f18-d9da-4619-bd01-094ab28fb74b.png)

After creating the VM, we can search for the type of log we want, in this case it is found within Windows Security logs with the Event ID 4625 (RDP Logon failure). 

![image](https://user-images.githubusercontent.com/74513606/166931385-eb1654f4-731f-4c88-b5aa-38938abb295a.png)

The next step is to improve the geodata of the logs. To do this we use a Powershell script together with an API from https://ipgeolocation.io/ which allows is to convert the IP of filed logons to geodata. 

![image](https://user-images.githubusercontent.com/74513606/166933014-bfd2a445-df35-4672-8d6d-ea20a8eb32f6.png)

Following this query, you can use a while loop to convert the raw logs using the API to the desired geodata. 

The resulting log looks like this. It is saved in ProgramData. Now the interesting part of using Azure features to properly analyze these logs can begin. 

![image](https://user-images.githubusercontent.com/74513606/166933749-32702794-b971-41e4-b04f-c37506e8e785.png)

First, we have to create a Custom Log inside Azure Log Analytics workspace which is linked to the Failed_RDP log file inside the VM. 

![image](https://user-images.githubusercontent.com/74513606/166937447-26c1ad42-6b18-4a93-99bd-f255dfc3bf6a.png)

There we can use the dummy values to train Custom Fields to automatically detect fields such as country, latitude  and username inside of the logs. This is necessary to later create queries who search after our custom fields. 

![image](https://user-images.githubusercontent.com/74513606/166937911-011420dd-b706-4132-8d21-7c6d88964102.png)

This KQL query searches for the parameters needed to visualize the logs inside of Sentinel. 

To make this possible, a custom Workbook has to be created with the KQL query shown above, inside the workbook, the queried data can be visualized as a map. 

After opening the firewall to all outside traffic and waiting a few minutes, the results can be seen!

# Results

![image](https://user-images.githubusercontent.com/74513606/166939166-66bdae02-2921-487c-904b-fffb85090948.png)

Most of the failed RDP logons came from multiple brute-force attempts coming from India. 

Either a brute force was attempted with the username ADMINISTRATOR, or a password spraying attack using a range of common default usernames.

This shows the critical importance of changing default credentials (and ofc not having Port 3389 open to the internet).

![image](https://user-images.githubusercontent.com/74513606/166939650-9b385017-6c53-4d7b-8b28-c0b43045afd1.png)


