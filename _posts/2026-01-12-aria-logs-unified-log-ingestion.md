---
Title: "Configuring Citrix NetScaler and Omnissa Horizon Log Ingestion in VMware Aria Operations for Logs"
Date: 2026-01-12
---

## Overview
In modern enterprise environments, logs are generated across a wide range of platforms—including infrastructure, applications, security devices, and network components. When these logs remain siloed, troubleshooting becomes time-consuming and operational visibility is limited.

This blog focuses on configuring VMware Aria Operations for Logs to ingest and centralize Citrix NetScaler and Omnissa Horizon logs for streamlined troubleshooting.
##
## NetScaler Configuration Steps

- Navigate to System > Auditing > Syslog > Servers tab, then click Add

<img width="1918" height="855" alt="image" src="https://github.com/user-attachments/assets/5da403c1-240c-49d8-a36f-940af11cf630" />

### Complete Add Process
  - Name: AriaLogs (your choice).

  - Server Type: Server IP; enter Aria Ops IP/hostname.

  - Port: 514.

  - Log Levels: ALL or select (e.g., INFO, DEBUG).

  - Click Create.

<img width="1530" height="346" alt="image" src="https://github.com/user-attachments/assets/a5b320d7-03e4-46d9-b323-1788a181065e" />

##

Goto Policies tab > click on Add

Name: AriaLogs-Policy > Click on Create.

<img width="857" height="627" alt="image" src="https://github.com/user-attachments/assets/7a995785-c754-4fa2-ad01-83983b0d6108" />


After creating, bind it globally (priority 1) to activate syslog forwarding appliance-wide.


Goto Policies tab > Click on Select action > Classic Policy Global Bindings

<img width="1530" height="357" alt="image" src="https://github.com/user-attachments/assets/2fbd15e5-f1b4-4c33-96cc-f04560256681" />

Click on Select policy

<img width="856" height="568" alt="image" src="https://github.com/user-attachments/assets/bad71daf-b4ff-4fd8-9477-9bd66075c00e" />

Select AriaLogs-Policy created earlier and click on Bind.

<img width="1908" height="567" alt="image" src="https://github.com/user-attachments/assets/7a0c0412-f3f5-41e6-8377-2effa8014501" />

##

### Verify the configuration and test log flow

- Login to NetScaler CLI and run nsconmsg -d current -g syslog
- Generate Test Traffic.

  <img width="836" height="238" alt="image" src="https://github.com/user-attachments/assets/9c812fa6-ed77-4e7a-b54b-58f48c4eb2bc" />

- NetScaler is actively forwarding syslog to Aria Operations for Logs.

### Check Aria Logs

- Login to ARIA Operations for Logs and navigate to Explore logs in the left pane.


<img width="1918" height="793" alt="image" src="https://github.com/user-attachments/assets/7956fc80-1c6b-48dd-a159-343a56461a13" />

- VMware Aria Operations for Logs dashboard showing successful NetScaler syslog integration.

- When we search for the 192.168.1.45 which is the VIP for load balanced Horizon connection servers, the following logs are generated in ARIA from NetScaler.

<img width="1847" height="415" alt="image" src="https://github.com/user-attachments/assets/ef985ec8-b985-4b3a-8787-baa95b9b8910" />

### Connection flow

  - Client: 192.168.1.4:53502 (Horizon client)

  - Target VIP (vServer): 192.168.1.45:443 

  - SNIP used by ADC: 192.168.1.126:45050

  - Backend server: 192.168.1.167:443 (Horizon connection server)

- Integration complete and healthy.

## 

## Omnissa Horizon Configuration Steps

- Login to ARIA Operations for Logs console and navigate to Management > Agents on the left pane and click on Download VMware Aria Operations for Logs Agent Version 8.18.0.

  <img width="1916" height="857" alt="image" src="https://github.com/user-attachments/assets/4faf0785-595e-4054-9ef3-c568b7fb1265" />


- Run the installer on the Horizon connection server

  <img width="1023" height="630" alt="image" src="https://github.com/user-attachments/assets/afaea626-2cd7-4832-b3e9-e19a8067b85f" />


- Provide the ARIA Operation for Logs server IP address

  <img width="635" height="498" alt="image" src="https://github.com/user-attachments/assets/369fed91-4b5b-4f86-ac5e-1ee03b2c0d63" />

- Navigate to C:\ProgramData\VMware\Log Insight Agent and edit the liagent.ini file.

<img width="865" height="323" alt="image" src="https://github.com/user-attachments/assets/978147a2-0c20-4e62-90f2-df3076c57e20" />



- proto=cfapi
- port=9543
- ssl=yes
  
- Add the following to the file.
  
```ini
[filelog|HorizonViewCS]
directory=C:\ProgramData\VMware\VDM\logs
include=log-*;debug-*
exclude=pcoip_perf*.txt;v4v*.log;wsnm_starts.txt
```
- Restart VMware Aria Operations for Logs Agent service.

- Login from Horizon client to generate traffic.

  <img width="1593" height="705" alt="image" src="https://github.com/user-attachments/assets/87bbffd8-1b90-4c6e-804d-042e3f8fc53f" />

- Horizon connection server started forwarding logs to Aria Operations for Logs.

<img width="1917" height="841" alt="image" src="https://github.com/user-attachments/assets/c324694e-c106-4044-b19d-3a33b155b8d5" />

### Horizon Login Failure: "Loading Failed" Error

- We get Loading Failed on Horizon client, Aria captures event bursts exactly when the client attempts login.

<img width="1596" height="593" alt="image" src="https://github.com/user-attachments/assets/96de9159-364a-4b39-b90e-6a87f02cc3b8" />












