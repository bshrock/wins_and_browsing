---
title: Chapter 12 Implementing and maintaining files
layout: book
---

# Chapter 12: Implementing and maintaining WINS servers in a WAN

The Windows Internet Name Service (WINS) resolves NetBIOS names to IP addresses using designated WINS servers. Because the WINS servers are well known, address resolution requests can be sent directly to these servers. The network traffic associated with broadcast browsing can be eliminated. Implementing WINS eliminates the need to configure the LMHOSTS file or configure your routers to pass UDP port 137 broadcasts for broadcast browsing. Because WINS does not require domain authentication, it can be used on WANs without the necessity of establishing or maintaining trust relationships.

WINS also enables NT 3.5x and WFW machines to browse remote domains without having a browse master for that remote domain on each subnet. Especially in networks with large numbers of domains or workgroups, you can have a significant number of browse masters on each subnet. WINS servers can reduce the number of browse masters required, especially in corporate networks where individual departments control their own servers.

Additionally, WINS reduces IP broadcast traffic on Microsoft WANs because the clients do not have to broadcast to locate the workstations. In routed networks, reducing or eliminating broadcast browsing traffic can significantly reduce network traffic, especially in IPX routed networks where SAP traffic can result in unacceptable network performance.

While broadcast browsing works well on small networks, it breaks down on larger networks. Browsing depends heavily on broadcast messages, and on the principle that all the clients will operate properly. Broadcast-based browsing can not readily recover from errors. WINS addresses several of these issues.

As with broadcast browsing, WINS has a server and a client. The server provides a central registry for all the machines and domains to register their NetBIOS addresses. The client provides the mechanism for the workstation to register its name. WINS also provides the capability of registering domain names, browse masters, and other unique addresses with the server. Once a resource is registered with the WINS server, all workstations will be able to locate these resources.

In addition to allowing dynamic entries from WINS clients, a WINS environment allows several types of manual database entries. Static addresses may be entered for machines that are not WINS enabled and would therefore not register with the WINS server. Multihomed entries must be used for machines which have more than one network card installed.

WINS server comes in the box with NT 3.51 and NT 4.0. The 4.0 version offers significant performance advantages over the 3.51 version, as well as stability enhancements such as self-packing databases.

WINS clients can be configured with the following:

* Windows NT 3.5 or later
* Windows 95
* Windows for Workgroups 3.11b running TCP/IP-32
* LAN Manager 2.2c for MS-DOS
* Microsoft Network Client 3.0 for MS-DOS.

Window NT and 95 WINS clients are included on the installation media. Windows for Workgroup WINS support is built into the TCP/IP-32 version 3.11b protocol stack, which may be downloaded from Microsoft's FPT site. The latter two clients are provided on the compact discs for versions 3.5 or later of Windows NT Server.

## WINS Server Configuration

Because your WINS clients are configured to directly access a specific WINS server, it is critical that the WINS server respond to their queries. The server must also respond in a timely manner or the client request will time out and return a failure to the WINS client. On the WINS server, the actual overhead associated with looking up the NetBIOS name and returning the address to the client is small. The most important tuning requirement is for speed and responsiveness.

When configuring your WINS servers, you need to consider the following elements:

- Hardware
  - Processor speed
  - Memory
  - Network adapter
- Software configuration
  - WINS event logging
  - File system [NTFS vs FAT]
- Network bandwidth

The WINS server does not need to serve that role exclusively; however, when designating other services to run on the server, remember that the WINS server should be responsive. Do not install other services that may involve extensive processor loads, disk access, or network connectivity. Examples of such services include domain controllers, print servers, SQL servers, SMS servers, or internet servers.

In general, if you have under 100 users registering and resolving addresses with a WINS server, you can use that server for other functions. However, for WINS hub servers or servers with more than 100 users, you should use dedicated WINS servers.

## Server hardware sizing

Typically, a WINS system will have at least two servers for redundancy and reliably. The WINS client may be configured with a primary and backup server. Although the WINS server can support up to 20,000 clients, Microsoft recommends two servers for every 10,000 WINS clients. Since test results have shown that a WINS server can handle an average of 1500 name registrations/minute and 750 queries/minute, this is a conservative estimate. Remember, however, that your peak load will occur when your workstations are first powered on. When your customers power on their workstations in the morning, the workstations will register with the WINS server, locate a domain controller, and log on. Computers may register 3 to 7 NetBIOS names, depending on the services running on those computers.

Several server specific configuration options improve the performance of your WINS servers. The performance increases at least by 25 percent by using a dual-processor SMP machine for WINS service. Even a 90 MHz Pentium computer can perform adequately as a WINS server, although a 166 MHz Pentium is recommended.

Use a busmastering network adapter that will cache network traffic when the processor is busy. Installing a second processor also improves network performance since multiple threads can handle network access and database queries.

To further improve database lookup speed, install enough memory to completely cache the WINS database. Generally each WINS database entry uses about 256 bytes, with each WINS client registering between 3 and 7 names. A good rule of thumb is that each client will use 1 KB or 1 MB for each 1000 WINS clients. To accurately determine the size of the WINS database, examine the size of the WINS.MDB file located at %SYSTEMROOT%\SYSTEM32\WIN. Your server's memory size should be at least 32 MB plus the size of your WINS database. If you are running additional NT services, you will require more memory.

## Server software configuration

The WINS server performance is reduced by at least 10 percent if detailed event logging is enabled. If logging is enabled, WINS will write and event log for all change to the WINS database. The location of this log file is specified in the registry at KEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Wins\Parameters\LogFilePath. By default, the log file path is %SystemRoot%\System32\WINS.

If detailed logging is used, informational messages will be included in the log file in addition to database changes. If you use WINS event logging, remember to plan for log file disk storage requirements. Occasionally, the WINS server should be stopped and the log files archived of deleted to conserve space.

While log files are useful for troubleshooting, they are not required during normal operation. The Event Log messages provide sufficient messages to ensure the WINS server is operational.

To change the event logging, run the WINS manager and select Server, Configuration. Press the Advanced button to see the logging options. Within the Advanced WINS Server Configuration section, there are two checkboxes: Logging Enabled and Log Detailed Events.

Figure 12.6 WINS Server Configuration

WINS server database lookups can be improved by using the NTFS file system on the volume which stores the WINS database. NTFS caching and directory structure improves the file access speed. Generally, the file system used, either NTFS or FAT, is selected at installation time. If you are running a DOS file system and wish to convert to NTFS, you may do so using the CONVERT utility.  From the command prompt, type CONVERT X: /FS:NTFS where X: is the drive you wish to convert.

WARNING: once you have converted the file system to NTFS, it may be inaccessible to other operating systems you have installed, including Windows 95 and DOS. The drive letters of your DOS or Windows 95 partitions may also change. Once you have converted a file system to NTFS, you can not convert it back.

## Network connectivity

Network connectivity is also a consideration in properly configuring your WINS server. While the actual WINS query and response are small, typically under 256 bytes, the WINS server maybe be receiving, processing, and replying to several requests simultaneously. If the WINS server does not answer client queries in a timely manner, the client may fail over to the secondary WINS server.

If the server is responding to several queries, it may lose network packets, which also results in poor response times since the WINS clients must retransmit their request. As stated in section 12.1.1, using a busmastering network adapter will help alleviate these problems.

If you are accessing the WINS server across a slow link, consider other network traffic across the link. If the IP packet times out, the client will usually switch to its secondary WINS server. However, client processes are waiting on the address resolution to complete, which may delay other processes on the WINS client computer. Your clients may experience poor system responsiveness.

While, a workstation using a dial-up link has sufficient bandwidth to use WINS address resolution, you should look closely at connectivity to remote offices using slow leased connections and dial up connections. While a WINS registration or query may be less then 256 bytes, your network bandwidth may be used for other functions, such as domain authentication. If general, a 20 workstation office should have a 64 KB line, while 50 workstations may have a 128 kb link.

If you can not meet these bandwidth requirements or if you have more than 100 users in a remote location, you may with to place a WINS server in that location for those users. The remote WINS server would then be configured to replicate with a centrally located WINS server.

When you use a remote WINS server with a central hub, you are trading of the WINS client traffic against the WINS replication traffic. The client traffic is one 256 byte packet per registration or query, while the replication traffic is one 256 byte packet every time a WINS client registration changes.

If you have 1,000 WINS clients registering each morning, the WINS servers would have to replicate 256 KB of new registrations before the local WINS server could resolve addresses properly. This example, while extreme, demonstrates the importance of examining your estimated replication bandwidth requirements.

## WINS service configuration

Once the WINS server is operational, you need to review the server configuration and ensure that its operational times are set properly. The default parameters are valid for most WINS servers. To review the configuration, run the WINS Manager, select the WINS server, and run Server, Configuration.

Figure 12.6 WINS Server configuration

The four WINS timers are Renewal Interval, Extinction Interval, Extinction Timeout, and Verify Interval. WINS uses these timer to determine when the database is cleaned of old registrations.

The WINS timers are checked during automatic WINS server maintenance periods, which is called Scavenging. Scavenging is initiated when the system is booted and regularly thereafter at a frequency of half the Renewal Interval. Scavenging may also be manually initiated by running the WINS manager and selecting Mappings, Initiate Scavenging.

Usually, a WINS client registers with the server when it is booted and unregisters when the client is shut down. However, sometimes clients are shut down improperly and the name is left registered.

The Renewal Interval specifies when the client must renew its name registration. The default is 144 hours. If the client does not renew its name registration within this period, the record will be automatically marked as release, just as though the client was properly shut down.

Records that are marked as released are not immediately removed from the database. Instead, they remain in the database until the Extinction Interval is reached. The default Extinction Interval is 96 hours.

During the scavenging cycle, Released records are marked as Extinct. Records that are already marked as Extinct are checked against the Extinction Timeout. If and Extinct record has been marked extinct longer then the Extinction Timeout, they are marked as Extinct, which is also known as Tombstoned. The default Extinction Timeout is 144 hours.

Extinct records are not immediately removed from the database. Instead, they are removed during the next scavenging cycle.

While the Renewal Interval, Extinction Interval, and Extinction timeout determine how records are marked as released, extinct, and removed from the database.

If replication is configured, then the Verification Interval specifies how frequently the records not belonging to this server should be verified. During the scavenging cycle, if the Verification Interval is exceeded, the WINS server find all records that belong it other WINS servers and then query those servers to see if the registration is still valid. Invalid records are immediately deleted. The default Verification Interval is 576 hours.

## Replication

In a typical WAN, you will have more than one WINS server. In addition to spreading the load across several servers, you should optimize network traffic across slow WAN links. Additional WINS servers also provide an online backup of your WINS database and secondary WINS servers for your WINS clients.

When the WINS client is configured to use a WINS server, it registers with that server and resolves names with that server. The WINS clients that are set to use that server only know about other clients set to use the same server. You probably want all your clients to be able to connect with each other, so you need to configure Replication between your WINS servers.

## Push/Pull Partners

Replication is the process by which WINS servers share their database with other WINS servers. When you configure your WINS server, you must set several parameters for each server. Every server has replication partners with which it will replicate. The two types of replication partners are push partners and pull partners. In a properly configured WINS system, each replication partner will be both a push partner and a pull partner.

In the replication relationship, the WINS push server contacts its partner to send any required database updates. The WINS pull server contacts its partner to request a push. If both replication partners are not set to push and pull, the all database updates are not necessarily replicated on both servers.

To select Replication partners and set their parameters, run the WINS Manager and select Server, Replication Partners. The Replication Partners screen shows all WINS servers of which this server knows. To add other WINS servers to this list, press the Add button and enter either the IP address of the server or its name. Remember: since replication is not yet configured, your WINS server may not be able to locate the other WINS servers by name.

Figure 12.8 WINS Replication Partner configuration screen

To select a push partner, select the WINS server with which you wish to replicate and, in the Replication Options, check the Push Partner box. To configure the push replication options, press the Configure button.

Figure 12.9 WINS Push Replication Partner configuration

The only configuration option for a push server is the update count. When the number of changes to the server's WINS database exceeds this count, the server will contact its replication partner and request that partner start a replication. Only database changes on this server apply to the update count, not updates received from other servers. The minimum update count is 20 records.

To select a pull partner, select the WINS server with which you wish to replicate and, in the Replication Options, check the Pull Partner box. To configure the Pull replication, options, press the Configure button.

Figure 12.8 WINS Pull Replication Partner configuration

The two configuration options for a pull server are the start time and the replication interval. The start time indicated the time at which the server will contact its replication partner to begin a replication. Once this initial replication is complete, the pull server will contact its partner at the interval specified to initial additional replications.

The default WINS push and pull replication intervals are conservatively set for a large WINS network. If you have a smaller network of less than 1000 users, you may wish to reset these values for make the servers more responsive and allow faster replication of new registrations. If you are replication across slow WAN links, you may also wish to adjust the values to optimize your network traffic.

The network traffic for replication is low. Before replication begins, the partners  will establish a TCP/IP session. This communication consists of about 20 packets totaling about 2000 bytes. Then the servers will exchange WINS database updates. These updates consist of about 128 bytes per record. Although the actually size of the transactions is small, if the transaction does not complete properly, the entire replication must begin again.

Until a computer's registration has been replicated to all WINS servers in the network, that computer can not reliably be located. The pull replication value is the most reliable indication of how long registrations will take to replicate. In a simple two WINS server network with a 20 minute pull replication interval, all registrations will be updated within 20 minutes.

In a WINS network with a hub and spoke pattern, using a 20 minute pull replication period, the registration may take up to 40 minutes to reach all servers: 20 minutes to reach the hub and 20 minutes to replicate to the other spokes.

The push replication value acts to allow updates under extraordinary circumstances, such as a large group of workstations powering on in the morning. In this case, the WINS server would receive a large number of registrations which it would wish to immediately replicate to its partners. In most situations, the default value of 20 is acceptable.

The best indicator of how to set the WINS replication parameters is how well they perform on your network. If records are not properly replicating, then the pull interval probably does not allow enough time for successful completion before the next interval begins. Start pull replication at 20 minutes and increase it until successful replications occur.

## WINS Database Version ID

The version ID is a unique number that identifies that database entry. When the WINS server is created, the WINS server's Version ID is initialized with the value 1. Every time the WINS server accepts a registration, the registered name is assigned the current version ID, then WINS server's Version ID is incremented. This mechanism allowed each name registration to have a unique identification number. Because each WINS server maintains its own version IDs, the  combination of the IP address of the WINS server accepting the registration and the registration record's Version ID uniquely identifies the name registration.

When the WINS server first accepts a registration, it initializes the creation time with the server's current time. When a WINS client reregisters with a WINS server, the creation time is updated to reflect the new registration. If a WINS client does not reregister by the renewal time, the WINS server will contact the client to request that it reregister. When the WINS servers share database information, this time is used to resolve any question of which server owns the registration or which registration is more current. For this reason, your WINS servers should synchronize their clocks from a single reliable time source. Ultimately, it does not matter is the time is accurate, as long as all your WINS servers maintain the same time.

Typically, networks containing Unix servers will maintain an NTP server providing a standard, network wide time. If an NTP server is not available, an external source such as the US Naval Observatory can be used. The NT Resource Kit includes TIMESERV.EXE, an NT service which allows the server clock to be synchronized with a number of different time sources. The TIMESERV documentation includes suggested public servers which can be used for synchronization.

Remember that each database entry, even if it has been replicated to other servers, belongs to the original server which registered that name. When the original server changes a record's status, this information is replicated to other WINS servers, which honor the actions of the registering server and update their databases accordingly.

## The Mechanics of WINS Database Replication

Database replication is based upon the version ID. When a WINS push server contacts a replication partner, it sends the push server's current version ID. The replication partner scans its database and builds a list of all known WINS servers and the highest version ID for each WINS server. The replication partner then returns this information to the push server. The push server then sends all records that its replication partner does not already have.

Figure 12.13 WINS replication using Version IDs

WINS1 |  |  | WINS2 |  |  |
---|---|---|---|---|---
Owner | Name | Version |Owner | Name | Version
WINS1 |Workstation11 | 101 | WINS1 | Workstation11 | 101
WINS1 |Workstation12 | 102 | WINS1 | Workstation12 | 102
WINS1 |Workstation13 | 103 | WINS1 | Workstation13 | 103
WINS1 |Workstation14 | 104 | WINS2 | Workstation20 | 200
WINS3 |Workstation31 | 301 | WINS3 | Workstation31 | 301
WINS3 |Workstation32 | 302 | WINS3 | Workstation32 | 303

Figure 12.13 shows the databases contained on two WINS servers, WINS1 and WINS2. WINS2 contains records for third WINS server, WINS3, however WINS3 is not shown.

When WINS1 requests a replication, it sends the highest version IDs it has for each server: WINS1 104 and WINS3 302. WINS2 then returns any records for these servers with higher version IDS: WINS2 200 and WINS3 303.


> The WINS servers can resolve cases in which a client originally registers with a WINS server then subsequently registers with a different WINS server. However, it is highly recommended that each workstation always register with the same WINS server.

When configuring WINS replication between your servers, you want to make sure that all servers eventually replicate with all other servers, so that all servers will have all addresses. Usually, a hub and spoke replication pattern is used. You should specifically avoid circular replication patterns.

In a hub and spoke pattern, the WINS servers with which clients register are designated as spokes. The spokes are not allowed to replicate with other spokes. Instead, the spokes replicate with hub servers, which act as central repositories.

Figure 12.11 Hub and Spoke Replication

## Hub and Spoke Replication

In a circular replication pattern, one of the WINS server's replication partners is set to replicate with another of the WINS server's replication partners. We can see an example in Figure 12.12. If Workstation1 is powered down without properly unregistering its name, the record would eventually be scavenged and removed from the WINS1 database. However, the record has already been replicated to WINS2 and WINS3 and, because of the replication pattern chosen, the record maybe be replicated from WINS3 to WINS1. In this manner, invalid WINS records can be endlessly pushed from server to server.

Figure 12.12 Circular Replication

For the same reasons, you should avoid replication patterns with more than two levels of servers. You should  design your WINS network for the smallest number of WINS servers to avoid excessive replication traffic.

## WINS Replication Structure Design

The ideal WINS network consists of dedicated WINS hub machines as the primary WINS repository, as shown in Figure 12.11. WINS spoke machines provide address resolution services to the WINS clients. The spoke machines accept name registrations, which are then replicated to a WINS hub from which they will be replicated to other WINS servers.

When designing your WINS replication structure, you should consider the placement of your WINS clients and their network connections to the spoke WINS server. The spoke WINS server should have at least 64KB available bandwidth for client address resolutions and replication with the hub. If other network traffic uses that link, ensure at least 64KB is usually available for WINS.

If you are using slow links to remote sites, you should generally consider locating the spoke WINS server at the remote location rather than resolving addresses against a spoke server across the link. As your network grows, you can add additional WINS spokes to handle the WINS client load.

The WINS hub machines should be located on a high speed network. Each hub machine will be configured to replicate with each other hub machine. Hub machines should not accept WINS registrations. Typically, one of the hub machines will be designated as the online backup.

## Regular Maintenance

Although WINS usually operates trouble free, it does require regular database maintenance. Because the database is dynamic, the WINS service must be shut down during these maintenance cycles. Typically, an unattended script is created which will shut down the WINS service, perform the maintenance, and restart the service.

The two most critical maintenance operations are compacting the WINS database and backup up the database.

## Packing the Database

WINS stores its database using the Jet database engine, which is included with Windows NT. As registrations are added and deleted, the database must be manually compressed to recover lost space and reindex the database. NT 4.0 upgrades the Jet engine to provide automatic database packing, but the NT 3.51 WINS databases must be manually packed. It is recommended that the database be packed when it exceeds 15 MB, although you'll probably see errors before this limit is reached.

You can manually compress the WINS database using the JETPACK utility. JETPACK is only installed if WINS is installed on the machine. Before compressing the database, you must shut down WINS. Create a batch file with the following commands:

```
CD %SYSTEMROOT%\SYSTEM32\WINS
NET STOP WINS
DELETE TMP.MDB
JETPACK WINS.MDB TMP.MDB
NET START WINS
```

In this example, we change to the WINS subdirectory, stop the WINS service, compress the database, and restart WINS. The JETPACK command line has two parameters. The first, **WINS.MDB**, is the WINS database. The second, **TMP.MDB**, is a temporary database that JETPACK creates.

JETPACK compresses the active database into the temporary database, deletes the active database, then renames the temporary database to the active database. Make sure that you do not have a file called TMP.MDB in the WINS directory.

Remember that packing the database may take several minutes and during this time WINS will be unavailable for address resolution. You should only compress the WINS database after hours or when workstations will not be registering or resolving with the WINS server.

Once the batch file is created, it can be scheduled to run using the NT AT command. Save the batch file to the %SYSTEMROOT% directory as COMPRESSWINS.BAT, then from the DOS prompt type:

```
AT 01:00 /every:SUNDAY "COMPRESSWINS.BAT"
```

Which tells the NT scheduler to run the batch file every Sunday at 1:00 AM.

## Backup and Restoration

Backing up your WINS databases is not as critical as with some systems. As long as you have more than one WINS server, you can always replicate your database back onto your corrupted WINS server. In the worst case, your clients must reboot their workstations to reregister with their primary WINS server. However, losing your database will disrupt operations and should be avoided.

By default, the WINS databases are stored in **%SYSTEMROOT%\SYSTEM32\WINS**. The files contained in their subdirectory depend on the version of WINS you are running. The primary WINS database is **WINS.MDB**. You may see other temporary working files, such as **TEMP.MDB** and **JET*.LOG**.

Some information is also stored in the Registry in **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\WINS**. The most critical information stored in this subkey is the current version ID of the WINS server.

The easiest way to backup your WINS database is to have WINS automatically back it up for you. Within the WINS Manager, from the Mappings menu, select the Backup Database command. In the Select Backup Directory dialog box (see Figure 12.3), enter the location for the backup. WINS will automatically backup the database for you every 24 hours. By default, no backup location is entered, which means that WINS does not automatically backup the database.

Screen capture 28

You may also set WINS to automatically back up the database when WINS is shut down. To select this option, start the WINS Manager, select your WINS server, and select Server, Configuration. From the Server Configuration screen, press the Advanced button, and check the Backup on Termination box (see Figure 12.4).

Screen capture 28

In the following case, to backup the database, you stop the WINS service then restart it. The batch file would look like this:

```
CD %SYSTEMROOT%\SYSTEM32\WINS
NET STOP WINS
NET START WINS
```

To restore your WINS database files, you must:

1. Stop the WINS service
2.  Delete the following files from your SYSTEMROOT%\SYSTEM32\WINS directory: WINS.MDB, TEMP.MDB, JET.LOG, and JET*.LOG
3. Move the backup copy of WINS.MDB into the WINS directory.
4. Restore the WINS Registry entries.
5. Restart the WINS service.

The primary problem with backing up your WINS database, is that in a dynamic environment, your information is probably not current. The static mappings and server entries may be current, since this information rarely changes, but workstation registrations may be inaccurate. The version ID information stored in the Registry will almost certainly be inaccurate, which will result in new registrations with restored WINS servers not replicating properly to other WINS servers.

Remember that the WINS server replicates its entries by sending its current version ID. If this version ID is lower then the highest version ID stored on its replication partner, the replication partner will not request updates.

If you have more than one WINS server, you may wish to restore your database from the active WINS server rather than restoring the old WINS database. If you wish to follow this approach, follow the instructions in the next section for migrating your WINS server to a new machine.

## Migrating WINS servers

Often, WINS servers are installed by individuals and departments before they are accepted into the corporation. As the number of WINS users increases, you may need to update your WINS infrastructure. Typically, you would implement hub WINS servers while upgrading your existing WINS servers. Perhaps you wish to upgrade your NT 3.51 WINS servers to NT 4.0.

While implementing new WINS servers is a trivial task, administratively you may not wish to update your WINS clients to use a different WINS server. The solution is to implement a new WINS server at the same IP address as the old WINS server. When implementing a new WINS server, there are three strategies for migrating the database.

The first strategy involves having all the WINS clients reboot, which will register them with the new WINS server. If you wish to follow this strategy, you will probably find it easier to just implement a new WINS server, insert it into the replication pattern, and reset your clients to use this address.

The second strategy is to install the new WINS server and move the database from the old WINS server to the new WINS server. This approach is covered in section 12.3.2, "Backup and Restoration."  However, WINS servers typically develop some data corruption or bad records, which WINS will gradually correct during routine WINS scavenging cycles. If you move the database from the old WINS server, you will also move the corrupt records. While your operations may not be disrupted, you will have to wait for WINS to correct these problems.

The third strategy is to install a new WINS server, establish replication partners, and let its partners replicate the full WINS database back to the new WINS server. In this section, we will discuss this strategy and the special considerations involved.

When you are replacing a WINS server with a new WINS server at the same IP address, you must make sure that you set the new server's beginning version ID to be higher than the old server. If you do not, its replication partners will not properly replication. As we iterate the installation procedures for the new WINS server we will clarify this point.

Before installing a new server, you should back up two Registry subtrees:

```
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\WINS\Parameters
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\WINS\Partners
```

The **Parameters** subkey contains the WINS configuration information which is ordinarily accessed through the WINS Manager program. One the new server, you can use the old values or set new values, but you must record the current version ID of the old WINS server. Since the Registry can store a maximum numeric value of **xFFFFFFFF**, the version ID is split into the high order value and the low order value. The two keys are as follows:

```
VersCounterStartVal_LowWord: REG_DWORD
VersCounterStartVal_HighWord: REG_DWORD
```

The **Partners** subkey contains information on the server's replication partners which would ordinarily be set through the WINS manager program. If you wish to establish a new replication pattern, you do not need to backup this information.

If you do not have access to the Registry of the old WINS server, you can determine the version ID by examining the WINS database of one of its replication partners. Run WINS Manager, select the replication partner, and pick Mappings, Show Database. Under Display Options, select the original WINS server, and Show Only Mappings from Selected Owner. Then under Sort Order, select Sort by Version ID. Scroll down through the database until you locate the highest version ID from this server (see Figure 12.5).


Figure 12.5 WINS database sorted by Version ID

As you recall, the WINS database uniquely identifies every record by the IP address of the WINS server that accepted the registration and the version ID that the WINS server assigned to the record. When a WINS server replicates with its partners, all replication is based on these two values. A WINS server asks its partner for its current version ID, locates its highest version ID from that partner, then pulls all records with version IDs higher than its current highest record number.

When you start the new WINS server, if the starting version ID is less than the version ID from its replication partners, the partners will not pull any records until the new WINS server reaches a version ID higher than they have on file. In other words, none of the new registrations with the new server will replicate and those workstations will be unreachable for anyone who is not using the new WINS server.

On the other hand, when you set the version ID on the new server to be higher than its previous version ID, the new server will pull all records that it does not have. In the case of the new WINS server with an empty WINS database, the new server will pull all of its records from its replication partner.

After installing the new WINS server with same IP address as the old WINS server, restore the two Registry subkeys that you backed up. For safety, before the new server replicates with its partners, increase the version ID to higher than its previous highest value.

## Summary

In this chapter, we have show how to properly configure your WINS server. Before any design is started, you must know how many WINS clients this server will support and how where those users will be located.

Each server must have a hardware and software configuration which will support its users. For smaller environments, the WINS service can be run on a nondedicated server, but if over 100 clients use a single server or if the server is a hub server, then a dedicated server should be used.

Once the WINS servers are configured and deployed, replication must be configured. Replication is the process by which WINS servers share their information with each other. Without replication, you do not have a single address space.

When your WINS system is operational, you will have to perform routine maintenance. The most critical operations are compressing and backing up the databases.

As your network grows, you may find you need to migrate your existing WINS servers to large, more capable, or dedicated servers. When migrating, the two primary approaches are moving the existing databases to new servers or configuring a new WINS server and replicating it with the old WINS server's partners.
