---
title: Chapter 13 TCPIP integration
layout: book
---

# Chapter 13: TCP/IP Integration

Integrating Microsoft network addressing with UNIX servers and workstations presents unique challenges. Microsoft networking resolves addresses through a variety of methods: broadcast, browse servers, WINS, or DNS. UNIX systems typically use only DNS. Although WINS and DNS have some similarities, they differ in computer name conventions, support for dynamic address resolution, and support for hierarchical naming structures to distinguish the two address resolution methods.

DNS was developed to support a global computer deployment. Where Microsoft uses a flat topology, DNS uses a hierarchical system. DNS names are composed of a root organization, a domain in that organization, and possibly subdomains. The root organizations, such as **.com**, **.org**, **.gov**, and **.net** are established and maintained by the National Science Foundation. Domains are established within a root organization and the responsibility for that domain is delegated to the organization that registered the name.

For example, **mybusiness.com** is a domain registered within the **.com** commercial organization. **mybusiness** has a DNS that contains records for all the machines in that domain. For administrative convenience, **mybusiness** may further subdivide their domain by creating subdomains, such as **accounting.mybusiness.com**.

Microsoft networking was developed for small LANs. In this environment, network communications use Server Message Block [SMB] protocols, while the network internals use NetBIOS program calls. In small networks, the computers had unique names that readily identified them. As Microsoft networking moved into corporate networks, they retained their flat naming conventions since NetBIOS is based on unique workstation names. While ensuring unique computer names in a single office is a trivial task, enforcing it in a 10,000 user country-wide WAN becomes almost impossible. Most organizations enforce a location-specific naming scheme incorporating the computer's location, such as using the building and office number.

For example, Microsoft networking may use a computer name of **nyf10c13** to identify the computer in the New York office, 10th floor, and cubical 13. DNS, because of its hierarchical naming convention, has more freedom in assigning names. This computer could be identified by location, as in **f10c13.ny.mybusiness.com**, or by function, as in **payroll.accting.mybusiness.com**.

## Machine naming considerations

Initiatives to integrate DNS and WINS are usually driven by the need to supply address resolution for Dynamic Host Configuration Protocol (DHCP) addressed computers. Without DHCP, an administration must manually assign an IP address to each computer on the network. If the network is resegmented or workstations moved, the administrator must manually reassign addresses to all users.

Unfortunately, DNS uses static address tables. These tables must also be manually maintain and, when the tables are changed, DNS must be reinitialized to recognize the changes. Since DHCP supplies IP addresses on demand, it is unfeasible to manual modify the DNS tables and reinitialize DNS.

If manual IP address assignments were used in an environment with a large population of mobile laptop users, each user would have to have a manually assigned address for each subnet to which they connected. When the laptop was moved between locations, it would need to be reconfigured for the new subnet. Addresses must remain reserved even if that user is only at a location for a few hours a week. When a user wants to connect to that mobile computer to access its resources, the user must know in which location that mobile user is located at that time to determine the computer's address.

When DHCP is implemented, the IP address is automatically assigned from a pool of available addresses. When the computer is shut down, and the address is no longer required, it can be released back to the pool.

While DHCP reduces the administrators work load, the administrator cannot manually update the DNS table every time a user powers their workstation on. Microsoft introduced WINS to resolve this problem. When the computer powers on and obtains a DHCP address, it then registers with the WINS server so other computers can find it. WINS supports Microsoft's flat directory structure.

Although administrators can use manually assigned addresses and manually maintain DNS entries for all machines, this introduces several problems. As previously described, mobile users are a problem because they must have several addresses. Much like any mobile users, they must inform people what office they will be located in that day and the number in that office. If the new office is in a different DNS subdomain, they must reset their TCP/IP network configuration.

Also, Microsoft networking uses address lookup much more heavily then UNIX systems. Where a large office might have 100 UNIX servers, it may have 1000 PCs. The PCs with Microsoft networking will perform address resolutions when a user clicks on network neighborhood, connects to a printer or file share, tries to locate a browse master, or responds to an election. Many DNS servers can not handle the increased load when Microsoft workstations are set to use DNS for all address resolution.

However, an integrated environment must support both WINS and DNS. Although WINS provides the superior solution for Windows networking, DNS provides several areas of support where WINS does not. WINS only provides one-to-one address resolution. Each machine has only one address and one name. Although WINS supports machines with multiple network cards and addresses, ultimately each Windows machine has only a single primary network address. That is, either several names are assigned to a single server or a single name is assigned to multiple servers.

While both DNS and WINS provide address resolution, remember that Microsoft networking is built around NetBIOS. Each Microsoft networking computer has one and only one machine name. All network programs on this machine and other machines connected to the network communicate using this NetBIOS name.

In comparison, a Unix machine uses TCP/IP addresses for communications. DNS provides a method of translating friendly names into IP addresses. Because of this, DNS can be used to define several names for the same machine, where each name would resolve to the same IP address.

DNS also provides additional functionality for Unix machines for which there is no parallel in the Microsoft networking environment. For example, while each WINS server contains the entire address database, DNS allows the responsibility for the address space to be split between servers. So DNS allows you to specify for which portions of the DNS address space a DNS server is allow to perform address resolutions, as well as other DNS servers to which it may refer clients.

DNS also provides support for Simple Mail Transfer Protocol [SMTP] electronic messaging through the use of special DNS entries called MX records. An MX record identifies which mail server will accept incoming mail for a site.


Item | DNS | WINS
---|---|---
Address assignment method | Static | Static or dynamic
Supported names | Hierarchical Domain Names (mycompany.com)| NetBIOS Station names
Applications supported | Internet applications (FTP, SMTP, Telnet, etc. | Microsoft Networking
SMTP mail support | Yes | No
Many names for 1 IP address | Yes | Limited
Many IP addresses for one name | Yes | Limited
Support multiple network adapters | Yes | Limited

## DNS

The major difference between WINS and DNS is that DNS provides a hierarchical name space. Because Microsoft networking uses a flat name space, WINS does not allow the address space to be segmented to support separate organizational groups. For example, the DNS may contain **acct.mybusiness.com** or even **ny.acct.mybusiness.com.**

Unix machines also use relative DNS address resolution. So if a machine's name is ny.acct.mybusiness.com, and the machine tried to locate a machine called RECEIVABLES, by default it will try to locate a machine called RECEIVEABLES.ACCT.MYBUSINESS.COM.  DNS also support full name resolution, where the machines name and full path can be specified.

However, WINS sees a single flat address space. Because WINS does not allow segmentation into subdomains, WINS does not allow the database to be distributed between separate WINS servers. Although multiple WINS servers can be implemented, each server must contain the entire database. To ensure all WINS servers contain the same information, WINS uses replication. When a workstation needs to resolve an address, it contacts its WINS server, which contains all addressing information.

In contrast, DNS designates specific DNS servers as authoritative for specific domains and subdomains. In this structure, each domain and subdomain has an identified and well known server which contains all the addressing information for that subdomain. For example, when you look up the machine **ny.acct.business.com**, the DNS client contacts the DNS server which provides address resolution for **business.com**. This DNS server then refers the DNS client to the DNS server responsible for address resolution for **accting.business.com**.

DNS's support for segmenting your address list provides several advantages. Because each DNS server is responsible for only specific addressing, you can better tune the DNS servers for your environment by limiting its address database. You can also limit access to a specific group of servers by limiting which clients can access the DNS for those servers.

To support address list segmentation, DNS supports three types of DNS servers: primary, secondary, and caching.

The primary DNS server is authoritative for all machines in its domain. The primary DNS server contains the name of the domain for which it is authoritative and the time-to-live for the addresses on that server.  When an address is resolved using the DNS server, the machine which resolves the address knows that it can cache the address information for the time-to-live period, after which it must contact the DNS server for updated information.

To balance the load between DNS servers, you can define secondary DNS servers. The secondary DNS server pulls its addresses list from the primary DNS server, which is called a zone transfer. Once the time-to-live on these addresses has expired, the secondary server must perform another zone transfer to refresh its list.

The caching DNS server resolves addresses using either the primary or secondary DNS server, then caches the results, which are used for future address resolution requests. After the time-to-live expires, the caching server will discard the address information.

The concept of authoritative DNS servers also provides another major difference between WINS and DNS servers. While a WINS server contains the entire database and is authoritative for the entire domain, DNS may spread responsibility throughout the DNS system. To reduce address lookups, WINS allows clients to cache resolved addresses. DNS also allows caching, but both servers and clients cache address lookups, which may cause problems.

To resolve an address using DNS, the client may be required to contact several DNS servers before it can locate the server that is authoritative for the subdomain it is seeking. Typically, the DNS client contacts its DNS server, which then refers the client to the authoritative DNS server. However, the DNS server may also retrieve the information from the authoritative DNS, cache the result, and return the result to the DNS client.

Each DNS server contains a time to live. When a client or server resolves an address, it will assume that this address is correct for the time to live. Until this time expires, the client will not recontact the DNS server to verify that the address is correct. The typical time-to-live is 72 hours.

In a dynamic environment where network traffic may be redirected, this time may be reduced to a few minutes. For example, when web server content is spread across several web servers, you may need the flexibility of quickly relocating content to another server. When changing an address on DNS, the changed value will not necessarily be reflected on the clients until this time to live expires. This is similar to the WINS replication times, where a changed value will not be updated on the WINS servers until it replicates.

Address resolution demonstrates the largest difference between WINS and DNS environments. WINS provides much better support for dynamic address changes.

The DNS server contains a database of names and their corresponding IP address. This database is stored in a manually maintained flat file, much like the LMHOSTS file. Whenever addresses change, the administrator must manually update the DNS database, then reinitialize DNS to read the new database.

Once the DNS database is reinitialized, the administrator must wait for the time to live period, after which the DNS servers and clients which have cached the address will contact the DNS server to update their address cache. The DNS server has no way of telling who has cached its address information so both the old and new addressing information must be maintained until the time of live has expired.

When the WINS database is changed, all clients using that database will receive the updated information. The administrator does need to wait until this information is replicated to all WINS servers on the network, however the administrator can manually trigger this replication process.

## DNS Design Considerations

When designing a DNS system, many considerations are similar to the WINS design process.

Try to keep your subdomains consistent. If you change your subdomains, you must refresh all your DNS servers, as well as refreshing any DNS servers or clients that may be caching this information. Remember that new information will not be updated until the time to live has expired. Any changes you make may have to coexist with the old system until all servers and clients are updated.

Design your subdomains to keep the name space as small as possible. This means avoid using a single flat name space and avoid placing large numbers of hosts in a single domain. If you keep the subdomains small, you have more flexibility in defining authoritative DNS servers and moving the subdomains between DNS servers.

Decide how you will create your subdomains. Will they be organization-specific, location-specific, or both? While it is attractive to maintain the organization focus, if your organization is distributed throughout multiple physical locations you may experience high WAN traffic while remote hosts look up addresses. You can distribute DNS servers on each side of the slow WAN link, but then you are faced with placing DNS servers for each organizational unit in each location. This is similar to the problems faced when designing NT domains and locating domain controllers.

Also, WINS integration is simplified if you design your DNS subdomains around physical locations. If you are using DHCP, which is subnet-specific, it is easier to integrate WINS and DNS if your DNS subdomains are also subnet- or location-specific.

DNS uses a name replication process called zone transfer. During a zone transfer, a DNS server contacts the authoritative DNS server and requests all records for which that server is authoritative. While WINS replications are based on which records have changed, zone transfers pull all records for which the time to live has expired, which is usually all records in the database. When planning the location of your DNS servers, you should consider the effect that zone transfers will have on your WAN links.

For example, if BIGCO has two locations, New York and LA, joined by a slow communications link, you would want to define two subdomains: NY.BIGCO.COM and LA.BIGCO.COM.  Since LA is a small sales branch with 10 people, a single subdomain is acceptable.

However, the NY office contains both accounting and sale personnel, as well as implementing DHCP. We may want to separate the NY.BIGCO.COM subdomain into three separate subdomains: ACCTING.NY.BIGCO.COM, SALES.NY.BIGCO.COM, and DHCP.NY.BIGCO.COM.

## Samba

Samba consists of a group of public domain programs which work together to provide Server Message Block (SMB) services for a UNIX machine. In other words, the UNIX server acts as a Microsoft networking server and a Microsoft networking client. Once Samba is properly configured, clients can access file shares and printers. Samba will also operate as a browse server, browse client, and WINS client.

The individual Samba components can be run independently, so Samba can provide file and print sharing without acting as a browse server. The primary components are as follows:

* **Smbd:** The server software that handles client connections, user authentication against a domain controller, and file permissions.
* **Nmbd:** The NetBIOS name server that provides browsing services and domain machine account management.
* **Smbclient:** The Microsoft networking client.

If your goal is to allow your UNIX servers and Microsoft servers to share files and printers, then Samba provides an excellent solution. Because Samba operates as a WINS client, the UNIX servers running Samba should be accessible to Microsoft WINS clients. Samba can also provide support for DNS address resolution and WINS proxy service.

However, Samba should not be used as a browse server if Microsoft browse servers are also running on the same network. Similarly, Samba should not be configured as a domain controller if Microsoft domain controllers currently exist.

An improperly configured Samba server will announce as a browse server and, in its default configuration, cast the highest possible election vote, which forces the other servers to accept Samba as the browse master. Even if the Samba server is behaving improperly and not responding properly to address resolution request, Samba will win any subsequent elections.

Figure 13.1 NT/Samba Integration

Figure 13.1 shows a network consisting of two locations: LA and New York, with the PDC located in LA. At the LA location, the PDC will act as the browse master. In the NY office, we wish to run SAMBA on the Unix server as the browse master. The Unix server is running all three SAMBA components.

Before the SAMBA server will properly function as a browse master, you must configure the SAMBA server to both be a member of the domain and to act as a browse server for the domain.

To join the domain, you must first create a machine account for the SAMBA server. From the PDC, run Server Manager and select Add to Domain. Select the computer type "Windows NT Workstation or Server," enter the name of the Unix machine, then press the Add button.

Adding a server to the NT domain

Screen capture 30

Next, you must modify the SAMBA configuration file on the Unix server. By default, this file is located at /usr/local/samba/lib/smb.conf. In the global section, modify the following two lines:

```
workgroup=DOMAIN
password server=Loki
```

Finally, on the Unix server, run **smbpasswd -j DOMAIN** to join the domain.

## DHCP/BOOTP

DHCP and BOOTP both provide a method for administrators to assign IP addresses to workstations. In addition to an IP address, the workstation also needs a minimum of its subnet mask and gateway address.  Both BOOTP and DHCP allow the IP client to be automatically configured with all the information the client needs to function properly, including gateway addresses, DNS servers, mail servers, news servers, and time servers. DHCP provides an evolutionary step up from BOOTP, providing added capabilities for dynamic leases.


Feature | BOOTP | DHCP
---|---|---
Dynamic address assignment | No | Yes
Addresses assigned based on | Network adapter MAC address | Network adapter MAC address or dynamic
Flexible lease time | No | Yes
Provide additional IP configuration information | Yes | Yes

BOOTP assigns IP addresses based on the network adapter's MAC address. This address is built into the network adapter and is unique. The administrator must manually create a table that lists each MAC address and the IP configuration information to use for that MAC address.

On networks with rapidly changing configurations, such as many remote users,
  transient users, or laboratories, the number of possible MAC addresses in the
  BOOTP table may exceed the available number of IP addresses on the subnet.

Figure 13.3 BOOTP and DHCP Configuration

To configure BOOTP, you must modify the BOOTPD configuration file, which is usually located at /bin/bootptab. For NY network configuration shown in Figure 13.3, the bootptab file would be:

```
# First, define a global template
# sm= the subnet mask
# ds= DNS server
global.template:\
 :sm=255.255.255.0:\
 :ds=128.25.40.2:\

# Second, define a hardware class for all ethernet machines
# gw = gateway address
# ht = hardware type

ethernet.template:\
:gw=128.25.40.1:\
:ht=ethernet:

# Third, define the address information for each machine
# tc = which hardware template to use
# ha = the MAC address of the network adapter
# ip = the IP address for this machine

Unix1.bigco.com:\
:tc=ethernet.template:\
:ha=AA005A7FF724:\
:ip=128.25.40.2:\

Unix2.bigco.com:\
:tc=ethernet.template:\
:ha=AA005A7FF730:\
:ip=128.25.40.3:\

Unix3.bigco.com:\
:tc=ethernet.template:\
:ha=AA005A7FF736:\
:ip=128.25.40.4:\
```

DHCP addresses this address problem by enabling automatic configuration of the TCP/IP-related parameters during startup using the standard DHCP process. DHCP provides safe, reliable, and simple TCP/IP network configuration; ensures that address conflicts don’t occur; and helps conserve the use of IP addresses through centralized management of address allocation. The system administrator controls how IP addresses are assigned by specifying a lease duration. A lease specifies how long a computer can use an assigned IP address before having to renew the lease with the DHCP server.

As with BOOTP, DHCP allows IP configuration information to be based on the machine network adapter's MAC address. Normally a server's IP address would be manually configured, however DHCP can be used to supply a static address.  Remember that since the lease is based on the network adapter's MAC address and each network adapter has a unique MAC address, the lease must be modified if the network adapter is changed.

To configure DHCP for the LA network shown in Figure 13.3, you should run the DHCP Manager, select your DHCP server from the server list, and select Scope, Create. From this screen, shown in Figure 13.4, you enter the starting and ending addresses of your DHCP range. Notice that we have made the range larger then the current number of workstations to accommodate future growth. You also enter the subnet mask and the lease duration on this screen. When you have finished entering the scope, press the OK button.

Figure 13.4 Creating a DHCP scope

Now we must define the DNS server and default gateway for this scope. Select the 128.25.30.0 scope from the DHCP server list, then select DHCP Options, Scope.  From the unused options, we select 003 Router and 006 DNS Servers, then press the Value button.

Figure 13.5 Adding DHCP Scope options

Select 003 Router in the Active Options subwindow, and press the Edit Array button. Within IP Address Array Editor, you may either enter the IP address of the router or its DNS name, as shown in figure 13.6. The DNS address is set identically, although obviously the DNS server can not be specified by DNS name.

Figure 13.6 Setting the Router address.

If you are configuring a scope which supports laptop users, you should set the lease time to a short duration, such as 1 day. For desktop users, you may wish to extend the lease duration to a week.

A functional DHCP system consists of three components:

* **Server** The server provides IP leases and information, such as addresses for the subnet masks, DNS servers, WINS servers, gateways, or time servers.
* **Client** The workstation client contacts the DHCP server to obtain, renew, or release an IP address and related information.
* **Relay agent** Since DHCP broadcasts are limited to the subnet, the DHCP relay agent forwards the DHCP packets between subnets. The relay agent usually runs on the routers.

DHCP allows an administrator to assign a range of IP addresses to a specific subnet. When a workstation on the subnet broadcasts an address request, the DHCP server assigns an address for a specific lease period. The workstation is responsible for renewing the lease before it expires and, if the workstation does not renew the address, the server will return the address to the available pool of addresses.

The DHCP standards provide a basic feature set, as well as option feature sets. Microsoft implements the basic set, where some UNIX implementations provide additional features, such as providing addresses based on DNS subdomain. If you use a UNIX-based DHCP implementation, the DHCP standards specify that the DHCP server provider should also provide clients that support this enhanced functionality.  Since the Microsoft DHCP client is built into their IP client, the DHCP provider would have to supply the complete TCP/IP stack.

BOOTP and DNS both use static, manually maintained tables. WINS and DHCP are both dynamic. In many companies, DHCP is being implemented to reduce the administrative overhead of manually assigning IP addresses.

WINS provides excellent support for DHCP, but the problem remains of how you can integrate the WINS address list and the DNS address list to provide a seamless address space for UNIX and Windows clients. Solutions to this challenge are discussed in the next section.

## WINS/DNS Integration

The primary challenge of WINS and DNS integration is to provide seamless integration between UNIX and Windows hosts and clients. Integrating Windows machines with DNS is trivial since Windows supports DNS address lookups. However, providing UNIX machines access to WINS registered Windows machines is a non-trivial and complex task.

There are two primary difficulties. First, since Microsoft networking uses a flat name space and DNS uses a hierarchical name space, the flat Microsoft networking names must be mapped into the DNS hierarchy. The Microsoft networking client configuration allows you to specify into which subdomain the machine should be mapped.

Second, the NetBIOS name lookup method must also support reverse name lookup, which uses the machine's IP address to lookup the machine's name. WINS, the NT 4.0 DNS, and Unix based DNS all allow proper reverse name lookup, however the NT 3.5 DNS does not properly support this function.

Reverse name lookup, also known as reverse address resolution, is frequently used for access control. When the client machine contacts the host, the host frequently must verify that the client machine is authorized for the transaction it is attempting. For example, rlogin, printing, and firewall applications frequently use reverse address resolution. The host uses the IP address of the client machine and queries DNS or WINS to locate the registered name for that machine. Once again, when using WINS to verify the machine name, that name can just be mapped into a DNS hierarchical name.

The Windows IP configuration allows you to specify which DNS servers to use for address resolution and reverse address resolution. The IP configuration also supports a separate DNS machine name and DNS domain name. If you are using DHCP, then the DNS domain name may be specified in the DHCP scope configuration.

In Windows NT, run Control Panel, Network. Select the Protocols tab, select TCP/IP protocol, and press Properties. Select the DNS tab and enter the appropriate Host Name, Domain, and DNS Search order.

Figure 13.7 Setting NT TCP/IP properties

In Windows 95, run Control Panel, Network. Select the TCP/IP protocol and press the Properties button. Select the DNS Configuration tab and enter the host name, domain name, and DNS servers.

Figure 13.8 Setting Windows 95 TCP/IP properties

To set a default domain name for DHCP scope, run the DHCP Manager, select the scope you wish to modify, and select DHCP Options, Scope. Add or modify Option 015 Domain Name.

When the Windows machine queries the DNS server for a machine's IP address, it will use the configured DNS domain name and limit the lookup to that specific domain. If desired, Windows may also be configured to use the WINS server for DNS address resolution. If WINS is used, the lookup may not be limited to the configured domain name, because WINS does not support the DNS hierarchy.

Typically, Windows machines will be configured with both WINS and DNS address resolution, using WINS for Windows hosts and DNS for UNIX hosts. WINS and DNS would provide address resolution and reverse address resolution for their respective hosts. When configuring a Windows machine for DNS address resolution, the DNS machine name specified does not have to be the same as the NetBIOS machine name.

To allow UNIX hosts access to WINS registered machines, Microsoft provides the Microsoft DNS Server. The DNS Server in the NT 3.51 Resource Kit is intended for use with Windows NT 3.51, while NT 4.0 ships with a DNS Server. Both DNS Server products enable you to define standard DNS name-to-address resolutions, as well as specifying that a WINS server should be used for address resolution. While the NT 4.0 DNS Server supports reverse address resolution, the NT 3.51 Resource Kit DNS Server does not.

The DNS configuration files specify for which DNS subdomain this DNS server is authoritative. When a host queries the Microsoft DNS server with a flat server name, the DNS server assumes the server name is located in the subdomain for which it is authoritative.

The simplest integration method is to place all statically addressed Windows machines in the DNS for the appropriate subdomain, which placing all the DHCP addressed machines in a separate subdomain. A Microsoft DNS server would be definitive for the subdomain of DHCP addressed machines and would be configured to use WINS for address resolution.

Unfortunately, this solution may not be practical for several reasons. If DHCP is used extensively in the organization, the DHCP subdomain may be very large, which violates the principle of keeping your DNS domains small. Also, if DHCP is used in several locations, you just deploy local DNS and WINS servers throughout the physical locations to provide address resolution for the extended subdomain.

If DHCP is used extensively, you may wish to divide your network into several zones, with each zone defined as an individual subdomain. Each subdomain would contain a WINS server accepting registrations for that subdomain and a DNS server that is definitive for that domain.

Under this scenario, the WINS servers would not replicate, since this would result in a single, flat name space. As a result, Windows clients configured for WINS resolution would be unable to locate machines registered in other locations, which severely limits your WINS server functionality. However, since the Windows servers usually use static IP addresses, they can be mapped in the DNS server. Workstations that would use the DHCP assigned addresses, usually do not need to support remote connections.

## Summary

In this chapter, we have discussed how to integrate Windows NT with Unix servers. The primary consideration is the integration of Windows flat name space with Unix's hierarchical addressing structure.

While Windows machines with static IP address can be manually entered into a Unix DNS, many organizations are adopting DHCP to allow dynamic IP address assignments. In these organizations, WINS must be implemented to support dynamic address resolution.

Until a dynamic DNS standard is developed and adopted by Microsoft, the only viable method of integrating DHCP addressed machines with an organization's existing DNS architecture is through the use of Microsoft 4.0 DNS using its WINS address lookup features.
