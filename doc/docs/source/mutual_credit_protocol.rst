Mutual Credit Protocol
======================

Prior reading
-------------

To get the most of this document, you are recommended to first read:

- :doc:`Economic idea <economic>`
- :doc:`Network structure <network>`

Intro
-----

At its core, **Offset allows to manage mutual credit balances at scale**.
Managing credit balance between only two people is a simple task, and can be
done using a pen and a paper. However, managing mutual credit for a whole
economy becomes more difficult, and requires specialized technology. 

Recall the idea of mutual credit. Consider three B, C, D. Suppose that B and C
maintain a shared balance, and also that C and D maintain shared balance. We
can draw the relationships between B, C, D using this simplified graph:

.. code:: text

   B --- C --- D

Suppose that initially, B and C set the balance between them to be 0, and that
C and D also set the balance between them to be 0. Assume that B wants to pay D
100 credits. In order to do this, the following needs to happen:

1. B and C set the balance between them to -100 (B owes C 100 credits)
2. C and D set the balance between them to -100 (C owes D 100 credits)

In total, we have the following balances:

- B: -100
- C: -100 + 100 = 0
- D: +100

Therefore, we consider the payment from B to D to be successful. The process
described here is the core of what Offst does.

If paying is that simple, why do we need a sophisticated protocol?
There are a few problems the Offset protocol attempts to solve:

- Correct incentives: What stops C from taking the money B has sent and run
  away with the money?

- Atomic payments along routes: What happens if in the middle of the payment, C
  suddenly become nonresponsive? How can we be sure that transactions are
  synchronized correctly, and don't get stuck?


The protocol described here is an abstraction of the communication protocol
used between nodes. For brevity's sake, we neglect a few matters:

- Route discovery (Done using index servers)
- How communication between nodes is relayed using relay servers.
- External "Token Channel" protocol
   - Allows nodes to keep communication even after disconnection.
   - Synchronizes communication between the two nodes.
   - Multiplexes multiple currency balances.
- Payments performed along multiple routes at the same time

The mutual credit protocol described here explains the inner workings of a
single currency channel between two nodes.

Protocol outline
----------------

Consider a network of 4 parties: B, C, D, E, where the pairs: (B,C), (C,D),
(D,E) are Offset friends.

.. code:: text

    B --- C --- D --- E

Suppose that B wants to send credits to E along this route. We call B
the **buyer**, and E the **seller**. The following is an outline diagram for the
payment protocol:

.. code:: text

    Invoice        <=====[inv]========    (Out of band)

    Request        ------[req]------->
    Response       <-----[resp]-------

    Commit         ======[commit]====>    (Out of band)
    Goods          <=====[goods]======    (Out of band)

    Collect        <-----[collect]----
    (Receipt)
                   B --- C --- D --- E

Where single arrows (``--->``) represent communication done between nodes, and
fat arrows (``===>``) represent communication done out of band. Out of band
communication could be using QR codes, sending an instant message on a mobile
phone, or using email. 

For example, in the diagram above, the invoice will be sent from E to B
using some external communication. However, the Request message sent from B to
E is sent along the chain of nodes ``B -- C -- D -- E``.

In the above diagram, the following operations occur:

1. E hands an Invoice to B. (Out of band)
2. B sends a Request message along a path to E.
3. E sends back a Response message along the same route to B.
4. B prepares a Commit message and hands it to E (Out of band).
5. E gives the goods to B (Out of band).
6. E sends a Collect message along the route to B.
7. B receives the Collect message, prepares a Receipt and keeps it.


From the point of view of an Offset user, this is how the transaction looks
like:

.. code:: text

    Invoice        <=====[inv]========    (Out of band)
    Commit         ======[commit]====>    (Out of band)
    Goods          <=====[goods]======    (Out of band)
                   B -- ...   ... -- E

The user only see the following steps:

(1) E hands an Invoice to B. (Out of band)
(4) B prepares a Commit message and hands it to E (Out of band).
(5) E gives the goods to B.

The event of B handing the commitment to E is "atomic". In other words, the
moment E receives the commitment, E knows for sure that he will receive the
money, and the transaction is considered successful.  Note however, that it
might take some time until the seller E will be able to collect his credits.

Message definitions
-------------------

Request message
~~~~~~~~~~~~~~~

.. code:: text

     -------[req]----->
    B --- C --- D --- E

This is the structure of the request message:

.. code:: capnp

    struct RequestSendFundsOp {
            requestId @0: Uid;
            # Randomly generated reqeustId [128 bits]
            srcHashedLock @1: HashedLock;
            # bcrypt(srcPlainLock), where srcPlainLock is of size 256 bits.
            route @2: FriendsRoute;
            # A route of friends that leads to the destination
            destPayment @3: CustomUInt128;
            # Amount of credits to pay the destination over this route.
            totalDestPayment @4: CustomUInt128;
            # Total amount of credits to be paid (Must match the invoice)
            # totalDestPayment > destPayment in cases of multi route-payments.
            invoiceId @4: InvoiceId;
            # A 256 bit value representing the invoice this request
            # intends to pay.
    }

The request message mainly verifies that there is enough capacity to
make the payment along the route (Including capacity for the transaction
fees). For example, if B wants to send 10 credits to E, then during the
request message passage from B to E:

-  B checks that B -> C has at least the capacity of 12 credits.
-  C checks that C -> D has at least the capacity of 11 credits.
-  D checks that D -> E has at least the capacity of 10 credits.

The extra credits are due to transaction fees to C and D (1 credit
each).

The Request message contains a hash lock: ``srcHashedLock``. This value
is generated by the buyer by generating a random ``srcPlainLock`` value
and hashing it: ``srcHashedLock := bcrypt(srcPlainLock)``. This
mechanism is used to ensure transaction atomicity: The seller can not
create a valid Collect message without knowing the secret value
``srcPlainLock``.

Response message
~~~~~~~~~~~~~~~~

If all went well during the Request stage, E sends back a Response
message along the same path, all the way back to B.

.. code:: text

     <------[resp]-----
    B --- C --- D --- E

.. code:: capnp

    struct ResponseSendFundsOp {
            requestId @0: Uid;
            destHashedLock @1: HashedLock;
            randNonce @2: RandNonce;
            signature @3: Signature;
            # Signature{key=destinationKey}(
            #   sha512/256("FUNDS_RESPONSE") ||
            #   sha512/256(requestId || randNonce) ||
            #   srcHashedLock ||
            #   destHashedLock ||
            #   destPayment ||
            #   totalDestPayment ||
            #   invoiceId
            # )
            #
            # Note that the signature contains an inner blob (requestId || ...).
            # This was done to make the size of the receipt shorter, as previously
            # this contained a full route.
    }

Note that the response contains a ``destHashedLock``. This value is
created by hashing a secret generated by the seller:
``destHashedLock := bcrypt(destPlainLock)``. This secret will only be
revealed when the Collect message is sent. We have this mechanism to
defend against fake Receipt generated by the buyer. (A valid receipt
must contain the secret ``destPlainLock``).

Cancel message
~~~~~~~~~~~~~~

A Cancel message may be sent back by any node during the Request period.
After Request message arrived at the seller node and before the Collect
message was sent, only the seller node may send a Cancel message. In
addition, any node may send a Cancel message to cancel ongoing
transactions in case of unfriending a node (As long as the Collect
message was not yet received).

After the Collect message was received, the transaction can not be
cancelled.

If any node could not forward the request message, or the destination
decided to cancel the transaction, a failure message will be sent back,
beginning from the failing node.

A cancel message can be sent as long as **receipt message** was not yet
sent.

.. code:: text

     <----[cancel]-----
    B --- C --- D --- E

.. code:: capnp

    struct CancelSendFundsOp {
            requestId @0: Uid;
    }

Commit message (Out of band)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After receiving a Response message, the source node of the payment
creates a Commit message. The Commit message is given to the
destination, and at that moment the payment is considered successful.

.. code:: text

    ======[commit]====>    (Out of band)
    B --- C --- D --- E

Upon receipt of a valid Commit message, the seller will give the goods
to the buyer, and send back (along the same route) a Collect message to
collect his credits.

.. code:: capnp

    struct Commit {
            responseHash @0: Hash;
            # = sha512/256(requestId || randNonce)
            destPayment @1: CustomUInt128;
            # Amount of credits paid in this Transaction.
            srcPlainLock @2: PlainLock;
            # The preimage of the hashedLock at the request message [256 bits]
            destHashedLock @3: HashedLock;
            signature @4: Signature;
            # Signature{key=destinationKey}(
            #   sha512/256("FUNDS_RESPONSE") ||
            #   sha512/256(requestId || sha512/256(route) || randNonce) ||
            #   srcHashedLock || 
            #   destHashedLock || 
            #   destPayment ||
            #   totalDestPayment ||
            #   invoiceId
            # )
    }

    struct MultiCommit {
            invoiceId @0: InvoiceId;
            # InvoiceId being paid.
            totalDestPayment @1: CustomUInt128;
            # The total amount being paid
            commits @2: List(Commit);
            # A list of confirmations. Each confirmation corresponds to a request
            # sent along one route.
    }

Note that the ``MultiCommit`` message may contain multiple ``Commit``-s,
each corresponding to one request. This allows fragmented payment along
multiple routes.

Verification of a MultiCommit message by the seller is done as follows:

-  InvoiceId matches an originally issued invoice.
-  For every Commit:

   -  The revealed lock is valid:
      ``bcrypt(srcPlainLock) == srcHashedLock``
   -  signature is valid.

-  Total of ``destPayment`` is correct (Equal the requested amount at
   the invoice).

Collect message
~~~~~~~~~~~~~~~

After receiving a confirmation message from the buyer, the destination
gives the goods to the buyer and sends back a Collect message to collect
his credits.

.. code:: text

     <----[collect]----
    B --- C --- D --- E

A Collect message completes the transaction. For example, when the
Collect message is sent from E to D, the credits that were frozen
between D and E become unfrozen, and the payment is irreversible. The
Collect messages continues all the way (along the original route) to the
source of the payment.

.. code:: capnp

    struct CollectSendFundsOp {
            requestId @0: Uid;
            srcPlainLock @1: PlainLock;
            destPlainLock @2: PlainLock;
    }

Note that the Collect message can only be sent by the seller after it
has received the confirmation, because the confirmation contains the
srcPlainLock.

When receiving a CollectSendFundsOp messages, the following should be
verified:

-  ``bcrypt(srcPlainLock) = srcHashedLock``
-  ``bcrypt(destPlainLock) = destHashedLock``

Receipt
~~~~~~~

Upon receiving the Receipt message, the source of the payment can
compose a Receipt.

.. code:: capnp

    struct Receipt {
            responseHash @0: Hash;
            # = sha512/256(requestId || randNonce)
            invoiceId @1: InvoiceId;
            srcPlainLock @2: PlainLock;
            destPlainLock @3: PlainLock;
            destPayment @4: CustomUInt128;
            totalDestPayment @4: CustomUInt128;
            signature @5: Signature;
            # Signature{key=destinationKey}(
            #   sha512/256("FUNDS_RESPONSE") ||
            #   sha512/256(requestId || sha512/256(route) || randNonce) ||
            #   srcHashedLock || 
            #   dstHashedLock || 
            #   destPayment ||
            #   totalDestPayment ||
            #   invoiceId
            # )
    }

The Receipt can be constructed only after the CollectSendFundsOp message
was received. Note that it is possible that a receipt can be constructed
only a long time after the confirmation message was given.

Cancellation
------------

Cancellation can happen at any time after the Request message was sent from the
buyer and before the Collect message was sent.

During the Request stage a cancellation message could be sent from any
node forwarding the Request message. However, after the Request message
arrives at the seller node, only the seller node may issue a
cancellation message. (This rule has one exception that happens during
unfriending, see below).

Examples for cancellation
~~~~~~~~~~~~~~~~~~~~~~~~~

-  An intermediate node cancels the transaction during the Request
   period. This can happen for example if there is not enough capacity
   for pushing credits forward.

.. code:: text

    Invoice        <=====[inv]========    (Out of band)

    Request        ---[req]---->
    Cancel         <--[cancel]--

                   B --- C --- D --- E

-  invoiceId is not recognized (by E):

.. code:: text

    Invoice        <=====[inv]========    (Out of band)

    Request        ------[req]------->
    Cancel         <-----[cancel]-----

                   B --- C --- D --- E

-  Commit took too long to arrive:

.. code:: text

    Invoice        <=====[inv]========    (Out of band)

    Request        ------[req]------->
    Response       <-----[resp]-------
    Cancel         <-----[cancel]-----

                   B --- C --- D --- E

-  Cancellation in Request period that happens due to unfriending nodes:

.. code:: text

    Invoice        <=====[inv]========    (Out of band)

    Request        ------[req]------->
                   B --- C --- D --- E
    Unfriend
    Cancel         <--[cancel]--
                   B --- C --- D     E

In the figure above: a request was sent from B to E. Next, D unfriends E
before E manages to send the response message. In that case D sends a
Cancel message for this transaction all the way back to B, and the
transaction credits are unfrozen.

-  Cancellation in Response period that happens due to unfriending
   nodes:

.. code:: text

    Invoice        <=====[inv]========    (Out of band)

    Request        ------[req]------->
    Response       <-----[resp]-------
                   B --- C --- D --- E
    Unfriend
    Cancel         <--[cancel]--
                   B --- C --- D     E


Atomicity
---------

Atomicity is guaranteed by using a `hash
lock <https://en.bitcoin.it/wiki/Hashlock>`__ created by the buyer:
``srcHashedLock``.

Assume that the node E issued an invoice and handed it to B.

B wants to pay the invoice. The payment begins by sending a Request
message along the path from B to E. The payment is considered successful
when B hands a MultiCommit message to E.

This means that we should examine the possibility of B waiting
indefinitely during the sending of Request and Response messages along
the route.

During this time (Request + Response period), B can discard the
transaction by walking away. E will not be able to make progress because
in order to send the Collect message, the correct srcPlainLock is
required, but E does not know it before B sends the MultiCommit message.

Also note that if B sends a valid MultiCommit message to E, the
transaction is considered successful, and B can not reverse it. This
happens because B reveals srcPlainLock at the MultiCommit message sent
to E.

Receipt verifiability
---------------------

A receipt is a proof that a certain invoice was paid. It can be verified
by anyone that possesses:

-  The invoice (``invoiceId`` + public key of seller)
-  The Receipt

Verification is performed by checking the signature (See description of
signature at the Receipt definition).

In order to make sure the buyer can not have a valid Receipt before the
payment actually completed, we use a hash lock that is issued by the
payment destination: ``destHashedLock``.

When the buyer receives a Response message it can not yet create a valid
Receipt, because the buyer doesn't yet know ``destPlainLock``. This
value is revealed only at the Collect message, when the payment is
considered to be successful.

Note: An alternative solution could be to let the seller sign a new
signature over the Collect message, but instead we chose to use a hash
lock, which is a less expensive cryptographic operation. Using a hash
lock also does not require access to the identity of the seller.

This leaves the whole protocol with only one cryptographic signature
over the Response message, signed by the seller.

Cancellation responsibility
---------------------------

Only the seller can issue a Cancel message (Sent from the destination
along a path to the source). A Cancel message will be sent by the Offset
node automatically for any incoming Request message that contains a non
recognized InvoiceId (TODO: Can this cause any issues?)

In addition, cancellation can be issued for a certain ``invoiceId`` from
the application level. Cancellation message should only be sent after
the invoice was issued and before a Commit message was received. It
might be possible for applications to cancel ``invoiceId``-s after a
certain amount of time.

-  Sending a Cancel message after a Commit message was received is
   considered a bad form for the seller, and can be seen as equivalent
   to not delivering the goods after a successful payment.

-  Sending a Cancel message after the goods were given to the buyer will
   cause the seller to lose credits.

The only way for the buyer to cancel a transaction is by never sending a
Commit message to the seller.

Multi-Route transactions
------------------------

Sometimes it might not be possible to send a payment along a single
route. In such cases it is useful to send the transaction along multipe
routes. The protocol allows sending a payment along multiple routes
atomically. This is done as follows:

1. Buyer gets an Invoice from the seller for a certain amount of
   credits.
2. Buyer sends a RequestSendFundsOp along a route.
3. A ResponseSendFundsOp or a CancelSendFundsOp message is returned.
4. Go back to (2) until the wanted amount of credits is acheived (for
   paying the invoice).
5. Buyer sends a MultiCommit message containing a list of all Commit-s
   for all the requests that got a valid response.
6. Seller verifies the MultiCommit message. If valid, the payment is
   accepted and the goods are handed to the buyer.
7. The Seller sends back CollectSendFundsOp messages for all requests.
8. Any CollectSendFundsOp message can be used to create a valid Receipt.
   (Two diferent constructed receipts will have the same invoiceId but
   different responseHash).

