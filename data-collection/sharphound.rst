SharpHound
==========

SharpHound is the official data collector for BloodHound. It is written
in C# and uses native Windows API functions and LDAP namespace functions
to collect data from domain controllers and domain-joined Windows systems.

Download the pre-compiled SharpHound binary and PS1 version at 
https://github.com/BloodHoundAD/BloodHound/tree/master/Ingestors

You can view the source code for SharpHound and build it from source
by visiting the SharpHound repo_.

.. _SharpHound repo: https://github.com/BloodHoundAD/SharpHound3

Basic Usage
^^^^^^^^^^^

You can collect plenty of data with SharpHound by simply running the binary
itself with no flags set:

::

   C:\> SharpHound.exe

SharpHound will automatically determine what domain your current user
belongs to, find a domain controller for that domain, and start the
"default" collection method. The default collection method will collect the
following pieces of information from the domain controller:

* Security group memberships
* Domain trusts
* Abusable rights on Active Directory objects
* Group Policy links
* OU tree structure
* Several properties from computer, group and user objects
* SQL admin links

Additionally, SharpHound will attempt to collect the following information
from each domain-joined Windows computer:

* The members of the local administrators, remote desktop, distributed COM,
  and remote management groups
* Active sessions, which SharpHound will attempt to correlate to systems
  where users are interactively logged on

When finished, SharpHound will create several JSON files and place them into
a zip file. Drag and drop that zip file into the BloodHound GUI and the
interface will take care of merging the data into the database.

The Session Loop Collection Method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

BloodHound uses graph theory to find attack paths in Active Directory, and
the more data you have, the more likely you are to find and execute attack
paths successfully. Much of the data you initially collect with SharpHound
will not likely change or require updating over the course of a typical red
team assessment - security group memberships, Active Directory permissions,
and Group Policy links change relatively rarely. That data can be collected
one time, and not again.

User sessions are different for two reasons:

1. Users, especially privileged users, log on and off different systems all
day, every day. How many systems does a typical help desk user or server
admin log into on any given day? 

2. The `way SharpHound's data collection works`_ necessitates scanning the
network several times to get more complete session information. Scannning
the network one time for user sessions may give you between 5 and 15% of
the actual sessions on the network.

.. _way SharpHound's data collection works: https://www.youtube.com/watch?v=q86VgM2Tafc

When you use the path finding function query in BloodHound to find a path
between two nodes and see that there is no path, 9 times out of 10 this is
because BloodHound needs more session data.

SharpHound's Session Loop collection method makes this very easy:

::

   C:\> SharpHound.exe --CollectionMethod Session --Loop

This will run SharpHound's session collection method for 2 hours, generating
a zip file after each loop ends. When done, collect all the zip files and
drag and drop them into the BloodHound GUI.

If you would like to specify a different loop time, use the --Loopduration
flag with the HH:MM:SS format to specify how long you want SharpHound to
perform looped session collection for. For example, if you want SharpHound
to perform looped session collection for 3 hours, 9 minutes and 41 seconds:

::

   C:\> SharpHound.exe --CollectionMethod Session --Loop --Looptime 03:09:41

Running SharpHound from a Non Domain-Joined System
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

While not an officially supported collection method, and not a colletion
method we recommend you do, it is possible to collect data for a domain
from a system that is not joined to that domain. To do so, carefully follow
these steps:

1. Configure your system DNS server to be the IP address of a domain controller
in the target domain.

2. Spawn a CMD shell as a user in that domain using `runas` and its `/netonly`
flag, like so:

::

   C:\> runas /netonly /user:CONTOSO\Jeff.Dimmock cmd.exe

You will be prompted to enter a password. Enter the password and hit enter.

3. A new CMD window will appear. If you type `whoami`, you will not see the
name of the user you're impersonating. This is because of the `/netonly` flag:
the instance of CMD will only authenticate as that user when you authenticate
to other systems over the network, but you are still the same user you were
before when authenticating locally.

4. Verify you've got valid domain authentiation by using the `net` binary:

::

   C:\> net view \\contoso\

If you can see the SYSVOL and NETLOGON folders, you're good.

5. Run SharpHound, using the `-d` flag to specify the AD domain you want to
collect information from. You can also use any other flags you wish.

::

   C:\> SharpHound.exe -d contoso.local

All SharpHound Flags, Explained
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SharpHound has several optional flags that let you control scan scope,
performance, output, and other behaviors.

CollectionMethod
----------------

This tells SharpHound what kind of data you want to collect. These are the most
common options you'll likely use:

* **Default:** you can specify default collection, or don't use the CollectionMethod
  option and this is what SharpHound will do. Default collection includes Active
  Directory security group membership, domain trusts, abusable permissions on AD
  objects, OU tree structure, Group Policy links, the most relevant AD object
  properties, local groups from domain-joined Windows systems, and user sessions.
* **DCOnly:** Collects data ONLY from the domain controller, will not touch other
  domain-joined Windows systems. Collects AD security group memberships, domain
  trusts, abusable permissions on AD objects, OU tree structure, Group Policy
  links, the most relevant AD object properties, and will attempt to correlate
  Group Policy-enforced local groups to affected computers.
* **ComputerOnly:** collects user sessions and local groups from domain-joined
  Windows systems. Will NOT collect the data collected with the DCOnly collection
  method.
* **Session:** just does user session collection. You will likely couple this with
  the --Loop option. See SharpHound examples below for more info on that.
* **LoggedOn:** does session collection using the privileged collection method. Use
  this if you are running as a user with local admin rights on lots of systems
  for the best user session data.

Here are the less common CollectionMethods and what they do:

* **Group:** just collect security group memberships from Active Directory
* **ACL:** just collect abusable permissions on objects in Active Directory
* **GPOLocalGroup:** just attempt GPO to computer correlation to determine members
  of the relevant local groups on each computer in the domain. Doesn't actually
  touch domain-joined systems, just gets info from domain controllers
* **Trusts:** just collect domain trusts
* **Container:** just collect the OU tree structure and Group Policy links
* **LocalAdmin:** just collect the members of the local Administrators group on
  each domain-joined computer
* **RDP:** just collect the members of the Remote Desktop Users group on each
  domain-joined computer
* **DCOM:** just collect the members of the Distributed COM Users group on each
  domain-joined computer
* **PSRemote:** just collect the members of the Remote Management group on each
  domain-joined computer

Domain
------

Tell SharpHound which Active Directory domain you want to gather information from.
Importantly, you must be able to resolve DNS in that domain for SharpHound to work
correctly. For example, to collect data from the Contoso.local domain:

::

   C:\> SharpHound.exe -d contoso.local

Stealth
-------

Perform "stealth" data collection. This switch modifies your data collection
method. For example, if you want to perform user session collection, but only
touch systems that are the most likely to have user session data:

::

   C:\> SharpHound.exe --CollectionMethod Session --Stealth

ComputerFile
------------

Load a list of computer names or IP addresses for SharpHound to collect information
from. The file should be line-separated.

LDAPFilter
----------

Instruct SharpHound to only collect information from principals that match a given
LDAP filter. For example, to only gather abusable ACEs from objects in a certain
OU, do this:

::

   C:\> SharpHound.exe --LDAPFilter "(CN=*,OU=New York,DC=Contoso,DC=Local)"

ExcludeDomainControllers
------------------------

This will instruct SharpHound to not touch domain controllers. By not touching
domain controllers, you will not be able to collect anything specified in the
`DCOnly` collection method, but you will also likely avoid detection by Microsoft
ATA.

RealDNSName
-----------

In some networks, DNS is not controlled by Active Directory, or is otherwise
not syncrhonized to Active Directory. This causes issues when a computer joined
to AD has an AD FQDN of COMPUTER.CONTOSO.LOCAL, but also has a DNS FQDN of, for
example, COMPUTER.COMPANY.COM. You can help SharpHound find systems in DNS by
providing the latter DNS suffix, like this:

::

   C:\> SharpHound.exe --RealDNSName COMPANY.COM

OverrideUserName
----------------

Rohan remind me why we added this

CollectAllProperties
--------------------

Collect every LDAP property where the value is a string from each enumerated
Active Directory object.

WindowsOnly
-----------

Limit computer collection to systems with an operating system that matches *Windows*

Loop
----

Instruct SharpHound to loop computer-based collection methods. For example,
attempt to collect local group memberships across all systems in a loop:

::

   C:\> SharpHound.exe --CollectionMethod LocalGroup --Loop

LoopDuration
------------

By default, SharpHound will loop for 2 hours. You can specify whatever duration
you like using the HH:MM:SS format. For example, to loop session collection for
12 hours, 30 minutes and 12 seconds:

::

   C:\> SharpHound.exe --CollectionMethod Session --Loop --LoopDuration 12:30:12

LoopInterval
------------

How long to pause for between loops, also given in HH:MM:SS format. For example,
to loop session collection for 12 hours, 30 minutes and 12 seconds, with a 15
minute interval between loops:

::

   C:\> SharpHound.exe --CollectionMethod Session --Loop --Loopduration 12:30:12 --LoopInterval 00:15:00

DomainController
----------------

Target a specific domain controller by its IP address or name for LDAP collection

LdapPort
--------

Specify an alternate port for LDAP if necessary

SecureLdap
----------

Connect to the domain controller using LDAPS (secure LDAP) vs plain text LDAP.
This will use port 636 instead of 389.

LdapUsername
------------

Use with the LdapPassword parameter to provide alternate credentials to the domain
controller when performing LDAP collection.

LdapPassword
------------

Use with the LdapUsername parameter to provide alternate credentials to the domain
controller when performing LDAP collection.

DisableKerberosSigning
----------------------

Disables LDAP encryption. Not recommended.

PortScanTimeout
---------------

When SharpHound is scanning a remote system to collect user sessions and local
group memberships, it first checks to see if port 445 is open on that system.
This helps speed up SharpHound collection by not attempting unnecessary function calls
when systems aren't even online. By default, SharpHound will wait 2000 milliseconds 
(2 seconds) to get a response when scanning 445 on the remote system. You can decrease
this if you're on a fast LAN, or increase it if you need to. For example, to tell
SharpHound to wait just 1000 milliseconds (1 second) before skipping to the next host:

::

   C:\> SharpHound.exe --PortScanTimeout 1000

SkipPortScan
------------

Instruct SharpHound to not perform the port 445 check before attempting to enumerate
information from a remote host. This can result in significantly slower collection
periods.

Throttle
--------

Adds a delay after each request to a computer. Value is in milliseconds (Default: 0)

Jitter
------

Adds a percentage jitter to throttle. (Default: 0) 

Building SharpHound from Source
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
