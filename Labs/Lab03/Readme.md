## Implement DCHP

### Implement DHCPv4

###  Objectives:
1. Build the Network and Configure Basic Device Settings
2. Configure and verify two DHCPv4 Servers on R1
3. Configure and verify a DHCP Relay on R2


### Solution

 1. [Build the Network and Configure Basic Device Settings](Readme.md#1-build-the-network-and-configure-basic-device-settings)

 2. [Configure and verify two DHCPv4 Servers on R1](Readme.md#2-configure-and-verify-two-dhcpv4-servers-on-r1)
 
 3. [Configure and verify a DHCP Relay on R2](Readme.md#3-configure-and-verify-a-dhcp-relay-on-r2)

 ### 1. Build the Network and Configure Basic Device Settings
 <br>

 Topology of the network
 <br>

 ![](/Labs/Lab03/pics/topology_eve.png)

 #### Addressing table

 |Device| Inteface| IP Address| Subnet Mask   |Default Gateway|
 |:----:|:-------:|:---------:|:-------------:|:--------------|
 |R1    | G0/0 <br>G0/1</br>G0/1.100</br>G0/1.200</br>G0.1/1000 |10.0.0.1<br>N/A<br>192.168.1.1</br>192.168.1.65</br>N/A</br>  |255.255.255.252<br>N/A<br>255.255.255.192</br>255.255.255.224</br>N/A</br>| N/A<br></br></br></br></br>|
 |R2|G0/0<br>G0/1</br>|10.0.0.2<br>192.168.1.97 </br>|255.255.255.252<br>255.255.255.240 </br>|N/A<br></br>|
 |S1|VLAN 200|192.168.1.66|255.255.255.224|192.168.1.65| 
 |S2|VLAN 1|192.168.1.98|255.255.255.240||
 |PC-A|NIC|DHCP|DHCP|DHCP|
 |PC-B|NIC|DHCP|DHCP|DHCP| 

 <br>        

#### VLAN Table

|VLAN |Name       |Interface Assigned|
|:---:|:---------:|:----------------:|
|1    |N/A        |S2:e0/1           |
|100  |Clients    |S1:e0/1           |
|200  |Management |S1: VLAN 200      |
|999  |Parking_Lot|S1:e0/2, e0/3     |
|1000 |Native     | N/A              | 

<br>

#### Step 1: Establish an addressing scheme:

   Subnet the network 192.168.1.0/24 to meet the following requirements:
  1. “Subnet A”, supporting 58 hosts (the client VLAN at R1) 
   <br>
   Subnet A:  **192.168.1.0/26, address range: 192.168.1.1 - 192.168.1.62**
   <br>
   *Record the first IP address in the Addressing Table for R1 G0/0/1.100*

  2. “Subnet B”, supporting 28 hosts (the management VLAN at R1 )
  <br>
   Subnet B:  **192.168.1.64/27, address range: 192.168.1.65 - 192.168.1.94**  
   <br>
   *Record the first IP address in the Addressing Table for R1 G0/0/1.200. 
   Record the second IP address in the Address Table for S1 VLAN 200 and enter the associated default gateway*
   3. “Subnet C”, supporting 12 hosts (the client network at R2)
   <br>
   Subnet C: **192.168.1.96/28, address range: 192.168.1.97 - 192.168.1.110**
   <br>
   *Record the first IP address in the Addressing Table for R2 G0/1*

   #### Step 2: Configure Inter-VLAN Routing on R1

   - Config of sub-interfaces on R1:

   ![](/Labs/Lab03/pics/R1_sub_int.png)

   - Switch ON G0/1 on R1

   ![](/Labs/Lab03/pics/R1_G01_ON.png)

   <b>


#### Step 3: Configure G0/1 on R2, then G0/0 and static routing for both routers

  - Configure interface G0/1 (Client network) on R2
  <br>

   ![](/Labs/Lab03/pics/R2_G01_ON.png)


  - Configure interface G0/0 on R2

  ![](/Labs/Lab03/pics/R2_G00_Conf.png) 


 - Configure interface G0/0 on R1

  ![](/Labs/Lab03/pics/R1_G00_Conf.png) 


- Configure a default route on each router pointed to the IP address of G0/0 on the other router.

![](/Labs/Lab03/pics/R1_def_route_to_R2.png)
![](/Labs/Lab03/pics/R2_def_route_to_R1.png)

-	Verify static routing is working by pinging R2’s G0/0 and G0/1 address from R1.

![](/Labs/Lab03/pics/Ping_R1_to_R2.png)

<b>

#### Step 4: Create VLANs on S1.

- Create and name the required VLANs on switch 1 from the table above

![](/Labs/Lab03/pics/Show_VLANs_S1.png)

-	Configure and activate the management interface on S1 (VLAN 200) using the second IP address from the subnet calculated earlier. Additionally, set the default gateway on S1

![](/Labs/Lab03/pics/S1_VLAN%20200.png)


- Configure and activate the management interface on S2 (VLAN 1) using the second IP address from the subnet calculated earlier. Additionally, set the default gateway on S2

![](/Labs/Lab03/pics/S2_VLAN%201_DG.png)

- Assign all unused ports on S1 to the Parking_Lot VLAN, configure them for static access mode, and administratively deactivate them. On S2, administratively deactivate all the unused ports.

S1:

![](/Labs/Lab03/pics/S1_Pariking_Lot_Inter.png)

S2:

![](/Labs/Lab03/pics/S2_Unused_shut.png)

<b>

#### Step 5: Assign VLANs to the correct switch interfaces

Assign VLAN 100 to S1`s port et0/1

![](/Labs/Lab03/pics/S1_assign_VLAN%20100.png)

VLAN table of S1:

![](/Labs/Lab03/pics/S1_VLANs.png)

VLAN table of S2:

![](/Labs/Lab03/pics/S2_VLANs.png)

<b>

#### Step 6: Manually configure S1’s interface F0/5 as an 802.1Q trunk.

setup of trunk on S1:

![](/Labs/Lab03/pics/S1_trunk_setup.png)

status of trunk on S1:

![](/Labs/Lab03/pics/S1_trunk_status.png)

### 2. Configure and verify two DHCPv4 Servers on R1

Configure and verify a DHCPv4 Server on R1. The DHCPv4 server will service two subnets, Subnet A and Subnet C

#### Step 1: Configure R1 with DHCPv4 pools for the two supported subnets. Only the DHCP Pool for subnet A is given below

- Exclude the first five useable addresses from each address pool.
<br>
Subnet A:  **192.168.1.0/26, address range: 192.168.1.6 - 192.168.1.62** <br>
Subnet C: **192.168.1.96/28, address range: 192.168.1.102 - 192.168.1.110**

Settings for DHCP first server 

![](/Labs/Lab03/pics/R1_First_DHCP.png)

*lease time initally was wrongly set for 1:12:30, eventually was changed to 2:12:30

The final DHCP config with two servers:

![](/Labs/Lab03/pics/DHCP_Config_R1.png)

Step 2: Attempt to acquire an IP address from DHCP on PC-A

PC-A has successfully recieved IP address from first DHCP server: 

![](/Labs/Lab03/pics/PC_A_DHCP_recieved.png)

### 3. Configure and verify a DHCP Relay on R2

Step 1: Configure R2 as a DHCP relay agent for the LAN on G0/1

Configure ip helper address on R2:
![](/Labs/Lab03/pics/R2_helper_address.png)

The helper address is IP of R1 G0/0

Step 2: Attempt to acquire an IP address from DHCP on PC-B

PC-B has successfully recieved IP address from first DHCP server: 
![](/Labs/Lab03/pics/PC_B_DHCP.png)


Step 3: Acqure DHCP  servers statistic from R1

DHCP statistic:
<br>

![](/Labs/Lab03/pics/R1_DHCP_statistic.png)

DHCP binding:

<br>

![](/Labs/Lab03/pics/R1_DHCP_binding.png)
