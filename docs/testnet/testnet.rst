*******
Testnet
*******

DEIP offers the Public Testnet for anyone to use. Public Testnet is the best place to start exploring DEIP blockchain. You can find all the relevant information about the public network in the next section.

What is Testnet
===============
Testnet is an alternative blockchain that is used for testing. Testnet has some very important properties which differ it from the mainnet:

* Testnet tokens **are separate and distinct from tokens in mainnet**. It allows application developers, users, and testers to experiment — without having to use real tokens or worrying about breaking the mainchain.
* Testnet **can be reset**. It is possible to reset all the supplied tokens and all the data stored in the testnet blockchain.

Run Public Testnet Node
=======================
There are two types of nodes in DEIP blockchain: **low-memory node** and **full node**.

Low-memory node only stores blockchain important data and is suitable for:

* seed nodes
* block producer nodes
* exchanges, etc.

Full node stores all the data (including data for supporting content web site) and has all plugins and APIs enabled.

=============
Initial setup
=============

Docker
------

1. Install Docker. Instructions for ubuntu 16-04 `here <https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04>`_, don't skip optional step 2 to run docker without ``sudo``.

Compose
-------

This step is optional. You can still easily run node without Compose.

1. Install Docker Compose. See instructions `here <https://docs.docker.com/compose/install/#prerequisites>`_.
2. To get the latest ``docker-compose.yml`` file, clone the repository::

    git clone https://github.com/DEIPworld/deip-testnet 
    cd deip-testnet

Pull latest images
------------------

If you want to use Docker without Compose, you need to pull actual images manually. All images reside in ``deipdev/tesnet`` repository and are tagged separately for node, full node and wallet.
Tag scheme is ``node_revision``, ``fullnode_revision`` and ``wallet_revision`` (for node, full node and wallet respectively). Actual revision is **a2aa3b6f**.

Pull actual docker images::
    
    docker pull deipdev/testnet:node_a2aa3b6f
    docker pull deipdev/testnet:fullnode_a2aa3b6f
    docker pull deipdev/testnet:wallet_a2aa3b6f

In case you want to use Compose, all you need to do is just run following command. It will automatically pull actual images::

    docker-compose pull

Now after we setup everything, we are ready to start the node.

=================
Node data storage
=================

By default node stores all data inside container in ``/var/lib/deipd`` folder.
Instruction how to map internal volume to a folder on host machine is **coming soon**.

===============
Low-memory node
===============

Run using Compose
-----------------

To make it easier to run a witness node, there is ``node-config.env`` that allows you easily set witness account name and private key.

``DEIPD_WITNESS_NAME`` - name of your witness account who will run the node 
``DEIPD_PRIVATE_KEY`` - your witness account private key

You can also set ``DEIPD_SEED_NODES`` parameter if you want your node to connect to already running nodes. Whitespace delimited list of seed nodes (with port).

Start the node::

    docker-compose up node

To start in detached mode::
    
    docker-compose up -d node

If your account is already selected to active witnesses list, your node will start producing blocks. If not, :doc:`follow this instructions <../blockchain interaction/cli-wallet/become-a-witness>` to become a witness.

Run using Docker
----------------

    docker run --env DEIPD_WITNESS_NAME=yourwitnessname --env DEIPD_PRIVATE_KEY=yourprivatekey deipdev/testnet:node_a2aa3b6f

To run in detached mode use ``-d`` flag::

    docker run -d --env DEIPD_WITNESS_NAME=yourwitnessname --env DEIPD_PRIVATE_KEY=yourprivatekey deipdev/testnet:node_a2aa3b6f


=========
Full node
=========

Run using Compose
-----------------

To run a full node no special configuration required. If you want to set seed nodes for full node, you can do it in ``fullnode-config.env``.

Launch the full node::

    docker-compose up full_node

To launch in detached mode::

    docker-compose up -d full_node

Run using Docker
----------------

    docker run deipdev/testnet:fullnode_a2aa3b6f

To run in detached mode use ``-d`` flag::

    docker run -d deipdev/testnet:fullnode_a2aa3b6f

======================
Attach to running node
======================

To see the output of node running in detached mode use docker attach command. You need to know container id or name to connect to it.

Get list of running containers:::

    docker ps

You will see id, name and some other properties of all running containers. Once you have the id (for example purpose we will use ``f92768f1dd89``):::

    docker attach f92768f1dd89

===========
Resync node
===========

You can force node to resync on startup. It means that your database will be purged and all blocks will be resynced.

Resync node using Compose
-------------------------

In node config (``config/node.env`` for witness node, ``config/fullnode.env`` for full node) uncomment the line:::

    # RESYNC_BLOCKCHAIN=1


Resync node using Docker
------------------------

Add ``-env RESYNC_BLOCKCHAIN=1`` option to your ``docker run`` command.