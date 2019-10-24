# Atomic Transactions

Atomic Transactions allow parties to trade with each other in a decentralised and trustless manner. In an atomic transaction, each party gets that they have bargained for or the transaction fails. 

For example, Alice and Bob could talk to each other and decide on a trade; Alice will give Bob a magic sword from inside of a game and Bob will give Alice 10 CHI. They create a transaction and each of them sign it. When that is complete, the transaction goes through. Alice gets 10 CHI and Bob gets the sword.

## Regtest

Here we will walk through performing an atomic transaction on the Xaya regtest private network.  

To follow along in this tutorial, you will need access to 2 regtest nodes. These can be separate computers or you can run the second node inside of a virtual machine (VM) such as Oracle's VirtualBox. 

Before proceeding here, read the [regtest](Regtestnet) tutorial and set up 2 nodes. You will need to mine regtest CHI to maturity, so it's easiest to simply mine 101+ blocks on each node so that you have at least 1 spendable output. 

# Create a Xaya Name on Each Node

To start, create a Xaya name on each node using [xaya-cli](xaya-cli).

    xaya-cli -regtest name_register "p/Alice" "{}"
    xaya-cli -regtest name_register "p/Bob" "{}"

From here on in, we'll refer to each node as the names we created on each node, i.e. Alice and Bob. 

Next, check to see the names in the wallet with the `name_list` RPC command.

    xaya-cli -regtest name_list

The response will be an empty array, i.e.:

    [
    ]

So, mine 1 block remembering to replace the regtest CHI address with your own.

    xaya-cli -regtest generatetoaddress 6 cVuy5TLydBa2aqGZ3VMW73XX3fFyTYweMx

Now the `name_list` command on Bob will return something like this:

    [
      {
        "name": "p/Bob",
        "name_encoding": "utf8",
        "value": "{}",
        "value_encoding": "ascii",
        "txid": "63ad6b5ff23a8b56a2591f7a9f1066b744568d5bc771f68c24cff5c9e0f76cfa",
        "vout": 0,
        "address": "cRL9Eyb4TAJuK81RK8dSkBFYvP5XVq42ap",
        "ismine": true,
        "height": 252
      }
    ]

## Prepare to Create the Transaction

Prior to creating the transaction for Alice and Bob to sign, we must gather some information. At this point we are working on the Bob computer/node. 

Bob will get Alice's name and the data we need with the name_show RPC command.

    xaya-cli -regtest name_show "p/Alice"

Which returns something like this:

    {
      "name": "p/Alice",
      "name_encoding": "utf8",
      "value": "{}",
      "value_encoding": "ascii",
      "txid": "f2e7816fb0935827966807bfaf89225548cc68b8f478a227f82eac5ee3450024",
      "vout": 0,
      "address": "chzkSAHVvhEETHRESUZjwboK5qXbJULZsp",
      "height": 252,
      "ismine": false
    }

The important parts there are the `txid`, the `vout`, and the `address` values. We'll use them shortly. 

That gives us data that we need for Alice, but we also need a change address to send Bob's change. We can use the same address that Bob is stored in with another `name_show` command.

    xaya-cli -regtest name_show "p/Bob"

Which returns:

    {
      "name": "p/Bob",
      "name_encoding": "utf8",
      "value": "{}",
      "value_encoding": "ascii",
      "txid": "63ad6b5ff23a8b56a2591f7a9f1066b744568d5bc771f68c24cff5c9e0f76cfa",
      "vout": 0,
      "address": "cRL9Eyb4TAJuK81RK8dSkBFYvP5XVq42ap",
      "height": 252,
      "ismine": true
    }

We'll use that `address` value below for Bob.

Next, we need to run `listunspent`. Because we're running on regtest, and because we've mined so many coins, this will be a very long list. Consequently we'll output this to a file.

    xaya-cli -regtest listunspent >listunspent.txt

Open the file and have a look inside. It is a JSON array of elements similar to this:

      {
        "txid": "02278c6c501329a1da6ada7ed9b22d4c9b4b2967e2fb01888d6d86f0dfbc3f06",
        "vout": 0,
        "address": "cVuy5TLydBa2aqGZ3VMW73XX3fFyTYweMx",
        "label": "First regtest address",
        "scriptPubKey": "76a91438b74db76af458f784fec5d00096f25fbd86034d88ac",
        "amount": 50.00000000,
        "confirmations": 256,
        "spendable": true,
        "solvable": true,
        "desc": "pkh([dc105b73/0'/0'/0']03593445c4a9111b2dd0aba3fe427545a09c6e5f9612f76741ddbef49d93ac83f0)#xn5nvqq7",
        "safe": true
      }

What we are looking for is an item with coins in it, i.e. the `amount` value is sufficient for our transaction. From this we need the `txid`, the `vout`, and the `address` values.

Now we begin to assemble the data we collected above into a transaction that we will create using xaya-tx. xaya-tx is an advanced utility that is distributed with the Xaya QT wallet. It's in the same folder as xaya-cli. 

For our transaction, we will need:

1. 2 `in` values of `txid`s followed by a colon and their `vout` values
1. 3 `outaddr` values of the `address`es from above
1. 1 `nameupdate` value consisting of 3 colon separated values: 1) `0`, 2) the name to update in hexadecimal, and 3) the value to update the name with in hexadecimal

The first `in` `txid` value is from the Alice `name_show` operation above, i.e.:

    in=f2e7816fb0935827966807bfaf89225548cc68b8f478a227f82eac5ee3450024

We add a colon and the `vout` value from the `name_show` result, i.e.:

    in=f2e7816fb0935827966807bfaf89225548cc68b8f478a227f82eac5ee3450024:0

The second `in` `txid` value is from Bob's `listunspent` value that we chose above, i.e.:

    in=02278c6c501329a1da6ada7ed9b22d4c9b4b2967e2fb01888d6d86f0dfbc3f06

We add a colon and the `vout` value from the `name_show` result, i.e.:

    in=02278c6c501329a1da6ada7ed9b22d4c9b4b2967e2fb01888d6d86f0dfbc3f06:0

We now need 3 `outaddr` values. The first spends the value of the Xaya name to the same address as the name, i.e. to Alice. All Xaya names cost 0.01 CHI. So, we have:

    outaddr=0.01:chzkSAHVvhEETHRESUZjwboK5qXbJULZsp

The second is Bob's payment of 10 CHI to Alice in the same format as above, i.e.:

    outaddr=10:chzkSAHVvhEETHRESUZjwboK5qXbJULZsp

The third is the change we pay to ourselves, i.e. to Bob. However, we need to know how much that is and we must do some math.

There are several important numbers we need. The 0.01 for the name is ignored.

1. The amount of CHI in the `listunspent` value that we chose above, i.e. 50 CHI
1. The amount Bob will pay to Alice, i.e. 10 CHI
1. The amount of tx fees that we wish to spend. This can be any amount. For our purposes here we will pay 0.001 CHI
1. The amount of change which is 50 - (10 + 0.001) = 39.999

Note that the tx fee is never explicitly stated. It is the difference between #1 above and the sum of #2 and #3, i.e. #4.

    outaddr=39.999:cRL9Eyb4TAJuK81RK8dSkBFYvP5XVq42ap

It's now time to figure out what our `nameupdate` value should be. As stated above, it is 3 colon separated values: 1) `0`, 2) the name to update in hexadecimal, and 3) the value to update the name with in hexadecimal.

You can use any number of different hex to string converters online.

    https://duckduckgo.com/?q=hex+to+string

1 is trivial.

2 is just p/Alice in hex, i.e.

    702f416c696365

Since Alice is selling a sword to Bob, Alice must perform a move in the game (i.e. perform a proper `name_update` operation) that transfers the sword to Bob. Our example here is arbitrary. For whatever game the move is for, it must match the game's logic and must be a valid move in the game. Our example is simple and minimalistic. An actual `name_update` value would likely be more complex for a sophisticated game. 

    {"g":{"x":{"send":{"Bob": "Magic sword"}}}}

In hex that is:

    7b2267223a7b2278223a7b2273656e64223a7b22426f62223a20224d616769632073776f7264227d7d7d7d

Assembling the above, we have the following:

    xaya-tx -regtest -create \
     in=f2e7816fb0935827966807bfaf89225548cc68b8f478a227f82eac5ee3450024:0 \
     in=02278c6c501329a1da6ada7ed9b22d4c9b4b2967e2fb01888d6d86f0dfbc3f06:0 \
     outaddr=0.01:chzkSAHVvhEETHRESUZjwboK5qXbJULZsp \
     outaddr=10:chzkSAHVvhEETHRESUZjwboK5qXbJULZsp \
     outaddr=39.999:cRL9Eyb4TAJuK81RK8dSkBFYvP5XVq42ap \
     nameupdate=0:702f416c696365:7b2267223a7b2278223a7b2273656e64223a7b22426f62223a20224d616769632073776f7264227d7d7d7d

Or on 1 line:

    xaya-tx -regtest -create  in=f2e7816fb0935827966807bfaf89225548cc68b8f478a227f82eac5ee3450024:0 in=02278c6c501329a1da6ada7ed9b22d4c9b4b2967e2fb01888d6d86f0dfbc3f06:0 outaddr=0.01:chzkSAHVvhEETHRESUZjwboK5qXbJULZsp outaddr=10:chzkSAHVvhEETHRESUZjwboK5qXbJULZsp outaddr=39.999:cRL9Eyb4TAJuK81RK8dSkBFYvP5XVq42ap nameupdate=0:702f416c696365:7b2267223a7b2278223a7b2273656e64223a7b22426f62223a20224d616769632073776f7264227d7d7d7d

The result of that command is a transaction in hex that must be signed. It will look something like this:

    0200000002240045e35eac2ef827a278f4b868cc48552289afbf076896275893b06f81e7f20000000000ffffffff063fbcdff0866d8d8801fbe267294b9b4c2db2d97eda6adaa12913506c8c27020000000000ffffffff0340420f0000000000505207702f416c6963652b7b2267223a7b2278223a7b2273656e64223a7b22426f62223a20224d616769632073776f7264227d7d7d7d6d7576a914bd4056c5414ffc4433607537b7c83a6e3a33d6cb88ac00ca9a3b000000001976a914bd4056c5414ffc4433607537b7c83a6e3a33d6cb88ac60a169ee000000001976a91406712471ae9e7ad746e7645588f0218d7622210588ac00000000

While not strictly necessary as we created the transaction ourselves, we can verify the content of the transaction using xaya-tx again. Run the following command using the hex from your own result.

    xaya-tx -regtest -json 0200000002240045e35eac2ef827a278f4b868cc48552289afbf076896275893b06f81e7f20000000000ffffffff063fbcdff0866d8d8801fbe267294b9b4c2db2d97eda6adaa12913506c8c27020000000000ffffffff0340420f0000000000505207702f416c6963652b7b2267223a7b2278223a7b2273656e64223a7b22426f62223a20224d616769632073776f7264227d7d7d7d6d7576a914bd4056c5414ffc4433607537b7c83a6e3a33d6cb88ac00ca9a3b000000001976a914bd4056c5414ffc4433607537b7c83a6e3a33d6cb88ac60a169ee000000001976a91406712471ae9e7ad746e7645588f0218d7622210588ac00000000

You will receive output similar to the following:

    {
        "txid": "6ebbf54e6103589495642f02bb5876ca1b4445e47c4faa6258e98a21c91ca2e2",
        "hash": "6ebbf54e6103589495642f02bb5876ca1b4445e47c4faa6258e98a21c91ca2e2",
        "version": 2,
        "size": 249,
        "vsize": 249,
        "weight": 996,
        "locktime": 0,
        "vin": [
            {
                "txid": "f2e7816fb0935827966807bfaf89225548cc68b8f478a227f82eac5ee3450024",
                "vout": 0,
                "scriptSig": {
                    "asm": "",
                    "hex": ""
                },
                "sequence": 4294967295
            },
            {
                "txid": "02278c6c501329a1da6ada7ed9b22d4c9b4b2967e2fb01888d6d86f0dfbc3f06",
                "vout": 0,
                "scriptSig": {
                    "asm": "",
                    "hex": ""
                },
                "sequence": 4294967295
            }
        ],
        "vout": [
            {
                "value": 0.01000000,
                "n": 0,
                "scriptPubKey": {
                    "nameOp": {
                        "op": "name_update",
                        "name": "p/Alice",
                        "name_encoding": "utf8",
                        "value": "{\"g\":{\"x\":{\"send\":{\"Bob\": \"Magic sword\"}}}}",
                        "value_encoding": "ascii"
                    },
                    "asm": "OP_NAME_UPDATE 702f416c696365 7b2267223a7b2278223a7b2273656e64223a7b22426f62223a20224d616769632073776f7264227d7d7d7d OP_2DROP OP_DROP OP_DUP OP_HASH160 bd4056c5414ffc4433607537b7c83a6e3a33d6cb OP_EQUALVERIFY OP_CHECKSIG",
                    "hex": "5207702f416c6963652b7b2267223a7b2278223a7b2273656e64223a7b22426f62223a20224d616769632073776f7264227d7d7d7d6d7576a914bd4056c5414ffc4433607537b7c83a6e3a33d6cb88ac",
                    "reqSigs": 1,
                    "type": "pubkeyhash",
                    "addresses": [
                        "chzkSAHVvhEETHRESUZjwboK5qXbJULZsp"
                    ]
                }
            },
            {
                "value": 10.00000000,
                "n": 1,
                "scriptPubKey": {
                    "asm": "OP_DUP OP_HASH160 bd4056c5414ffc4433607537b7c83a6e3a33d6cb OP_EQUALVERIFY OP_CHECKSIG",
                    "hex": "76a914bd4056c5414ffc4433607537b7c83a6e3a33d6cb88ac",
                    "reqSigs": 1,
                    "type": "pubkeyhash",
                    "addresses": [
                        "chzkSAHVvhEETHRESUZjwboK5qXbJULZsp"
                    ]
                }
            },
            {
                "value": 39.99900000,
                "n": 2,
                "scriptPubKey": {
                    "asm": "OP_DUP OP_HASH160 06712471ae9e7ad746e7645588f0218d76222105 OP_EQUALVERIFY OP_CHECKSIG",
                    "hex": "76a91406712471ae9e7ad746e7645588f0218d7622210588ac",
                    "reqSigs": 1,
                    "type": "pubkeyhash",
                    "addresses": [
                        "cRL9Eyb4TAJuK81RK8dSkBFYvP5XVq42ap"
                    ]
                }
            }
        ],
        "hex": "0200000002240045e35eac2ef827a278f4b868cc48552289afbf076896275893b06f81e7f20000000000ffffffff063fbcdff0866d8d8801fbe267294b9b4c2db2d97eda6adaa12913506c8c27020000000000ffffffff0340420f0000000000505207702f416c6963652b7b2267223a7b2278223a7b2273656e64223a7b22426f62223a20224d616769632073776f7264227d7d7d7d6d7576a914bd4056c5414ffc4433607537b7c83a6e3a33d6cb88ac00ca9a3b000000001976a914bd4056c5414ffc4433607537b7c83a6e3a33d6cb88ac60a169ee000000001976a91406712471ae9e7ad746e7645588f0218d7622210588ac00000000"
    }

There is a lot to unpack there so for the sake of brevity it is left up to the reader to check the `vin` and `vout` values to see that the proper coins/items are sent to the proper recipients. 


