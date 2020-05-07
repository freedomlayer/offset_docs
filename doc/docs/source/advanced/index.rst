Index
=====

Index servers have the role of indexing all the nodes in an Offset network.
Upon request, an Index server can return routes between nodes with a requested
amount of credit capacity.

Anyone can run an index server instance. However, index servers are only useful
if they share information with each other. We say that the **index servers form a
federation**. 

In the current implementation of Offset, every node reports his current status
to only one index server. This index server than forwards the information about
this node to all the other index servers he knows about.

If you run your own index server but do not register it with other index
servers, your index server will only have partial information about the nodes
in the network, and therefore your index server will not be useful for finding
routes between any pair of nodes.

Index servers only share data if they were configured to do so. For two index
servers A and B to share data, the following must hold:

- A registered B as a trusted index server
- B registered A as a trusted index server

This document describes how to set up an index server on your desktop computer.
You can use the same instructions to run your index server in the cloud.

Recommended prior reading:

- :doc:`Network structure <../theory/network>`
- :doc:`Initial setup <setup>`


Running your own index server
-----------------------------

We begin by presenting all the commands required to configure and run an index
server:

.. code:: sh

   # Public facing index server's address
   INDEX_ADDRESS="www.myindex.com"

   # Public facing index server's port, used by nodes
   INDEX_PORT_CLIENT=3456

   # Public facing index server's port, used to federate with other index
   # servers
   INDEX_PORT_SERVER=6543

   # Create a directory for the index server's data
   mkdir index
   # Create a directory for trusted index servers.
   mkdir index/trusted

   # Create an identity for the index server
   stmgr gen-ident --output index/index.ident

   # Generate index ticket to be used by nodes
   stmgr index-ticket \
           --address $INDEX_ADDRESS:$INDEX_PORT_CLIENT \
           --idfile index/index.ident \
           --output index/index_client.index

   # Generate index ticket to be used by other index servers
   stmgr index-ticket \
           --address $INDEX_ADDRESS:$INDEX_PORT_SERVER \
           --idfile index/index.ident \
           --output index/index_server.index

   # Register other trusted index servers
   # Note that those servers also have to register 
   # our index server tickets as # trusted
   cp other_index1.index index/trusted/
   cp other_index2.index index/trusted/

   # Start index server
   stmgr --idfile index/index.ident \
         --lclient $INDEX_ADDRESS:$INDEX_PORT_CLIENT \
         --lserver $INDEX_ADDRESS:$INDEX_PORT_SERVER \
         --trusted index/trusted &


To be able to federate with another index server, you and the other index
server's operator should exchange index servers tickets. Your ticket is found
at ``index/index_server.index``.

Assuming that you received the file ``other_index1.index`` from the other index
server operator, you should copy this file to your ``index/trusted`` directory.
It is crucial that both you and the other index server operator add each
other's ticket to the ``index/trusted`` directory.

To configure nodes to use your index server, provide them with index server
ticket for node clients: ``index/index_client.index``. This file can also be
added using the Offset app.

Index commands explained
------------------------

TODO
