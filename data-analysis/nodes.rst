Nodes
=====

Nodes represent principals and other objects in Active Directory.
BloodHound stores certain information about each node on the node
itself in the neo4j database, and the GUI automatically performs
several queries to gather insights about the node, such as how
privileged the node is, or which GPOs apply to the node, etc. Simply
click the node in the BloodHound GUI, and the "Node Info" tab will
populate with all that information for the node.

Users
^^^^^

At the top of the node info tab you will see the following info:

* **USERNAME@DOMAIN.COM**: the UPN formatted name of the user, where
  USERNAME is the SAM Account Name, and DOMAIN.COM is the fully
  qualified domain name of the domain the user is in.
* **Sessions**: The count of computers this user has been observed
  logging onto. Click this number to visually see the connections
  between those computers and this user
* **Sibling Objects in the Same OU**: the number of other AD users, groups,
  and computers that belong to the same OU as this user. This can be
  very helpful when trying to figure out the lay of the land for an
  environment
* **Reachable High Value Targets**: The count of how many high value
  targets this user has an attack path to. A high value target is by
  default any computer or user that belongs to the domain admins,
  domain controllers, and several other high privilege Active Directory
  groups. Click this number to see the shortest attack paths from this user
  to those high value targets.
* **Effective Inbound GPOs**: the count of GPOs that apply to this user.
  Click the number to see the GPOs and how they apply to this user.
* **See user within Domain/OU Tree**: click this to see where the user
  is placed in the OU tree. This can give you insights about the
  geographic location of the user as well as organizational placement
  of the actual person.

Node Properties
---------------

* **Display name**: The Active Directory display name for the user
* **Object ID**: The user's SID. In neo4j this is stored as the user's
  objectid to uniquely identify the node
* **Password Last Changed**: The human-readable date for when the user's
  password last changed. This is stored internally in Unix epoch format
* **Last Logon**: The last time the domain controller you got this data from
  handled a logon request for the user
* **Last Logon (Replicated)**: The last time any domain controller handled
  a logon for this user
* **Enabled**: Whether the user object is enabled in Active Directory. Fun
  fact: if you control a disabled user object, you can re-enable that
  user object.
* **AdminCount**: Whether the user object in Active Directory currently,
  or possibly ever has belonged to a certain set of highly privileged
  groups. This property is related to the AdminSDHolder object and the
  SDProp process. Read about that here: https://adsecurity.org/?p=2053
* **Compromised**: Whether the user is marked as Owned. You can mark any
  user in the BloodHound GUI as Owned by right-clicking it and clicking
  "Mark User as Owned".
* **Password Never Expires**: Whether the UAC flag is set for the user in
  Active Directory to not require the user to update their password
* **Cannot Be Delegated**: Whether the UAC flag is set on the user in 
  Active Directory to disallow kerberos delegation for this user. If
  this is "True", then the user cannot be abused as part of a kerberos
  delegation attack
* **ASREP Roastable**: Whether the user can be ASREP roasted. For more info
  about that attack, see https://github.com/GhostPack/Rubeus#asreproast


Extra Properties
----------------

This section displays some other information about the node, plus all other
non-default, string-type property values from Active Directory if you used
the --CollectAllProperties flag. The default properties you'll see here
include:

* **distinguishedname**: The distinguished name (DN) of the user
* **domain**: The FQDN of the domain the user is in
* **name**: The UPN-formatted name of the user
* **passwordnotreqd**: Whether the UAC flag is set on the user object to
  not require the user to have a password. Note that this does not
  necessarily mean the user does *not* have a password, just that the user
  is allowed to not have one
* **unconstraineddelegation**: Whether the user is allowed to perform
  unconstrained kerberos delegation. See more info about that here:
  https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/

Group Membership
----------------

This section displays stats about Active Directory security grops the user
belongs to:

* **First Degree Group Memberships**: AD security groups the user is
  directly added to. If you typed `net user david.mcguire /domain`, for
  example, these are the groups you'd see this user belonging to.
* **Unrolled Group Membership**: Groups can be added to groups, and those
  group nestings can grant admin rights, control of AD objects, and other
  privileges to many more users than intended. These are the groups that
  this user effectively belongs to, because the groups the user explicitly
  belongs to have been added to those groups.
* **Foreign Group Membership**: Groups in other Active Directory domains
  this user belongs to

Local Admin Rights
------------------

* **First Degree Local Admin**: The number of computers that this user
  itself has been added to the local administrators group on. If you were
  to type `net localgroup administrators` on those systems, you would see
  this user in the list
* **Group Delegation Local Admin Rights**: AD security groups can be added
  to local administrator groups. This number shows the number of computers
  this user has local admin rights on through security group delegation,
  regardless of how deep those group nestings may go
* **Derivative Local Admin Rights**: This query does not run by default
  because it's a very expensive query for neo4j to run. If you press the play
  button here, neo4j will run the query and return the number of computers
  this user has "derivative" local admin rights on. For more info about
  this concept, see http://www.sixdub.net/?p=591

Execution Privileges
--------------------

* **First Degree RDP Privileges**: The number of computers where this user
  has been added to the local Remote Desktop Users group.
* **Group Delegated RDP Privileges**: The number of computers where this user
  has remote desktop logon rights via security group delegation
* **First Degree DCOM Privileges**: The number of computers where this user
  has been added to the local Distributed COM Users group
* **Group Delegated DCOM Privileges**: The number of computers where this
  user has group delegated DCOM rights
* **SQL Admin Rights**: The number of computers where this user is very
  likely granted SA privileges on an MSSQL instance. This number is inferred
  by the number of computers listed on the user's serviceprincipalnames
  attribute where an MSSQL instance is referenced
* **Constrained Delegation Privileges**: The number of computers that trust
  this user to perform constrained delegation. This number is inferred by
  insepecting the msDS-AllowedToDelegateTo property on the user object in
  Active Directory and getting a count for how many computers are listed
  in that attribute

Outbound Object Control
-----------------------

* **First Degree Object Control**: The number of objects in AD where this
  user is listed as the IdentityReference on an abusable ACE. In other words,
  the number of objects in Active Directory that this user can take control
  of, without relying on security group delegation
* **Group Delegated Object Control**: The number of objects in AD where this
  user has control via security group delegation, regardless of how deep those
  group nestings may go
* **Transitive Object Control**: The number of objects this user can gain control
  of by performing ACL-only based attacks in Active Directory. In other words,
  the maximum number of objects the user can gain control of without needing
  to pivot to any other system in the network, just by manipulating objects
  in the directory

Inbound Object Control
----------------------

* **Explicit Object Controllers**: The number of principals that are listed
  as the IdentityReference on an abusable ACE on this user's DACL. In other
  words, the number of users, groups, or computers that directly have control
  of this user
* **Unrolled Object Controllers**: The *actual* number of principals that have
  control of this object through security group delegation. This number can
  sometimes be wildly higher than the previous number
* **Transitive Object Controllers**: The number of objects in AD that can achieve
  control of this object through ACL-based attacks

Groups
^^^^^^

At the top of the node info tab you will see the following info:

* **GROUPNAME@DOMAIN.COM**: The UPN formatted name of the security group, where
  GROUPNAME is the group's SAM Account Name, and DOMAIN.COM is the fully qualified
  name of the domain the group is in
* **Sessions**: The number of computers that users belonging to this group have
  been seen logging onto. This will include users that belong to this group through
  any number of nested memberships. Very useful for targetting users that belong
  to a particular security group
* **Reachable High Value Targets**: The count of how many high value targets this
  group (and therefore the users belonging to this group) has an attack path to.
  A high value target is by default any computer or user that belongs to the domain
  admins, domain controllers, and several other high privilege Active Directory
  groups. Click this number to see the shortest attack paths from this user to
  those high value targets.

Node Properties
---------------

* **Object ID**: The SID of the group. The group's SID is stored internally as its
  objectid
* **Description**: The contents of the description field for the group in Active
  Directory.
* **Admin Count**: Whether the group object in Active Directory currently, or
  possibly ever has belonged to a certain set of highly privileged groups. This
  property is related to the AdminSDHolder object and the SDProp process. Read
  about that here: https://adsecurity.org/?p=2053

Extra Properties
----------------

This section displays some other information about the node, plus all other
non-default, string-type property values from Active Directory if you used the
–CollectAllProperties flag. The default properties you’ll see here include:

* **distinguishedname**: The distinguished name (DN) of the group
* **domain**: The FQDN of the domain the group belongs to
* **name**: The UPN formatted name of the group

Group Members
-------------

* **Direct Members**: The number of principals that have been directly added to
  this group. If you typed `net group GROUPNAME /domain`, these are the
  principals you would see in that output
* **Unrolled Members**: The actual number of users that effectively belong to
  this group, no matter how many layers of nested group membership that goes
* **Foregin Members**: The number of users from other domains that belong to this
  group

Group Membership
----------------

* **First Degree Group Membership**: The number of groups this group has been
  added to
* **Unrolled Member Of**: The number of groups this group belongs to through
  nested group memberships
* **Foreign Group Membership**: Groups in other domains this group has been added
  to

Local Admin Rights
------------------

* **First Degree Local Admin**: The number of computers this group itself has been
  added to the local administrators group on
* **Group Delegated Local Admin Rights**: The number of computers this group (and
  the members of this group) has admin rights on via nested group memberships
* **Derivative Local Admin Rights**: This query does not run by default because
  it’s a very expensive query for neo4j to run. If you press the play button here,
  neo4j will run the query and return the number of computers this group has
  “derivative” local admin rights on. For more info about this concept, see
  http://www.sixdub.net/?p=591

Execution Privileges
--------------------

* **First Degree RDP Privileges**: The number of computers where this group has
  been added to the local Remote Desktop Users group.
* **Group Delegated RDP Privileges**: The number of computers where this group has
  remote desktop logon rights via security group delegation
* **First Degree DCOM Privileges**: The number of computers where this group has
  been added to the local Distributed COM Users group
* **Group Delegated DCOM Privileges**: The number of computers where this group has
  group delegated DCOM rights

Outbound Object Control
-----------------------

* **First Degree Object Control**: The number of objects in AD where this group is
  listed as the IdentityReference on an abusable ACE. In other words, the number of
  objects in Active Directory that this group can take control of, without relying
  on security group delegation
* **Group Delegated Object Control**: The number of objects in AD where this group
  has control via security group delegation, regardless of how deep those group
  nestings may go
* **Transitive Object Control**: The number of objects this group can gain control
  of by performing ACL-only based attacks in Active Directory. In other words, the
  maximum number of objects the group can gain control of without needing to pivot
  to any other system in the network, just by manipulating objects in the directory

Inbound Object Control
----------------------

* **Explicit Object Controllers**: The number of principals that are listed as the
  IdentityReference on an abusable ACE on this group’s DACL. In other words, the
  number of users, groups, or computers that directly have control of this group
* **Unrolled Object Controllers**: The actual number of principals that have control
  of this object through security group delegation. This number can sometimes be
  wildly higher than the previous number
* **Transitive Object Controllers**: The number of objects in AD that can achieve
  control of this object through ACL-based attacks

Computers
^^^^^^^^^

At the top of the node info tab you will see the following info:

* **COMPUTERNAME.DOMAIN.COM**: The fully qualified name of the computer
* **Sessions**: The total number of users that have been observed logging onto this
  computer
* **Reachable High Value Targets**: The count of how many high value targets this
  computer has an attack path to. A high value target is by default any computer or
  user that belongs to the domain admins, domain controllers, and several other high
  privilege Active Directory groups. Click this number to see the shortest attack
  paths from this computer to those high value targets
* **Sibling Objects in the Same OU**: the number of other AD users, groups, and
  computers that belong to the same OU as this computer. This can be very helpful
  when trying to figure out the lay of the land for an environment
* **Effective Inbound GPOs**: the count of GPOs that apply to this computer. Click
  the number to see the GPOs and how they apply to this computer
* **See Computer within Domain/OU Tree**: click this to see where the computer is
  placed in the OU tree. This can give you insights about the geographic location of
  the computer as well as the purpose and function of the computer

Node Properties
---------------

* **Object ID**: The SID of the computer. We store this in neo4j as the computer's
  objectid to uniquely identify the node
* **OS**: The operating system running on the computer, according to the corresponding
  property on the computer object in Active Directory
* **Enabled**: Whether the computer object is enabled
* **Allows Unconstrained Delegation**: Whether the computer is trusted to perform
  unconstrained delegation. By default, all domain controllers are trusted for this
  style of kerberos delegation. For information about the abuse related to this
  configuration, see https://www.harmj0y.net/blog/redteaming/another-word-on-delegation/
* **Compromised**: Whether the computer is marked as Owned. You can mark any computer in
  the BloodHound GUI as Owned by right-clicking it and clicking “Mark Computer as Owned”.
* **LAPS Enabled**: Whether LAPS is running on the computer. This is determined by
  checking whether the associated MS LAPS properties are populated on the computer
  object
* **Password Last Changed**: The human readable time for when the computer account's
  password last changed in Active Directory
* **Last Logon (Replicated)**: The last time any domain controller handled a logon
  for this computer. In other words, the last time the computer authenticated to the
  domain

Extra Properties
----------------

This section displays some other information about the node, plus all other non-default,
string-type property values from Active Directory if you used the –CollectAllProperties
flag. The default properties you’ll see here include:

* **distinguishedname**: The distinguished name (DN) of the computer
* **domain**: The fully qualified name of the domain the computer is in
* **name**: The FQDN of the computer
* **serviceprincipalnames**: The list of SPNs on the computer. Very useful for determining
  any non-default services that may be running on the computer, such as MSSQL

Local Admins
------------

* **Explicit Admins**: The count of principals that have been directly added to the local
  administrators group on the computer. If you typed `net localgroup administrators` on
  the computer, these are the principals you would see listed in that output
* **Unrolled Admins**: The real number of principals that have local admin rights on this
  computer via nested group memberships
* **Foreign Admins**: The number of users from other domains that have admin rights on
  this computer
* **Derivative Local Admins**: The count of users that can execute an attack path relying
  on admin rights and token theft to compromise this system. For more information about
  this attack, see http://www.sixdub.net/?p=591

Inbound Execution Privileges
----------------------------

* **First Degree Remote Desktop Users**: The number of principals that have been granted
  RDP rights to this system by being added to the local Remote Desktop Users group
* **Group Delegated Remote Desktop Users**: The real number of users that have RDP access
  to this system through nested group memberships
* **First Degree Distributed COM Users**: The number of principals added to the local
  Distributed COM Users group
* **Group Delegated Distributed COM Users**: The number of users with DCOM access to this
  system through nested group memberships
* **SQL Admins**: The number of users that have SA privileges on an MSSQL instance running
  on this system. This is determined by inspecting the serviceprincipalname attribute on
  user objects in AD

Group Membership
----------------

* **First Degree Group Memberships**: AD security groups the computer is directly added to.
* **Unrolled Group Membership**: The number of groups this computer belongs to through
  nested group memberships
* Foreign Group Membership: Groups in other Active Directory domains this computer belongs
  to

Local Admin Rights
------------------

* **First Degree Local Admin**: The number of computers that this computer itself has been
  added to the local administrators group on.
* **Group Delegation Local Admin Rights**: This number shows the number of computers this
  computer has local admin rights on through security group delegation, regardless of how
  deep those group nestings may go
* **Derivative Local Admin Rights**: This query does not run by default because it’s a very
  expensive query for neo4j to run. If you press the play button here, neo4j will run the
  query and return the number of computers this computer has “derivative” local admin rights
  on. For more info about this concept, see http://www.sixdub.net/?p=591

Outbound Execution Privileges
-----------------------------

* **First Degree RDP Privileges**: The number of computers where this computer has been
  added to the local Remote Desktop Users group.
* **Group Delegated RDP Privileges**: The number of computers where this computer has remote
  desktop logon rights via security group delegation
* **First Degree DCOM Privileges**: The number of computers where this computer has been added
  to the local Distributed COM Users group
* **Group Delegated DCOM Privileges**: The number of computers where this computer has group
  delegated DCOM rights
* Constrained Delegation Privileges: The number of computers that trust this computer to
  perform constrained delegation. This number is inferred by insepecting the
  msDS-AllowedToDelegateTo property on the computer objects in Active Directory and getting a
  count for how many computers are listed in that attribute

Inbound Object Control
----------------------

* **Explicit Object Controllers**: The number of principals that are listed as the
  IdentityReference on an abusable ACE on this computer’s DACL. In other words, the number of
  users, groups, or computers that directly have control of this computer
* **Unrolled Object Controllers**: The actual number of principals that have control of this
  object through security group delegation. This number can sometimes be wildly higher than
  the previous number
* **Transitive Object Controllers**: The number of objects in AD that can achieve control of
  this object through ACL-based attacks

Outbound Object Control
-----------------------

* **First Degree Object Control**: The number of objects in AD where this computer is listed as
  the IdentityReference on an abusable ACE. In other words, the number of objects in Active
  Directory that this computer can take control of, without relying on security group delegation
* **Group Delegated Object Control**: The number of objects in AD where this computer has
  control via security group delegation, regardless of how deep those group nestings may go
* **Transitive Object Control**: The number of objects this computer can gain control of by
  performing ACL-only based attacks in Active Directory. In other words, the maximum number of
  objects the computer can gain control of without needing to pivot to any other system in the
  network, just by manipulating objects in the directory

Domains
^^^^^^^

At the top of the node info tab you'll see this information:

* **Users**: The total number of user objects in the domain
* **Groups**: The total number of security groups in the domain
* **Computers**: The total number of computer objects in the domain
* **OUs**: The total number of organizational units in the domain
* **GPOs**: The total number of group policy objects in the domain
* **Map OU Structure**: Click this to see the entire tree structure, including all OUs, users,
  and computers

Node Properties
---------------

* **Object ID**: The SID of the domain. We map this internally in neo4j to a property called
  objectid to uniquely identify the node
* **Domain Functional Level**: The functional level of the Active Directory domain. This becomes
  particularly relevant in certain attack scenarios, such as resource-based constrained
  delegation

Extra Properties
----------------

This section displays some other information about the node, plus all other non-default,
string-type property values from Active Directory if you used the –CollectAllProperties flag. The
default properties you’ll see here include:

* **distinguishedname**: The distinguished name (DN) of the domain head object
* **domain**: The fully qualified name of the domain
* **name**: The name of the domain, this is what is displayed in the node label

Foreign Members
---------------

* **Foreign Users**: Users from other domains that have been added to security groups in this
  domain
* **Foreign Groups**: Groups from other domains that have been added to security groups in this
  domain
* **Foreign Admins**: Users in other domains that have been granted local admin rights on
  computers in this domain
* **Foreign GPO Controllers**: Users in other domains that have been granted control of group
  policy objects in this domain

Inbound Trusts
--------------

* **First Degree Trusts**: The number of other domains that directly trust this domain
* **Effective Inbound Trusts**: The number of other domains that trust this domain through
  trusting other domains that trust this domain. Easier to understand by clicking the number

Outbound Trusts
---------------

* **First Degree Trusts**: The number of domains tha thtis domain directly trusts
* **Effective Outbound Trusts**: The number of domains this domain trusts by trusting other
  domains

Inbound Object Control
----------------------

* **First Degree Controllers**: The number of principals that are listed as an IdentityReference
  on an abusable ACE on the domain head object. In other words, the number of principals that
  have direct control of the domain head. Control of this object is incredibly dangerous, as
  it gives principals the ability to perform the DCSync attack, or grant themselves any
  privileges on any object in the directory
* **Unrolled Controllers**: The real number of principals that have control of the domain head
  through nested security groups
* **Transitive Controllers**: The number of principals that can gain control of the domain head
  by executing an ACL-only attack path, without the need to pivoting to any other computers in
  the domain
* **Calculated Principals with DCSync Privileges**: The number of principals that have the
  DCSync privilege, which is granted with the combination of two specific rights, GetChanges
  and GetChangesAll

GPOs
^^^^

OUs
^^^
