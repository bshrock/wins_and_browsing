---
title: Chapter 14 Troubleshooting files
layout: book
---

# Chapter 13: Troubleshooting and resolving WINS problems

Although WINS usually operates trouble free, it can develop problems. Usually these problems prevent proper name-to-address resolution, although they may manifest themselves in a variety of ways

In addition to workstations being unable to locate or connect to servers, you may experience domain authentication problems or WINS replication errors. To successfully resolve these problems, you must be able to identify the fact that you are having WINS problems. Once the problem is identified as WINS related, you must be able to isolate and correct the cause.

The most common cause of WINS related problems is an incorrect WINS database replicating its information to the other WINS servers in the system. Once the problem WINS server is identified, it can be corrected, rebuilt, or removed from operation. The records for this WINS server must then be removed from the other WINS servers in the system.

In addition to the WINS Manager included with NT server, the NT Resource Kit contains a number of utilities which allow the administrator to isolate and correct WINS server problems. These utilities are discussed further in section 14.5.  

Since most WINS server problems first appear on workstations using the WINS clients, the ability to isolate and correct WINS server problems depends on understanding how these problems appear to the WINS client.

## Isolating WINS Problems

WINS servers are susceptible to a variety of problems, including network connectivity issues, server replication problems, and improper workstation registration or releases. To your WINS clients, most of these problems will appear as errors in resolving names or locating resources on the network. To resolve these problems, you need to eliminate the basic, obvious problems and progressively advance to more serious problems.

Before beginning extensive troubleshooting procedures, you should ask yourself:

* is the WINS client properly configured with the proper WINS server address?
* Is the WINS client's TCP/IP parameters properly configured?
* Can the WINS client ping the WINS server?
* Is the WINS service running on the WINS server?
* Is the WINS server's TCP/IP parameters properly configured?
* Can the WINS server be pinged from the WINS client?

## Cannot Connect to WINS Server

If your WINS clients cannot resolve addresses, the WINS server may be unreachable. To resolve addresses, the workstation must be able to communicate with its own installed network drivers, through the physical network cabling to the default network gateway, and to the WINS server. Proper communications can be verified by pinging the workstation’s local address, the default gateway, and the WINS server.

First, we must determine these three IP addresses: the local workstation, the default gateway, and the WINS server.

From Windows for Workgroups or Windows NT, use the IPCONFIG command. From the command prompt, type IPCONFIG /ALL. First, the command will list some general IP information about the system, such as the machine name and DNS server addresses. Next, the IPCONFIG command lists each network adapter installed in the machine and the IP information for that network adapter.

Screen capture 37

## IPCONFIG output

In the example shown, the workstation has two network adapters installed: EE161 [Intel EtherExpress 16] and NdisWan4 [a RAS connection]. Since the workstation’s RAS connection was not active when the capture was performed, the RAS IP configuration does not contain any IP information.

From Windows 95, use the WINIPCFG command. The output is similar to the IPCONFIG command, except presented in a windowed application. From the Start button, select Run, and enter the command WINIPCFG. Press the More Info button.

Screen capture 38

## WINIPCFG output

In this example, the IP address information for the EtherExpress adapter is:

```
IP Address: 207.91.93.243
Default Gateway: 207.91.93.1
Primary WINS server: 207.91.93.12
```

To verify network connectivity, ping the workstation address, default gateway, then the WINS server. Which ping test fails will help you determine where your network connectivity problem is located. To ping an address, from the command prompt, type PING _address_.

Screen capture 39

## PINGing the WINS server

If you can not ping the workstation address, then probably your workstation network configuration is incorrect. Verify that you have the correct network adapter selected, as well as the correct interrupt and address information for that adapter.

If you can not ping the gateway address, then your computer’s network adapter is probably not connected properly to the network. Verify that the network cable is connected from the network adapter to the network port. Verify that your network hub is working properly.

If you can not ping the WINS server, then you probably have network connectivity problems between the WINS server and the network. Follow the same sequence of ping tests, but this time ping from the WINS server to the workstation. Try another workstation to see if you can ping the WINS server from there.

Note: in the workstation example used here, the WINS server is located on the same subnet as the workstation, so we would not have to test the connectivity from the workstation to the default gateway.

If the ping tests show proper IP connectivity between the workstation and WINS server, try running WINS Manager and connecting to the WINS server. If you cannot connect using WINS Manager, try stopping and restarting the WINS service. Check the event log to ensure that the WINS server did start properly and is running without error messages.

If the workstation and WINS server have IP connectivity and the WINS server appears to be running properly, you may need to backup the WINS database and reinstall the WINS services.

## Duplicate Name Error Messages

When the WINS client tries to connect to a network resource, you may receive a "duplicate name" error. Usually this occurs when the workstation name is registered more than once in the database. Typically, you have a static record in the WINS database and a workstation has dynamically registered the name. The two resolutions to this problem are as follows:

1. Examine the WINS database to locate the static records. When you have identified which server owns the record, you must run the WINS Manager against that server and delete the record on the owning WINS server. The record can only be deleted on the server which owns the record. Once the record has been deleted, the server must be scavenged to remove the record, then the deletion must be replicated to the other servers in the network. Instructions on how to scavenge the server are provided later in the chapter in the section, "Scavenging the Database."
2. Configure the server to allow dynamic registrations to overwrite the static entries. To set this parameter, run the WINS Manager and select the WINS server for which you wish to set Migrate On. From the menu, select Server, Configuration and press the Advanced button. Check the Migrate On/Off checkbox. You should set this option on all servers.

Screen capture 40

## Disabling WINS server Migration

When Migrate On is selected and the server receives a registration that conflicts with a static registration, the server will challenge the machine at the static address. If the static address does not respond to the challenge, then the WINS server will allow the dynamic registration.

## Network Path Not Found on WINS Client

If your WINS client receives the message "network path not found," then the workstation which it is seeking may not have properly registered with the WINS server. If the workstation has properly registered with its WINS server, then the registration may not have properly replicated to the WINS server which your client is using.

To resolve this problem, you must determine with which WINS server the workstation is registering and which WINS server your client is using for address resolution. This information can be obtained using the WINIPCFG command in Windows 95 or the IPCONFIG command in DOS and Windows NT. Instructions for using these commands is located in section 14.1.

Run WINS Manager, select the WINS server which the WINS client is using for address resolution, and try to manually locate the workstation's registration. If client’s  WINS server does not have a registered name, then the workstation’s WINS server is not properly replicating with the WINS client’s WINS server. Move on to the following section: "Server Not Replicating with its Partners."

## Server Not Replicating with its Partners

If WINS servers are not properly replicating, the cause is usually pretty basic. The actions to take to ensure proper replication are as follows:

* Verify network connectivity between the two servers [see section 14.1 for ping test procedures]
* Verify that the server and its replication partners are all configured as push and pull partners.
* Manually trigger a replication and check the event log to verify that the replication completed properly.

If the server’s are properly configured for replication, but are still not replicating properly, your options are extremely limited. Your best solution is to back up the WINS databases, deinstall and reinstall WINS on both servers, and restore the databases. Since this solution maybe be undesirable for a number of reasons, you may wish to take the easy way out.

If the list of servers that are not replicating is small and if they have fixed IP addresses, you can manually add the addresses as static addresses.

To add a static address, run the WINS Manager and select the server on which you wish to add the static addresses. From the menu, select Mappings, Static Mappings, and press the Add Mappings button. Enter the name and IP address of the machine, then press the Add button.

Screen capture 41

## Removing Bad WINS Servers

If a WINS server has become completely corrupt, you may need to remove the server from your network entirely. The records which that server owns, however, have been replicated to all the other WINS servers. Even if you remove the WINS server from your system, you must remove all the records that that server owns also.

To delete a WINS server, you should first run WINS Manager, select the server which you wish to delete, and from the menu select Server, Replication partner. In Figure 14.6, server 207.91.93.243 is configured as a push and pull partner with 207.91.93.12. To remove this replication partnership from server 207.91.93.243, uncheck the Push Partner and Pull Partner check boxes, and press OK.

Screen capture 42

## Removing the Replication partner

Remember that for each of that server's replication partners, you must also run WINS Manager and remove the discontinued server from their partner’s replication list.

Once the discontinued server is removed from the replication pattern, you must remove that discontinued WINS server's records from the database stored on each server.

Using WINS Manager, connect to each remaining WINS server then select Mappings, Show Database. From the Selected Owners list, select the discontinued server, then press the Delete Owner button.

Screen capture 43

## Deleting an WINS server’s records

### Removing Entries from Non-Existent WINS Servers

In some cases, you may be unable to delete a WINS server's records from your database, because WINS will not delete entries that are owned by other WINS servers that are not functioning. Records owned by the no longer functioning server will be endlessly replicated through your network.

To remove a nonfunctioning server's entries from the database, you must create a new Registry key. From the Registry Editor, go to the HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WINS\Partners key and create a new value:

```
Value Name: PersonaNonGrata
Data Type:  REG_MULTI_SZ
Value:      IP address of non-existent WINS server that is the owner server of the unwanted records being replicated.
Default:    (None)
```

After creating the value, you must stop and restart the WINS service for the value to take effect.

Once a server has been added to the **PersonaNonGrata** value, you can run the WINS Manager and use the Delete Owner button to remove the server's records from the WINS server. The value must be added and the owner deleted from all WINS server. Adding the **PersonaNonGrata** key just prevents a WINS server from accepting any WINS entries which are owned by servers on the **PersonaNonGrata** list.

As always, any registry changes are immediately saved to the registry. Before making any manual modifications to the registry, you should make a backup.

### Removing Non-Existent Servers That Continue to Appear in WINS Manager

A similar problem may occur when removing WINS servers. After you have used WINS Manager to delete the server from the Owner list, the server may reappear. This problem occurs when you improperly remove a server from the replication pattern.

Once you have removed a server's replication partners, you must also run WINS Manager against its partners and remove the deleted server from their replication lists. If you do not, then the replication partners will keep the server active and will not allow it to be removed from the Owner's list.

To correct the problem, you must stop replication on all WINS servers. To stop replication, use the WINS Manager to connect to all WINS servers and remove their WINS partners, as described in section 14.2. Before deleting the replication partners, document the replication parameters for all WINS servers. Then, using the procedures outlined in section 14.2,  remove the decommissioned servers from each WINS server in the network. Once the decommissioned server has been removed from all WINS servers, use the WINS Manager to reinstate all WINS replication partnerships.

### Performing Mass WINS Server Record Deletions

Sometimes, you need to remove a large number of corrupt WINS records from the database. For example, if you replace an existing WINS server with a newly installed WINS server, you may forget to reset the version ID on the new server. When this server begins replicating with its partners, it will begin with version ID 1. However, the previous server running at the same IP address probably had a much higher version ID.

To correct this problem, you would need to reset the version ID and delete all the new records from the database. To perform mass record deletions, you can use two utilites from the NT Resource Kit: WINSCL and WINSDMP (discussed later in the chapter in the section, "NT Resource Kit Tools").

WINSDMP allows you to save the WINS server’s database as a standard text file. This program will not modify the WINS database in any way. WINSDMP servers two primary purposes. First, since the WINS database is dynamic and may change while you are examining it, WINSDMP provides a snapshot of what the database looks like at that instant. Second, WINSDMP allows you to import the database into a database or spreadsheet, where you can better search and sort it for anomalies.

WINSCL allows you to perform extensive modification on the WINS database from the command prompt. Although you can use the WINS manager, WINSCL provides a much better interface for this purpose. You should backup your WINS database before using WINSCL, since all changes will be immediate and irreversible.

First, you must identify the corrupt records. You can use the WINSDMP utility to dump the WINS database to a standard comma delimited (**.csv**) file. Import the **.csv** file into a spreadsheet. Sort this spreadsheet by WINS server owner [column L] and version ID [column H].

Screen capture 44: WINS database sorted by WINS server and Version ID

Usually you will notice the lower version ID numbers are newer records then the higher version ID numbers. You'll want to delete the records which have improperly been assigned lower version IDs. In the example shown, the WINS server’s IP address was changed after the server began receiving registrations. The registrations received under the old IP address, 10.1.1.1, must be removed.

Once you have identified the version ID range that needs to be deleted, you can use the WINSCL utility to delete this range (refer to the section "WINSCL.EXE").

### Recovering WINS Databases

In environments with a single WINS server, or when you do not have access to the WINS database from the replication partners, you may have no choice except to try to recover the existing WINS database. When trying to recover a single database, your only options are to compact and scavenge the existing database or to delete the existing database and reregister the WINS clients. Compacting the WINS database is covered in Chapter 12, "Integrating DNS with WINS" in the section entitled "Packing the Database."

Scavenging the database is a low risk, low impact option since the WINS server initiates scavenging as a normal part of its housecleaning operations. However, you may want to force the WINS service to perform these operations immediately rather than wait.

Manually deleting and recreating the database is a more disruptive since the WINS server is unavailable during this operation. Also, all WINS clients must reregister and those registrations must be replicated.

Consistency checking is also a disruptive operation, but the WINS servers remain available during the process. However, since consistency checking greatly increases network traffic and processor utilization on all the WINS servers, it should not be used during production hours.

### Scavenging the Database

WINS database scavenging is a normal scheduled WINS service activity. The WINS service automatically starts a database scavenge when the service is started and thereafter at an interval equivalent to half of the WINS configuration Renewal Interval.

During the scavenge cycle, the WINS service cleans the WINS database. How records are cleaned depends on the state of the record and whether the WINS server owns the record or the record is replicated from another WINS server. Table 14.1 details what actions are performed based on the record’s state.

Table 14.1 Scavenging cycle actions

Record's state  Owned ..	If Replicated..
Active	If Renewal interval	If Verified <br>
	has expired, then 	interval has<br>
	marked Released	expired then<br>
		Re-verify
Released	If Extinct interval	N/A<br>
	has expired, then 	<br>
	marked Extinct
Extinct	If Extinct	If Extinct timeout <br>
	timeout has expired, 	has expired, then <br>
	then Deleted	Deleted
Deleted	Deleted	Deleted

Record's state | Owned ..	| If Replicated..
---|---|---
Active | If Renewal interval has expired, then marked Released| If Verified interval has expired then Re-verify
Released | If Extinct interval has expired, then marked Extinct| N/A
Extinct | If Extinct timeout has expired then deleted | If Extinct timeout has expired, then Deleted
Deleted | Deleted | Deleted

Although scavenging is automatic, if extensive registrations, releases, or manual record deletions have occurred on a WINS server, you may wish to manually initiate a scavenge cycle.

To manually initiate scavenging, run the WINS Manager and select the server you wish to scavenge. From the menu, select Mappings, Initiate Scavenging.

Screen capture 45: Initiating Scavenging<

Remember: during scavenging the server checks the state of every record, including contacting other WINS servers for records which is does not own. This process may take a lot of time.

When manually scavenging a database, you should remember several key points:

* If you manually delete a registration, the registration should be deleted from the server that owns that registration.
* Depending on the record's state, several scavenge cycles may be required to delete the record from the WINS database. Allow enough time for the scavenge cycle to complete before initiating another scavenge cycle.
* Replicated copies will not be removed from the other WINS servers until the record is successfully deleted from the WINS server which owns that record.
* Once records are deleted from the WINS server which owns the record, that server must replicate with its partners before the record will be deleted from the other WINS servers.

## Manually Recreating a WINS Database

If the WINS database has extensive corruption, you may wish to delete the entire database and create a new database. In this case, after the new database file is created, all WINS clients must reboot and reregister with the WINS server.

Although both Windows NT 3.51 and 4.0 use the same database file name, WINS.MDB, and the same location, systemroot%\system32\wins, the recreation process is different for the two operation systems as detailed in the two sections that follow.

## Manually Reinitializing a WINS Database in NT 3.51

Before following this procedure, you must have the original NT 3.51 source files. You will be restoring files from the original NT source CD. Before making any changes to the WINS database, you must first stop the WINS service.

* From the Control Panel, run the Services applet. Select the Windows Internet Name Service and click Stop.
* You must rename the old WINS database. From the command prompt, type:

```
REN %systemroot%\SYSTEM32\WINS\WIN.MDB %systemroot%\SYSTEM32\WINS\WIN.OLD
```

* Now that the old, corrupt WINS database has been renamed, you must restore the WINS database from the original NT 3.51 source CD. Assuming the NT CD is in the D: drive, from the command prompt type:

```
copy D:\i386\System.md_ %SystemRoot%\System32\WINS\System.md_
del %SystemRoot%\System32\WINS\System.mdb
expand %SystemRoot%\System32\WINS\System.md_ %SystemRoot%\System32\WINS\System.mdb
```

* You may now restart the WINS service. From the Control Panel, run the Services applet, select the Windows Internet Name Service, and click Start.

## Manually Initializing a WINS Database in NT 4.0

When NT 4.0 WINS service starts, it will check for the existence of a WINS database and, if that database is not found, WINS will create a new database. As with NT 3.51, before making any changes to the WINS database, you must first stop the WINS service.

* From the Control Panel, run the Services applet. Select the Windows Internet Name Service and click Stop.
* You must rename the old WINS database. From the command prompt, type:

```
REN %systemroot%\SYSTEM32\WINS\WIN.MDB %systemroot%\SYSTEM32\WINS\WIN.OLD
```  
* Now that the old, corrupt WINS database has been renamed, you may restart the WINS service, which will automatically create a new WINS database.
* From the Control Panel, run the Services applet, select the Windows Internet Name Service, and click Start.

## Consistency Checking

When a WINS server becomes unsynchronized with its replication partners, it may become unable to determine which records are still active in the database and which records should be deleted. When this occurs, a consistency check should be performed between the replication partners.

Consistency checking can be manually performed using the WINS command-line tools from the NT Resource Kit. The two tools supplied for this purpose are WINSCL and WINSCHK (discussed later in the chapter).

When using NT 4.0, you can set the WINS service to automatically perform consistency checking. During this process, the WINS service will replicate all records from an owner to determine if its records for that owner are synchronized properly. This operation consumes a lot of processor time and network bandwidth, which may not be desirable if your WINS server is not dedicated or is across a slow network link from its replication partners.

> Keep in mind that in verifying the database, WINS will use the most current version of all records.

During the database consistency check, WINS will not delete a record if one of the partners with whom it is verifying does not own the record, since WINS does not know whether it or its partner is most current. Records that are not owned by one of the replication partners will only be deleted when the server which owns that record replicates the deletion to all WINS servers in the network.

To enable NT 4.0 automatic consistency checking, open the Registry and, within the **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services WINS\Parameters key**, create a key called **ConsistencyCheck**.

Within the **ConsistencyCheck** key, you may create the following values:

* TimeInterval
* SpTime
* MaxRecsAtATime
* UseRplPnrs

```
Name: TimeInterval
Type: REG_DWORD
Value: No Of Seconds
> Description: Specifies the time interval at which WINS will do a consistency check.
Default: 24 hours

Name: SpTime
Type: REG_SZ
Value: hh:mm:ss
Description: Specifies the specific time in hh:mm:ss format at which the first consistency check will be done. Thereafter, it will be done periodically at TimeInterval seconds.
Default:</b> 2:00:00 (2 AM).

Name: MaxRecsAtATime
Type: REG_DWORD
Value: No Of Recs
Description: The maximum number of records that will be replicated in one consistency check cycle. WINS does consistency checks on the records of each owner of WINS. After checking one owner it either goes on to the next one in its list of owners or stops based on the MaxRecsAtATime value.
Default: 30000

Name: UseRplPnrs
Type: DWORD 0 or non-zero value
Description: If set to a non-zero value, WINS will contact only its pull partners when doing consistency checks on the records in its database. If the owner whose records need to be checked is a Pull Partner, it will be used. Otherwise, a random pull partner will be used.
```

## WINS Disaster Recovery

As is usually the case, the best way to prevent a disaster is to plan for a disaster. If the WINS clients are properly configured with primary and secondary WINS servers and the primary server fails, they will automatically switch to the secondary server. If either the primary or secondary WINS servers remain functional, the WINS clients will be able to perform address registration and address resolution.

However, if you need to reinstall or replace a WINS server, the most important information you must have available is documentation of your servers’ replication patterns. Unless you properly re-establish your replication partners after returning your WINS server to operation, your WINS registrations will not properly replicate.

If you are not able to return the WINS server to service with its database intact, you have several options to regenerate the WINS database.  If the replication partners are properly configured, the partners can push the database back to the reconstructed WINS server.

Since the WINS clients dynamically register with the WINS server, the WINS database can easily be recovered by having your workstations reboot and reregister. While this disrupts your normal user operations, it is not disastrous.

## Planning and Documenting Your Replication Pattern

In any multiple WINS server environment, your most critical information will be your replication pattern. The WINS database can be quickly restored to a server or a new WINS server created and clients reregistered, but if the server cannot quickly be restored to the replication pattern, it is useless. Remember that other WINS servers in the pattern are receiving registrations and this information must be provided to the recovered WINS server's clients.

Once a replication pattern is finalized, that pattern should be documented with both diagrams and written documentation. For each WINS server, you should include the following:

* Push and pull partners
* Push partner update counts
* Pull partner start time and replication interval

Figure 14.10 Sample WINS replication documentation

Server: | Partner Name | Push partner | Pull partner | *
---|---|---|---|---
 | | Replication count: | Start time: | Interval time:
NYC | | | |
 | LA |500 | 1:00 | 0:30:00
  | Seattle | 500 | 1:00 | 0:30:00
LA | | | |
 | NYC | 500 | 1:10 | 0:30:00
  | Seattle | 500 | 1:10 | 0:30:00
Seattle | | | |
 | LA | 500 | 1:20 | 0:30:00
  | NYC | 500 | 1:20 | 0:30:00


This information is available from the WINS Manager or directly from the Registry. The critical Registry information is stored in **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Wins\Partners**.

One strategy that is frequently used in WINS systems is a dedicated online WINS backup server. This server accepts push replications from all the hub servers, but does not otherwise replicate or accept registrations.

## Regenerating Servers and Reregistering Clients

As noted earlier, WINS clients can easily be rebooted to register with a new WINS server. If the primary WINS server is down, they will register with their secondary WINS server. When the primary is returned to server, any registration differences between the primary and the secondary will be resolved during WINS replication.

However, if a WINS server is permanently removed from service, the WINS client configuration must be modified on all WINS clients. Manually changing the configuration can be time consuming. If the clients are configured to use DHCP, then the DHCP scope configuration can be modified to supply information on the new WINS servers.

Remember that the DHCP clients only retrieve DHCP configuration information when they renew their leases. They will continue to use the old information until either the lease renews, typically half way through the lease period, or the address is manually renewed using the **IPCONFIG** command on the workstation.

## Recovering the Existing Database

Remember also that each WINS server contains a full copy of the WINS database. This database can be backed up and, if necessary, restored from another WINS server. Chapter 12 provides instructions on how to backup and restore a WINS database in the section "Backup and Restoration."

## NT Resource Kit Tools

The NT Resource Kit provides several command line tools to supplement the WINS Manager. These utilities provide functionality that is either not available in WINS Manager or difficult to accomplish from WINS Manager.

The tools are:

* WINSCL: WINS server administration
* WINSDMP: dump a WINS server’s database
* WINSCHK: consistency check

### WINSCL.EXE

WINSCL, which requires WINS administrative permissions to run, provides a command-line utility for managing your WINS servers. Most activities that can be performed using the WINS Manager program can be performed from the command prompt using WINSCL. Unlike WINS Manager, WINSCL cannot be used to configure a WINS server or configure WINS replication partners.

WINSCL can perform many activities, including:

* Monitor WINS activities
* Examine WINS databases
* Initiate replication or scavenging
* Register or query a name
* Backup or restore a database

The version of WINSCL included in the NT 4.0 Resource Kit also offers the capability to perform consistency checking on the WINS database.

> Consistency checking the WINS database results in a full replication of the WINS database. This operation may generate a large amount of network traffic and result in performance degradation for all other network users, as well as reduced WINS name resolution performance.

When using WINSCL to delete a machine's name registrations from the database, you must specifically remove all registered names. For example, the workstation LOKI would register LOKI[00], LOKI[20], and LOKI[03].

To delete all three entries, you would run the command:

```
WINSCL T
```

When prompted, enter the WINS server's IP address. When prompted for a command, enter

```
DN LOKI
```

 WINSCL will prompt you to enter 1 to select the sixteenth character or 0 to not enter the sixteenth character. Enter 1, then enter the sixteenth character: 20.

Repeat this process for the registered names LOKI[00] and LOKI[03].

The names are case sensitive and, if the wrong case is used, WINSCL may indicate that the operation was successfully completed even if the operation failed. To verify that a registered name was properly deleted, refresh and view the WINS database

### WINSDMP.EXE

WINSDMP is a command-line utility that allows you to dump the entire WINS database to a standard comma delimited (**.csv**) format. By default, the program will dump the database to the screen, however the output can be redirected to a text file using standard pipe commands.

For example:

```
WINSDMP ZEUS c:\data\winsdump.txt
```

would connect to the WINS server ZEUS, dump its database, and save the contents to the file **c:\data\winsdump.txt**. The WINS server's IP address can be used instead of its name. The fields of the dumped file, in order, are as follows:

1. The IP address of the WINS server which owns the record
2. Registered name
3. Sixteenth character in hexidecimal
4. Name length
5. Type of record (unique, multihomed, special group, normal group)
6. State of the record (active, released, tombstone)
7. Version ID (high order word)
8. Version ID (low order word)
9. Static or dynamic flag
10. Time stamp
11. Number of IP addresses in the record
12. List of IP addresses

### WINSCHK

WINSCHK provides WINS database consistency checking, supplementing WINSCL with the capability to optionally flag possible causes of database inconsistencies. Possible causes may include asymmetrical replication topologies or a high rate of communications failures.

WINSCHK also allows you to perform the following operations:

* Check name and version number inconsistencies in the databases
* Monitor replication activity
* Verify replication topology


WINSCHK can be run either interactively , scheduled process, or as a background process. If scheduled, it can log any errors found to the file WINSTST.LOG. If run as a background process, it will log its activities to MONITOR.LOG.
