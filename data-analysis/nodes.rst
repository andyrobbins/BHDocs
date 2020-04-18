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

Execution Privileges
--------------------

Outbound Object Control
-----------------------

Inbound Object Control
----------------------

Words about user nodes
----------------------

Groups
^^^^^^

Words about group nodes

Computers
^^^^^^^^^

Words about computer nodes
