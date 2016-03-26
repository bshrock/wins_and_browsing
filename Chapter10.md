---
title: Chapter 10 Browsing in a Novell Environment
layout: book
---

# Chapter 10: Browsing in a Novell Environment

Browsing in a Novell network presents several problems, primarily relating to Novell's use of proprietary network protocols: Internetwork Packet Exchange (IPX) and Server Announcement Protocol (SAP).

Novell developed IPX and SAP specifically for use with their network operating system.  As Novell's acceptance increased and their servers began appearing on corporate WANS, Novell added performance enhancements to provide better routing, however the basic architecture of these protocols limits its effectiveness.

Much like NetBEUI, IPX was originally intended for use on a single subnet, and no provisions were made for addressing workstations outside that subnet. Both NetBEUI and IPX use an arbitrary and unique name to identify the workstation. NetBEUI uses an administrator assigned name, while the IPX name is usually hard coded onto the network adapter by the adapter's manufacturer.

Large networks are usually split into logical subnets to reduce network traffic. In addition to a unique workstation address, IPX requires a subnet address that identifies on which subnet the computer is located. Like the workstation address, the IPX subnet address is an arbitrary value. Unlike the workstation address, the administrator must assign the IPX subnet address. All workstations on a subnet have the same subnet address.

Figure 1: IPX network

Let's take a second to compare this to the TCPIP addressing. Using IP, each workstation has a unique numerical network address. The first part of the address identifies the computer's subnet and the second part identifies the computer on that subnet. IP's addressing scheme allows must easier communication between subnets than IPX's addressing.

Figure 2: TCPIP network

To enable communications between subnets, routers are used to connect the subnets together. A router listens to the traffic on all of subnets to which it is connected and learns the addresses of those computers. When the router hears a network packet which it knows it on another subnet, it will rebroadcast that packet on the target machine's subnet.

To operate properly, a router must maintain an internal table entry for each workstation containing each workstation's address and subnet address. If the network contains more than one router, the routers must be able to exchange information to build a routing table, which tells the routers how to get to each subnet on the network.

Figure 3: IPX network routing table

NetBEUI can not be routed, because the workstation address does not contain a network address number. IP can be easily routed, because the first part of each workstation address contains the network number. The router knows all addresses starting with the same number belong on the same subnet. IPX is more difficult to route, because the workstation address does not include the subnet information. The router must store the table of all workstations on the network.

Figure 4: TCPIP network

As the network increases in size, the routers require more processor time to maintain the IPX routing table and synchronize that table with other routers, which leaves less processor time for routing. To overcome this problem, many IPX networks use bridging, where the routers simply rebroadcast all network traffic to all subnets. Essentially, you have one large network, which introduces both network bandwidth and router capacity problems.

Novell and Microsoft networking share a common problem: how do computers find out about other computers.  On both network operating systems, the servers announce themselves when they boot. Novell servers announce themselves using SAP.  All Novell servers listen for SAP announcements and store the information locally for address resolution. This contrasts with Microsoft, in which the browse servers store this information and provide centralized address resolution services.

To provide compatibility with Novell, Microsoft workstations configured with IPX also use SAP for browsing.  However, this causes several problems for both Novell and Microsoft servers.

Since most Microsoft networking workstations also contain a server component, they will use SAP to announce themselves as Novell servers. The routers will repeats these SAP announcements to all subnets, resulting in high network overhead. Since Microsoft workstations regularly rebroadcast announcement packets, this traffic is routine and ongoing.  When a Microsoft workstation calls for an election, all Novell servers will respond by reannouncing through SAP. These announcements may cause a SAP storm, which are similar to Election Wars.

While Novell was originally built around IPX, Microsoft was originally built around NetBEUI. Internally, Microsoft uses the NetBIOS interface, which is so closely tied to NetBEUI that most people confuse the two.

When Microsoft networking uses the IPX protocol, it usually uses a variant called NWLink, which is NetBIOS calls encapsulated within IPX protocol frames. When you install IPX on a NT workstation, you are installing IPX and NWLink, which increases the complexity of the network stack.

Since Novell does not recognize or support NWLink, Microsoft introduced NWBLink, which provides full Microsoft networking functionality using standard Novell network compatible protocols.  This protocol is also referred to as Direct Mode.

Computers running Direct Mode can only communicate with other computers running Direct Mode or with Novell servers. Microsoft supports Direct Mode on WFW, Windows 95, and Windows NT. They do not support Direct Mode on LAN Manager machines, which only supports NWLink.

## 10.1 Configuring Direct Hosting on WFW

WFW has two IPX protocols you may install: IPX/SPX Compatible Transport (NWLink) and IPX/SPX Compatible Transport with NetBIOS (NWBLink). To enable Direct Hosting on Windows for Workgroups, install the "IPX/SPX compatible transport" rather than the "IPX/SPX Compatible Transport with NetBIOS".

To install this protocol, open Program Manager, open the Main program group, and run Windows Setup. Select the Options menu and select Change Network Settings, which opens the Network Setup window. Press the Drivers button to open the Network Drivers window. Press the Add Protocol button to show the available protocols. From the menu, select the IPX/SPX Compatible Transport and press the OK button.

Also, you must make the following modification to your **System.ini** file:

```
[Network]
DirectHost=yes
```

To disable Direct Hosting on WFW, perform the reverse action: select the "IPX/SPX Compatible Transport with NetBIOS" protocol and set **DirectHost=no** in your **System.ini** file.

It is important to realize that while the WFW Direct Host drivers can communicate with any NT machine which has the IPX protocol configured, it can only communicate with Windows 95 or WFW workstations if those computers have the Direct Host drivers loaded.

Also, the Direct Host drivers do now allow connectivity with Novell file servers. You must continue to use the Novell supplied network drivers if you require Novell connectivity.

## 10.2 Configuring Direct Hosting on Win95

Windows 95 Direct Hosting uses the standard IPX network protocol.  To install IPX, press the Start button, Settings, and Control Panel. From the Control Panel, run Network. You will see the Network Configuration tab, which shows the installed network clients and protocols.  Select IPX/SPX-compatible Protocol and press the Properties button.

The NetBIOS tab shows a single checkbox: I want to enable NetBIOS over IPX/SPX. Check the box for Direct Hosting or leave it unchecked or normal NetBIOS over IPX operation.

Once you have installed Direct Hosting, may selected either SAP Advertising or Workgroup advertising. Workgroup Advertising uses the same broadcast method used by workgroups on Microsoft networks. If you select Workgroup Advertising, you must have at least one browse master per subnet. SAP advertising uses SAP announcements.

When using Direct Mode, you need to choose either Workgroup Advertising or SAP Advertising as the browsing method that the computers will use..

To specify your browsing preference, you must have File and Printer Sharing for NetWare Networks installed. From the Control Panel Network applet, chose File and Print sharing for NetWare Networks, and press the Properties button. The Advanced tab shows your options for SAP Advertising and Workgroup advertising.

When using Workgroup Advertising, the configuration menu has the following options:

* **Disabled:** The computer will not be added to the browse list and can not be seen by any client using any browsing method.
* **Enabled:** May be master. The computer will be added to the browse list. It may be promoted to browse master if a preferred browse master is not available. The equivalent of **MaintainServerList= AUTO**.
* **Enabled:** Preferred master. This computer is the preferred master browser, but it will still lose the election if a computer with a higher bid votes. The equivalent of **MaintainServerList= YES**.
* **Enabled:** Will not be master. This computer will be added to the browse list, but it cannot be promoted. The equivalent of **MaintainServerList= No**.

The other option when configuring Direct Hosting in a Windows 95 environment is SAP Advertising, which is used by Novell NetWare 2.15 and above, 3.x, and 4.x servers to announce themselves on the network.

SAP advertising has a theoretical limit of 7000 systems and a practical limitation of about 1500 systems. When using SAP Advertising, browser elections, workstation announcements, and browse master announcements will also generate responses from your Novell servers. When implementing SAP Advertising, remember that you will be generating additional SAP broadcasts which may subsequently generate SAP storms. Also, since all Windows 95 machines using SAP Advertising will appear on your Novell server list, your Novell servers must maintain a larger server list.

From the SAP configuration menu, the following options are available:

* **Disabled:** This computer will not announce and NETX or VLM clients can not see the server or connect to it. Win95 clients using Client for NetWare can see and connect if Workgroup Advertising is enabled on the peer server.
* **Enabled:** This computer will announce itself and it will appear on the Entire Network list. NETX, VLM, and Win95 Client for Network clients can see it using any browsing method.

If you want your workstation using the Novell network clients to attach to your Window 95 workstation's resources, you should use SAP advertising. Although you will be able to see the Workgroup Advertising computers that will appear on your Novell SLIST, Novell workstation client functionality when connecting to these workstations will be severely limited.

Obviously you cannot log on to Windows 95 workstations using either the NETX or VLM clients, because Windows 95 does not supply authentication services. The VLM client, however, does allow you to run the **LOGIN /NS** command from the Windows 95 command prompt. You can map and access Windows 95 shares using either Novell client using the normal Novell MAP commands.

## 10.3 Browsing Across Remote and slow links

With the advent of cheap, high-speed dial-up network communications, many companies are implementing dial-on-demand, WAN links. These links create problems for network browsing and domain authentication.

By default, the browse master will contact the domain controller every 12 minutes. Even assuming the link is set to disconnect on no traffic, the modem may dial almost continuously, particularly if there are frequent elections on the subnet. Domain authentication traffic generates the same type of usage pattern.

You can change the frequency at which the master browser attempts to connect with the domain master browser by changing the **MasterPeriodicity** key, which affects how frequently the master browser will contact the PDC. A value of 86400 seconds tells the master browser to contact the domain master browser once a day.

Determining the optimal value for this key depends on your specific network configuration. On most networks, you want the servers to show on the master browse list, but the workstations are less important. Since servers are usually left running and they usually have fixed addresses, they will show on the browse list almost constantly. Any time the browse list is replicated, it will contain accurate information for most servers.

If workstations are properly shut down, they unregister from their browse master and are removed from the list. If workstations are frequently rebooted during the day, they may not appear on the browse list when it is replicated to remote subnets.

Remember also that the PDC uses the browse list to locate the BDCs for account synchronization. If the PDC can not synchronize BDC for an extended period, that BDC will become untrusted.

If your primary concern is that users be able to reach the file servers, a replication frequency of 2 [7200 seconds] hours should be sufficient. If a server does not appear on the browse list, you can correct the problem and expect the corrected list to replicated within 4 [14400 seconds] hours.

If user's share information from their workstations, you will want a faster replication time to ensure that all workstations are available. A MasterPeriodicity of 1 hour [3600 seconds] should be sufficient.

The MasterPeriodicity key is located in the registry in the HKEY_LOCAL_MACHINE\SYSTEM\CurrectControlSet\Services\Browser\Parameters tree. It is a REG_DWORD with a value, in seconds, between 300 and x418937. The default value is 12 minutes. If you change this value, you must reboot the machine for it to take effect.

Administrators may also consider stopping the browser service after hours. To disable the browsing service on an NT server, type **NET STOP BROWSER**. This command can be scheduled using the **NT AT** command.

Stopping the browser service may have severe implications for your network. The browse server will not provide address resolution for browse clients. If you have your workstations configured for broadcast address resolution, they will broadcast to locate workstations and servers are required. Similarly, any time a user opens the Network Neighborhood, their computer will broadcast to locate other computers.

Broadcasts are usually confined to a single subnet. If your remote network consists of more than one subnet, you may be unable to locate computers outside your subnet.

When the browser service is restarted, if the MasterPeriodicity time has expired, the server will immediate attempt to retrieve a new browse list.

Remember: if a workstation can not contact a browse master, it will call an election. All workstation on which a browser service is running will vote in that election. If you do not disable the browser service on all machines, a new browse master will be elected, which will then try to retrieve the browse list. When you restart the shut down browser service, you will force another election.

Administrators may also want to disable the subnet browsing entirely on the remote subnet. If the subnet is small, and the browsing traffic would be minimal, administrators can disable service entirely. This requires disabling the service all machines on the subnet. Your clients will then user broadcast address resolution and will be unable to connect to resources on other subnets unless you use either WINS or LMHOSTS files.

## 10.4 LMHOSTS Files

LMHOSTS files enable you to manually and statically map machine names to IP addresses for NetBIOS name resolution. Using an LMHOSTS file, you can define the workstation's most frequently accessed servers, which might include browse masters, domain controllers, and commonly used file servers. By prioritizing the workstation using the LMHOSTS file first, you can eliminate the network browsing traffic to locate these commonly accessed servers.

Although the preferred solution to IP browsing would be WINS implementation or broadcast-based browsing, you may need to use LMHOSTS files to provide browsing functionality for some of your subnets. LMHOSTS files should be implemented on subnets where browse servers can not communication regularly with the domain master browser. If you have multiple browsing domains but no not have browse masters for all domains on all subnets, you may also wish to implement LMHOSTS files.


> Please note that support for LMHOSTS domain browsing has not been formally documented or tested by Microsoft, and may not be available in future versions of the client or server operating systems. It is suggested that you not implement browsing using exclusively LMHOSTS files.

### 10.4.1 Using LMHOSTS Files

On an NT machine, the LMHOSTS file is stored in **%WINROOT%\SYSTEM32\DRIVERS\ETC**, while in WFW and Windows 95 machines the file is located in the Windows program subdirectory. The LMHOSTS file is not created by default, but the directory contains **LMHOSTS.SAM**, a sample LMHOSTS file. The format for the LMHOSTS file is the same for all versions of Windows.

The LMHOSTS file contains entries mapping IP addresses to machine names, in the following format:

```
10.1.1.1 Computer1<br>
10.1.1.1 Computer2 #PRE #DOM: domain
```

The **#PRE** option tells the computer to preload the entry into memory, rather than scanning the LMHOSTS file each time.

The **#DOM:** entry tells the computer that this entry designates a domain controller. Both **#PRE** and **#DOM:** must be capitalized. If you are using an LMHOSTS file exclusively for browsing, you must list an entry for each domain controller in the domain.

LMHOSTS also supports centrally located LMHOSTS files as shown in the following example:

```
  #INCLUDE C:\WINNT\LOCAL.TXT
    10.1.1.1 Server1 #PRE #DOM: domain
    10.1.1.1 Server2 #PRE #DOM: domain
    #BEGIN ALTERNATE
    #INCLUDE \\SERVER1\SHARED\LMHOSTS
    #INCLUDE \\SERVER2\SHARED\LMHOSTS
    #END ALTERNATE
```

The **#INCLUDE** statement forces the system to parse a remotely maintained LMHOSTS file as if it were local. The locally located #INCLUDE file may be specified using the drive letter and subdirectory, but remotely located LHMOSTS files must be specified using the full UNC path. If the file is on a remote subnet, you must add an entry to the LMHOSTS file that defines this server's address. The entry must use the #PRE option to ensure it is preloaded **#BEGIN_ALTERNATE** groups multiple **#INCLUDE** statements. **#END_ALTERNATE** marks the end of an **#INCLUDE** statement grouping. NT will try to contact the servers listed in the **#BEGIN_ALTERNATE** section to locate the specified LMHOST file. Once any file is successfully processed, the rest of the section is skipped.

When an NT computer configured with LMHOSTS wins an election, the newly elected browse master tries to register with the domain master browser. To locate the Primary Domain Controller (PDC), the new browse master locates every computer in the LMHOSTS file with the **#DOM:** domain flag and queries the computer with the **NetGetDcName** API. Only the PDC will respond. The master browser then contacts the PDC to get the domain browse list. To improve browsing in this scenario, place your PDC near the top of the LMHOSTS file.

Windows 95 and WFW master browsers cannot query with the **NetGetDcName** API and would therefore not support exclusively using LMHOSTS file to locate the PDC. You can support these systems by manually entering a domain registration entry in the LMHOSTS file.

When the PDC registers its workstation name, it also registers a special name as the domain master browser. This special name is the PDC's domain name with a 16th character of hex 1B. To support WFW and Windows 95 machines, you can enter this special name into the LMHOSTS file.. The format of the LMHOSTS entry is:

```
"Computer2 \0x1b" #PRE #DOM:domain
```

where the computer name between the quotes is padded with spaces to a total of 15 characters. The **\0x1b** is the sixteenth character in hex.

### 10.4.2 Precautionary Measures when Browsing with LMHOSTS Files

There are a number of cautions when browsing exclusively using LMHOSTS files.

If you promote a different domain controller to PDC, you would have to change the LMHOSTS **\0x1b** entry for that domain and reboot the servers.

All computers require a LMHOSTS file with entries for all domain controllers and proper **\0x1b** entries for all domains. The PDC requires an entry for each master browser, which implies you have prioritized your subnet workstation to provide a stable subnet master browser.

If your LMHOSTS file becomes extremely long, you may want to consider using a common LMHOSTS file accessed across the network or distributed to all servers automatically. If you use a centralized LMHOSTS file, remember the file must be accessible at all times. If you replicate the file to all servers, remember entries marked **#PRE** are only preloaded at system boot time.

> Seeing a computer in the browse list does not imply you can connect to it. If it is on your local segment, you can connect via broadcast; if it is on a remote segment, you will need an LMHOSTS entry for it.
