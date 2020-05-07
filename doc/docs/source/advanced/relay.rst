Relay
=====

A relay server is used to relay communication between Offset nodes.
We use relays because most user devices are behind `NATs
<https://en.wikipedia.org/wiki/Network_address_translation>`_, and therefore do
not own a public address. As a result, it is usually very difficult for two
user devices to communicate directly.

Make sure to go through the required :doc:`initial setup <setup>` first.

Running your own relay
----------------------

To set up a relay, run:

.. code:: sh

   # Public facing relay address
   RELAY_ADDRESS="www.myrelay.com"

   # Public facing relay port
   RELAY_PORT=4567

   # Create a directory for the relay's data
   mkdir relay

   # Generate a new identity for the relay
   stmgr gen-ident --output relay/relay.ident

   # Generate relay ticket
   stmgr relay-ticket \
           --address $RELAY_ADDRESS:$RELAY_PORT \
           --idfile relay/relay.ident \
           --output relay/relay.relay

   # Start the relay:
   strelay --idfile relay/relay.ident --laddr 0.0.0.0:$RELAY_PORT &


Relay commands explained
------------------------

We first set ``RELAY_ADDRESS`` as a public facing address for the relay.
``RELAY_ADDRESS`` is the public facing address of the relays. Nodes will
connect to the relay through this address.  

If the scope of your experiment is your local machine, you can use
``127.0.0.1``. If you are running a test at home that includes your mobile
phone, you might be able to use your address in your private LAN network. 

If you want to deploy your relay to the cloud, you should set ``NODE_ADDRESS``
to your public facing IP, or possibly to your domain name. If you are using a
domain name, note that you do not need to obtain a certificate, because Offset
uses a different type of authentication.

``RELAY_PORT`` is the public facing port of the relay. You can pick any port
number that you want that is at least 1024 [1]_.

Next, we create a directory named ``relay``, that is going to hold all of our
relay's data.


The following command:

.. code:: sh

   stmgr gen-ident --output relay/relay.ident

Generates a new identity for the relay. An identity is a pair of private and
public keys. When nodes connect to the relay, the relay will use his private
key to prove his identity.

Next, we generate a relay ticket:

.. code:: sh

   stmgr relay-ticket \
           --address $RELAY_ADDRESS:$RELAY_PORT \
           --idfile relay/relay.ident \
           --output relay/relay.relay

A relay ticket is a file containig the relay's public address and public key.
The ticket file allows nodes to connect to the relay. A relay ticket file can
also be used inside the Offset mobile app, to configure a new relay.

Finally, the last command:

.. code:: sh

   strelay --idfile relay/relay.ident --laddr 0.0.0.0:$RELAY_PORT &

Starts the relay. The ``&`` sign at the end of the command means that the
command will run in the background. If this is not what you want, you can omit
it.

.. [1]
   In most operating systems, ports below 1024 are usually reserved, and
   require administrator priviledges to use.

