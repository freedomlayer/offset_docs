Node
====

An Offset node is roughly the equivalent of an Offset card.

An Offset node that lives inside a mobile device has a major disadvantage: when
the mobile device enters "sleep mode", or loses Internet connection, the node
becomes offline. As a result, transactions can not be forwarded using the node.

This document describes how to set up a node on your desktop computer. You can
use the same instructions to run your Offset node in the cloud.

Recommended prior reading:

- :doc:`Network structure <../theory/network>`
- :doc:`Initial setup <setup>`

Running your own node
---------------------

We begin by presenting all the commands required to have a basic node running.
We will then explain what the commands do.

.. code:: sh

   # Public facing node address
   NODE_ADDRESS="www.mynode.com"

   # Public facing node port
   NODE_PORT=9876

   # Create a directory for the node's data
   mkdir node

   # Create a subdirectory for the node's trusted applications.
   mkdir node/trusted

   # generate node's identity
   stmgr gen-ident --output node/node.ident

   # Init node database
   stmgr init-node-db --idfile node/node.ident --output node/node.db

   # Generate node tickets:
   stmgr node-ticket  --address $NODE_ADDRESS:$NODE_PORT \
           --idfile node/node.ident \
           --output node/node.ticket

   # Create a directory for the app's data
   mkdir app

   # Generate app's identity
   stmgr gen-ident --output app/app.ident

   # Generate app ticket:
   stmgr app-ticket --idfile app/app.ident \
           --pbuyer --pconfig --proutes --pseller \
           --output app/app.ticket

   # Fill in apps tickets at the corresponding nodes' "trusted" directories
   cp app/app.ticket node/trusted/

   # Create node entries:
   stmgr node-entry \
           --address $NODE_ADDRESS:$NODE_PORT \
           --appid app/app.ident \
           --nodeid node/node.ident \
           --output node/node.rcard

   # Start node:
   stnode --database node/node.db \
           --idfile node/node.ident \
           --laddr 0.0.0.0:$NODE_PORT \
           --trusted node/trusted &


This is all your need to do to have your own node up and running.  To connect
your mobile phone to your node, you need to transfer the ``node/node.rcard``
file to your mobile phone (You can also scan it as a QR
code).

Node commands explained
-----------------------

Let's begin explaining the above commands.

``NODE_ADDRESS`` is the public facing address of the node. Applications will
connect to the node through this address.  

If the scope of your experiment is your local machine, you can use
``127.0.0.1``.

If you are running a test at home that includes your mobile phone, you might be
able to use your address in your private LAN network. 

- Linux: ``ifconfig`` or ``ip addr``
- Windows: ``ipconfig``
- macOS: ``ifconfig``

Your local IP address will usually look like ``10.100.101.2`` or
``192.168.0.5``, but it might be something else.

If you deploy your node to the cloud, you should set ``NODE_ADDRESS`` to your
public facing IP, or possibly to your domain name. If you are using a domain
name, note that you do not need to obtain a certificate, because Offset uses a
different type of authentication.

``NODE_PORT`` is the public facing listening port of the node. Usually you
should be able to select any port that you like that is larger or equal to 1024 [1]_. 

Next, we create a directory for the node's data, and invoke:

.. code:: sh

   stmgr gen-ident --output node/node.ident

This command creates a new identity to be associated with the node. An identity
is a key pair: A private key and a public key. All transactions issued through
this node will be signed using this identity.

Next, we create an initial database for the node:

.. code:: sh

   stmgr init-node-db --idfile node/node.ident --output node/node.db

The node's database contains the full state of the node. It contains, for
example, all the current balances, configured friends, configured relay servers
and index servers. The command above will create an empty new node database.

Next, we create a node ticket:

.. code:: sh

   stmgr node-ticket  --address $NODE_ADDRESS:$NODE_PORT \
           --idfile node/node.ident \
           --output node/node.ticket

A node ticket is a file containing the node's public address and public key.
This information allows Application to securely connect to the node.

We continue to create an Application. We first create the directory ``app``,
which is going to contain all of the application's files.

As for the node, we begin by generating an identity file for the application:

.. code:: sh

   stmgr gen-ident --output app/app.ident

Next, we create an application ticket:

.. code:: sh

   stmgr app-ticket --idfile app/app.ident \
           --pbuyer --pconfig --proutes --pseller \
           --output app/app.ticket

The application's ticket contains the the application's public key, and
permissions. In the command above we gave the application all the possible
permissions: buying, configuration, routes query and selling.

The application's ticket is then stored at the node's trusted directory:

.. code:: sh

   cp app/app.ticket node/trusted/

By storing the application's ticket in this directory, we register the
application with the node. If we skip this step, the node will not be willing
to communicate with the application.

Next, we create a node entry, also known as a "remote node" file:

.. code:: sh

   stmgr node-entry \
           --address $NODE_ADDRESS:$NODE_PORT \
           --appid app/app.ident \
           --nodeid node/node.ident \
           --output node/node.rcard

The remote node file allows an Offset mobile app to connect to this as an
Offset application.

The node is not running yet. To run the node, we invoke:

.. code:: sh

   stnode --database node/node.db \
           --idfile node/node.ident \
           --laddr 0.0.0.0:$NODE_PORT \
           --trusted node/trusted &

The `&` sign at the end of the command means that the command will run at the
background. If this is not what you want, you may omit the sign.

Resulting files tree
--------------------

These are the files you should have after running the above commands:

.. code:: sh

   app/
   ├── app.ident
   └── app.ticket

   node/
   ├── node.db
   ├── node.ident
   ├── node.rcard
   ├── node.ticket
   └── trusted
       └── app.ticket


.. [1]
   In most operating systems, ports below 1024 are usually reserved, and
   require administrator priviledges to use.

