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
