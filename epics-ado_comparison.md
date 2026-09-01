# Difference between EPICS and ADO infrastructures
Created by Andrei Sukhanov.

### Naming convention
- EPICS: Flat.<br>
Disadvantage: The flat naming for large systems requires strict policy for naming convention and additional tools like ChannelFinder.
- ADO: Device:Parameter.<br>
Advantage1: Naming convention is intrinsically device-centric. No additional naming convention required.<br>
Advantage2: ADO parameters have aliases, they can be addressed with system or generic name.

### Introspection
Ability to find information about device features.
- EPICS:<br>
Channel Access lacks introspection completely. PVAccess supports limited introspection, it is based on server IP address, not device name. For large system ChannelFinder is needed.
- ADO:<br>
Introspection is supported intrinsically. Very useful tool provided: adoPet.

### Name service
- EPICS:<br>
An external tool ChannelFinder, requires many pre-requisites. To support ChannelFinder, device need to host a RecSync client to expose its properties. This complicates the structure of the IOC. It is not clear if ChannellFinder could be in sync with Python-based IOCs (e.g. p4p).
- ADO:<br>
The cnslookup is integral part of the ADO framework. There is no need for tools like ChannelFinder. The cnslookup requires central database which is core part of the ADO infrastructure. Interface between devices and database is simple, it is built into the communication protocol.

### Communication protocol
- EPICS:<br>
Both Channel Access and PVAccess protocols are custom. The PVAccess is very complicated. Due to its complexity, the pure-Python implementation of the PVAccess still does not exist.
PVaccess supports RPC-type requests, this functionality does not seem to be very demanding as it can be implemented using standard requests.
- ADO:<br>
The communication protocol is based on RPC standard, and it is using very small subset of the RPC features. The protocol is less complex as EPICS. There exist pure-Python and Java implementations of the protocol. Most of the recent ADO managers are written in Python.<br>
The RPC relies on rpcbind system service. For systems where rpcbind is prohibited the internal program ID resolution is provided.

### Precursors for secure access control
- EPICS:<br>
Protocol does not propagate user ID and program ID to server.
- ADO:<br>
User ID and Program ID of the client are available to server as part of client request.
This simplifies the implemetation of secure access control.

### Parameter properties
- EPICS:<br>
The concept of DB records is counter-intuitive and unnecessary complicated. Some of its features (e.g. communication between DB records) are rarely used in modern applications.
- ADO:<br>
ADO parameters follows the simple rules: parameter has properties, the properties are exposed for inquiry and modifications.

### Server executable
- EPICS:<br>
The IOC is build with an internal shell and a state machine. They seems to be unnecessary complications.

### Data logging
- EPICS:<br>
Archiver Apliances. Set of custom tools.
- ADO:<br>
Set of custom tools and scripts. Provides similar functionality as Archiver Apliances. The data extraction tool is Java-based DataServer.<br>
Typical data volumes, Run24: **280 GB of compressed data per day, 300 TB per year**.<br>
Retrieval time for years-old data from long term data storage: 1-2 hours.
Disadvantage1: Data ingestion processes are based on outdated SDDS file format.
Disadvantage2: There is no data storage policy, all logged data a get stored in long term storage system. 
- Alternative:<br>
Consider to deploy an off-the-shelf time series database like **TDengine or InfluxDB**.

### GUI
- EPICS:<br>
Advantage: The CS Studio make it possible to create impressive GUI. 
Disadvantage: There is no simple tools like Gpm, adoPet, Pet.
- ADO:
Disadvantage. The Pet interface is very limiting.
The impressive GUI can be created using Python and QT, but this requires basic Qt expertise.
 
### Central Database.
- EPICS:<br>
There is no central database in EPICS.
- ADO:<br>
The database is core component of the ADO infrastructure.
The database is currently SYBASE, which is planned to be replaced by PostgreSQL or MySQL.


### Set History
Keeping track of who and when modified sensitive parameters.
- ADO:
Modification of all sensitive parameters is recorded and information on who and when have modified is stored in the database. It is part of the client API. This feature is very useful for system and device recovery.

### Parameter cache
When restarted, each device recovers to previous state.
- ADO:
Every time the critical parameter gets modified, its value is written to a file (parameter cache). When device get restarted, all critical parameters are recovered from the file. The critical parameters have special feature bit.



