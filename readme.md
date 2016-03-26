---
title: Browsing and WINS
Layout: book
---
# Broadcast Browsing Address Resolution

## Chapter 8: Broadcast Browsing:

### Client Traffic in Browsing Environments

The workstation's browse client performs several routine tasks critical to successful browsing. When the workstation boots, the browse client registers with the browse master and locates backup browsers, which it later uses when locating other machines on the network. Without these normal browse client actions, the workstation can not locate other machines or be located by other machines.

### Server Traffic in Browsing Environment

Before a workstation can either register or locate other machines, potential browse servers must agree on what role they will play. Potential browse servers must identified, browse masters elected, and browse lists replicated. Until this hierarchy is established, workstations can not locate any network resources.

### Elections

A network contains several browse servers and potential browse servers. Before workstation registrations can be accepted, browse lists compiled, or workstation names resolved, a browse master must be elected on each subnet to control the entire browsing process.

### Tuning the Browsing Network

Robust, reliable browse servers are critical to proper browsing. The primary consideration is knowing how to identify potential browse servers and prioritize them for proper operation. Reliable browse servers reduce network traffic and improve address resolution.

### Browsing in WFW Environments

Many legacy environments still contain Windows for Workgroups workstation. While WFW was not originally designed for browsing, subsequent performance enhancements have added this support. If you have WFW workstation on your network, you must understand how WFW supports browsing as well as potential problems you may face.

### Browsing in Windows 95 Environments

Windows 95 incorporates full browse client and server functionality. Since most corporations have deployed Windows 95 as their standard workstation, you must understand how to properly configure the client and server components.

### Browsing in Windows NT Environments

Windows NT introduced support for WAN wide browsing, and continues to provide the cornerstone of a properly functioning browsing network. Any WAN wide browsing must have an NT primary domain controller and, for stability, should contain NT browse servers. This section describe the extensive NT browsing configuration options.


## Chapter 9: Complete WINS Environment

### WINS client traffic

WINS uses central registration servers which are known to all workstations. Although the browse clients no longer must broadcast to locate browse servers, the client must still register with the server.

### WINS database replication

WINS uses stable and designated servers, rather than relying on elections. For this reason, the network traffic and complexity associated with server announcements, elections, and browse list replication no longer exist. You must understand, however, how WINS servers communicate and replicate their databases.

### Legacy operating systems

Windows 95 and NT 4.0 fully support WINS, however, older operating systems do not. You must understand the limitations on these operating systems and how to integrate them with your WINS systems.

### Multihomed machines

Machines with multiple networ   cards present unique connectivity problems. Microsoft networking, which is built around NetBIOS, assumes that each network name has a single network address associated with it. While broadcast browsing doesn’t support multihomed machines, WINS does support these configurations.

## Chapter 10: Partial WINS Environment

### Browsing in a Novell Environment

Browsing in Novell environments presents special problem, primarily because of Novell and NT integration issues. However, the IPX protocols can also introduce routing issues.

### Browsing Across Remote and slow links

Access from remote locations causes a number of problems. When using either terminal servers or routers with slow links, you must plan support for slow browse server links. When using RAS, browsing does not function at all, but alternate address resolution methods are used.

### LMHOSTS Files

Browsing was originally intended for local area networks, where bandwidth is readily available for browsing, elections, and browse list replication. With remote links, unreliable connectivity, or other networking problems, you may not be able to support a normal browsing environment. LMHOSTS files provide support for these environments.

## Chapter 11: Troubleshooting Browsing

### Architectural Problems

This section will identify theoretical and design problems and solutions that can proactively correct problems.

### Election Wars

Most browse server problems are corrected when elections result in better browse servers, but sometimes this isn't the case. This section covers how to identify problem workstations and servers and correct the problems.

### Workstation Connectivity Problems

Most authentication and connectivity problems are caused when workstations can’t locate other computers. This section covers how to identify and correct these problems.

### Problems Browsing Novell Servers and IPX Networks

This section covers browsing issues unique to Novell and IPX networks, including problems locating and connecting to Novell servers, SAP storms, and direct hosting.

# WINS Address Resolution

## Chapter 12: WINS Design

### WINS server configuration

Designated WINS server must have the processor, memory, and network bandwidth to provide adequate response times to the WINS clients. While a WINS server does not need to be dedicated to the role, some services should not be run on the WINS server.

### Replication

For stability and to improve response times, you should have several WINS servers for your clients. You must configure your WINS servers to communicate and replicate their database information to each other. Choosing the proper replication pattern is critical to your server's stable operation.

### Regular Maintenance

Regular maintenance is critical to WINS server operation. The databases must be compressed, backed up, and checked for consistency.

### Migrating WINS servers

As your WINS environment grows, you should know how to migrate these servers to more capable machines.

## Chapter 13: WINS Integration

### Machine Naming considerations

The primary integration consideration between Windows networking and other TCP/IP machines is the difference in naming conventions. While DNS supports a hierarchical naming structure, Microsoft networking uses a flat directory structure.

### DNS

While DNS and WINS provide similar functionality, they have major differences. This section discusses how DNS operates and how it is configured, as well as some of the issues which may arise when integrating WINS and DNS.

### Samba

Samba provides Microsoft networking for UNIX-based machines. Once the Samba server is installed and running, it provides disk and print shares, as well as browse server support. This section discusses configuration issues and possible problems that arise when using Samba.

### DHCP

The primary advantage of WINS is its integration with DHCP for dynamically assigned addresses. Since DHCP typically drives the requirements for WINS and DNS integration, we will discuss how DHCP operates and how it integrates with WINS.

### WINS/DNS Integration

This section discusses WINS and DNS integration strategies to provide seamless address resolution for both Microsoft networking and UNIX systems.

## Chapter 14: WINS Troubleshooting

### Isolating WINS problems

Before you can identify and correct the problems associated with WINS servers, you must be able to identify the cause of your problems. This section assists you in isolating your WINS problems.

### Removing Bad WINS servers

Sometimes a WINS server's database becomes so corrupt that the simplest recovery is to switch the clients to another server and completely remove the server from operation.

### Recovering WINS database

Because each WINS server contains the complete database, any single server failure can be recovered by restoring the database from another WINS server.

### WINS disaster recovery

While the WINS database can be regenerated by having the clients reregister with the servers, you must plan for server recovery. This section tells you what you should back up and how, as well as how to implement the new servers.

### Resource Kit Utilities

The NT Resource Kits provide several utilities for troubleshooting and correcting WINS problems. This section discusses the utilities, their purpose, and how to use them.
