Offset is not a blockchain
==========================

The recent rise in popularity of digital currencies might make it difficult to
understand what makes Offset different in its approach to money and payments.
We present here the differences between Offset and a blockchain based digital
currency.

Summary
-------

.. list-table:: Offset - Blockchain [1]_ comparison
   :header-rows: 1

   * - 
     - Blockchain
     - Offset

   * - :ref:`global-consensus`
     - Yes
     - No

   * - :ref:`security-foundation`
     - Proof of work (Mining)
     - Trust between people

   * - :ref:`origin-of-money`
     - Created through mining
     - Created and destroyed by users. Automatically adjusts to market size.

   * - :ref:`incentives`
     - Rewards first adopters
     - Early and late adopters have the same money creation power

   * - :ref:`transaction-speed`
     - A few mintues, up to a few hours
     - Less than a few seconds

   * - :ref:`transaction-certainty`
     - Certainty increases as time progresses, but never reaches 100%
     - 100% certainty when completed

   * - :ref:`storage`
     - Blockchain size increases at a rate of a few GBs every month
     - Small constant size (A few KBs)

   * - :ref:`fees`
     - High fees due to mining difficulty
     - According to users' settings along route

   * - :ref:`Receive payments when offline <recipient-online>`
     - Yes
     - No


.. _global-consensus:

Global consensus
----------------

Global consensus is a core idea powering blockchain based digital currencies:
All the participants in a blockchain network try to maintain together a single
view of the current state of balances for all users. Blockchain based currencies
usually use proof of work as a technology to acheive consensus: the ledger that
took the most effort to create is chosen as the global truth.

TODO: Add image for longest chain rule (Bitcoin)

Contrast with blockchain currencies, Offset does not attempt to acheive global
consensus. Instead, every Offset user maintains synchronized balances with a few
selected Offset friends. In other words, each Offset user has a local view of
his own balances, and not a global view of the balances of all Offset users. **It
turns out that secure payments are possible even without a global consensus
system!**

Offset's approach makes it much more efficient than its blockchain counterparts.
Transactions are much faster, and the storage required for every user is of
a very small constant size.

.. _security-foundation:

Security foundation
-------------------

Decentralized network can be subverted when populated by large amounts of
identities all belonging to a single malicious adversary. This kind of attack
is called a `Sybil attack <https://en.wikipedia.org/wiki/Sybil_attack>`_. We
compare here the mitigations used in blockchain systems and in Offset against
sybil attacks.

Blockchain systems use proof of work as a safeguard against Sybil attacks. This
idea can be simply described as: "one processor, one vote". **blockchain
networks rely on the fact that computation power is rare.**

Therefore an adversary has to gain meaningful computation power before he can
obtain influence over a blockchain network. In blockchain based network, having
large computation power can provide an adversary with the ability to double
spend money.

Offset does not make use of Proof of work. Instead, Offset uses trust between
people as a safeguard against Sybil attacks. In order to use Offset, a user
has to set up mutual credit lines with a few Offset friends. Friends should be
chosen carefully! Friends will usually be people the user has real world
familiarity with, or possibly a trusted local hub.

For every Offset friend, the user sets up a credit limit. The credit limit is
the maximum amount of money the friend might owe the user. It is also the
maximum amount that the user will lose in case the relationship with this
friend is lost. Hence, **Offset relies on the fact that real life relationships
are rare**. An Offset user can spend money from his mutual credit relationships
and disappear, but it will cost him relationships that might be more
valuable than the money he spent.

.. _origin-of-money:

Origin of money
---------------

Money creation in blockchains
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Blockchain systems have a `mining
<https://en.wikipedia.org/wiki/Bitcoin#Mining>`_ mechanism for the creation of
new money. Mining is a computationaly expensive process that fills multiple
roles:

* Inserting new money (Miners are rewarded with the newly created money)
* Maintaining the blockchain consensus.

Blockchains are usually designed such that mining is initially more rewarding
to miners, and as time goes by it becomes less and less profitable. For
example, in Bitcoin, mining is designed to become `50% less profitable every
210000 blocks <https://en.bitcoin.it/wiki/Controlled_supply>`_, and the total
amount of Bitcoins ever created is limited to about 21 million.

This property of blockchains makes it more appealing for people to join early,
with the hope of becoming rich as more users join the network.


Money creation in Offset
~~~~~~~~~~~~~~~~~~~~~~~~

Money in Offset is created and destroyed by users. Offset is designed so that
the money supply changes to match the market. As the market expands, the money
supply increases. When the market shrinks, money is destroyed. Therefore, **You
will not become rich by joining Offset early**.

The total sum of balances in Offset is always zero. Consider two Offset
friends: Bob and Charli. If Bob's balance with respect to Charli is ``x``, then
Charli's balance with respect to Bob is ``-x``. The sum of those two balances
is always ``0``.

We count the amount of money in an Offset network by summing all the positive
balances. For example purposes, consider again the two Offset friends: Bob and
Charli. Suppose that initially the balance between Bob and Charli is ``0``.


.. image:: images/bob_charli_mutual_0.svg
  :alt: Zero balance between Bob and Charli

Next, assume that Bob buys a chocolate bar from Charli for the price of $2. Now
the balance between Bob and Charli is -$2 from Bob's point of view, and +$2
from Charli's point of view. In the moment of purchase, new money was created
by Bob. In this case we can say that the total amount of money in the market is
$2.

.. image:: images/bob_charli_mutual_2.svg
  :alt: Creation of money by Bob's purchase

The money created by Bob's purchase will be destroyed when a complete buying
cycle is complete: For example, Charli will use the newly created money to buy
something from Dan, which will use the money to buy something from Eve, which
will eventually buy services from Bob. When Eve buys from Bob, the money is
destroyed.

TODO: Add image demonstrating destruction of money.

.. _incentives:

Incentives
----------

Most blockchain based digital currencies reward first adopters: New money is
easier to create in the beginning. Therefore people want to join early, in
the hope of becoming rich when late users join the network.

In Bitcoin, for example, mining is designed to become `50% less profitable every
210000 blocks <https://en.bitcoin.it/wiki/Controlled_supply>`_, and the total
amount of Bitcoins ever created is limited to about 21 million.

Contrast with blockchain based currencies, **you will not become rich by joining
Offset early**. Early and late Offset users have the same money creation
power. 

The money supply in Offset matches the size of the market, and so
Offset currencies stick to their original value. Unlike blockchain based
currencies, there is no point in speculating or gambling on the future value
of Offset currencies.

Offset offers new users interest free credit, based on trust. A new user can
start using Offset by establishing Offset friendship with another Offset
user. New users do not need to spend any (traditional) money to start playing
the Offset "game".

.. _transaction-speed:

Transaction speed
-----------------

In a blockchain based digital currency, every batch of transactions has to
propagate through all the participants of the blockchain network. As a means of
avoiding money `double spending
<https://en.wikipedia.org/wiki/Double-spending>`_, participants in the
blockchain network have to perform expensive `proof of work
<https://en.wikipedia.org/wiki/Proof_of_work>`_ and achieve global consensus
over the new state of the blockchain.

A blockchain transaction is considered complete only when there is enough
certainty that it will stay inside the blockchain, and this might take a long
time to happen. 

For example, in Bitcoin new blocks are added to the blockchain
at a rate of about 1 block every 10 minutes. For small transactions most users
will want to wait at least one block, and for larger transactions where stronger
certainty is required, users will sometimes prefer to even wait 6 blocks (about
one hour).

Offset transactions are very efficient with respect to their blockchain based
counterparts. This is possible because Offset does not rely on a global
consensus to operate.

It usually takes no more than a few seconds for an Offset transaction to
complete. an Offset transaction will usually pass through only a few computers
in the network that are relevant to the transaction. Offset doesn't have to
maintain any shared ledger, and therefore no consensus or proof of work are
required.

TODO: Add image demonstrating comparison between an Offset payment and a
blockchain payment, from networking point of view.

.. _transaction-certainty:

Transaction certainty
---------------------

TODO

.. _storage:

Storage
-------

To operate a blockchain, every network node has to
store the full blockchain. For example, the size of the bitcoin blockchain
in May 2020 is more than 270GB, and it keeps growing in the rate of about 5GB
every month. 

Offset is storage efficient. In comparison, every Offset user has to save only a
few Kilobytes of information about his balances and current state, and that
amount stays constant.

TODO: Add an image comparing a blockchain storage against Offset saved balances

.. _fees:

Fees
----

TODO


.. _recipient-online:

Receive payments when offline
-----------------------------

The blockchain approach allows users to collect payments even when they are
offline. For example, it is possible to send money to a Bitcoin address even if
the recipient is not connected to the Internet.

One downside of Offset design is that Offset users have to be online in order
to collect payment. This happens because Offset payments require the recipient
to sign using his private key. The recipient is the only one knowing his
private key, and therefore he has to be online in order to collect the incoming
payment.

Mutual credit
-------------

The core idea powering Offset payments is `mutual credit
<https://en.wikipedia.org/wiki/Mutual_credit>`_: A synchronized balance
maintained between two people. Offset does not make use of any blockchain
technology. You can learn more about the economic ideas behind Offset
:doc:`here <economic>`.

TODO: Add image demonstrating mutual credit.


No single legder
----------------

Most blockchain currencies rely on a single ledger containing the current
balances of all the users and the history of all transactions in the network.
This single ledger is usually called "the blockchain". Offset does not maintain
such a ledger. Instead, every Offset user keeps his own balances locally.


.. [1] 
   There are many blockchain based digital currencies, therefore the comparison
   might fail to generalize over all of them. When in doubt, the comparison
   refers to the characteristics of Bitcoin.
