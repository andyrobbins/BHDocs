SharpHound
==========

SharpHound is the official data collector for BloodHound. It is written
in C# and uses native Windows API functions and LDAP namespace functions
to collect data from domain controllers and domain-joined Windows systems.

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
