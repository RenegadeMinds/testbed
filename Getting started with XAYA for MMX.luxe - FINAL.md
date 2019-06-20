# Getting Started with XAYA for MMX and .luxe Domains

The following guide is to help .luxe domain name resellers get started with XAYA and .luxe domains. For further usage instructions beyond this document, refer to the documentation in the [luxe GitHub repository here](https://github.com/xaya/luxe).

## Building Binaries

You must build 3 binaries from source:

1. xayad
1. libxayagame
1. luxed


## Build xayad

xayad is the daemon for the XAYA blockchain and wallet.

For instructions to build xayad on Ubuntu, see [here](https://github.com/xaya/xaya_tutorials/wiki/Build-xayad).


## Build libxayagame

libxayagame is the daemon for dapps and is required to build luxed. Together with luxed, they form the Game State Processor (GSP). [See here for more information about GSPs](https://github.com/xaya/xaya_tutorials/wiki/The-Game-Processor).

For a tutorial on how to build libxayagame on Ubuntu, see [here](https://github.com/xaya/xaya_tutorials/wiki/How-to-Compile-libxayagame-in-Ubuntu).

## Build luxed 

luxed is the Game State Processor (GSP) logic for MMX .luxe domains. Run the following commands to build luxed.

    cd ~/
    git clone https://github.com/xaya/luxe
    cd luxe
    ./autogen.sh
    ./configure
    make
    sudo make install
    sudo ldconfig

For more information about GSPs, see [this tutorial](https://github.com/xaya/xaya_tutorials/wiki/The-Game-Processor).

## Run xayad with correct options
You must first create a XAYA configuration file, i.e. xaya.conf. Create the file as shown below.

    mkdir ~/.xaya
    nano ~/.xaya/xaya.conf

In the xaya.conf file, enter the following. Make certain to create your own username and password.

    rpcuser=username
    rpcpassword=password
    rpcallowip=127.0.0.1
    zmqpubgameblocks=tcp://127.0.0.1:28332

Run xayad. (It is recommended to run this in a screen. This way you can check the status in real time, at least the first time around.)

    xayad 

**NOTE:** You must wait for xayad to fully sync. This may take an hour or more the first time you sync, depending on your computer’s performance and network speed. You can check the sync status by running the xaya-cli command shown below. xaya-cli acts as a JSON-RPC client and formats commands correctly.

    xaya-cli getblockchaininfo

You can check [https://explorer.xaya.io/](https://explorer.xaya.io/) to see the current block.

## Run luxed

Create a directory for the luxe data.

    cd ~/
    mkdir .luxe

Run luxed with the following options. It is probably wise to run luxed in a screen so that you can detach and go back to it to check the real time logs. Make certain to replace `username` and `password` with the ones you used in your xaya.conf file. 

    luxed --xaya_rpc_url=http://username:password@127.0.0.1:8396 --game_rpc_port=8200 --enable_pruning=-1 --datadir="~/.luxe" -alsologtostderr

The luxed daemon will begin syncing with the XAYA blockchain. You can use the RPC interface to check its status as shown below.

    curl --data-binary '{"jsonrpc": "2.0", "id":"curltest", "method": "getcurrentstate"}" -H content-type: text/plain;' http://127.0.0.1:8200/

A response that shows "state":"up-to-date" means it is synced.

## JSON-RPC Interfaces
Both xayad and luxed expose a JSON-RPC interface. xayad uses port 8396 and luxed uses port 8200 if run with the settings in our examples above.

You can use xaya-cli, curl, or any other JSON-RPC client that you wish for xayad. More information about xaya-cli is available [here](https://github.com/xaya/xaya_tutorials/wiki/xaya-cli) and [here](https://github.com/xaya/xaya_tutorials/wiki/XAYA-RPC-Methods).

You can use curl or any other JSON-RPC client that you wish for luxed. The examples here use curl, but you are free to use any JSON-RPC client you wish.

## Name Clarifications

For our purposes here, there are 2 distinct types of names:

- XAYA names
- Domain names, i.e. .luxe domains

XAYA names exist purely on the XAYA blockchain. They are not related to domain names, but on the XAYA blockchain, they perform a similar function, i.e. they are unique identifiers.

The second type of name for our purposes here is the .luxe domain name. These are part of the ICANN system. As a reseller, to enable blockchain functionality for a .luxe domain name, you must create an association for the .luxe domain name using your XAYA name. That is, you use your XAYA name to create an entry on the XAYA blockchain and that entry’s value includes data for a .luxe domain. This will become clearer below where associations are explained.

## Create a XAYA Name

You will first need a XAYA address to receive some CHI. (CHI is the native cryptocurrency used on the XAYA blockchain.) 

Run the following command.

    xaya-cli getnewaddress

This will return a public address that you can use to receive CHI. (For testing purposes you can contact the XAYA team who can send some CHI to you.) The address will look like this: CKfbkuoLNuZfGv8qHurNJ852Jj7FgoAc1w.

Once you have received some CHI, you can check your balance with the `xaya-cli getbalance` command. 

Next, you must create a XAYA name that will be your “account” for the luxe system.

Run the following command where `<my name>` is your chosen XAYA name. 

**NOTE:** All XAYA names for the luxe system must start with “p/”.

    xaya-cli name_register "p/<my name>" "{}"

For example, your command may look like the following.

    xaya-cli name_register "p/appleexchange" "{}"

If you have successfully registered your chosen XAYA name, the return value will be a transaction ID similar to the one shown below.

    28d489d22ec3908978ce678746ad270a60254d8edd4a978d286d1251ebfb8043

You can use any JSON-RPC client that you wish, such as curl. 
The following command performs the same function as above (remember to replace `<my name>`), but uses curl instead. Replace `user:password` with the user and password in xaya.conf. 

    curl --data-binary '{"jsonrpc": "1.0", "method": "name_register", "params": ["p/<my name>", "{}"] }' http://user:password@127.0.0.1:8396/

**Always remember to create a backup.**

It is imperative that you backup your wallet in safe locations. The XAYA wallet follows the same methods as Bitcoin and all Bitcoin RPCs work with XAYA. You must make sure that you never lose your wallet or you will lose all control of coins and names that exist in it.

You can backup your wallet with the following command where “/path/to/backup/wallet.dat” is the location to save the backup file. 

    xaya-cli backupwallet /path/to/backup/wallet.dat

## Send Your XAYA Name to MMX

In order for MMX to properly register you as a reseller, you must inform them of the XAYA name you’ve chosen for your business. Consult with your management for the proper MMX contact. 

This must be completed in order for the luxed operations below to function properly, i.e. MMX must register you as a reseller before any luxed operations will work.

## Associate Domain Names

There are several operations for associating .luxe domain names. The following are examples. See the luxe Name Management documentation for further details.

First, make sure your luxe GSP is synced. This can be verified with the `getcurrentstate` RPC method. 

You can use curl to query luxed as shown below.

    curl --data-binary '{"jsonrpc": "2.0", "id":"curltest", "method": "getcurrentstate"}" -H content-type: text/plain;' http://127.0.0.1:8200/

The returned JSON should include the following name/key pair.

    "state":"up-to-date"

**NOTE:** The luxed daemon is read-only. All actions are performed on the XAYA blockchain.

In other words, to read information, you query the luxe GSP database through the luxed daemon. To write to the luxe GSP database, you must send those updates to xayad, which are entered into the XAYA blockchain then processed in the luxe GSP and entered into its database. 

![LUXED-VS-XAYAD-READ-VS-WRITE](LUXED-VS-XAYAD-READ-VS-WRITE.PNG)

<!-- ## Create and Associate New Domain Names in the luxed GSP

To enable blockchain functionality for .luxe domain names, they must be entered into the system by an authorised .luxe domain name reseller.  -->

## Create New Domain Names via Reseller

To avoid discrepancy between the MMX system and the XAYA blockchain, registrars/resellers are not allowed to create new domain names on the XAYA blockchain directly. The creation of new domains is done by MMX after receiving orders/requests from the registrar/reseller through the traditional process. This is the same for the transfer and deletion process. 

To create new domains with DNS functionality, resellers must:

1. Check the availability of the domain through the Registrar API. The registrar then checks the availability in the MMX system. If it is available, then the API returns it as available.
1. Submit a create request through the registrar API. The registrar then submits the create request to the MMX system. 
1. Once the name has been created successfully in the MMX system, MMX will then create the same name in the XAYA blockchain and assign the name to the registrar, the reseller and the owner. MMX then informs the registrar that the create request has been completed. 
1. The reseller receives confirmation through the registrar API about the completion of the purchase. 

To create new domains without DNS functionality (on-chain only), the reseller must:

1. Check the availability of the domain through the registrar API. The registrar then checks the availability with the MMX system. If it is available, then then the API returns it as available.
1. Submit an on-chain create request through the registrar API. The registrar then submits the create request to the MMX system. 
1. MMX reserves the name in the MMX system, creates the same name in the XAYA blockchain, and assigns the name to the registrar, the reseller and the owner. MMX then informs the registrar that the create request has been completed. 
1. The reseller receives confirmation through the registrar API about the completion of the purchase. 

**NOTE:** The registrar and reseller should send their XAYA name to MMX for assigning the created name on the blockchain correctly. If the owner doesn’t have a XAYA name, the registrar/reseller can assign it later once the owner creates his XAYA name. The MMX system would need to add the XAYA name as additional/optional information requested for the create request. 

<!-- ### Create New .luxe Domain Names on the Blockchain

The following command creates the .luxe domain names "foo" and "bar" in the system. Remember to replace `<my name>` with the XAYA name that you chose and sent to MMX above under “Send Your XAYA Name to MMX” (and they have confirmed it has been added as a reseller).

    xaya-cli name_update "p/<my name>" "{\"g\":{\"luxe\":{\"an\": [\"foo\", \"bar\"]}}}"

**NOTE:** It takes 30 seconds on average for a transaction to be mined into the XAYA blockchain, i.e. after you’ve created your XAYA name, it must be mined into the blockchain before you can use it. -->

### Create Associations

Once a .luxe domain name has been entered into the system as shown above, data can be associated with the domain. This data can be virtually anything. One example is associating a Bitcoin address with the domain for payments. 

You can create associations for .luxe domain names as shown below. Remember to replace `<my name>` with the XAYA name you chose above.

    xaya-cli name_update "p/<my name>" "{\"g\":{\"luxe\":{\"a\": {\"foo\": null, \"bar\": {\"btc\": \"123abc\", \"eth\": null}}}}}"

That command creates several associations. 

For the domain name “foo”, it removes all associations, i.e. sets them to null.

For the domain name “bar”, it sets “btc” to “123abc” and deletes the “eth” association by setting it to null. 

**NOTE:** XAYA blocks are mined at a rate of approximately 1 per 30 seconds. If you try to update a name successively, and a new block has not been mined, you may get a `there is already a pending update for this name` error. To reduce this restriction you can batch multiple commands in 1 name_update and/or request to have more p/names to use for reselling .luxe domain names from MMX.

<!-- ### Transfer an Existing Name

You can transfer an existing name to another .luxe domain reseller and/or a new domain owner.

    xaya-cli name_update "p/<my name>" "{\"g\":{\"luxe\":{\"tn\": {\"nametotransfer\": {\"o\": \"new owner\", \"r\": \"new reseller\"}}}}}"

That command transfers the name “nametotransfer” to its new owner (“new owner”) at the “new reseller” registrar.

See the [Name Management documentation](https://github.com/xaya/luxe/blob/master/doc/moves.md#name-management) for further details.

### Delete an Existing Name

You can delete one or more existing domain names from the system as shown below.

    xaya-cli name_update "p/<my name>" "{\"g\":{\"luxe\":{\"dn\": [\"foo\", "\bar"\]}}}"

That command deletes the domain names “foo” and “bar”. 

See the [Name Management documentation](https://github.com/xaya/luxe/blob/master/doc/moves.md#name-management) for further details. -->

## How to Lookup Names

To look up data associated with .luxe domain names, you must query the luxed daemon using the luxed binary's JSON-RPC interface with a JSON-RPC client such as curl. The following command uses curl and shows how to query the daemon for the name “bar” and the key “btc” with the `lookup` command.

    curl --data-binary '{"jsonrpc": "2.0", "id":"curltest", "method": "lookup", "params": {"name": "bar", "key": "btc"} }' http://127.0.0.1:8200

Above under “Create Associations” we created an association for the name "bar" and the key "btc" with the value "123abc", so the command immediately above would return a result similar to the one shown below.

    {
      ...context information...,
      "data": "123abc"
    }

## HTTP Proxy for Lookups

The `luxe-http-rest.py` file provides a convenient RESTful proxy to retrieve results. For example, the following RESTful URL would return the value stored for the .luxe domain name “name” in its key “key”. 

    /luxe/lookup/name/key

For example, the previous example above under “How to Lookup Names” looked up the value for the name/key pair “bar” and “btc”. The RESTful example would then be as follows.

    /luxe/lookup/bar/btc

For further information about the HTTP proxy, [refer to HTTP REST Proxy for `luxed`](https://github.com/xaya/luxe/blob/master/http-rest/README.md).





































