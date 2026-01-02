---
Title: "Load Balancing Omnissa Horizon Connection Servers using Citrix NetScaler ADC"
Date: 2026-01-02
Categories: [Omnissa, Horizon, Citrix, ADC, Load Balancing]
---

## Overview
This article explains how to load balance Omnissa Horizon Connection
Servers using Citrix NetScaler ADC in an enterprise environment.

## Architecture
- 3 × Omnissa Horizon Connection Servers.
- Citrix NetScaler ADC (VPX/MPX).
- SSL termination and persistence considerations.
<img width="1918" height="502" alt="image" src="https://github.com/user-attachments/assets/8b8c4c6f-182f-43a6-9d32-12c1664d43de" />

## Step 1: Create the SSL Monitor
This monitor checks Connection Server health via HTTPS GET /favicon.ico.
1. Go to Traffic Management > Load Balancing > Monitors > Add.
   <img width="1918" height="866" alt="image" src="https://github.com/user-attachments/assets/33bf9014-9fd7-4663-8150-825774aa57b7" />
2. Name: mon-horizon-cs-ssl.
3. Type: Select HTTP.
4. Interval: 30 seconds (recommended)
5. HTTP Request: GET /favicon.ico.
6. Check Secure.
7. Expand Advanced Parameters:
   - Destination Port: 443.
   - TROFS Code: 503 (handles quiesce/out-of-service gracefully).
8. Click Create.
   <img width="1916" height="767" alt="image" src="https://github.com/user-attachments/assets/0e7ec47a-b589-4d6a-893c-5aa196db385a" />

## Step 2: Create Server Objects
Define your Connection Servers.

1. Go to Traffic Management > Load Balancing > Servers > Add.
   
<img width="1903" height="758" alt="image" src="https://github.com/user-attachments/assets/aafc897f-b534-4ba8-be21-96e3faef11ab" />


2. Click Create after each.
   
   <img width="870" height="717" alt="image" src="https://github.com/user-attachments/assets/9549db66-b917-4a18-851e-e598399837ab" />

## Step 3: Create the Service Group

Groups your servers on TCP 443.

1. Go to Traffic Management > Load Balancing > Service Groups > Add.

   <img width="1913" height="767" alt="image" src="https://github.com/user-attachments/assets/ed538d40-e0fb-4b7e-84d5-1d8c890af439" />

2. Name: lb-svcgrp-horizon-cs-443
3. Protocol: SSL_BRIDGE. Click OK.
4. Goto Service Group Members and expand it.
   -Select Server Based.
   -Add all 3 servers (conn-svr-1, etc.).
   -Port: 443.
   -Click Create for each.
5. Bind monitor:
       - Click Monitors on the right panel.
       - Bind mon-horizon-cs-ssl
       - Click Bind > Done.

<img width="1918" height="823" alt="image" src="https://github.com/user-attachments/assets/13e30947-8d04-49bf-9f60-631b31696de9" />

<img width="1915" height="832" alt="image" src="https://github.com/user-attachments/assets/1831af9d-abe2-432b-99f8-23c335f7f7c9" />

<img width="1918" height="822" alt="image" src="https://github.com/user-attachments/assets/3d43ba81-f6a4-4030-b5a0-d902821ec505" />


## Step 4: Create the Load Balancing Virtual Server (VIP)

1. Go to Traffic Management > Load Balancing > Virtual Servers > Add.

<img width="1916" height="756" alt="image" src="https://github.com/user-attachments/assets/9e3c3a9c-0883-4463-9cec-73b9bdd40b67" />

2. Name: prod-vs-horizon-cs-443
3. Protocol: SSL_BRIDGE.
4. IP Address: Your internal VIP (e.g., 192.168.1.100).
5. Port: 443. Click OK.
6. Goto Services and Service Groups.

<img width="1915" height="782" alt="image" src="https://github.com/user-attachments/assets/db3957a7-07f0-4b79-a9ce-e1970fe8e374" />

7. Bind lb-svcgrp-horizon-cs-443.

<img width="1910" height="526" alt="image" src="https://github.com/user-attachments/assets/c1a7020b-b7ed-4748-acb2-d120dfe0b471" />

<img width="1916" height="668" alt="image" src="https://github.com/user-attachments/assets/440a965b-465d-48b2-9a67-28f9b5156cef" />


## Step 5: Configure Source IP Persistence

1. Edit the virtual server prod-vs-horizon-cs-443
2. Under Persistence, select SOURCEIP.
3. Timeout: ≥600 minutes
4. Click OK.

<img width="1513" height="607" alt="image" src="https://github.com/user-attachments/assets/c1bdd7ad-dd74-4397-8791-d6d510413ade" />

## Step 6: Horizon Connection Server Configuration

Edit locked.properties on each Connection Server to allow load-balanced access.

1. Path: C:\Program Files\Omnissa\Horizon\Server\sslgateway\conf\locked.properties (create if missing).
2. Add:
        - allowUnexpectedHost=true
        - checkOrigin=false
        - enableCORS=false
3. Restart Omnissa Horizon Connection Server service.


## Step 7: DNS and Testing

- Point internal DNS (e.g., horizon.internal.company.com) to the VIP.
- Test:
        - Access Horizon Client/Admin Console via VIP.

        - Check NetScaler stats: Traffic Management > Load Balancing > Virtual Servers (status UP, connections distributing).

        - Monitor logs: Disable a server in Horizon Console—monitor should mark it DOWN.



<img width="1680" height="621" alt="image" src="https://github.com/user-attachments/assets/8ffaa966-aa33-49bb-bcaa-c50a9fb196af" />


<img width="1221" height="777" alt="image" src="https://github.com/user-attachments/assets/4b83f2ee-4d41-4c63-a50c-5e2f0c61817f" />


<img width="1193" height="247" alt="image" src="https://github.com/user-attachments/assets/01fd3ca8-c782-4187-9175-cfa60c876818" />


<img width="1917" height="867" alt="image" src="https://github.com/user-attachments/assets/f516afa9-531a-4fb5-bef2-feb1a0212340" />



## Analyzing Omnissa Horizon Client Traffic Using nstcpdump on Citrix ADC

<img width="1612" height="802" alt="image" src="https://github.com/user-attachments/assets/a3bdfb2d-887b-405f-929a-71de65bf0dad" />





        
