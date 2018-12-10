******
Wallet
******


How to start a wallet
=====================

Using Docker
------------
1. Install Docker. Instructions for ubuntu 16-04 `here <https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-16-04>`_, don't skip optional step 2 to run docker without ``sudo``.
2. Pull actual wallet image::

    docker pull deipdev/testnet:wallet_latest

3. Start wallet::

    docker run -it -v ~/wallet:/var/lib/wallet deipdev/testnet:wallet_latest -s ws://82.196.2.5:8090 -w /var/lib/wallet/wallet.json

* ``-v ~/data:/var/lib/wallet`` maps ``~/wallet`` folder to ``/var/lib/wallet`` folder inside container (this is wallet working directory).
* ``-s ws://82.196.2.5:8090`` specifies node wallet will connect to. ``ws://82.196.2.5:8090`` is address of Public Testnet seed full node maintained by DEIP.
* ``-w /var/lib/wallet/wallet.json`` specifies wallet file name. Please notice, that path to the file is **inside** the container.

All settings defined above are automatically set in Compose config. You can still change them manually if you want to.

Using Compose
-------------

This step is optional. You can still easily run wallet without Compose.

1. Install Docker Compose. See instructions `here <https://docs.docker.com/compose/install/#prerequisites>`_.
2. To get the latest ``docker-compose.yml`` file, clone the repository::

    git clone https://github.com/DEIPworld/deip-testnet 
    cd deip-testnet

3. Pull the latest docker image::

    docker-compose pull

4. Execute command:::

    docker-compose run --rm wallet

After this wallet will start and connect to node at 82.196.2.5 (Full-Node supported by DEIP). You can specify different address using ``-s ws://your_node_ip:your_node_port`` (i.e. ``-s ws://127.0.0.1:8090``) if you want to connect to different node. Wallet file is saved to ``data`` folder.

To stop the wallet, press ``Ctrl + P, Ctrl + Q``.

Create account
==============

In order to create an account, any other user should issue a transaction for account creation. Account who will issue this transaction will also pay account creation fee, which will be credited to new accounts Common tokens balance.
For testnet users we prepared `shared accounts <https://github.com/DEIPworld/deip-testnet/blob/master/testnet-shared-accounts.txt>`_ that you can use for testing and creation of new accounts

Create account with keys
------------------------

1. Start wallet :doc:`see this tutorial <./wallet>`
2. Set password for wallet file::

    set_password yourSecretPassword

3. Unlock the wallet::

    unlock yourSecretPassword

4. Import private key of one of the shared accounts. Let's take 'alice' for this example::

    import_key 5JGoCjh27sfuCzp7kQme5yMipaQtgdVLPiZPr9zaCwJVrSrbGYx

5. Generate new keys. You can use up to 4 keys for owner, active, posting authorities and memo; or you can use single key and update them later::

    suggest_brain_key

.. Attention:: Make sure you saved the keys!

For this example, let's say following keys were generated::

    {
    "brain_priv_key": "CHOLANE MORTIER MIZZLY ELBOW ADAYS SUNKET SUBBASS SHIMMER FLEWS KRAL EXPORT PELTATE UNBUSH CRUCIFY SULK ANNUAL",
    "wif_priv_key": "5JEHAET62qWq1Mep3qZhWe2QhTDzF2uzWwgg6DGCify56VBrUkR",
    "pub_key": "DEIP6ioSfo5gbaP3YJ3G7ivXATXSLbFLURsB4Y1MmgBFjfepW9qm6u"
    }

6. Create account. This method takes following parameters: creator name, new account name, json metadata (can leave empty), owner key, active key, posting key, memo, fee and boolean showing whether or not you want to broadcast this transaction::

    create_account_with_keys "alice" "yourAccountName" "" DEIP6ioSfo5gbaP3YJ3G7ivXATXSLbFLURsB4Y1MmgBFjfepW9qm6u \
    DEIP6ioSfo5gbaP3YJ3G7ivXATXSLbFLURsB4Y1MmgBFjfepW9qm6u DEIP6ioSfo5gbaP3YJ3G7ivXATXSLbFLURsB4Y1MmgBFjfepW9qm6u \
    DEIP6ioSfo5gbaP3YJ3G7ivXATXSLbFLURsB4Y1MmgBFjfepW9qm6u "10.000 TESTS" true

After transaction was included in block, your account should be created. To verify your account exists run following command:::

    get_account "yourAccountName"

As result you should see info about your newly created account.

7. Now you can import key of your new account to wallet (see step 4) and use it.

Become a witness
================

1. Сreate an account using wallet and remember the private active key
2. Import your account key to wallet
3. To become a witness execute ``update_witness`` command with following parameters: account name, url of page describing your intentions and ideas about supporting the network, your block signing key (public key), your proposed chain properties (account creation fee & maximum block size), boolean showing whether or not you want to broadcast this transaction. Chain properties object can be empty (default account creation fee & maximum block size values will be used), or user defined in form ``{"account_creation_fee":"1.000 TESTS","maximum_block_size":65536}``::

    update_witness "yourAccountName" "yourAccountUrl" DEIP6ioSfo5gbaP3YJ3G7ivXATXSLbFLURsB4Y1MmgBFjfepW9qm6u {} true

Once your transaction submitted and included in block, you can verify your account is in block producers list now. To gel a list of all block producers run ``list_witnesses`` command with parameters: lower bound block producer name (leave empty if you want to list all block producers), limit of how many entries to show::

    list_witnesses "" 1000

As result you should see your account name in the list.
4. To become an active block producer you should be selected into active block producers list. To do it, you must vote for yourself. You can use any of shared accounts.::

    vote_for_witness "alice" "yourAccountName" true true

Now you can verify your account received votes by running ``get_witness`` command::

    get_witness "yourAccountName"

When your account gets enough votes, you can start a block producer node by providing ``DEIPD_WITNESS_NAME`` and ``DEIPD_PRIVATE_KEY`` parameters in ``node-config.env`` and it will start block production.

Available commands
=============================

To get full list of all commands supported by wallet, execute ``help`` command while running wallet.

To get detailed information about command and all parameters, execute ``gethelp command_name``, i.e. ``gethelp create_account``

create_account
--------------

This method will genrate new owner, active, and memo keys for the new account which will be controlable by this wallet. There is a fee associated with account creation that is paid by the creator. The current account creation fee can be found with the 'info' wallet command.

Parameters:

* creator: The account creating the new account (type: const std::string&)
* newname: The name of the new account (type: const std::string &)
* json_meta: JSON Metadata associated with the new account (type: const	std::string &)
* fee: The fee to be paid for account creation. It is converted to Common tokens for new account (type: const asset &)
* broadcast: true if you wish to broadcast the transaction (type: bool)


create_account_with_keys
------------------------

This method is used by faucets to create new accounts for other users which must provide their desired keys. The resulting account may not be controllable by this wallet. There is a fee associated with account creation that is paid by the creator. The current account creation fee can be found with the 'info' wallet command.

Parameters:

* creator: The account creating the new account (type: const std::string&)
* newname: The name of the new account (type: const std::string&)
* json_meta: JSON Metadata associated with the new account (type: const std::string&)
* owner: public owner key of the new account (type: const public_key_type&)
* active: public active key of the new account (type: const public_key_type&)
* posting: public posting key of the new account (type: const public_key_type&)
* memo: public memo key of the new account (type: const public_key_type&)
* fee: The fee to paid for account creation. It is converted to Common tokens for new account (type: const asset&)
* broadcast: true if you wish to broadcast the transaction (type: bool)

update_witness
--------------

Update a witness object owned by the given account.

Parameters:

* witness_name: The name of the witness account. (type: const std::string&)
* url: A URL containing some information about the witness. The empty string makes it remain the same. (type: const std::string &)
* block_signing_key: The new block signing public key. The empty string disables block production. (type: const public_key_type &)
* props: The chain properties the witness is voting on. (type: const chain_properties &)
* broadcast: true if you wish to broadcast the transaction. (type: bool)

vote_for_witness
----------------

Vote for a witness to become a block producer. By default an account has not voted positively or negatively for a witness. The account can either vote for with positively votes or against with negative votes. The vote will remain until updated with another vote. Vote strength is determined by the accounts vesting shares.

Parameters:

* account_to_vote_with: The account voting for a witness (type: const std::string&)
* witness_to_vote_for: The witness that is being voted for (type: const std::string&)
* approve: true if the account is voting for the account to be able to be a block produce (type: bool)
* broadcast: true if you wish to broadcast the transaction (type: bool)

transfer
--------

Transfer funds from one account to another.

Parameters:

* from: The account the funds are coming from (type: const std::string&)
* to: The account the funds are going to (type: const std::string&)
* amount: The funds being transferred. i.e. "100.000 TESTS" (type: const asset&)
* memo: A memo for the transactionm, encrypted with the to account's public memo key (type: const std::string&)
* broadcast: true if you wish to broadcast the transaction (type: bool)

transfer_to_common_tokens
-------------------------

Transfer DEIP into a vesting fund represented by Common Tokens. Common Tokens are required to vesting for a minimum of one coin year and can be withdrawn once a week over a 13 weeks withdraw period.

Parameters:

* from: The account the DEIP is coming from (type: const std::string&)
* to: The account getting the Common Tokens (type: const std::string&)
* amount: The amount of DEIP to vest i.e. "100.00 TESTS" (type: const asset&)
* broadcast: true if you wish to broadcast the transaction (type: bool)

create_vesting_contract
-----------------------

Create new vesting contract

Parameters:

* creator: The account who creates vesting contract (type: const std::string&)
* owner: The account who owns tokens from contract (type: const	std::string&)
* balance: Amount to vest (i.e. "1.000 TESTS") (type: const asset&)
* vesting_duration_seconds: Duration of vesting in seconds (type: const	uint32_t&)
* vesting_cliff_seconds: Duration of vesting cliff in seconds (type: const uint32_t&)
* period_duration_seconds: Duration of withdraw period in seconds (funds will be available every period, i.e. every 3 months) (type: const uint32_t&)
* broadcast: (type: const bool)

withdraw_vesting_balance
------------------------

Withdraw from vesting contract. Only withdraws the amount available for withdrawal

Parameters:

* vesting_balance_id: The account who created vesting contract (type: const int64_t&)
* owner: The account who owns tokens from contract (type: const std::string&)
* amount: Amount to withdraw (i.e. "1.000 TESTS") (type: const asset&)
* broadcast: (type: const bool)

get_active_witnesses
--------------------

Returns the list of witnesses producing blocks in the current round (approx. 1 minute and 3 seconds or 21 Blocks)