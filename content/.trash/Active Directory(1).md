Q: What is Active Directory (Domain Services)
- stores all domain objects (user acc, groups, comp. acc)
- searchable directory
- method to apply config/sec settings for obj
A: ADDS is the current iteration of Active Directory that can be administered through Azure. 

## What are the logical components?

|**Logical component**|**Description**|
|---|---|
|Partition|A partition, or naming context, is a portion of the AD DS database. Although the database consists of one file named Ntds.dit, different partitions contain different data. For example, the schema partition contains a copy of the Active Directory schema. The configuration partition contains the configuration objects for the forest, and the domain partition contains the users, computers, groups, and other objects specific to the domain. Active Directory stores copies of partitions on multiple domain controllers and updates them through directory replication.|
|Schema|A schema is the set of definitions of the object types and attributes that you use to define the objects created in AD DS.|
|Domain|A domain is a logical administrative container for objects such as users and computers. A domain maps to a specific partition and you can organize the domain with parent-child relationships to other domains.|
|Domain tree|A domain tree is a hierarchical collection of domains that share a common root domain and a contiguous Domain Name System (DNS) namespace.|
|Forest|A forest is a collection of one or more domains that have a common AD DS root, a common schema, and a common global catalog.|
|OU|An OU is a container object for users, groups, and computers that provides a framework for delegating administrative rights and administration by linking Group Policy Objects (GPOs).|
|Container|A container is an object that provides an organizational framework for use in AD DS. You can use the default containers, or you can create custom containers. You can't link GPOs to containers.|

## What are the physical components?

Physical components in AD DS are those objects that are tangible, or that describe tangible components in the real world.

|**Physical component**|**Description**|
|---|---|
|Domain controller|A domain controller contains a copy of the AD DS database. For most operations, each domain controller can process changes and replicate the changes to all the other domain controllers in the domain.|
|Data store|A copy of the data store exists on each domain controller. The AD DS database uses Microsoft Jet database technology and stores the directory information in the Ntds.dit file and associated log files. The C:\Windows\NTDS folder stores these files by default.|
|Global catalog server|A global catalog server is a domain controller that hosts the global catalog, which is a partial, read-only copy of all the objects in a multiple-domain forest. A global catalog speeds up searches for objects that might be stored on domain controllers in a different domain in the forest.|
|Read-only domain controller (RODC)|An RODC is a special, read only installation of AD DS. RODCs are common in branch offices where physical security is not optimal, IT support is less advanced than in the main corporate centers, or line-of-business applications need to run on a domain controller.|
|Site|A site is a container for AD DS objects, such as computers and services that are specific to a physical location. This is in comparison to a domain, which represents the logical structure of objects, such as users and groups, in addition to computers.|
|Subnet|A subnet is a portion of the network IP addresses of an organization assigned to computers in a site. A site can have more than one subnet.|


Q: What are ADDS forests and domains?
- hint
- 