---
title: Chapter 8 Broadcast Browsing
layout: book
---
# Chapter 8: Broadcast (Plain) Browsing

Microsoft network browsing was first developed and introduced with LAN Manager and continues relatively unchanged with Windows 95 and NT 4.0. Browse servers store the workstations' name and network addresses. When a workstation need to communicate with another workstation, it contacts the browse server rather than broadcasting a request to all workstations.

The term broadcast-based browsing applies because all workstations broadcast to register with the browse server and to locate browse servers. Browsing functionality has been enhanced, but it continues to provide backward compatibility for Windows for Workgroups and LAN Manager. Implementation details between different OS versions may cause differences in how browsing behaves. Interactions between different OS versions on the network produce interesting effects and unique problems. Later sections in the chapter will address these issues.

Microsoft networking is built around NetBIOS, a software interface that requires a unique NetBIOS name to be assigned to each computer. Each computer belongs to a workgroup, which also has a unique NetBIOS name. NetBIOS names are limited to 15 characters, with a sixteenth character provided for special NetBIOS workgroup names. This chapter frequently refers to these special NetBIOS names as _<workgroup><hh>_, where the NetBIOS name is "workgroup" and the sixteenth character contains the hexidecimal value "hh". For example, _<domain><1D>_ or just _<1D>_. Microsoft networking does not distinguish between workgroup names and domain names because the concept of domains did not exist when broadcast browsing was defined.

Originally, the only supported network protocol was NetBEUI, which uses the NetBIOS name as the unique network address for each computer. To provide support for Novell networks, IPX was adapted. Because Microsoft networking requires NetBIOS, Microsoft supplied IPX encapsulated NetBIOS, also called NWLINK. Microsoft also supplies TCP/IP encapsulated NetBIOS. NetBEUI is not routable, and the other supported protocols only use local subnet broadcasts. In other words, browsing only works across a single physical subnet.

With the introduction of Windows NT, Microsoft addressed the single physical subnet limitation by introducing the concept of a domain master browser. This single computer gathers information from all the browser services on the WAN, consolidates the list, and updates all the browser servers.

Another primary architectural limitation is the browser service's use of network protocols and workgroup name registrations. In operating systems prior to NT, the browser server and client are combined. This unified service binds to a single network protocol, usually the primary protocol on the first network adapter that initializes. Therefore, each physical subnet may contain several browser service networks, each containing a unique subnet of the machines on that subnet. Specifically, each workgroup and protocol combination will have its own browser servers. These servers will only talk with other servers registering with the same protocol and workgroup combination.

Assume you are running Windows 95 and NT servers. All your Windows 95 workstations are in the CORP workgroup. They all run IP and half have IPX installed to communicate with the Novell servers. The NT servers all run IP and are in a domain called BUSINESS.

Potentially, you would have three different browsing systems on each subnet: CORP IP, CORP IPX, and BUSINESS IP. It's unlikely that any computers would be able to find anyone from another browsing system.

Every computer that uses Microsoft networking includes a browser that provides both client and server components. The browser *client* component provides a mechanism by which a computer using Microsoft Networking announces itself on the network, registers its name, and locates other computers and network resources. The browser *server* component provides the registration database, name resolution services, and a method of sharing this information with other server components. In NT, the client and server are separate components, but under previous operating systems the client and server components are functional parts of the same network component. However, all workstations are both clients and servers.

Browser servers are usually named by the role which they serve: potential browse master, browse master, backup browse master, and domain browse master.

On each subnet, one of the browser services becomes the browser *master* through a process called an election. The master browser maintains the database of registered computers and redistributes the list to the backup browse masters. The browser client contacts the master browser when the workstation registers its name. When the browse client needs to locate a workstation, it contacts a backup browser rather than the master browser. A potential browse master is not currently a browse server, but can change its role if either the browse master requests it become a backup browse master or if an election is called.

The basic browsing processes are easily understood and easily implemented on a network. The difficulties arise in that all computers on the network participate in browsing. Furthermore, browsing depends on all computers on the network operating correctly, particularly the server components. Unfortunately, on a large WAN with thousands of computers, this likelihood of optimum operation is small.

This chapter focuses on the pure broadcast-based browsing environment. First, you will learn about the browser client transactions, followed by coverage of the browser service traffic. Because elections involve all computers on the network, we will discuss elections separately from the client and server components. Finally, we will describe how to tune your computers and network to optimize browsing.

## 8.1 Normal Browser Client Transactions

The browser client component provides a mechanism by which a computer using Microsoft Networking announces itself on the network, registers its name, and locates other computers and network resources. Although LAN Manager uses browsing, this chapter focuses primarily on Windows for Workgroups, Windows 95, and Windows NT.

When Window for Workgroups was released, it supported only local subnet browsing. NT's introduction defined the method for browsing WANs. The NT release CD includes WFW network clients that support this functionality. NT browsing has been enhanced several times. By the time Windows 95 was release, Microsoft had developed WINS as a preferred workstation location method. Windows 95 updates contain bug fixes for browsing problems, but no major features or enhancements.

All browser client versions support the same basic feature set, which allows the workstation to register its presence on the network and to locate other computers. These simple tasks require that the browser client be able to locate the backup browse masters and retrieve its list of computers. The following sections will describe the network traffic associated with these actions. When remotely connecting to the network, workstations perform these routine actions slightly differently; this is covered in section 8.1.5.

### 8.1.1 Client Host Announcements

Every computer running Microsoft Networking is a potential browse master. Every computer is a client. When the networking is initialized, the computer loads its name and workgroup name, initializes the network adapters, binds the protocols to the adapters, and starts the network client. The browse client broadcasts its machine name and workgroup name on the network. Generally, this broadcast uses the first network adapter and the first bound protocol on that adapter.

Figure 8.1 Normal Registration

Now that the network code is initialized, the computer broadcasts a request for a browse master for its workgroup or domain. The Announcement Request is broadcast to _[1D]_. When the master browser detects the Announcement Request, it responds with a Local Master Announcement, which asks the clients to declare an election if required. The destination is _[1E]_ and the packet contains the Local Master Announcement, the announcement interval, the local master browser's name, and the type of server (domain controller, NT, potential browser, and master browser).

If there is a browse master present, it responds to the workstation with its name. If no browse master responds, an election will be called.

The computer then queries the browse master for the names of the backup browsers. If more then three backup browsers are returned, the workstation will randomly select three backup browsers.

At this point, the computer has fully initialized its NIC and network code. Everyone on the subnet knows the computer is there. The browse master knows the computer's unique computer name. The computer has a list of backup browsers that it may use to find computers on other subnets.

### 8.1.2 Locating a Backup Browse Master

The next operation to examine is how your computer finds another computer on the network. When your computer registers with a browse master, it receives the address of several backup browsers, from which it selects three at random. Your computer uses these backup browsers to locate network resources.

Because the network is dynamic, your backup browser may have shut off or been demoted by the master browser. If your cached backup browsers do not respond, your browser client must locate another backup browser.

The browser client broadcasts a _GetBackupListRequest_ to the _[workgroup] [1D]_ record. The subnet master browser returns a list of backup browsers. If the workstation is a member of a domain that does not have a master browser on the subnet, the browser client unicasts the _GetBackupListRequest_ to the primary domain controller, which returns a list of available backup browsers The browser client picks three browse servers from the list presented and caches them for future use.

Figure 8.2 Locating backup browsers

### 8.1.3 Retrieving the Browse List

All name retrieval is performed by querying the browser servers on your subnet. Therefore, all browsing is dependent upon having local subnet browser servers for your workgroup, as well as on other workgroups that contain machines you want to contact.

When the browser client wants to contact a remote machine, it contacts one backup browser randomly selected from the cached list of three. The workstation sends a _NetServerEnum_ API to the backup browser, which returns a list of NetBIOS names. In addition to the list of workstations in the workgroup, the backup browser returns a list of all other workgroups of which it knows.

If the backup browser does not respond, the workstation unicasts to the master browser to obtain the browse list. If the computer which the browser client trying to locate is in the workgroup, the browser client should now have its address. The workstation can be contacted directly.

If neither the backup browser nor the master browser responds to the browser client request, the browser client calls for an election (the election process is covered in section 8.3, "Elections").

Figure 8.3 Retrieving the browse list

If the browse client is trying to obtain the address of a machine in a different workgroup, the process is very similar. For sake of convention, different workgroups will be referred to as remote workgroups.

The browse client broadcasts a request for the _[remote workgroup][1D]_ name. A backup browser for the remote workgroup returns the remote workgroup's NetBIOS name list. If a remote workgroup's backup browser does not respond to the broadcast, the browser client contacts the remote workgroup browse master directly to obtain the NetBIOS name list.

There are other ways to obtain browse lists, as well as differences in how some operating systems obtain lists. The following paragraphs cover these other methods of obtaining browse lists.

All the Microsoft operating systems support network functionality from the command prompt using the _NET_ command. The _NET VIEW_ command returns a list of available servers by directly querying the master browser, rather than querying the backup browsers. The list returned will be more accurate because the master browser may not have yet updated the backup browser.

In an IP environment, the browser client contacts the browse server using TCP/IP encapsulated NetBIOS. Establishing communication between the browser client and browse server involves several steps, as follows:

1. The browser client must first establish a TCP/IP session with the selected browse server, establish a NetBIOS session, negotiate the SMB protocol, establish a null session to IPC$, and request a browse list.
2. Using the NT browser client, the communication is completed via a NetBIOS session and RPC calls.
3. If the client is a WFW or Windows 95 client, the request is completed using the _NetServerEnum_ API call, which is sent at both the physical and packet layers

Using WFW, the NetBIOS name list is split into two packets. The first packet contains the workgroup and domain list. The second packet contains the listing of the local subnet machines in your workgroup. Windows NT and Windows 95 queries result in a single unified list.

Using Windows 95, after the browser client obtains the address for the workstation, it will also query the workstation directly to obtain a list of that workstation's shared resources. WFW and NT will not automatically retrieve the workstation's shared resources list, but they have the capability to retrieve this information. WFW use the _NetShareEnum_ API call; Windows 95 and Windows NT use RPC calls.

### 8.1.4 Shutting Down the Workstation

If your workstation's operating system is properly shut down, then the browser client notifies the browse master that the workstation is going offline. The browse master will notify the backup browsers that the browse list has changed, and the backup browsers will request the updated NetBIOS name list immediately. The master browser will also update the domain master browser's list during the next scheduled synchronization cycle.

Typically, the updated workstation list may take 15 to 30 minutes or longer to fully propagate throughout the network. Not only must the information be updated on the domain master browser, but also it must in turn be updated on the remote subnet's master browsers, which must then update their backup browsers. During this time, computers on remote subnets may be unaware that your workstation has shut down.

If you do not properly shutdown your operating system, the master server does not know that you are no longer available. The master browser must figure out that you are no longer available.

Normally, a computer will announce itself to the master browser every 12 minutes. If the computer misses 3 announcement cycles, the browse master will remove the computer from its list.

If you are a portable user who moves between subnets, remote users may find it difficult to reach your workstation because the browser servers must propagate the old registration being deleted and the new registration being added.

## 8.2 Browsing Across a Remote Connection

When remotely accessing a Microsoft network, you would establish a point to point protocol (PPP) dial-up connection using either a terminal server or an NT server running Microsoft's Remote Access Server. The two access methods are radically different in functionality and how browsing works across the remote link.

Note that this section discusses the case in which a single computer sustains a single remote network connection. The workstation may be running multiple protocols across this connection. Typically, either analog or ISDN dial-up is used, but the connection could also be via a serial cable, X.25, or a dedicated communications line. The common characteristic is that the remote computer initiates this connection using Microsoft's dial-up networking client.

The Microsoft dial-up networking client supports NetBEUI, NWLINK, and IP. The IP connectivity supports either a fixed IP address or Dynamic Host Configuration Protocol (DHCP) assigned addresses.

If the remote computer is using a connectivity device that requires that the remote computer uses a network interface, then that remote computer should be considered to be on a local subnet. Browsing will function as described in the section 8.1.

### 8.2.1 Remote Connectivity via Terminal Servers

When establishing remote connectivity using a terminal server, after you have dialed in and established your PPP connection, you have a physical cable connection to the network. Your workstation operates just as though it was locally attached to the subnet on which the terminal server resides. Note that you must configure the terminal server so that all dial-in ports reside on the same physical subnet as the terminal server, rather than using virtual LANs.

Figure 8.4 Remote connectivity via terminal server

When using a terminal server, browsing functions as described in section 8.1. You may experience problems with network latency. If these problems are severe and the browser client does not receive replies from the browser servers on the subnet, it may assume the browser servers are dead and call for an election.

Because the computer sees itself as actually residing on the local subnet, it may vote in elections and, potentially, become a browser master or backup browser. If the remote connection speed is too slow to respond to browse list requests, other browse clients on the subnet may call for elections. Remote workstations should be prioritized to not act as browse servers.

In Windows for Workgroups, you modify the SYSTEM.INI file to add the SlowLanas parameter to the [Network] section. The format is as follows:

```
[Network]
SlowLanas=LANA[,LANA …]
```

Where LANA refers to the LANA numbers located in the PROTOCOL.INI file. In Windows 95, the SlowLanas entry has moved to the Registry.

### 8.2.2 Remote Connectivity via RAS

You can also establish remote connectivity using Microsoft's Remote Access Service (RAS) , which operates more like a proxy server than a terminal server. The RAS server receives the packet across the remote connection, uses the RAS server's address resolution methods to locate the target computer, and establishes a session with the target computer on behalf of the remote computer. The RAS server will use any installed address resolution methods to locate the target server, including browsing, DNS, WINS, or LMHOSTS files. The RAS server may use different network protocols for the dial-in session from your computer and for the network session to the target computer.

Assume, for example, that you have a remote computer with NetBEUI protocol only. Your LAN computer's has only IP. Your RAS server has both NetBEUI and IP installed. Your remote workstation establishes its connection using NetBEUI. When the remote workstation requests a browse list, the RAS service contacts the local subnet's browse servers for the browse list, as described above, and sends that browse list to the remote workstation.

Figure 8.5 Remote connectivity via RAS server

If your remote machine cannot locate computers on the network, you are experiencing browsing problems between the RAS server and the network. The remote computer only requests connections from the RAS server; it does not participate in either browsing or registration.

For this same reason, computers on the local network cannot attach to computers that have dialed in because the dialed in computers have not registered with a browse master.

## 8.3 Normal Browser Server Transactions

Every workstation contains a browse server. When the workstation initializes, the browse server announces itself on the network and determines which role its browse server will perform. This process is called an election. While elections are technically part of the server traffic, elections will be discussed in greater detail in section 8.3.

Depending on which browse server role the workstation assumes, it may accept registrations, synchronize with the domain browse master, or synchronize the backup domain controllers. Section 8.2 discusses the network transactions produced browse servers, which types of servers generate them, and when they occur.

### 8.3.1 Server Traffic Host Announcements

As discussed in section 8.1.1, each computer announces itself on the network when initializing its network code. WFW and Windows 95 clients use an integrated browser client and server, which NT has as a separate client and server.

If the computer has a server component, it will initiate a browser announcement to its local browser master indicating that it can receive client requests. The destination for the browser announcement is _[workgroup][1D]_. This announcement indicates to the browse master that the server is available to be browsed. The packet contains the Host Announcement command, an announcement interval, the announcing computer's name, and the server type (WFW, Win95, NT workstation, NT server).

Windows 95 has a server component installed if you have selected the "share files and printers" checkbox on the network configuration button.

The computer may initiate multiple Host Announcement frames specifying the computer as a browser workstation, browse server, domain controller, or potential browse server. The computer will rebroadcast its announcements every 12 minutes.

### 8.3.2 Browse Master Initialization

When a new browse master is elected, it does not have a browse list. The browse master initiates two broadcasts to force all workstations on the local subnet to announce themselves.

The first Announcement Request broadcast destination is _[workgroup][00]_. All browse clients using this workgroup or domain name initiate a host announcement, which registers the workstation with the browse master.

The second Announcement Request broadcast destination is _<01><02>_MSBROWSE<02><01>_. A broadcast to this special NetBIOS name forces all browse masters on the subnet to register their _[1D]_ name with the newly initialized browse master. Browse masters use this method to located browse master for other workgroups which are operating on this subnet. Remember that not all workgroups may have browse masters on all subnets.

The browse master then registers with the domain master browser, which retrieves the complete browse list. In return, the domain master browser issues a request to the browse master to retrieve that browse master's list.


Figure 8.6 Browse Master initialization

The browse master repeats the _<01><02>_MSBROWSE_<02><01>_ broadcast every 15 minute intervals to maintain its list of other workgroups. Notice that a newly elected browse master may take up to 15 minutes to complete its initialization tasks, during which time the backup browsers may not have a complete browse list. Therefore, you should avoid rebooting master browsers or taking other steps that may initialize an election.

### 8.3.3 Browse Master Replication

The browse client registers its computer name with the browse master, but the client uses the backup browse masters to locate other computers. Before the client can locate another computer on the network, the browse master must replicate the computer's registration to the backup browse masters. This process is called replication.

If you have a single subnet, browser server replication is simple. When the browse list changes, the master browse server sends an update notification to the backup browse servers notifying them that an updated list is available. The backup browsers are responsible for requesting a copy of the updated list.

If you have WAN wide browsing enabled, replication becomes difficult. Because browsing broadcasts are limited to the local subnet, Microsoft introduced the concept of the domain master browser, which gathers all the NetBIOS name lists from the master browsers, merges the lists, and redistributes the merged list to the master browsers.

The domain master browser is always the primary domain controller for the domain, and the Primary Domain Controller is always a domain master browser If you have workstations that do not use domains, they can only access subnet browsing. To enable them for WAN browsing, you must change their workgroup name to be the same as the name of the domain. Note that this is only possible in LAN Manager, WFW, and Windows 95. NT does not allow a workgroup and a domain with the same name to coexist.

If you do not have a primary domain controller available, you cannot browse outside your subnet unless you are using WINS.

Every 15 minutes, the master browser contacts the domain master browser, transmits its NetBIOS name list, and receives the master list from the domain master browser. The browse master sends only the names of its local subnet workstations, not all the workstations of which it knows.

The domain master browser continually updates this master list as it receives updates from the master browsers. The master domain browser also contains machines it knows about from other sources. The domain master browser may find out about improperly registered machines through normal domain authentication network traffic. For example, the network may have a subnet with only Windows for Workgroups machines. Because WFW master browsers cannot replicate their list to the domain master browser, the domain master browser will not have the list of any machines on this network. When one of these WFW machines logs into the domain, however, the domain controller will learn the workstation's name and add it to its browse list.

Before computers on other subnets can contact your computer, your registration information must be propagated from your browse master to the other browse masters and backup browsers on the WAN. The domain master browser must merge your master browser's information into its master browse list, then wait for the other master browsers to contact it during their 15 minute synchronization cycle.

Once the browse master gets the updated list from the domain master browser, the browse master will update all its backup browsers. Remember that the workstations use the backup browsers to locate computers. Until the backup browser on the remote subnet has your computer's registration information, your computer cannot be located.

Due to the length of this process, your computer may not be accessible to everyone for 15 to 30 minutes. If your workstation frequently reboots, computers may be looking for you at your old addresses until the browser lists propagate throughout your network.

Replication problems can arise from machines having identical names across a network, browse masters not properly gathering the lists, overload conditions on the browse masters or domain master browser, or any number of other conditions. Most browsing problems occur because browse lists are not properly propagated across the network. Chapter 11 will discuss replication troubleshooting and corrective procedures.

## 8.4 Maintaining Browse Lists

Every 12 minutes, the browse servers announce their presence to the subnet with a **<domain>[1D]NetBIOS** name. The master browser monitors the network for these broadcasts and, when one is received, the master browser adds the server to the browse list.

At the same interval, the master browser broadcasts the special name **<01><02>MSBROWSE<02><01>**, which causes all browse masters from other workgroups or domains to register with the browse master. This is how browse masters discover other domains and workgroups.

### 8.4.1 Elections

Most computers have the capability to be a browser server. An election is the process by which the computers determine which of them will become the master browser. Elections are automatically initiated by any browser client when that client feels an election should be called. All browse clients, which means all workstations, act independently when deciding if they should call an election and when they call the election. All workstations running a browser server component participate in the election. All workstations monitor the election and honor the outcome.

Because the browsing process depends on individual computers monitoring the browse masters and backup browsers, calling elections when errors occur, and implementing the election results, the election process must be robust and self-correcting.

Elections are initiated by a couple of different methods, which consist of the following:

1. A network client makes a request to the browse master and the browse master does not respond in a timely manner. Perhaps the browse master has not yet been elected, but more likely the browse master has either crashed or is overloaded, which prevents it from responding in a timely manner. In either case, the browser client calls an election.

2. If a backup browser does not communicate with the browse master or does not receive a browse list for a while, the backup browser will initiate a broadcast for the master browser. If the master browser does not respond, the backup browser calls for an election.

3. NT servers call for an election when initialized. If the primary domain controller boots or if a new primary domain controller is promoted, an election will be called. Elections determine the most capable machine to act as master browser. Because an NT machine is more robust then either WFW or Windows 95, it initiates an election.

### 8.4.2 The Election Process

When the computer calls for an election, it will also enclose its election bid, which establishes the high bid. In order of precedence, the election bid is based on the following criteria:

1. The computer's OS/domain role
2. Browser/OS specific criteria
3. Workstation name
4. Current time alive

Using these criteria, the computer prepares a 4 byte word which is its election bid. Specific information on how this bid is determined is described in section 8.3.2. The bids are compared as integers, with the highest integer winning the election. The theory is that the higher the election bid, the more likely the chance the computer will make a good browse master.

Election packets are broadcast using the **domain[1E]** NetBIOS name. The packet contains the Election command, the election criteria, the system up time, and the local host name.

When the other computers receive the election request, they will compute their own bid. If a computer's bid is higher than the current bid, then that computer will transmit its own bid. That computer then becomes the potential browse master.

The process continues until a computer submits an unchallenged bid. To ensure that nobody has a higher bid, the winning computer may call up to an additional 3 elections. Once the election is complete, the new browse master will initiate three separate actions (covered in section 8.2):

1. Broadcast its presence to the subnet.
2. Issue a request for all subnet computers to register with itself.
3. Contact the domain master browser to get a complete browse list.

After it has been elected, the master browser must announce itself. The master browser sends a broadcast to **<01><02>MSBROWSER<02><01>**, which announces the master browser to other domains and workgroups. The packet contains the Workgroup Announcement command, the announcement interval, the workgroup or domain name, and a flag indicating whether this registration is for a workgroup or domain.

The master browser repeats these broadcasts every 15 minutes. If a domain or workgroup has not been announced for 3 announcement periods, it will be removed from the browse list.

Immediately after an election, the browse master has not received the registrations of all local machines or synchronized with the domain master browser. The browse master broadcasts a request for all workstations on the local subnet to announce themselves, then contacts the domain master browser to announce itself. The domain master browser then initiates a replication to the master browser, which must then update the backup browse masters. Until this replication cycle completes, the local subnet workstations may have problems locating machines outside the subnet. Full synchronization may not occur for 15 to 30 minutes.

To reduce network traffic during the election process, each operating system has a built-in delay before it submits its election bid. Because older operating systems are less likely to win an election, their delay is longer in the hope they quickly will receive a bid higher than theirs.

The built-in delays for election bids are as follows:

* Windows for Workgroups: Several hundred ms
* Existing master browsers and Primary Domain Controllers (PDCs): 100 ms
* Existing backup browsers and Backup Domain Controllers (BDCs): 200 to 600 ms
* Others: 800 to 3000 ms

### 8.4.3 Election Criteria

The election bid consists of a 4-byte packet containing the operating system type [1 byte], election version [2 bytes], and operating specific criteria [1 byte]. Each workstation calculates its own election bid. The workstation can submit a bid that does not reflect the true configuration of the machine. For example, Samba is preconfigured to vote as an NT version 5.0 server.

The following hexidecimal mask shows the different fields which make up the bid and in which position they are located:

* Operating System Type: 0xFF000000
* Election Version: 0x00FFFF00
* OS specific: 0x000000FF

### 8.4.4 Operating System Type

The highest order byte represents the operating system used. Because NT computers can support a larger number of workstation registrations, it has a higher priority then either Windows 95 or Windows for Workgroups. The byte order is as follows:

* Windows NT Server	0x20000000
* Windows NT Workstation	0x10000000
* Windows for Workgroups	0x01000000
* Windows 95	0x01000000

### 8.4.5 Election Version

Every operating system upgrade potentially upgrades the browse server component also. The election version is not related to the operating system or the version of that operating system. The election version indicates which version of the browser election protocol is used.

### 8.4.6 Operating System Specific Information

The lower order byte represents this workstation's ability to server as browse server. In addition the computer's role, such as PDC or WINS server the computer may be prioritize through registry or INI file entries. The byte order is as follows:

* PDC	0x00000080
* WINS System	0x00000020
* Preferred Master	0x00000008
* Running Master	0x00000004
* MaintainServerList = Yes	0x00000002
* Running backup browser	0x00000001

If two computers have identical election bids, then the server that has a greater uptime wins, followed by the lexically lowest system name as a tiebreaker.

In the example, we have a subnet with an NT domain controller, an NT workstation, several Windows 95 workstations, and a few Windows for Workgroup workstations.

The NT domain controller is an NT server, so it will win the election. If it were unavailable, the NT workstation would have the highest value operating system and would therefore win.

But what if neither of these NT servers were running.  The Windows 95 and WFW machines have the same value for the operating system version. The election specific criteria can not be determined from examining a computer, but if all computers are running the latest service pack or patch.

Because the Windows 95 machine knows how to contact the domain master browser and the WFW machines does not, you hope the Windows 95 machine wins the election. To guarantee the Windows 95 machine wins, you must set the WFW machine options to **MaintainServerList=No**.

### 8.4.7 Problems with the Election Process

The election process should guarantee the most capable machine is elected as the browse master. If the elected machine cannot fulfill its duties, a new election should be held to locate a more capable machine. But what if there isn't a more capable machine on the network?

Windows for Workgroups was never designed to provide network browsing services. It will not act as browse master for more than 20 to 40 computers. Windows 95 will support about 100 to 200 workstations on a subnet. NT will support thousands of computers per subnet.

If you have a subnet with only Windows for Workgroups machines, you are stuck with a Windows for Workgroups browse master. If you have 50 computers, the browse master will quickly become overloaded and unresponsive. Inevitably, one of the workstations will call an election.

The winning Windows for Workgroups machine will also overload and yet another election is called. On a heavily loaded subnet with no capable machines, you will experience continuous elections. Your browse master may never stay up long enough to synchronize with the domain master browser and obtain a complete network browse list. On large WANs, just retrieving the browse list may overload the browser master and cause it to crash.

Elections cause performance and stability problems on computers, not so much from the election network traffic. Rather, elections interfere with other computer operations. While the election is taking place, the operating system is monitoring the election traffic to see if it should vote.

Even if the computer has already been outvoted, it must watch the network traffic until the election is resolved. Particularly on operating systems that are single threaded, this process may slow or cripple the operating system.

If the PDC goes down, the domain has lost its domain master browser. The master browsers will notice when they try to announce to the domain master browser and fail. After 3 announcement cycles, the master browsers will lose all entries except their local subnet machines. Even if a new machine is promoted or the PDC restored, the new domain master browser takes at least 15 minutes to obtain updated browse lists from the master browsers, merge the lists, and return the master list to the browse masters.

### 8.4.8 Browser Wars

Browser wars occur when a misbehaving workstation continuously calls elections. Usually the workstation is experiencing network problems such that it cannot resolve network names or browse the network, yet it still has network functionality. When the workstation cannot initiate a session with its backup browsers or browse master, it forces an election. Because the problem is with the workstation rather than the browse master, the workstation will not continuously force elections to which the other workstations must respond.

Browser wars are common on networks with Windows for Workgroups workstations. Although the simplest way to resolve the problems is to configure your workstations so they will not act as browse masters, your underlying network design and workstation configuration problems will remain unresolved.

Browser wars are also common when the browse master is overloaded or unstable. Assume your browse master is an NT server that has a loose network cable. The server occasionally loses network connectivity. While the server is disconnected, a workstation requests a browse list and receives no response. The workstation will call an election and a new browse master will be elected.

When the NT server returns online, it will call for a new election. During the course of this election, the NT server may once again lose connectivity. In addition to the network traffic and workstation delays associated with an election, your computers take the performance hit associated with gathering the workstation information, replicating to the domain master browser, and replicating to the backup browsers.

## 8.5 Tuning the Browsing Network

In tuning the browsing network, your main priorities are improving the stability of the browse servers and reducing the overall network traffic. The browser servers generate a lot of traffic when they synchronize, but the primary performance problems result from server elections and the subsequent server synchronization. The five general methods for reducing browsing traffic are as follows:

* Identify those computers you wish to use as browse servers
* Reduce the network subnet workgroups and protocols used
* Reduce the number of browser entries
* Increase the amount of time between browser updates
* Eliminate unnecessary browser elections

If allowed to run its course, browser elections will eventually select the most capable machines to act as browse servers. These elections may disrupt your network for some time while your workstations slug it out. And every time you reboot systems, you will start new elections. In addition to the network traffic generated, elections slow down and may lock up your workstations.

### 8.5.1 Identifying Computers Viable as Browse Servers

For each subnet on your network, identify the most capable machines to serve as browse master and also identify machines to server as backup browsers. These should be NT machines, or at least Windows 95 machines. The selected machines should remain on continuously. You will require one browse master and one backup browser for every 16 WFW machines, 32 Windows 95 machines, or 32 NT machines. Disable this functionality on all machines that you do not want to act as browse servers.

On WFW workstation, the browse server and browse client are a single component. You may disable the browse server by editing the SYSTEM.INI file. In the [Network] section, add or modify the following line:

```
[Network]
MaintainServerList = No
```

The browser service can also be completely disabled by removing VBROWSE.386 DLL from the workstation.  You can either unselect the "sharing Enabled" checkbox in the Network Control Panel or you may directly edit the SYSTEM.INI file and remove "VBROWSE.386" from the Network= line.

Windows 95 stores its browser configuration settings in the registry. Although you can directly modify the registry, it is recommended you go through the appropriate graphical interface to modify system settings.

As with WFW, the easiest method to disable the browse server is to disable file and printer sharing.  From Control Panel, Network, click the File and Print Sharing button and unselect both boxes.

If you wish to use the Windows 95 machine as a browse master, it does supply the equivalent of the MaintainServerList entry. File or Print sharing must be enabled.  In Control Panel, Network, you will see a list of the network components installed. Select File and Print Sharing from the list and click on the Properties button.  From the Properties list, select BrowseMaster and select No.

On NT servers and workstations you can remove the machine from the browse list by simply stopping the NT browser service.

Windows for Workgroups browse masters will support only 10 to 30 workstations, while even Windows 95 supports only 100 to 200. Once these limits are exceeded, the server becomes unresponsive and the browse clients will call for an election. WFW machines should NEVER be used as browse servers. In Windows NT, the browser service provides much higher performance, but prior to NT 4.0 the browser database is limited to a single 64K segment.

### 8.5.2 Reducing Subnet Workgroups and Protocols

You can also tune your browsing network by reducing the number of workgroups and protocols on each subnet. If you have a subnet of two workgroups with two protocols loaded on each workstation, then you have four browse networks operating on that subnet: one for each workgroup and protocol combination. Using a single network protocol may cut the number of browse servers in half.

If possible, avoid using the IPX protocol for browsing. A browse server configured with IPX uses SAP broadcasts for all server announcements and discovery. In addition to producing large amounts of network traffic, any Novell servers on the network will respond to these SAP broadcasts by announcing themselves using SAP packets. The resulting SAP storms may degrade network performance.

Any large IPX network is a tradeoff of router performance maintaining the IPX routing tables vs. bridging all packets. But increasing the size of your subnets increases both the loads on your existing browse servers and the volume of SAP broadcasts on the subnet. In large routed IPX networks, router performance may degrade as routers update and propagate their router lists after a SAP storm.

For the best browsing performance on your network, use the TCP/IP protocol.

### 8.5.3 Reducing Browser Entries

In organizations with large networks and large numbers of workstations, the browse list will contain hundreds or thousands of computers. Not only does replicating these browse entries to each subnet increase network traffic and reduce the browse server's responsiveness, but the users are presented with browse lists of all the machines on the network when they only want to look at a few.

Windows for Workgroups and Windows 95 workstations both have a server component. If you have file or print sharing enabled on these workstations, they will register with the browse masters and their registration information will be propagated to all master browsers. If you disable file and print sharing on these workstations, they will not register. NT machines will always register with the browse masters.

### 8.5.4 Increasing Time Between Browser Updates

You can reduce the browse server synchronization traffic by changing the frequency of that synchronization. While NT's browser service allows extensive customization, WFW and Windows 95 allow none. On an NT server, the **MasterPeriodicity** and **BackupPeriodicity** registry entries configure the browser service announcement and replication times.

While it may be tempting to just disable all browsing on the local subnet, you will just force your clients to directly contact the domain master browser to locate other workstations. While this does centralize your overhead in a central location, your PDC may not be able to accept the additional overhead.

### 8.5.5 Eliminating Unnecessary Browser Elections

Of course, you can also do performance tuning by eliminating unnecessary browser elections. When an election is called, it requires all computers to participate, even if they are not actively sending their election packets, they are waiting for the result. Once the election is complete, the workstations must all register and this information must be synchronized to all the servers.

To reduce elections, stabilize your browse master so that it is always available to your browser clients. Make sure your browse master is appropriately sized for the number of machines you must support.

Monitor your event logs for machines calling elections. If you get several on the same subnet, profile the browse masters to discover why. Look at the machine that keeps calling the elections to determine why. Perhaps it has networking problems.  Troubleshooting browse server problems are discussed further in Chapter 11.

### 8.5.6 Browsing in WFW Environments

Because of performance and architectural limitations, WFW machines should never be used as browse servers unless absolutely necessary. You should disable the browser service on these machines.

The browser service can also be completely disabled by removing **VBROWSE.386 DLL** from the workstation. You can either unselect the "sharing Enabled" checkbox in the Network Control Panel or you may directly edit the SYSTEM.INI file and remove "**VBROWSE.386**" from the **Network=** line as follows:

```
[386#Enhanced]
Network =
```

When an election occurs, the workstation enters a waiting state which it awaits the election outcome. Even though the criteria for a WFW workstation are too low to vote in or win an election, the workstation will be waiting until the election is complete. During this time, a WFW workstation will be locked and unresponsive.

WFW uses three primary configuration files: **PROTOCOL.INI**, **SYSTEM.INI**, and **WIN.INI**. Each file is broken into sections, which begins with a header line in the format:

```
[SectionName]
followed by several parameters in the format:
ParameterName=Value
```

and contains one line for each parameter.

The **SYSTEM.INI** file contains the following parameters:

```
[Network]
MaintainServerList= [Yes|No|Auto]
```

* **Yes** means the workstation will vote in elections as a potential browser.
* **No** means the workstation will not vote in elections at all.
* **Auto** means the workstation will act as a browse master or backup browser if required.

These values affect not only the bid submitted during elections, but also whether or not the workstation will submit a bid at all.

If you have a known slow network connection, such as a dial-up network connection, you do not want the computer acting as a browse master for the remote network. The browse server may be set to not participate in election on the slow network link. The SlowLanas option allows you to specify the LANA numbers that should be ignored. The PROTOCOL.INI file contains the network protocols in use and the LANA numbers which refer to those protocols:

```
[Network]
SlowLanas=LANA[,LANA …]
```

Also remember that WFW machines cannot browse a WAN unless they have a domain master browser. If you are using WFW machines, change their workgroup name to the name of an existing domain. The WFW machines will now use the browsing system associated with that domain rather than running a separate system of its own.

### 8.5.7 Browsing in Windows 95 Environments

Windows 95 stores its browser configuration settings in the Registry. Although you can directly modify the Registry, it is recommended you go through the appropriate graphical interface to modify system settings.

As with WFW, the easiest method to disable the browser service is to disable file and printer sharing. From the Network option in the Control Panel, click the File and Print Sharing button and unselect both boxes.

If you wish to use the Windows 95 machine as a browse master, it does supply the equivalent of the **MaintainServerList** entry. File or Print sharing must be enabled. In the Network option of the Control Panel, you will see a list of the network components installed. Select File and Print Sharing from the list and click on the Properties button. From the Properties list, select BrowseMaster. In the value box, you can select Yes, No, or Automatic

Selecting the Yes option indicates that the workstation will vote in elections as a potential browser. Selecting the No option indicates that the workstation will not vote in elections at all. Selecting the Auto option indicates that the workstation will act as a browse master or backup browser if required. The selection of the Yes, No, and Auto options affects not only the bid submitted during elections, but also whether or not the workstation will submit a bid at all.

If you are running Windows 95 under IPX, you have a number of configuration options available. These options are discussed in the Chapter 10, "Other Browsing Environments."

### 8.5.8 Browsing in Windows NT Environments

In Windows NT, the browse server is implemented as a service, with extensive performance tuning available via the Registry. Some Registry entries may not appear by default and must be created. Microsoft changes Registry entries in Service Packs; read the Service Pack README file for changes.

The browser service is divided into two sections, which are described as follows:

* **User mode.** This mode is part of the LAN Manager server service. This portion is responsible for maintaining the browse list, remoting the **NetServerEnum** API, and managing the computer's role as a browser.
* **Kernal mode.** This mode is the Datagram receiver and receives and manages datagrams.

Although the NT browse service performs the same fundamental functions as Win95 or WFW as a browse master, the underlying architecture is completely different. In addition to responding to the normal NetBIOS calls, NT server's browse service will spawn RPCs between servers to perform routine functions.

Also, the NT browse service requires user authentication. If your NT browse masters are members of a domain, they will only talk to other NT servers who are part of the domain. If you are using Win95 or WFW as a master browser, and you have disabled the Guest account on your NT browse masters, the current logged in user must be an authenticated domain user.

Similarly, if you wish to browse a remote domain, either they must have Guest account enabled or your domain must trust their domain.

## 8.6 NT Registry Entries

The Computer Browser service provides extensive tuning capability though the Registry. The applicable registry keys are located in 3 locations:

```
HKEY_LOCAL_MACHINE\ SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Browser\Parameters
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\DGRcvr\Parameters
```

### 8.6.1 HKEY_LOCAL_MACHINE\ SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters

```
Lmannounce
Type: REG_DWORD
Possible values: 0,1
```

If you have LAN Manager 2.x clients on your network, you must set this value to 1 force NT to use LAN Manager compatible server announcements. Otherwise, leave the default setting of 0.

### 8.6.2 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Browser\Parameters

```
BackupPeriodicity
Type: REG_DWORD
Possible values:(300 to x418937)
```

This value specifies, in seconds, how frequently the backup browse master should contact the browse master. The default value is 720 seconds (12 minutes). Changes will not take effect until you reboot the computer.

```
CacheHitLimit
Type: REG_DWORD
Possible values:(0 to 256)**
```

When the browse client wishes to retrieve the browse list, it sends a NetServerEnum request to the backup browser. This value specifies how many times the client should send a NetServerEnum query before it caches the response. The default value is 1, which means all addresses are cached as they are received Operation is also affected by the QueryDriverFrequency registry parameter, which determines how often the cache is cleared.

```
CacheResponseSize
Type: REG_DWORD
Possible values: (0 to xffffffff)
```

This value specifies how many names should be cached at one time. The default value is 10. Each network transport has its own separate cache. To disable name caching, set the value to 0.

```
IsDomainMaster
Type: REG_SZ
Possible values: True or False
```

Setting this value to True selects this server to be a Preferred Master Browse, which increase its election criteria and its chances of becoming a browse master.

```
MaintainServerList
Type: REG_SZ
Possible Values: Yes, No, Auto
```

This value determines whether or not this computer can become a browse server. The default value is Auto.

Yes means the computer will act as a browse server. When it boots, it will register with the browse master. The computer will participate in elections and, depending on the election outcome, may become a browse master or backup browse server.

No means the computer will register only as a browse client, not as a browse server or potential browse server. It will not participate in elections.

Auto means the computer will register as a potential browse server. When it registers with the browse master, the browse master will tell this computer is it should register as a browse server or browse client.

```
MasterPeriodicity
Type: REG_DWORD
Potential values: (300 to x418937)
```

This value specifies how frequently a master browser contacts the domain master browser for browse list updates. The default is 720 seconds [12 minutes], with a minimum value of 300 seconds [5 minutes] and a maximum of 0x418937 seconds [about 47 days]. The browse service monitors this key and any changes are immediately implemented.

```
QueryDriverFrequency
Type: REG_DWORD
Potential values(0 to 900)
```

The browse master maintains an internal name cache that is use to respond to NetServerEnum request. This value indicates, in seconds, how frequently the browse master invalidates this special cache and queries the browser driver for a current browse list.

The default value is 30 seconds. Reducing this time results in a more current browse list for your clients, but at the expense of increased processor utilization, which may result in a less responsive browse master.

### 8.6.3 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\DGRcvr\Parameters

```
BrowserServerDeletionThreshold
Type: REG_DWORD
Potential values:(0 to 0xffffffff)
```

If more than this number of servers are flushed from the cache in a 30 second period, an event will be generated. The default value is 0xffffffff.

```
BrowserDomainDeletionThreshold
Type: REG_DWORD
Potential values:(0 to 0xffffffff)
```

If more than this number of servers are flushed from the cache in a 30 second period, an event will be generated. The default value is 0xffffffff.

```
FindMasterTimeout
Type: REG_DWORD
Potential value:(0 to 0xfffffff)
```

This value specifies, in seconds, the maximum time the FindMaster requests should take. The default value for this entry is 0xffffffff

```
GetBrowserListThreshold
Type: REG_DWORD
Potential values: any whole number
```

This value represents the threshold that the Browser uses before logging an error indicating that too many of these requests have been "missed." If more requests than the value of **GetBrowserServerList** are missed in an hour, the Browser logs an event indicating that this has happened. The default value of this entry is 0xffffffff (That is, never log events.)

```
MailslotDatagramThreshold
Type: REG_DWORD
Potential values: an whole number
```

This value represents the threshold that the Browser uses before logging an error indicating that too many of these requests have been "missed." If more mailslots than the value of **MailslotDatagramThreshold** are missed in an hour, the Browser logs an event indicating that this has happened. The default value for this entry is 0xffffffff (That is, never log events.)

## 8.7 Summary

This chapter examined the browser components, their functions, and how to stabilize browse server performance.

The browser client registers the workstation's name with the browse master and uses the backup browse masters to locate other computers. The client cannot be tuned and, as long as it can find a browse server, it functions reliably.

The browse server accepts the client's name registrations and replicates this information to the other browse servers. These replications depend on a browse server hierarchy of backup browse masters, browse masters, and a domain master browser. Elections are the process by which backup browse masters and browse masters are determined.

The browse server election and replication processes are complicated and full of potential problems. Tuning is performed primarily through selecting reliable machines and prioritizing them to win elections.

Although WINS provides a superior browsing solution, broadcast browsing is always running in the background. Understanding the interactions between WINS and broadcast browsing is critical to troubleshooting and resolving browsing problems.

Broadcast browsing runs on almost every Microsoft network, but it is not optimal for most networks.
