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

* **First Degree Locavl Admin**: The number of computers that this user
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

Words about computer nodes
