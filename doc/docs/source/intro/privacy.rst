Privacy
=======

        “Arguing that you don't care about the right to privacy because you
        have nothing to hide is no different than saying you don't care about
        free speech because you have nothing to say.” 
        -- Edward Snowden

Privacy is a core value in the design of Offset. This document discusses what
information Offset allows you to keep private, and what bits of information
might leak when you use Offset.

Summary
-------

* The only unique id representing your Offset identity is a public key.

* Offset does not require any of the following information:

  * Real name
  * Bank account information
  * Credit card number
  * Phone number (Sim card is also not required)
  * Email address

* Transactions

  * In most cases when you buy something with Offset, your identity will be kept secret.
  * When you sell something using Offset, the buyer learns your public key.

* Index servers 

  * Know your public key
  * Know the public keys of your friends
  * Know approximate balances between you and your friends.
  * Know when you are online

* Relays

  * Know your public key
  * Might know the public keys of some of your friends

* Offset friends

  * Know your public key
  * Know when you are online
  * Usually do not know your IP address

* Strangers

  * Can't do much if don't know your public key

  * If a stranger knows your public key, he might be able to deduce:

    * The public keys of your friends
    * Approximate balances between you and your friends.
    * When you are online


* The Offset app 

  * Contains no tracking code
  * Does not send your information to any third party, except for the ones you select explicitly.



Identity
--------

When you create a new Offset credit card, a new key pair (private and public
key) is created for you automatically. Offset does not require that you provide
your real name, id number, bank account, credit card number, phone number or
email address. Therefore, non of those pieces of information are linked to your
Offset identity.

The newly created public key is the only unique id representing your identity
in almost all of Offset operations.

Transactions
------------

When you initiate **a sale** by creating an invoice and sending it, you expose your
Offset public key. This happens because your public key is included inside the
invoice file. Unless your public key is provided, the buyer (And the rest of
nodes along the transaction chain) will not be able to verify your signature.

When you buy from someone using Offset, you send the funds along a chain of
Offset nodes. The seller can only see the public key of the last node on the
chain, and therefore if the chain is of size at least 3, the seller will not be
able to learn the buyer's public key. Hence, the buyer can have certain privacy
when buying goods.

Index servers
-------------

Index servers has the job of knowing the full state of an Offset network,
allowing nodes to find routes of certain credit capacity. As such, index
servers need to know about every Offset friendship, and an approximate balance
for every currency in those friendships.

As a result, given a node's public key, an index server can know whether a node
is online, list the node's friends public keys, and provide approximate
balances with those friends.


Relays
------

Relays facilitate communication between nodes. A node typically listens on
multiple relays to accept TCP connections from friends. As communication
between nodes is usually not direct, in most cases Offset friends can not learn
each other's IP address.

As a node is constantly connected to a few relay servers, a relay server can 
know whether or not a node is online. (Though not every relay will have this
information about every node).


Friends
-------

Your Offset friends know your Offset public key (It is included inside your
friend ticket).

Knowing the liveness status of an Offset friend is crucial for the operation of
Offset, because Offset nodes should not forward transactions to friends that
seem to be offline. Your Offset friends get information from their relays about
your liveness status. Therefore, Offset friends can always know if you are
online. 


Strangers
---------

A stranger that does not know your Offset public key will not be able to obtain
any information about your presence in Offset.

A stranger that knows your Offset public key can query index servers, to be
able to deduce some information like your friends' public keys and possibly
deduce approximate balances between you and your friends.


Offset app
----------

The Offset app does not include any tracking functionality, and does not send
your information to any third party without your explicit consent. There is no
"Offset app backend", as Offset is fully decentralized.

When you connect with an index server, relay or an (Offset) friend, you share
some information with them, as described earlier in this document.
