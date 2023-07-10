# Topology:



**The objective of this exercise is to create a simulated network, using an ASA connected to an ISP router.**


This exercise was made using Cisco Packet Tracer to simmulate a real network. However, the tool does have some limitations.

![image](https://github.com/AfonsoFerreira2223/ASA_network/assets/114146560/8d225f76-b718-48a9-a427-858f5337e895)


This topology includes:

|Networks:|  Hosts:|
|--|--|
| inside network (172.16.1.0/24) |Three PCs on the inside  network using ASA as the DHCP server
outside network (94.65.3.0/24| An internal web server connected to the ASA using address 192.168.1.2 |
|server network (192.168.1.0/24) |Two servers on the internet (97.65.3.100 and 97.65.3.101)
| DNS server (8.8.8.0/24) | A computer on the 95.65.3.0/24 network receing an Ip from the ISP router
| A web server and ftp server on the internet (97.65.3.0/24)|A dns server with the Ip address 8.8.8.1

---------


|Asa| ISP Router |
|--|--|
| Outside Ip address 94.65.3.2 | interface connected to the ASA with Ip 94.65.3.1
Inside Ip address 172.16.1.1 | interface connected to the DNS server with address 8.8.8.8 |
|Server address 192.168.1.1 | interface connected to the two internet servers with address 97.65.3.254
