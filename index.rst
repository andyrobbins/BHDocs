BloodHound: Six Degrees of Domain Admin
=======================================

.. meta::
   description lang=en: Identify and execute attack paths in Active Directory

.. image:: /images/bloodhound-logo.png   
   :align: center
   :width: 300px
   :alt: BloodHound logo

BloodHound uses graph theory to reveal the hidden and often unintended
relationships within an Active Directory environment. Attackers can use
BloodHound to easily identify highly complex attack paths that would otherwise
be impossible to quickly identify. Defenders can use BloodHound to identify
and eliminate those same attack paths. Both blue and red teams can use
BloodHound to easily gain a deeper understanding of privilege relationships in
an Active Directory environment.

Getting Started
---------------

**Install**   

Depending on which operating system you're using, install Neo4j, then
download the BloodHound GUI. You can also build the BloodHound GUI from source.

OS-specific instructions: 
:doc:`Windows <installation/windows>` |
:doc:`OSX <installation/osx>` |
:doc:`Linux <installation/linux>`

**Collect your first dataset**

BloodHound is a data analysis tool and needs data to be useful. The officially
supported data collection tool for BloodHound is called SharpHound. Download
SharpHound or build it from source to collect your first data set. From a
domain-joined system in your target Active Directory environnment, collecting
your first dataset is quite simple:

::

   C:\> SharpHound.exe

Executing Advanced Queries
   Words about cypher here

Contributing Back
   Words about contributing back here

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: Getting Started
   
   getting-started/installation
   getting-started/example-db
   getting-started/collecting-first-dataset
   getting-started/executing-advanced-queries
   getting-started/contributing-back

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: Installation

   installation/windows
   installation/osx
   installation/linux

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: Data Collection

   data-collection/sharphound
   data-collection/bloodhound-py

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: Data Analysis

   data-analysis/bloodhound-gui
   data-analysis/nodes
   data-analysis/edges

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: Cypher Query Language

   cypher-query-language/intro-to-cypher
   cypher-query-language/cypher-query-gallery

.. toctree::
   :maxdepth: 2
   :hidden:
   :caption: Blue Team Use Cases

   blue-team-use-cases/auditing-permissions

Core Components
---------------

Words about core components here

Features
--------

Pathfinding
   Words about path finding here

Node Exploration
   Words about exploring nodes here
