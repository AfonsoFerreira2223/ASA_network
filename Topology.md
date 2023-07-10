# Topology:



**The objective of this exercise is to create a simulated network, using an ASA connected to an ISP router.**


This exercise was made using Cisco Packet Tracer to simulate a real network. However, the tool does have some limitations.

![image](https://github.com/AfonsoFerreira2223/ASA_network/assets/114146560/8d225f76-b718-48a9-a427-858f5337e895)


This topology includes:

|Networks:|  Hosts:|
|--|--|
| inside network (172.16.1.0/24) |Three PCs on the inside  network using ASA as the DHCP server
outside network (94.65.3.0/24| An internal web server connected to the ASA using address 192.168.1.2 |
|server network (192.168.1.0/24) |Two servers on the internet (97.65.3.100 and 97.65.3.101)
| DNS server (8.8.8.0/24) | A computer on the 95.65.3.0/24 network receiving an Ip from the ISP router
| A web server and ftp server on the internet (97.65.3.0/24)|A dns server with the Ip address 8.8.8.1

---------


|Asa| ISP Router |
|--|--|
| Outside Ip address 94.65.3.2 | interface connected to the ASA with Ip 94.65.3.1
Inside Ip address 172.16.1.1 | interface connected to the DNS server with address 8.8.8.8 |
|Server address 192.168.1.1 | interface connected to the two internet servers with address 97.65.3.254



### Inside network:


There are three PCs on the inside interface of the ASA, which will all receive an IP address from DHCP

As such, the only configuration done here was to set the pc NIC settings to DHCP instead of static

![image](https://github.com/AfonsoFerreira2223/ASA_network/assets/114146560/84389b31-801c-47fe-b88e-b1b50e39e7a9)



### Configuring the ASA


The first thing to do is to create and configure the interfaces.

    Int g1/3
	ip add 172.16.1.1 255.255.255.0
    nameif inside
    security-level 0
    no shut
    exit

These were the commands used to create an interface named Inside. In this instance, I am going to leave all the interfaces of the ASA set to security-level 0 for now.

Now that the interface is created, it's time to tell the ASA it will serve as a DHCP server for the 172.16.1.0/24 network.

    dhcpd address 172.16.1.10-172.16.1.15 INSIDE
    dhcpd dns 8.8.8.8 interface INSIDE
    dhcpd enable INSIDE

This will generate a pool of 5 IPs in the 172.16.1.0/24 range and assign them to the three computers on this network. It will also tell them which dns server to use, which will be configured later on.


As for the other interfaces, the process is similar to the first one.
```
Int g1/1

ip add 94.65.3.2 255.255.255.0
nameif outside
security-level 0
no shut
exit
```

```
Int g1/2
ip add 192.168.1.1 255.255.255.0
nameif server
no shut
exit
```

Now that all the interfaces are created, let's move to the next step.

The PCs might not be getting a DHCP address yet. This is because the ASA is most likely blocking ports 68 and 67, which are used for DHCP.

The commands to create an access-list for the inside interface to let DHCP traffic through are:

    access-list inside permit tcp any 172.16.1.0 255.255.255.0 eq 68
    access-list inside permit tcp any 172.16.1.0 255.255.255.0 eq 67

Since we will be using these PCs to access the internet, let's modify the access-list now, to accept ports 80, 443, 53, and 20/21.

The commands are exactly the same as the above, the only difference will be the port number.
Also, keep in mind port 53 should be udp instead of tcp.

Let's also allow icmp traffic to flow in the network, as it is useful to verify connectivity:

    access-list inside permit icmp any any

And finnaly apply the rules with the command 

    access-group inside in interface inside

This will bind the access-list rules with the interface inside, which is where they should be applied.

Now in order to access the internet, these PCs need to translate their internal IPs into routable ones. 

We can configure dynamic NAT by using the interface of the ASA as their public IP:

    object network inside-nat
    subnet 172.16.1.0 255.255.255.0
    nat (inside, outside) dynamic interface
    exit

**Note: You can always allow ip traffic from a specific Ip range in case something is not working due to the firewall. To know if the ASA is blocking traffic, you can use simulation mode in packet tracer, which will show the path packets take and if they are being denied by a firewall configuration**



The configuration will be slightly different for the server network. This is because this web server is part of a DMZ, and as such some access to the outside will be denied.

Computers trying to access this web server from the outside network can't use a private Ip address, so the ASA will have to perform NAT to let the server reach the internet. 

Configuring static NAT on the server interface of the ASA:

    object network server-static
    host 96.65.3.1
    
    object server-static-outside
    host 192.168.1.2
    nat (server,outside) static server-static

Now the server in the dmz zone (192.168.1.2) can be accessed through the internet.

