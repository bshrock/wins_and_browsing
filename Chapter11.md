---
title: Chapter 11 Troubleshooting Browsing
layout: book
---

# Chapter 11: Troubleshooting Browsing

Browsing is extremely dynamic and, for the most part, self-correcting. When browse servers fail, workstations will elect new browse servers. Re-elections will occur until stable browse servers are elected. As browse servers go down, browse clients locate new browse servers to use.

Because most problems on a browsing network will resolve themselves, administrators usually don't need to worry about troubleshooting. Eventually, the servers and workstation will reach a dynamic equilibrium. Administrators only need to correct situations in which the servers cannot correct themselves.

Network connectivity problems are the primary causes of all browsing problems. These problems are sometimes intermittent, but the type of browsing problems you are having can help you identify the network problems. Even if you can't identify the underlying cause, you can usually correct the symptoms.

Workstation connectivity problems are the next most likely causes of network problems. Browsing depends on everyone working properly, and a problem workstation can wreak havoc on your network. You must identify and correct the problem workstation to resolve these problems.

Browsing problems are not usually pre-existing. Usually, they appear out of the clear blue sky, leveling all network functionality in the path. When troubleshooting these problems, you have to follow basic troubleshooting procedures to identify what has changed:

* When did the problems appear?
* Did the problems appear on more than one workstation?
* What changes occurred between the time this configuration worked and the time it stopped working?
* What network configuration changes occurred?
* Were any new workstations or servers added to the network?
* Has new hardware or software been added to the workstation?
* Is the network connection working?

To aid you in troubleshooting your browsing problems, let's briefly review the broadcast browsing process.

* The workstation boots and broadcasts its name registrations.
* The workstation broadcasts a request to retrieve a list of browse servers that it will use to resolve addresses.
* If a browse server does not respond to the workstation, or if the workstation feels the browse server's response is not quick enough, then the workstation will call an election.
* When an election is called, all potential browse servers respond with their election tokens. The highest token wins the election and becomes the new browse master.
* When a new browse master is elected, it contacts the domain master browser to register and retrieve the domain browse list. Then, it will broadcast a request for all machines on the local subnet to announce themselves.

Sections 11.1 through 11.4 provide an analysis of and solutions to common problems related to the topics highlighted in the Chapter Roadmap at the beginning of the chapter. These topics are as follows: architectural problems, election wars, workstation connectivity problems, and Novell server and IPX network problems.

## 11.1 Architectural Problems

For browsing to function properly, every subnet must have a properly functioning browse master. The subnet browse masters gather the information on the workstations located on that subnet, and pass that information to the domain master browser, which then updates all the subnet master browsers.

Remember that WAN browsing requires a domain controller to gather and maintain the WAN wide browsing list. If you are using workgroups only, then the browse list will only contain the machines on the local subnet. Even if machines from the workgroup reside on several subnets, each subnet's browse list will be separate and machines from one subnet will be unable to locate machines from another subnet.

Figure 11.1 Functional Broadcast Browsing

Each subnet must contain a properly configured and robust browse master. Recall that the browse master must gather the machine registrations for the local subnet, update the domain master browse with that list, and retrieve the domain wide list from the domain master browser. If the subnet's browse master does not perform these functions correctly, several problems can occur.

If the browse master does not accept registrations or does not properly update the domain master browser, machines on the browse master's local subnet will not be able to locate machines outside the local subnet.

Figure 11.2 Client not registering properly with browse master

If the browse master does not properly register with the domain master browser or retrieve the master browse list from the domain master browser, machines on the browse master's local subnet will not be announced properly and other machines will be unable to locate them.

In this section, we'll tell you how to identify and designate reliable browse masters to provide reliable browsing for your entire network. In designating browse masters, you will correct the common problem of machines not appearing on your browse list. However, if your network is large, you will want to restrict the machines that appear on your browse list so the browse list size and browse server replication traffic do not get out of hand.

Most browsing problems share common causes and problem resolutions. This section and the rest of the chapter covers the tools available to identify and resolve browsing problems.

### 11.1.1 Designating Appropriate Workstations as Browse Servers

The most important architectural decision is which machines will be browse servers and where they will be located on your network. A browse server should be a high availability machine with reliable network connectivity. In addition to planning which machines will act as browse servers, you must identify one of these browse servers to function as the browse master for that subnet. In case the designated browse master is unavailable for any reason, including scheduled downtime, you should also identify a browse server that will operate as the browse master's backup. These designated browse servers should not be Windows 95 or Windows for Workgroups machines, as they may cause a variety of architectural problems.

When designating browse servers, the preferred operating systems, in order, are:

* Windows NT server
* Windows NT workstation
* Windows 95

The most important limitation is the number of workstations that can be supported by these operating systems. Windows for Workgroups can typically only support 40 to 50 workstations, although this number varies widely depending on the workstation configuration. Windows 95 can support a maximum of about 100 workstations for each subnet. By comparison, Windows NT Server or Workstation can support several thousand workstations on each subnet.

Windows for Workgroups and Windows 95 workstations also present authentication problems if you're running a domain structure. Recall that WAN browsing only exists if you're in a domain architecture with the domain PDC acting as the domain master browser.

The subnet browse master must establish a connection with the domain master browser to register and to retrieve the browse list. These transactions require authentication with the primary domain controller. When a WFW or Windows 95 subnet browse master establishes a session with the domain master browser, it will first try to authenticate as the current logged in user. If that fails, the subnet browse master will fall back to the domain guest account, which is by default active. If the domain guest account is disabled, the workstation will be unable to connect, update the domain master browser with its subnet browse list, or retrieve the domain browse list. Of course the user may not always be logged into the workstation, in which case there is no security context which the workstation can use to log in to the domain controller.

You must also look at the network protocols that are available for the domain master browser and the subnet browse master to communicate, as well as the protocols the workstations are using to communicate to the subnet browse master. Remember that browsing occurs on all active network protocols, so you must designate browse servers for each network protocol. To make your browse servers more robust, you should limit your subnet browse masters and domain master browser to a single network protocol.

Although browsing occurs on the NetBEUI protocol, NetBEUI is not routable and can not be used for browsing outside the subnet. Whenever possible, NetBEUI should be abandoned in favor of IPX or IP. In large networks, IP is generally the preferred solution.

Workstations with multiple network protocols will attempt to communicate with a browse master on the first network protocol installed. If the workstation's connection to the browse master fails, the workstation will fall back to subsequent network protocols. However if the browse master is slow in responding, the workstation may also switch to its other installed network protocols. The process of consecutively trying different network protocols until a common protocol is finally found may result in connection speed problems, which may result in slow browsing.

If you cannot use an NT Workstation or an NT Server machine on each subnet, then you should implement Windows 95 workstations as browse masters. Be sure to have a sufficient number of browse masters running for the number of workstations that may be on the subnet. If you are using WFW or Windows 95 browse servers, you should have one browse master plus one backup browse master for each 20 workstations. If you are using NT browse servers, a browse master and a backup browse master for each 100 workstations is sufficient. Although NT can provide browsing services for a much larger number of machines, the browse master will identify and designate a backup browser for each 100 machines.

When identifying browse servers, you should also identify machines that should never operate as browse servers. This list should include all of your Windows for Workgroups machines, as well as any workstation which is shut down or rebooted during the working day. Remember that if your browse master is rebooted, an election will be held. Until the new browse master completes the synchronization with the domain master browser, you may experience browsing problems.

You should also specifically exclude any workstation that is heavily used by a single user. Many common office productivity packages can hog the processor, resulting in poor browse server responsiveness. When the browse server does not respond quickly enough, other workstations will call an election.

On WFW workstation, the browse server and browse client are a single component. You may disable the browse server by editing the **SYSTEM.INI** file. In the **[Network]** section, add or modify the following line:

```
[Network]
MaintainServerList = No
```

The browser service can also be completely disabled by removing VBROWSE.386 DLL from the workstation.  You can either unselect the "sharing Enabled" checkbox in the Network Control Panel or you may directly edit the SYSTEM.INI file and remove "VBROWSE.386" from the Network= line.

Windows 95 stores its browser configuration settings in the registry. Although you can directly modify the registry, it is recommended you go through the appropriate graphical interface to modify system settings.

As with WFW, the easiest method to disable the browse server is to disable file and printer sharing.  From Control Panel, Network, click the File and Print Sharing button and unselect both boxes.

If you wish to use the Windows 95 machine as a browse master, it does supply the equivalent of the MaintainServerList entry. File or Print sharing must be enabled.  In Control Panel, Network, you will see a list of the network components installed. Select File and Print Sharing from the list and click on the Properties button.  From the Properties list, select BrowseMaster and select No.

On NT servers and workstations you can remove the machine from the browse list by simply stopping the NT browser service.

### 11.1.2 Remote Subnet Browsing Problems

If you cannot browse a remote subnet, the browse masters on that subnet are probably not properly updating the domain master browser with their browse list. This could occur for several reasons including protocol issues and domain authentication issues.

Most operating systems support browse servers on a single network protocol; typically the first bound network protocol. AN NT Workstation or Server will operate as a browse master on multiple protocols. The WFW and Windows 95 machines will not. In either case, only a single protocol should be installed for browse servers.

Browse servers require proper domain authentication to communicate with the primary domain controller. Either the user login to these machines must always be a valid domain user or the Guest account must be enabled in your NT domain. If you have multiple NT domains, each domain operates as a separate browsing network. The browse lists will not be replicated between domains; however, the subnet browse masters should be able to locate browse masters for the other domains.

For separate domains to communicate properly, they must have a trust relationship between one another. Windows NT uses trust relationships to allows one domain to trust another domain's authentications. For example, when John from the NY domain tries to access the accounting files in the LA domain, the LA domain controller knows how to ask the NY domain controller if John is really who he says he is. The NY domain controller is responsible for authenticating John, and the LA domain controller will trust the NY domain controller's verification. . To improve cross domain browsing, a browse server for each domain should be located on each subnet, however this is not a requirement.

For the trust relationship to function properly, the domain controllers in the two domains must be able to communication. Both sets of domain controllers should share a common network protocol.

If you suspect you are having browsing problems due to protocol related issues, you can use the BROWSEMON utility, which is included in the NT Resource Kit, to verify proper browse server operation. When you run BROWSEMON, the utility will identify the subnet browse master, the protocols which the browse master is using, and for which domain this browse master is functioning. If the subnet contains more than one browse master, as is the case if you have multiple protocols or domains on the subnet, then BROWSEMON will identify all active browse masters on the subnet.

To improve browse server operations, check the network configuration for all browse servers on the subnet to verify that they are using the same network protocol. If a browse server has multiple network protocols installed, prioritize the protocols so that all browse servers have the primary network protocol selected.  To ensure that the subnet browse master can communicate properly with the domain master browser, verify that the domain master browser also has the same primary protocol installed.

Any machine operating as a browse server for multiple network protocols will experience decreased performance since, in general, the Windows networking architecture experiences slower performance with multiple network protocols. It is recommended that a single network protocol be used for the entire WAN.

If the master browser does not respond quickly enough, which is extremely common with WFW and Windows 95 workstations, the workstation negotiates a common protocol between workstation and browse master, which may be time-consuming and result in elections being called.

To select the default network protocol in Window NT, open the Control Panel and select Network. NT 3.5x has a Bindings button on the control panel, while NT 4.0 has a Bindings tab. Within the Bindings screen, you will see each NT service and the protocols installed for that service. After selecting the Server service, select the protocol you wish to use as the default browsing protocol and use the Move Up button to move it to the top of the protocol list.

> When there are workgroups in the network, workgroup clients will not be able to see remote workgroups and members in those workgroups. This is because workgroups are not WAN aware and workgroups separated by routers are considered to be discrete workgroups even if they have the same name. Workgroup clients can, however, browse remote domains if there is at least one member of the remote domain on the local subnet. This domain member announces the presence of that domain on its subnet and thus workgroup clients know about the existenc   of that domain and will get the browse list from that domain member.

In figure 11.4, we have two subnets, each containing a workgroup called Workgroup1. Each workgroup is separate and neither workgroup can located members from the other. However, since subnet1 contains a member of Domain1, the workgroup members on subnet1 can locate members of domain1.

Figure 11.4 Workgroup browsing

If you have a WFW workstation or Windows 95 workstation which doesn't use domain authentication, you must set the workgroup name to the same as the domain name. If you do not, you will have a standalone workgroup with no WAN browsing capabilities.

So set the workgroup name in Windows 95, run Control Panel, select Network, and select the Identification tab. In the Workgroup field, enter the name of the domain that you wish to use for browsing.


In Windows for Workgroups, edit the SYSTEM.INI file. In the Network section, change to Workgroup= line to the name of the domain that you wish to use for browsing.

### 11.1.3 Reducing Computers that Appear on the Browse List

On a large WAN, your browse list may become too large. Browse servers must maintain the domain browse list and replicate this list. WFW and Windows 95 browse servers will be unable to maintain a large list, resulting in slow performance, browser elections, incomplete lists, and other problems. Also, you may wish to limit the list for reasons of security and user convenience.

Only workstations with a browse server component will register with the browse master and will be replicated on the browse list. Only browse servers appear on the browse list. If you do not wish your workstation to appear on the browse list, you should disable the browse server component on your workstation.

On a WFW workstation, you must edit the **SYSTEM.INI** file. In the **[Network]** section, add or modify the following line:

```
[Network]
MaintainServerList = No
```

The browser service can also be completely disabled by removing **VBROWSE.386 DLL** from the workstation. You can either unselect the "sharing Enabled" checkbox in the Network Control Panel or you may directly edit the **SYSTEM.INI** file and remove **"VBROWSE.386"** from the **Network=** line.

Windows 95 stores its browser configuration settings in the Registry. Although you can directly modify the Registry, it is recommended you go through the appropriate graphical interface to modify system settings.

As with WFW, the easiest method to disable the browse server is to disable file and printer sharing. From Control Panel, Network, click the File and Print Sharing button and unselect both boxes.


If you wish to use the Windows 95 machine as a browse master, it does supply the equivalent of the **MaintainServerList** entry. File or Print Sharing must be enabled. In Control Panel, Network, you will see a list of the network components installed. Select File and Print Sharing for Microsoft Networks from the list and click on the Properties button. From the Properties list, select BrowseMaster and select Disabled.

On NT servers and workstations, you can remove the machine from the browse list by simply stopping the NT browser service. To stop the browser service, run the Control Panel and select Services. From the list, select the Computer Browser service and press the Startup button. Under Start up type, select Disabled and press OK.

## 11.2 Election Wars

The most problematic browsing problem results from election wars, in which workstations constantly cause elections. The resulting network traffic and browse list problems can paralyze your network. This section discusses how to identify the causes and correct them.

If a browse server does not respond to the browse client's request, the client will call for an election. Theoretically, the elections will ultimately result in a more responsive and reliable browse master. The entire election process is based on the theory that the browse clients and browse servers will all operate properly and obey the rules. Unfortunately, theory sometimes doesn’t match reality and the election process breaks.

If a workstation does not get a response from the browse master, it will call a new election. The cause could be common workstation connectivity problems, such as too many installed network protocols, multiple adapters, or lost network packets to the browse master.

The same networking problems can occur on the browse server, as well as additional server related problems. If the browse server doesn’t have the capacity to maintain the browse list, it may be unresponsive. If the browse server is rebooted or a new NT workstation boots, an election will also be called.

When browse clients and browse servers experience problems that the election process cannot resolve, continual browse elections occur. This situation is called an election war. Election wars continue until the administrator identifies and resolves the problems that caused it.

First, we will describe how to recognize that you have an election war. The common symptoms, such as incomplete browse lists or slow browse server response, normally occur on networks anyway.

Second, we will describe how to identify what is causing the election war. The common causes are poor network connectivity, inappropriate browse servers, or Samba servers.

Finally, we will describe how to resolve the problems causing the election wars.

### 11.2.1 Recognizing an Election War

Election wars are simply continuous undesirable browser elections. The symptoms are typical of the transitory symptoms experienced when browser servers fail. However, because browser elections correct most of the problems, they are not usually noticed. In an election war, you will see a unique constellation of problems.

When a new subnet browse master is elected, it gathers the names of the machines on the subnet, locates the domain master browser, updates the domain master browser, and retrieves the complete domain browse list. Once the subnet browse master retrieves the browse list, it will update the subnet backup browsers. The browse clients actually use the backup browsers to retrieve the browse list.

If the subnet browse master fails, the backup browse servers still have a complete browse list. Browse clients can continue to look up names until the entries begin to time out, which is usually 15 minutes. Ordinarily a new browse master would be elected during this time. However, during an election war, the browse master is not available long enough to complete its initialization sequence and the entries in the backup browse masters time out. As this happens, the workstation can no longer locate machines outside their subnet.

Users outside your subnet will also experience problems locating machines on your subnet. Since the subnet browse master does not remain active long enough to collect the subnet workstation list and update the domain master browser, local subnet names will start expiring on remote subnet browse masters.

If you have a backup domain controller on your subnet, you may experience domain authentication problems. The primary domain controller will be unable to locate the local backup domain controller and therefore will lose synchronization with the PDC. User password and profile updates will not be replicated to the local BDC. Although the local BDC will properly authenticate a user when they log on, the remote domain controllers will not acknowledge this authentication. The user will appear to log in properly and connect to their resources on the local servers, however they will be unable to connect to resources on remote servers.

The local workstations will become unresponsive or sluggish. Even if they are not viable browse master candidates, the network subsystems will become unresponsive while the workstations wait for the election results, register with the new browse master, and retrieve the backup browse master list. Connections to servers on remote subnets will also appear sluggish.

On large subnets, WFW or Windows 95 workstations that have been elected as browse masters will be unable to properly fill the role. In extreme cases, they may become unstable and reboot. You may notice a sequence in which a workstation locks up, followed a few minutes later by another workstation locking up.

You can run BROWSEMON from the NT Resource Kit, which shows the current browse master and backup browsers. As new browse servers are elected, BROWSEMON will update to reflect this fact.

### 11.2.2 Isolating the Computer Responsible for an Election War

If you don't already have an NT machine on the subnet, you should install one on the subnet. This may resolve the election war, however the event log messages will help you identify the machine that is calling the elections.

If you have an NT machine on the subnet, you can examine its event logs for event ID 8033, which occurs each time that an election is forced. Typically, this event only occurs immediately after the NT machine reboots. To determine when the machine was rebooted, examine the event log for event 6005: EventLog started, which is always the first event logged when the machine is booted. The Browser error message description will show that the browser has forced an election, the network protocol, and the network adapter. If the election is being caused by workstation related problems, the election will always be called by the same workstation. You should disable the browse server component on the problem workstation.

In this screen capture, the browser service has been started on \Device\NetBT_EE161. NetBT refers to NetBIOS over TCPIP, while EE161 identifies the network adapter. Information on the network adapter can be found by examining the registry. Run REGEDT32 and switch to \HKEY_LOCAL_MACHINE\CurrentControlSet\Services. The contains information on the network adapter driver while the EE161 key contains information specific to the first EE16 adapter, such as the interrupt and IO address.

If the event logs do not identify a specific workstation, look for a workstation that continually reboots, locks up, or experiences significant performance problems. Once again, disable the browse server component on this workstation.

If the problem does not appear to be limited to a single workstation, look for a common network protocol on which the elections occur. You can use a network protocol analyzer or Microsoft Network Monitor to look for anomalies with traffic on this protocol, such as excessive traffic from a single network source.

### 11.2.3 Eliminating Election Wars

If you can identify a single workstation causing the continuous election wars, you should disable the browse server component on that workstation. Instructions on how to disable the browser service on different operating system can be found in section 11.1.3.

If the problem is not isolated to a single workstation, you should approach the problem as an architectural browsing problem. Identify and prioritize your browse masters to provide reliable coverage.

## 11.3 Workstation Connectivity Problems

From a user perspective, the most common problem is being unable to locate a computer on the browse list. Windows 95 and NT 4.0 both have the Network Neighborhood, which shows the browse list and allows the user to select a computer from the list. Users reporting that they are unable to locate a computer usually means that the computer doesn't appear on the list.

When a computer boots, it contacts the local browse master to register its name. The browse master then needs to update the domain master browser, which updates the other master browsers, which then update their backup browsers. Although the local browse master will have the registration information immediately after the workstation registers, the browse master may not update the domain master browser for 5 to 15 minutes. Propagating the registration throughout the entire network may take up to 30 minutes. Until a subnet's backup browse master is updated, local workstations will not see the computer on their list.

Even though the computer does not appear on the local browse list, a user should be able to locate the computer by manually entering the computer name using Uniform Naming Conventions [UNC]. For example, you can map a network drive by right clicking on My Computer and selecting Map Network Drive. The Shared Directories window displays the  browse list If the computer you are seeking does not appear on this list, you can manually enter the computer name and share name in the Path line.

If the backup browser does not find the computer on the local browse list, the backup browser will contract the domain master browser to query the name. Depending on the address resolution method chosen, the workstation and browse servers may broadcast to locate the workstation.

If no computers appear on the browse list except the local computer, the workstation probably is not properly connected to the network. Check the network adapter and cabling.

If you cannot locate the computer by typing in the computer name, the registration has probably not replicated from the local browse master to the domain master browser. However several other problems can cause the same problem.

The machine's workgroup name could be different from the domain name. In this case, the registration information would not be replicated to the domain master browser. Instead, the machine would call and election for the workgroup name. You would then have browse servers running for the domain name and the workstation's workgroup name. If you run BROWSEMON, it will show all browse servers running on the local subnet and all the workgroup names for which they are gathering registrations.

Remember also that there are browse servers for each network protocol on the subnet. Each workstation will register on its primary protocol only. BROWSEMON will show the browse servers for each network protocol. Change the binding order on your workstations so they use the same default network protocol. Instructions for changing the protocol binding order can be found in section 11.1.2.

> RAS presents unique browsing problems. Remember that a RAS connection really uses the browse capabilities of the RAS server. When a RAS user retrieves the browse list, they are actually retrieving the RAS server's browse list. If a workstation does not appear on the browse list you should be able to manually enter the workstation name to locate it. When you manually enter the name, the RAS server will use its address resolution methods, which may include broadcast, contacting the local browse master, and contacting the domain master browser.

> Although the computer dialing into the RAS server can locate computers on the local area network, the reverse is not true. The computer dialing into the RAS server does not register with a browse server, therefore workstations on the network will not be cable to locate the remote workstations or connect.


## 11.4 Problems Browsing Novell Servers and IPX Networks

### 11.4.1 NetWare network traffic overview

Browsing IPX networks presents its own unique set of problems. A workstation's IPX network address consists of a workstation address and a network address. The workstation address uniquely identifies the workstation, while the network address uniquely identifies the subnet. On an IP network, the workstation's IP address also contains subnet information. However, the router can simply apply a binary mask to identify the subnet on which the workstation resides. So the router only needs to keep a list of subnets and how to reach them.

To route IPX packets, the router must maintain a table of workstation addresses, the subnets on which those workstations are located, and how to route packets to get to each unique subnet. On each subnet, all computers must use the same IPX network number or network traffic will not be routed properly.

As the network size and number of workstations grows, maintaining the routing table requires increasing amounts of router processor time. Ultimately, the router spends so much time maintaining the workstation and routing tables that the router's performance is adversely affected. To overcome this problem, most large IPX networks use bridging rather than routing. In other words, the routers forward all network traffic to all network segments rather than just forwarding the appropriate packets. The network design trades router processor utilization against network bandwidth utilization.

### 11.4.2 Reducing SAP and RIP announcements

In addition to the overhead from maintaining this routing list, the router generates network traffic as it propagates its routing tables to all other routers on the network. After all, every router must know how to reach every workstation. These tables are communicated using SAP or Routing Information Protocol (RIP) protocols. Netware 4.x may also use NetWare Link Service Protocol [NLSP], which reduces the network overhead. Novell servers use the Service Access Point (SAP) protocol to announce themselves on the network. Browsing also uses SAP.

When using IPX, all election and browse table replication use SAP. Calling an election also results in the Novell servers and the routers announcing themselves, and can also trigger routing table replication. Similarly, rebooting a Novell server produces a SAP announcement from that server, which triggers an election. The effect is similar to an election war that includes the Novell servers and routers. On large IPX networks, these SAP storms can use most of the available network bandwidth.

Even if IPX is not the primary protocol on your network, recall that your workstation's browse client will pause during an election to await the election results. Therefore, Novell server announcements and router table synchronization can adversely affect the performance of your Microsoft networking workstations just as election wars affect your workstations.

Even if your workstations do not participate in the SAP storms, browsing will be affected by the bandwidth problems caused by the SAP storms. The network congestion reduces the response time from the browse servers, causing the workstations to call for elections. The congestion can also cause browse server synchronization problems, resulting in incomplete browse lists, problems locating workstations, and domain synchronization problems.

If you are using RAS and IPX, the remote workstations are considered to be on a separate subnet. If the RAS client's IPX network number is not configured, the RAS server will use RIP to identify an unused IPX network number, which will subsequently be used for the remote workstations. The process of locating an unused network number and subsequently responding to RIP traffic for that address may significantly degrade the performance of the RAS server and remote workstation.

To work around this problem, you need to configure your workstation's networking carefully. If you do not need connectivity to Novell servers, do not load IPX protocols on your workstations. If you must use IPX, install IP as your primary network protocol. Use IP as the only protocol on your domain controllers, which will ultimately result in forcing your browse server synchronization traffic onto the IP protocol.

Recall that only workstations with a browse server component register with the browse master and participate in elections. Plan which workstations you wish as browse servers and disable the browse server component on all other workstations.

On large IPX networks, consider abandoning broadcast browsing and implement WINS instead. You can then disable the browse server component on all Microsoft networking components. Not only will they not announce themselves, but also they will not respond to SAP broadcasts from routers or Novell servers.

### 11.4.3 IPX Frame selection

When configuring your NT workstations, you should also consider which IPX frame type you select. By default, NT uses autoselect, which monitors all selected IPX frame types.

With NT 3.51 prior to SP5, the IPX network drivers took care of the internal IPX routing between the different frame types. In the configuration screen, there is an entry for the internal IPX network number that is used for this functionality; however, the IPX network number is not used. With NT 3.51 SP 5 and NT 4.0, the NT routing is used to internally route IPX frame types. The IPX network number in this configuration is used if you have selected IPX autoselect. If this number is not unique on your network, it will result in SAP storms and routing errors.

To overcome these potential problems, you should only select the IPX frame types you use, standardize on a single IPX frame type if possible, and use a unique IPX network number for each NT machine configured with IPX.

To select the IPX frame type, open the Control Panel and select Networks. Select the Protocol tab, Nwlink IPX/SPX protocol, and press the Properties button. This window shows the computer's Internal Network Number, which is used by NT to internally route IPX packets, as well as the IPX frame types selected. If the Auto Frame Type Detection is selected, NT will receive all IPX frame types. If Manual Frame Type Detection is used, NT will only use the selected frame types. Frame types can be added by pressing the Add button.

When you are using BROWSEMON, you will notice that the **\Device\NWLnkIpx** transport always has an unknown Master Browser. This is not usually a cause for concern. Recall that although browsing usually uses IPX encapsulated NetBIOS, or NWLink, you can also configure it to use Direct Hosting, which uses IPX directly for browsing.

When you use IPX, BROWSEMON may show two network interfaces: **\Device\NWLnkIpx [Direct Hosting on IPX]** and **\Device\ NWLnkNb [NWLink]**.Any server running BROWSEMON and Direct Hosting will always display an unknown browse master for Direct Hosting enabled NT browse masters. The converse is also true. In other words, workstations running only Direct Hosting will be unable to retrieve a browse list unless it has at least one other protocol configured which supports browsing. Also, browse servers running that protocol must exist on its local subnet.

## 11.5 Summary

This chapter has discussed the importance of proper browse server architecture in preventing common browsing problems. For each subnet, robust and reliable browse servers should be identified and configured.

The primary benefit of reliable dedicated browse servers is to eliminate Election Wars, which result when workstations repeatedly call elections for new browse servers. The workstations call elections when the existing browse servers do not respond to address resolution requests in a timely manner.

A solid browse server architecture should eliminate most browsing problems, after which remaining problems with individual workstations can be identified and corrected. Most individual workstation browsing problems can be traced to improper protocol selection and configuration.

Novell server connectivity presents special problems because of the complications of IPX protocols and routing. Understanding how to configure IPX protocols, select frame types, and set the internal network number will help eliminate IPX routing problems and reduce network traffic associated with router updates.
