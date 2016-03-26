---
title: Chapter 9 WINS Environments files
layout: book
---

# Chapter 9: WINS Environments

As we've seen, broadcast-based browsing has a number of problems. These problems are primarily associated with the cooperative nature of browsing. Both servers and clients are expected to obey and honor the same set of rules when announcing, registering, and electing browse masters.

WINS overcomes these problems by providing an NT-based NetBIOS name registry database service. This service may run on any NT server, allowing the server to be sized appropriately for the environment. WINS also uses the JET database engine which allows higher capacity than the existing browser services and better scalability for larger environments. In WINS, replication partners also eliminate the performance issues associated with browser server replication and elections.

Technically speaking, WINS is not browsing at all; however, WINS serves the same function as browsing and it is important to understand how browsing and WINS perform the same basic registration and lookup functions. In this chapter, we will discuss how WINS performs normal browsing functions.

Implementing WINS does not eliminate broadcast browsing. If even a single computer on the network is using broadcast-based browsing, then that computer will cause elections. For a true WINS environment, all computers on the network must have browsing disabled.

As we've seen, broadcast browsing provides a simple, out of the box name resolution solution. For many companies, it provides all the functionality they need. Most problems will eventually correct themselves, and if the network is small enough, nobody will notice.

WINS does provide a technically superior solution, however. Since the WINS servers are designated and known, the network traffic and workstation slowdowns associated with broadcast browsing is eliminated. WINS servers are easily monitored and performance tuned, resulting in consistent, reliable address resolution service. Organizations with large WANs are almost required to implement WINS to ensure proper domain synchronization and user authentication.

Most networks fall in the middle. WINS offers a number of advantages even in smaller networks. Development sites often want separate NT domains for development, testing, and internal information systems. While broadcast browsing depends on domain authentication for database replication, WINS does not. A single WINS server could be used across the entire organization. An Internet available WINS server can provide a virtual browsing environment for a widely dispersed workforce.

On the other hand, if you need to keep organizational units separate, you can implement separate WINS servers for each department. Separate, non-replicating WINS servers help keep browse lists down to manageable sizes.

Consider the example of EgoSoft, a software development business with locations in two cities. The software development is predominantly in LA with marketing and sales in NY. Except for a few executives, the NY people have computers. The development people have laptops and frequently telecommute and travel.The NY users could comfortably work with broadcast browsing. When they turn on their computers in the morning, elections are held, the new browse masters notify the PDC, and within the hour the complete browse list has been distributed to all browse masters. Since the workstation addresses are fairly static, all workstation will be able to locate the servers and workstation on this network. If the browse masters have problems, the workstations will call for new elections.

The LA users will probably experience a lot of network connection problems.  Since they frequently reboot their computers during program testing, elections are called almost continuously, which means the browse masters will rarely have a complete workstation and server list. Browsing problems may prevent the domain controllers from properly locating the PDC and replicating.

Also, the LA telecommuters will be able to attach to and use network servers, but other network users will be unable to attach to and use the resources on the remote workstations. This is because the remote workstations are not registering with the browse masters.

Since the developers in LA frequently add and remove workstations for testing, they do not want users inadvertently connecting to their test machines. The IS department also doesn't want these servers showing up on the corporate browse list.

Figure 9.1 shows a diagram of this network. The production network has WINS servers in both LA and NY. All local workstation register with their local WINS server. Since the two WINS servers replicate with each other, all production workstations appear on the corporate browse list. The test environment has its own WINS server that does not replicate with the production WINS servers. The test machines which register with the test WINS server are invisible to the production machines.

Figure 9.1 Corporate WINS with a private  WINS server

## 9.1 WINS Client Traffic

WINS uses central registration servers which are known to all workstations. Although the browse clients no longer must broadcast to locate browse servers, the client must still register with the server.

##9.1.1 Name Registration

Throughout this section, we will assume the workstation booting up is a member of a domain. WINS is a pure IP service and does not depend on user authentication. WINS will serve any workstation that knows its address. Logging into a domain, however, will register additional NetBIOS names.

As seen in figure 9.2, WINS' name registration process is much simpler than with broadcast browsing. Instead of locating a browse master, registering, and having that registration replicated, the workstation simply contacts the WINS server directly.

Figure 9.2 Name registration with WINS

During the workstation configuration, you may specify a primary and secondary WINS server. If the primary WINS server is unavailable, the workstation will switch to the secondary WINS server. Every WINS configuration should have at least two WINS servers since real time replication provides your only operational backup. If the WINS server is not on the local subnet, you must specific a default gateway that will allow connectivity to the WINS server. If you are using DHCP, the DHCP server may dynamically provide the WINS server and gateway addresses.

When the workstation boots up, it will first register its unique workstation name with a WINS server. Because the workstation knows the address of the WINS server, it does not have to broadcast to locate the local browse master.

When the workstation boots up, it will send a Name Registration Request directly to the WINS server. How the request is handled by the WINS server depends on whether or not the workstation name is already registered, with which WINS server the name is registered, and the status of the registration record.

If the name is not yet registered, the WINS server creates a registration record with a version ID unique to that WINS server, a time stamp of the current time plus the Renewal Interval, and the WINS server ID. The WINS server then sends a Positive Name Registration Response to the workstation.

If the name is registered and the IP address is the same, the WINS server determines if the registration record belongs to that server. If the WINS server owns the record, the IP address is the same, and the record is active, the WINS server updates the record time stamp and returns a Positive Name Registration Response. Otherwise, if the WINS server owns the record, it creates a new registration record and returns a positive response.

If the name is registered with a different IP address, the WINS server sends the workstation a Wait for Acknowledgement Response, then issues a Name Query Request to the address of the currently registered workstation. If no response is received, the WINS server will rebroadcast the request two more times at 500 ms intervals. If the currently registered workstation exists, it sends a Positive Name Query Response to the WINS server. The WINS server returns a Negative Name Registration Response to the workstation seeking registration.

If the WINS server does not receive a reply to its Name Query, it will return the workstation a Positive Name Registration Response.

If the WINS server does not own the record, but the IP address is the same, the WINS server will create a new registration record for the workstation.

The workstation registers the **workstation<00>** name, which uniquely identifies this workstation. The WINS server returns an acknowledgement that the workstation may have this registration. The workstation then registers the **workstation<03>** name, which allows the messenger service to locate the computer. If connecting via RAS, the workstation also registers the **workstation<21>**.

If the computer is a member of the domain, some of its services may try to locate a domain controller to authenticate their service accounts and begin operations. When the user logs in interactively, the logon service also contacts a domain controller to authenticate the user.

The services may register additional workstation names with WINS, such as:

```
\\computer_name [00h]</b> Workstation Service
\\computer_name [03h]</b> Messenger service
\\computer_name [06h]</b> Remote Access Service
\\computer_name [1Fh]</b> NetDDE service
\\computer_name [20h]</b> Server service
\\computer_name [21h]</b> RAS client
\\computer_name [Beh]</b> Network Monitoring Agent
\\computer_name [BFh]</b> SMS Network monitoring utility
\\username[03h] </b>currently logged on user
\\domain_name[1Bh]</b> PDC
\\domain_name[1Dh]</b> Master browser
\\domain_name[00h] </b>Workstation service
\\domain_name[1Ch]</b> domain controller
\\domain_name[1Eh]</b> used for browser elections
```

When the user logs on to the domain, the workstation queries the WINS server for **domain<1C>**, which returns a list of up to 25 domain controls. One of the domain controllers will be the primary domain control (PDC) and the others will be backup domain controllers (BDCs).

The workstation decides which domain controller it will use, then contacts the WINS server. It queries the **domain controller<00>** name to obtain the domain controller's address. The method by which the workstation contacts the domain controller and logs on varies depending on the client operating system.

Once a workstation has logged onto the domain, it registers the **username<03>** NetBIOS name, which is used by the Messenger service to locate the user. This name is not unique; it will be registered for every workstation on which the user logs on.

If the workstation is also participating in broadcast-based browsing, it will query the WINS server for browse server addresses and may register additional NetBIOS names. This information is covered in the next chapter.

### 9.1.2 Name Renewal

When the workstation registers with the WINS server, its record is assigned a time to live. The workstation has all responsibility for renewing its registration within the time to live period. Ordinarily, the workstation will automatically try to renew with its assigned WINS server halfway through the time to live period. A name renewal follows exactly the same procedure as a new name registration.

If the workstation does not renew, the record is marked as extinct. Extinct records may remain in the database for some time before they are purged, so even workstations that are experiencing temporary or long term connectivity problems may still be found.

In contrast, in broadcast browsing a workstation must announce itself every 12 minutes. Every time the workstation reboots and reregisters, this information must be fully replicated throughout the system.

### 9.1.3 Name Release

When the workstation shuts down gracefully, it will contact the WINS server and release its registered names. The WINS server marks the record as released and updates the time stamp with the current time plus the extinction interval. If the WINS database is not the owner of the registration, it will make itself the registration owner.

The WINS service on NT 4.0 handles the release slightly differently. The record is marked released with the time stamp of the current time, plus extinction time out, plus extinction interval. This avoids problems where the initial registration was not with the current WINS server, otherwise the release may not be properly replicated to the original WINS server.

A name registration is also released if the workstation does not renew within the time to live interval. In this case, the WINS server marks the record as released during its normal database maintenance procedures.

### 9.1.4 Name Query

In a broadcast browsing environment, the workstation must occasionally broadcast for a list of browse servers which the workstation will use for address resolution.

In WINS, queries are sent directly to the primary and secondary WINS servers. NT 3.5 and the older WFW clients query the primary WINS server and, if it does not respond, the secondary WINS server. NT 3.51, NT 4.0, Windows 95, and the WFW 3.11b TCPIP client will query the secondary WINS server if the primary WINS server does not have a registration for that NetBIOS name. This assists in extremely dynamic environments where a name registration may not have replicated yet.

Name queries are the primary challenge in a mixed WINS and broadcast browsing network. These integration issues will be discussed in the next chapter.

## 9.2 WINS Database Replication

In a broadcast environment, replication between backup browsers, browse masters, and domain master browsers can use a lot of bandwidth and workstation processor utilization.

Increased database efficiencies mean that WINS servers can service larger number of name registrations than browse servers. WINS servers also designate specific replication partners and designate when these replications will occur. These improvements reduce infrastructure requirements considerably. Rather than having a browse master and backup browse servers on each subnet, a single centrally located WINS server can supply address resolution for up to 10,000 workstations.

WINS replication server partners are designated as push partners or pull partners. The push partner is configured with parameters designating when replication will occur, which can be either a specific number of database changes or a specific time internal. When a replication is required, the push partner contacts its replication partner and sends its database changes.

A pull partner operates similarly, except that when replication is required, it requests replication from its partner. Its partner then pushes it the database updates.

In each server replication pair, each server is usually both a push and pull partner of the other. During the replication, the pull partner scans its database and builds a table of each WINS server and the highest version ID it has for that WINS server. Then, for each WINS server in its database, the pull partner requests any database records with higher version IDs.

Although each replication partner usually pushes and pulls, special cases may exist. For example, if you have a WINS server that is designated as an online spare. Since this WINS server would not have clients registering with it, the server does not need to push database changes. However, it does need to pull changes from the other WINS hubs.

Figure 9.3 shows a typical WINS configuration with a dedicated online WINS backup server. Since client register with both WinLA and WinNY, these two servers must replicated their databases. However, Backup has no client registrations and therefore no database changes that it must replicate. Instead Backup pulls database changes from both WinLA and WinNY.

Figure 9.3 Online WINS backup

Replication will be covered in more detail in the next section.

## 9.3 Legacy Operating System Support

WANs require a staged approach, and some sites may continue to rely on broadcast browsing. Older versions of Windows must be accommodated. In these networks, many seemingly unconnected problems can be traced to address resolution problems. Resolving these problems requires understanding how WINS and browsing run on integrated networks.

LAN Manager 2.x clients and Windows NT 3.1 clients and servers are b-node clients, and will not register with a WINS server. So if the environment needs a b-node name resolution through WINS, the names should be added to the WINS server as static entries.

To create a static entry, use the WINS Manager. Select the WINS server on which you wish to create the static mapping. Once you have created the entry on this WINS server, it will be replicated to this server's partners. From the menu bar, select Mappings, Static Mappings, and press the Add Mappings button. Type in the server name and IP address.

### 9.3.1 WFW version 3.11

The WFW TCPIP 3.11b network drivers provide a browser client that supports WINS name registration and resolution. However, the browser client remains WINS unaware. While NT 3.5 or higher will query the WINS database to retrieve a browse list, WFW will not. A WFW browse master will only supply the local subnet browse list. However, if you type in the name of the server, the WFW WINS client will query the WINS server for that name.

As with NT 3.1, the WFW WINS client is unaware of the **Domain<1B>** name and cannot browse remote domains.

### 9.3.2 NT Version 3.1 (pre-WINS)

NT Advanced Server 3.1 implemented b-node address resolution, which was what WFW had implemented at the time. However it also provided support for domain-wide browsing by implementing the concept of a domain master browser.

The NT 3.1 server was prioritized to win elections on the subnet. Upon becoming master browser, the server contacts the primary domain controller using a directed datagram, <u>MasterBrowserAnnouncement</u>. The domain master browser then remotes a <u>NetServerEnum</u> call to the browse master, which collects the name list from the browse master.

To use the cross subnet browsing, a WFW workstation changes its workgroup name to the name of the domain. NT domain members are also supported. However, NT workgroups cannot join because NT does not allow a workgroup and domain with the same name to exist. If an NT workgroup is implemented across two or more subnets, it will function as separate workgroups

To ensure that the browse master can locate the domain master browser, each browse master should have an LMHOSTS file containing the names of all domain controllers with the <u>#DOM</u> and <u>#PRE:</u> parameters. To guarantee that the domain master browser can retrieve the browse list from the browse master, the domain master browser must either cache the browse master's name or have the browse master's name in an LMHOSTS file.

The #DOM flag designates a domain controller so the workstation knows which servers it can use to authenticate.  When the computer uses the LHMOST file to resolve addresses, it scans the file from top to bottom.  The #PRE flag design designates a server whose entry should be precached to avoid scanning the LMHOST file each time the address is required. Any entries with #DOM should also use #PRE, as in the following sample LHMOSTS file.

```
10.1.1.2 PrimaryDC #PRE #DOM:domain
10.1.1.2 BackupDC #PRE #DOM:domain
10.1.1.3 PrimaryWINS #PRE
10.1.1.4 Server
```

When an NT 3.1 workstation browses a remote domain, it broadcasts a **GetBackupListReq** to the name **DOMAIN[1d]**, not special **DOMAIN[1b]** name which WINS aware clients use. If there is no subnet browse master for that remote domain, the workstation will directly contact its domain master browser, which will remote the **NetServerEnum** API to the foreign domain master browser and return the result to the workstation.

### 9.3.3 NT Version 3.5 with WINS

NT Server 3.5 included the WINS service, which allowed domain independent browsing and name resolution. If WINS was implemented on each PDC, workstations could resolve addresses for machines in other domains, even if there was no local subnet browse master for that domain. WINS also eliminated the requirement that the domains have either a trust relationship or an unsecured Guest account.

If an NT 3.5 workstation is implemented on subnets as a browse master, it could provide address resolution for both broadcast registered and WINS registered workstations.

Each WINS enabled domain master browser registers the **domain<1B>** name with the WINS server. All domain master browsers occasionally pull all **<1B>** names from WINS. First, they perform a wildcard lookup for addresses of all names ending with **<1B>**. Then they perform a reverse name lookup for each address to obtain the name of the PDC.

The domain master controller adds the foreign domain names to their browse lists and updates the browse masters. If the subnet workstation needs to locate a computer from a foreign domain, the browse master supplies the name of the foreign domain's domain controller. The workstation can directly contact the foreign domain controller to resolve the name.

When a WINS enabled 3.5 workstation browses a foreign domain, it first queries WINS for the **domain<1B>** address, which returns the PDC for the domain. Then the workstation sends a **GetBackupListReq** to the PDC, which returns a list of browse masters for that domain. The workstation picks one of the browse masters at random and remotes a **NetServerEnum** call to that name to retrieve the browse list.

NT 3.51 does contain a bug that requires that all domain controllers for a domain must register with the same WINS server. If they do not, the domain controllers may not properly replicate. This was fixed in NT 3.51 Service Pack 5. Refer to Microsoft KnowledgeBase article Q140978.

### 9.3.4 Proxy Agent

The WINS proxy agent supplies address resolution for non-WINS enabled b-node workstations. The workstation running proxy agent must be WINS enabled and be properly configured to perform WINS queries. The proxy listens for subnet IP broadcast name queries.

The proxy accepts the query and checks its cache to see if it has already retrieve the address. If the proxy agent has not retrieved the address, or if the cached name has expired, the proxy server queries the WINS server for an address and caches the response.

If the proxy agent can resolve the address via its cache or querying the WINS server.The proxy agent will then broadcast the queried name's address unless one of the following conditions is met.

First, if both addresses are in the same subnet, the proxy server will not respond to the name query broadcast. In other words, since the requesting workstation is on the same subnet as the queried workstation, the proxy agent assumes the queried workstation will reply directly to the requesting workstation. The proxy agent can tell if the queried workstation is on the same subnet by comparing the proxy agent's network address with the queried workstation's address.

Second, the proxy server will also not respond to the query if it cannot resolve the address of the requested workstation.  

The default proxy server cache time is 6 minutes, with the minimum time being 1 minute.

The proxy server does not accept broadcast name registrations nor does it register names with the WINS server. However, when the proxy server detects a host registration broadcast, it will query the WINS server to verify that the name is not already registered.

By default, the proxy server will not send a Negative Name Registration Response, but it may be configured by creating the following key:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Netbt\Parameters
  EnableProxyRegCheck REG_DWORD
  Value = 1
```

Similarly, when the proxy agent does not accept name release broadcasts. However, when it receives a name release broadcast, it flushes the name from its cache but takes no other action. This default action can not be changed.

In Windows NT 3.51, to configure a computer as a WINS Proxy Agent, check the "Enable WINS Proxy Agent" box in the Advanced properties of the TCP/IP protocol in the Network Control Panel.

In Windows NT 4.0, you must manually change the following Registry parameter to 1 to enable the WINS proxy agent:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Netbt\Parameters\
  Value Name: EnableProxy
  Value Type: REG_DWORD
  Values: Boolean (0 or 1)
  Default: 0
```

Restart the computer after changing this Registry setting to enable the WINS proxy agent.

## 9.4 Multi-Homed Machines

Servers which contain more than one network adapter, also know as multi-homed servers,  cause connectivity problems whether they are configured to use WINS or not. Although Microsoft networking supports multiple network adapters, NetBIOS only supports a single name per workstation. WINS does provide special support for multihomed servers, however. The servers must use static IP addresses on all network adapters. Use the WINS administrator program to manually add both IP addresses using the MULTIHOMED address type.

To create a WINS entry for a multihomed machine, use the WINS Manager. Select the server on which you wish to create the static entry. From the menu bar, select Mappings, Static Mappings, and press the Add Mappings button. After selected the Type/Multihomed button, enter the name of the server and its IP addresses. The down arrow enters the IP number to the list, while the up arrow is used to prioritize the addresses.

Since a multihomed server may send from any network adapter, you cannot predict which adapter it will use. Statically mapping the addresses ensures that any reverse name lookup will properly return the correct address. This entry type in no way load balances across the adapters, except in cases where the application will randomly select one of the addresses. However, you may be able to statically map each adapter to a different name and load balance by setting different workstations to different names. For most applications this will not work because the server and client will be using different NetBIOS names.

Prior to NT 4.0, the PDC could not be a multihomed server. On initialization, the browser server will bind to one adapter or the other, but which adapter it will bind to can not often be determined. NT 4.0 provides a new Registry parameter that allows the computer browser to be disabled on one or more network adapters.

To enable this feature, you must add the following key:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Browser\Parameters\
  UnboundBindings REG_MULTI_SZ
  Value: NetBT_&lt;name of network adapter driver to be disabled&gt;
```

If you wish to disable the browsing service on more than one adapter, enter each driver name on a separate line in the value field. The network driver name is the name which is referenced in the Registry under HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services.

Each network adapter will contain a subtree for each unique adapter model, such as EPRO, and a subtree for each installed adapter of that type, such as EPRO1 and EPRO2. To disable the browse service on the second adapter, the Registry value would be NetBT_EPRO2.

To verify that the parameter was entered correctly, reboot the server and inspect the event log. Prior to adding the parameter, you would have two entries showing an election: one for each adapter. Now you should only see one browser election called.

When you create a static WINS entry for the PDC, list only the IP addresses for the network adapters that remain active.

## 9.5 Summary:

This chapter has discussed how WINS functions and contrasted with the broadcast browsing.  While broadcast browsing dynamically designates browse masters, WINS uses designated WINS servers. Since the WINS servers are well known and can be tuned for better performance, WINS is the recommended address resolution method.

Most designs should use more than one WINS server, which provides both load balancing and online backups. To ensure that the workstation list is complete, WINS servers designate replication partners. At designated times, the push partner contacts its pull partners, which then pulls any database changes from its push partner. In larger WINS networks, dedicated hub servers are usually created to provide stable replication.  The WINS clients usually register with WINS spokes, which replicate with the hubs.

When implementing WINS, your legacy operating systems may not fully support WINS for address resolution or WAN wide browsing. You may need to implement NT browse masters or WINS proxy agents on each subnet to ensure stable browsing.
