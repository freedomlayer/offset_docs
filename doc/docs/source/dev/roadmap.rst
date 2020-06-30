Roadmap
=======

Some rough ideas about the future of Offset

Protocol
--------

* Allow payment using a single out of band file exchange (invoice file), instead
  of two files exchange (invoice + commit).

* Allow removing currency after it was added. Might require a zero balance.

* Currently the buyer pays fees. Maybe this is a wrong model and the seller
  should pay fees instead? Check incentives and safety issues around this.

* Extend/rethink index server protocol.
   * Add cyclic routes search for rebalancing?
   * Strategy for unfriending: creating a zero balance with a friend by
     rebalancing.

* Rethink numbering of token channel messages and increments during
  inconsistencies. Can this mechanism be cheated?

* Log money gained through commisions? Might be required for tax
  reports? How to do this elegantly and securely?

* Allow payment without an invoice, like a donation?

Economics
---------

* Decide on the single currency vs multiple currency debate

* Dealing with regulations: KYC (Know Your Customer), AML (Anti Money
  Laundering) and filing taxes?


Offset core
-----------

* Update compiler and dependencies

* Implement bidirectional connection attempt between nodes, along relays.
  Currently only one node attempts to connect to the other node, according to
  the order of public keys.

* Solve relay/index server stall bug? Seems to be solved when servers are restarted.

* Serialization: Should we change ``capnp`` communication serialization to something else? Possibly ``protobuf``?

* Solve issue of running ``stcompact`` on very old android devices (FiatJaf + Blackhole bug)

* Implement better searching algorithm for index servers. [`Related issue <https://github.com/freedomlayer/offset/issues/218>`__]

* Allow seeing relays connectivity status (Online/Offline). Currently only
  possible for index servers.

* Implement Offset application permissions. 

* Encrypt private key on disk: stnode + stcompact

* Use a better database [`Issue <https://github.com/freedomlayer/offset/issues/143>`__]
   * for node's database
   * For stcompact (Mobile app database)
   * Possibly use sqlite3?

* Transactions interface
   * Should different node's applications know about each other's transactions? 
   * Should there be an option to list all pending incoming/outgoing transactions? If so, how to do this efficiently?

* Safe erasure of secret data [`Issue <https://github.com/freedomlayer/offset/issues/29>`__]

* Exponential backoff for all retrying connectors [`Issue <https://github.com/freedomlayer/offset/issues/144>`__]

* Enhance signature security
   * Should add textual constant prefix to all signatures?
   * Is the signature malleable? Is this an issue? What can be done to solve?

* Replace ``ring`` cryptographic library with something else, due to testing
  issues. [`Issue <https://github.com/freedomlayer/offset/issues/167>`__]
  **Closed by** [`PR 300 <https://github.com/freedomlayer/offset/pull/300>`__]

* Add automatic tool to check security issues with Rust dependencies
  (cargo-audit) [`Issue <https://github.com/freedomlayer/offset/issues/241>`__],
  **Closed by** [`PR 301 <https://github.com/freedomlayer/offset/pull/301>`__].

Offset mobile app
-----------------

* Update compiler and dependencies

* Better colors for the Offset mobile app (Should be consistent with website)

* Choose better defaults
   * Card should be enabled when created?
   * Friend should be open when created?
   * Currency should be open when created?

* Allow sending Offset data as text? (Currently can send as files or QR codes)

* Merge incoming/outgoing transactions into one screen (Requested by awpcrypto)

* Keep purchase history (Incoming/Outgoing) forever.

* Add transaction time and date (stcompact)

* Password for locking/unlocking the Offset app.

* Support Importing/Exporting 
   * Nodes (Private key + database)
   * Transactions history?

* Make it easier to add friends, while still keeping it safe?

* Implement bill splitting using Offset? [`Issue <https://github.com/freedomlayer/offset/issues/266>`__]

* Include different assets for different architectures during build in an elegant way (Currently done using a bash script).

* Add mobile app to F-Droid store [`Issue <https://github.com/freedomlayer/offset_mobile/issues/14>`__]


Documentation
-------------

* High quality video tutorials

* Document multi-currency feature.

* Document security considerations: What makes Offset secure?

* Add FAQ page


Community
---------

- Add a discussion group / bulletin board for the project.
- Create a dummy website to check payments with a dummy currency.
