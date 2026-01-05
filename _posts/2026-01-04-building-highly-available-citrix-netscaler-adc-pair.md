## Building a Highly Available Citrix NetScaler ADC Pair

High availability is a critical requirement for any production-grade load balancing infrastructure. A NetScaler HA deployment uses two appliances working as a synchronized pair to ensure continuous service delivery. One node operates as the primary, actively handling client connections and backend traffic, while the secondary node remains in a monitoring and standby role. In the event of a failure or service disruption on the primary node, the secondary node automatically takes over, maintaining uninterrupted application access.

## Environment Details

The configuration demonstrated in this blog is based on a real-world internal deployment. 
All IP addresses and hostnames shown are representative and used for documentation purposes only.

### Citrix NetScaler ADC

- Platform: Citrix NetScaler VPX
- Version: NS14.1: Build 51.72.nc
- Licensing: Platinum Edition
- Deployment Model: High Availability (Active / Passive)
- Hypervisor: VMware vSphere (ESXi)


### Network Configuration

- Internal subnet: 192.168.1.0/24
- Virtual IP (VIP): 192.168.1.45
- HA heartbeat: Default (NSIP-based)

### Load-Balanced Application

- Application: Omnissa Horizon Connection Server
- Number of servers: 3
- Protocol: TCP 443 (SSL_BRIDGE)
- Persistence: SOURCEIP

### Scope of This Blog

- Focuses on configuring High Availability for Citrix NetScaler ADC
- Internal user access only
- Does not cover:
  - Unified Access Gateway (UAG)
  - External access
  - GSLB or multi-site HA
 
## High Availability Configuration Steps

### Step 1: Edit the HA settings on HA Node 2 (in this example NSVPX-PROD-02)


- Go to System > High Availability > Nodes > Edit

<img width="1917" height="592" alt="image" src="https://github.com/user-attachments/assets/f9ddd31d-d665-4d11-8d07-b8c2e90bd7f8" />

- On High Availability Status > Select STAY SECONDARY (Remain in Listen Mode) > Click OK

<img width="923" height="776" alt="image" src="https://github.com/user-attachments/assets/7e0a3ff5-7248-4e22-b32a-83b159ad046b" />

### Step 2: Edit the HA settings on HA Node 1 (in this example NSVPX-PROD-01)

- Go to System > High Availability > Nodes > Edit
- On High Availability Status > Select STAY PRIMARY > Click OK

<img width="857" height="762" alt="image" src="https://github.com/user-attachments/assets/fbdbe440-0674-4216-9a84-a60f61c7f946" />


### Step 3: Add HA Node 2 on Node 1 (NSVPX-PROD-01)

- Go to System > High Availability > Nodes > Add

<img width="865" height="852" alt="image" src="https://github.com/user-attachments/assets/461156f4-ec6c-45ec-a69b-55d9bf695f9d" />



<img width="1918" height="587" alt="image" src="https://github.com/user-attachments/assets/d3b1ff6a-7f5a-4441-830b-454d2665442f" />


### Step 4: Change the Node state on Node 1(NSVPX-PROD-01)

- Go to System > High Availability > Nodes > Select Node 1 > Edit

<img width="1917" height="592" alt="image" src="https://github.com/user-attachments/assets/4b267178-308f-46f4-b6f8-eaab8c7eb990" />


-  On High Availability Status > Select ENABLED (Actively Participate in HA)

<img width="857" height="717" alt="image" src="https://github.com/user-attachments/assets/94f6dc7a-5ddc-4599-a9bf-0023eb4e8fb7" />



### Step 5: Change the Node state on Node 2(NSVPX-PROD-02)

- Go to System > High Availability > Nodes > Select Node 2 > Edit
- On High Availability Status > Select ENABLED (Actively Participate in HA)


### HA Nodes Status on Node 1(NSVPX-PROD-01)


<img width="1918" height="542" alt="image" src="https://github.com/user-attachments/assets/2bf0392a-4c62-44ce-8aed-54410b99488f" />


### HA Nodes Status on Node 2(NSVPX-PROD-02)


<img width="1918" height="538" alt="image" src="https://github.com/user-attachments/assets/7f6f9af5-887f-4782-910a-8e44f8767b4e" />




## Failure Testing and Validation


### Test : Primary Node Service Failure Simulation

This test verifies automatic failover during an unexpected service disruption.

1. On the Primary node, simulate a failure by:
   - Disabling all interfaces **or**
   - Powering off the VM
2. Monitor the Secondary node behavior

**Expected Outcome:**
- Secondary node automatically assumes the **PRIMARY** role
- VIP ownership transfers seamlessly
- Applications remain accessible to users

In this test, we disabled the NIC on Node 1


<img width="1040" height="557" alt="image" src="https://github.com/user-attachments/assets/ed4e62f0-ba52-4865-8342-be16e94a7a05" />



On Node 2, HA status now changed to Primatry as shown below


<img width="1918" height="545" alt="image" src="https://github.com/user-attachments/assets/316b417f-417c-4b4f-930b-a7c5623fc1ca" />




To validate High Availability behavior at the network level, nstcpdump.sh was executed simultaneously on both NetScaler nodes while Horizon client traffic was generated. As shown above, client traffic destined for the VIP (192.168.1.45:443) is captured only on the active node (NSVPX-PROD-02). The passive node (NSVPX-PROD-01) does not receive any client traffic, confirming that VIP ownership and data-plane processing are exclusive to the active HA node.




### Packet capture demonstrating that Horizon client traffic is processed exclusively by the active NetScaler HA node




<img width="1918" height="642" alt="image" src="https://github.com/user-attachments/assets/8e3a8724-82d8-41bd-ad5b-f116b91f8342" />


### Observations
	-NSVPX-PROD-01 (Inactive / Secondary Node)
	-No TCP SYN, ACK, or application data packets were captured
	-Only the tcpdump initialization output was displayed
	-No client traffic was observed

This confirms that the Secondary node does not process client data-plane traffic while in the passive role.

	-NSVPX-PROD-02 (Active / Primary Node)
	-TCP three-way handshake (SYN / SYN-ACK / ACK) was observed
	-TLS and application data packets were captured
	-Continuous traffic flow was visible for the Horizon client session

This confirms that:

	-The VIP (192.168.1.45) is owned by the active node

	-Client traffic is processed exclusively by the Primary node

### Summary

Packet-level validation using nstcpdump.sh provides a reliable and transparent method to verify NetScaler HA behavior.
The results confirm correct VIP ownership, proper HA role enforcement, and adherence to NetScaler Active/Passive design principles.

This validation step is strongly recommended as part of production HA testing and operational acceptance.










  

