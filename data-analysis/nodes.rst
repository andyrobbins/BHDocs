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

* `USERNAME@DOMAIN.COM`: the UPN formatted name of the user, where
  USERNAME is the SAM Account Name, and DOMAIN.COM is the fully
  qualified domain name of the domain the user is in.
* Sessions: The count of computers this user has been observed
  logging onto. Click this number to visually see the connections
  between those computers and this user
* Sibling Objects in the Same OU: the number of other AD users, groups,
  and computers that belong to the same OU as this user. This can be
  very helpful when trying to figure out the lay of the land for an
  environment
* Reachable High Value Targets: The count of how many high value
  targets this user has an attack path to. A high value target is by
  default any computer or user that belongs to the domain admins,
  domain controllers, and several other high privilege Active Directory
  groups. Click this number to see the shortest attack paths from this user
  to those high value targets.
* Effective Inbound GPOs: the count of GPOs that apply to this user.
  Click the number to see the GPOs and how they apply to this user.
* See user within Domain/OU Tree: click this to see where the user
  is placed in the OU tree. This can give you insights about the
  geographic location of the user as well as organizational placement
  of the actual person.

Node Properties
---------------

Extra Properties
----------------

Group Membership
----------------

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
