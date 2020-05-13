Initial Setup
=============

Hey, welcome to the advanced usage tutorial for Offset. Going through this
tutorial requires minimal familiarity using the `command line interface
<https://en.wikipedia.org/wiki/Command-line_interface>`_ on your computer.

You will also need a working desktop computer. Offset should work correctly on
Windows, Linux and macOS.

Download
--------

To use this tutorial, you first need to download an Offset release suitable for
your system. You can download Offset from the `releases page
<https://github.com/freedomlayer/offset/releases>`_. 

The downloaded file is a compressed file. Extract the files. You should now
have a directory tree similar to this:

.. code:: text

   .
   ├── bin
   │   ├── stcompact
   │   ├── stctrl
   │   ├── stindex
   │   ├── stmgr
   │   ├── stnode
   │   └── strelay
   ├── LICENSE-AGPL3
   ├── LICENSE-APACHE
   ├── LICENSE-MIT
   └── README.md

Installation
------------

No special installation is required. Copy the resulting directory to any placed
you would like on your system. For your convenience, you are recommended to add
the ``bin`` directory `to your working path
<https://askubuntu.com/questions/109381/how-to-add-path-of-a-program-to-path-environment-variable>`_.

To make sure that everything works correctly, run ``stmgr -V`` in your terminal. You
should see output like this:

.. code:: sh

   $ stmgr -V
   stmgr 0.1.0
