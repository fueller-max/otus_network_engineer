## Implement DCHP

### Implement DHCPv6


### Objectives
1. Build the Network and Configure Basic Device Settings
2. Verify SLAAC address assignment from R1
3. Configure and verify a Stateless DHCPv6 Server on R1
4. Configure and verify a Stateful DHCPv6 Server on R1
5. Configure and verify a DHCPv6 Relay on R2

#### Background / Scenario
The dynamic assignment of IPv6 global unicast addresses (GUA) can be configured the following three ways:
*	Stateless Address Auoconfiguration (SLACC)
*	Stateless Dynamic Host Configuration Protocol for IPv6 (DHCPv6)
*	Stateful DHCPv6
When using SLACC to assign IPv6 addresses to hosts a DHCPv6 server is not used. Because a DHCPv6 server is not used when implementing SLACC, hosts are unable to receive additional critical network information, including a domain name server (DNS) address as well as a domain name.

When using Stateless DHCPv6 to assign IPv6 addresses to host, a DHCPv6 server is used to assign the additional critical network information, however the IPv6 address is assigned using SLACC.
When implementing Stateful DHCPv6, a DHCPv6 server assigns all network information, including the IPv6 address.
The determination of how hosts obtain they dynamic IPv6 addressing is dependent on flag setting contain within the router advertisement (RA) messages.


### Solution

1. [Build the Network and Configure Basic Device Settings](Readme.md#1-build-the-network-and-configure-basic-device-settings)

2. [Verify SLAAC address assignment from R1](Readme.md#2-verify-slaac-address-assignment-from-r1)

3. [Configure and verify a Stateless DHCPv6 Server on R1](Readme.md#3-configure-and-verify-a-stateless-dhcpv6-server-on-r1)

4. [Configure and verify a Stateful DHCPv6 Server on R1](Readme.md#4-configure-and-verify-a-stateful-dhcpv6-server-on-r1)

5. [Configure and verify a DHCPv6 Relay on R2](Readme.md#5-configure-and-verify-a-dhcpv6-relay-on-r2)

### 1. Build the Network and Configure Basic Device Settings
 <br>

 Topology of the network
 <br>

 ![](/Labs/Lab03/v6/pics/topology_eve.png)


 #### Addressing table

 |Device| Inteface| IP Address| 
 |:----:|:-------:|:---------:|
 |R1    | G0/0<br></br>G0/1</br> |2001:db8:acad:2::1 /64<br>fe80::1</br>2001:db8:acad:1::1/64</br>fe80::1 |
 |R2|G0/0<br></br>G0/1</br>|2001:db8:acad:2::2/64<br>fe80::2</br>2001:db8:acad:3::1 /64</br>fe80::1|
 |PC-A|NIC|DHCP|DHCP|
 |PC-B|NIC|DHCP|DHCP|


<br>

 

 #### Step 1: Enable IPv6 Globally
  <br>

  ![](/Labs/Lab03/v6/pics/R1_en_IPV6.png)
  
  <br>

  ![](/Labs/Lab03/v6/pics/R2en_IPV6.png)


 #### Step 2: Configure interfaces and routing for both routers.
 
 <br>

 G0/0 interface of R1:

 ![](/Labs/Lab03/v6/pics/R1_G00_IP_addr.png)

  <br>

 G0/0 interface of R2:

 ![](/Labs/Lab03/v6/pics/R1_G01_IP_addr.png)

  <br>

  G0/0, G0/1 interface of R2:

 ![](/Labs/Lab03/v6/pics/R2_G00_G01_IP_addr.png)


  Default routes:
  <br>

  ![](/Labs/Lab03/v6/pics/R1_def_route.png)

  <br>
  
  ![](/Labs/Lab03/v6/pics/R2_def_route.png)

   Show brief ipv6 interfaces of R1:
   <br>

   ![](/Labs/Lab03/v6/pics/R1_show_IPv6_brief.png)

   Show brief ipv6 interfaces of R2:
   <br>
   
   ![](/Labs/Lab03/v6/pics/R2_show_IPv6_brief.png)
  
   G0/0 ipv6 interface of R1:
   <br>

   ![](/Labs/Lab03/v6/pics/R1_G00_%20ipv6_Interface.png)

   All other interfaces are simular

   Ping R2 ip(2001:db8:acad:2::2) from R1:
   <br>

   ![](/Labs/Lab03/v6/pics/R1_ping_R2.png)

   Ping R1 ip(2001:db8:acad:2::1) from R2:
   <br>

   ![](/Labs/Lab03/v6/pics/R2_ping_R1.png)


### 2. Verify SLAAC address assignment from R1

ipv6 settings on PC-A are set to auto. After a moment the PC recieved ipv6 address:

![](/Labs/Lab03/v6/pics/PC_A_ipv5_SLAAC.png)

As we can see the PC gets the ipv6 address 2001:db8:acad:1:2050:79ff:fe66:6805

The first 64 bits comes from router`s RA (Router Advertisiment): 2001:db8:acad:1
The second 64 bit are formed via EUI-64 based on PC MAC address:

MAC address: 00:50:79:66:68:05, after EUI-64 comes: 2050:79ff:fe66:6805 (added fffe in the middle + set 7th bit to "1")


### 3. Configure and verify a Stateless DHCPv6 Server on R1

As we can see from previous screenshot there is no DNS server provided. SLAAC cannot provide DNS by itslef. We need DHCPv6 to provide DNS information for the hosts.


#### Step1: provide stateless DHCPv6 

![](/Labs/Lab03/v6/pics/R1_DHCP_stateless.png)

![](/Labs/Lab03/v6/pics/R1_DHCP_on_G01.png)

Check PC-A ip settings.

Unfortunatelly, looks like there is a bug in VPC implementation and DNS info is not presented:

![](/Labs/Lab03/v6/pics/PC_A_ipv5_SLAAC.png)


We configure another router R3(as dhcp v6 clinet) instead of VPC to test IPv6 address assigment + DNS information:

![](/Labs/Lab03/v6/pics/R3_IPv6_SLAAC.png)

<br>

![](/Labs/Lab03/v6/pics/R3_DNS_recieved.png)

As we can see, the R3 has GUA + DNS information as expected.

### 4. Configure and verify a Stateful DHCPv6 Server on R1

Create a DHCPv6 pool on R1 for the 2001:db8:acad:3:aaaa::/80 network. This will provide addresses to the LAN connected to interface G0/1 on R2. As a part of the pool, set the DNS server to 2001:db8:acad::254, and set the domain name to STATEFUL.com.

![](/Labs/Lab03/v6/pics/R1_DHCP_statefull.png)


### 5. Configure and verify a DHCPv6 Relay on R2

#### Step 1: Examine PC-B the SLAAC address that it generates

![](/Labs/Lab03/v6/pics/PC_B_IPv6_from_SLAAC.png)


As we can see the IPv6 address prefix is 2001:db8:acad:3 (according to IPv6 address of G0/1 R2)

#### Step 2: Configure R2 as a DHCP relay agent for the LAN on G0/1

![](/Labs/Lab03/v6/pics/R2_DHCPv6_realy.png)

#### Step 3: Attempt to acquire an IPv6 address from DHCPv6 on PC-B

We are using Win10 host to see IPv6 IP/DNS assigment from DHCPv6 server.
<br>

![](/Labs/Lab03/v6/pics/Win10_DHCP_IPv6.png)

As we can see IPv6 address has prefix 2001:db8:acad:3:aaa  whch corresponds to DHCP`s 
server pool.
Also we can see DNS settings recieved from DHCP server.












   
    

    





     


