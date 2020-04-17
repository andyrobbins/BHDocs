The BloodHound GUI
==================

After setting up neo4j and collecting data with SharpHound, you're
ready to explore the data with the BloodHound GUI.

.. note:: Want to follow along? Explore the example dataset by loading
   the example db locally, or connect to the hosted example dataset at
   bolt://206.189.85.93:7687

Connecting to your Database
^^^^^^^^^^^^^^^^^^^^^^^^

When you open BloodHound for the first time, you will be prompted to
log into a neo4j database:

.. image:: ../images/neo4j-login.png
   :align: center
   :width: 700px
   :alt: BloodHound Login Screen

Enter the IP address of your neo4j server (or localhost if neo4j is running
locally), the neo4j user name, and neo4j password.

.. note:: Following along with the example database? The username is neo4j
   and the password is BloodHound. The online example is hosted at
   bolt://206.189.85.93:7687

You can optionally check the box next to "Save Password". This will store
your neo4j credentials on disk, and BloodHound will automatically try to
authenticate to neo4j with that info the next time you open the GUI.

After authenticating, you'll be presented with the standard GUI, including
the graph rendering area, search bar, etc. If you want to connect to a different
neo4j instance, you can disconnect from your current instance by clicking
the "Database Info" tab, then clicking "Log Out/Switch DB":

.. image:: ../images/log-out.png
   :align: center
   :width: 700px
   :alt: Database logout button

Importing Data
^^^^^^^^^^^^^^

When SharpHound finishes collecting data, it will create a ZIP file which
contains individual JSON files. The easiest way to import that data is to
very simply drag and drop the ZIP into the BloodHound GUI:

.. image:: ../images/import-data.gif
   :align: center
   :width: 700px
   :alt: Import data with drag and drop

When finished, you'll see an alert at the top that says "Finished processing
all files"

You can also import the individual JSON files one at a time, and even import
multiple zip files at the same time.

The data import process is almost exclusively using `merge` operations, meaning
if the data you're importing is already in the database, there will be no change.
For example, if BloodHound already knows a certain user belongs to a group,
and you re-import group membership data, there won't be any change: BloodHound
will simply keep the same information it had before.

The data import process will also never `remove` data from the database: if a
user belonged to a group, then no longer does after a new data collection, the
data import process will not remove the data - you'll still see the user as a
part of that group.

Some data will be over-written, such as properties on objects. For example, if
a user's last logon time stamp is updated in AD, the data import process will
update that property on the user node.

Searching for Nodes
^^^^^^^^^^^^^^^^^^^

Path Finding
^^^^^^^^^^^^

Keyboard Shortcuts
^^^^^^^^^^^^^^^^^^

Running Raw Cypher Queries
^^^^^^^^^^^^^^^^^^^^^^^^^^
