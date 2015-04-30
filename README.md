<img width="250" src="http://bitcore.io/css/images/bitcore-node-logo.png"></img>
=======

[![NPM Package](https://img.shields.io/npm/v/bitcore-node.svg?style=flat-square)](https://www.npmjs.org/package/bitcore-node)
[![Build Status](https://img.shields.io/travis/bitpay/bitcore-node.svg?branch=master&style=flat-square)](https://travis-ci.org/bitpay/bitcore-node)
[![Coverage Status](https://img.shields.io/coveralls/bitpay/bitcore-node.svg?style=flat-square)](https://coveralls.io/r/bitpay/bitcore-node)

## Prerequisites

* **Node.js v0.10.0-v0.12.x** - Download and Install [Node.js](http://www.nodejs.org/download/).

* **NPM** - Node.js package manager, should be automatically installed when you get node.js.

* **Fully-synced Bitcoin Core** - Download and Install [Bitcoin Core](http://bitcoin.org/en/download)

`bitcore-node` needs a trusted Bitcoin Core instance to run. It will connect to it
through the RPC API and bitcoin peer-to-peer protocol.

Configure Bitcoin Core to listen to RPC calls and set `txindex` to true.
The easiest way to do this is by copying `./config/bitcoin.conf` to your
bitcoin data directory (usually `~/.bitcoin` on Linux, `%appdata%\Bitcoin\` on Windows,
or `~/Library/Application Support/Bitcoin` on Mac OS X).

Bitcoin Core must be running and fully synced before running `bitcore-node`. We're planning
to remove the need of running Bitcoin Core separately. [More info](https://github.com/bitpay/bitcore-node/issues/57).

## Quick Install
  Check the Prerequisites section above before installing.

  To install `bitcore-node`, clone the main repository:

    $ git clone https://github.com/bitpay/bitcore-node && cd bitcore-node

  Install dependencies:

    $ npm install

  Run the main application:

    $ npm start

  Then open a browser and go to:

    http://localhost:8080

  Please note that the app will need to sync its internal database
  with the blockchain state, which will take some time. You can check
  sync progress at `http://localhost:8080/v1/node`.


## Configuration

`bitcore-node` is configured using [yaml](http://en.wikipedia.org/wiki/YAML) files.
The application defaults are in the [api/config/](api/config/) folder.

To run the app with different configurations, simply do:
```sh
# to start a testnet instance
NODE_ENV=testnet npm start

# to start a livenet instance
NODE_ENV=livenet npm start

# start a custom configuration instance (will usee foo.yml)
NODE_ENV=foo npm start
$  
```

A sample configuration file would contain:

```
# Sample configuration file with defaults for livenet
BitcoreHTTP:
  port: 8080                # http api port
  logging: true             # enables request logging
  BitcoreNode:              
    LevelUp: ./db           # path to database location
    network: livenet        # bitcoin network (livenet, testnet)
    NetworkMonitor:
      host: localhost       # p2p host
      port: 8333            # p2p port
    RPC:
      host: 127.0.0.1       # rpc ip
      port: 8332            # rpc port
      user: user            # rpc username
      pass: password        # rpc password
      protocol: http        #http, https
      #rejectUnauthorized: false
      #disableAgent: true
```

## Synchronization

The initial synchronization process scans the blockchain from Bitcoin Core
to update addresses and balances. `bitcore-node` needs exactly one
trusted bitcoind node to function.
[There are plans to expand this to more than one](https://github.com/bitpay/bitcore-node/issues/58).
Bitcoin core must have finished downloading the blockchain before running `bitcore-node`.

While `bitcore-node` is synchronizing the web API can be queried
(the sync process is embedded in the webserver), but there will be missing data
and incorrect balances for addresses. The 'sync' status is shown at the `/v1/node` endpoint.

While synchronizing the blockchain, `bitcore-node` listens for new blocks and
transactions relayed by the bitcoin network. Those are also stored on `bitcore-node`'s database.
In case `bitcore-node` is turned off for a period of time, it will automatically catch up on restart.


### Database

To store the blockchain and address information indexes, [LevelDB](http://leveldb.org/) is used.
By default these are stored on the project's root folder, under the name `db/` for livenet and 
`tesnet-db` for testnet. This can be changed using the configuration files.

## Development

To run `bitcore-node` in development:

```$ NODE_ENV=development gulp```

To run the tests

```$ gulp test```


## API

By default, `bitcore-node` provides a REST API at `/v1`.
The end-points are detailed in the following document (please review and comment):

https://docs.google.com/document/d/1rXQdfr8VDBSzheEn0KynnCKCMB0oAsMunkjdG4aY-18

Here is a small summary via examples:

### Node routes
GET '/v1/node'
```
{
  "sync": 0.0011844682935139,
  "peerCount": 1,
  "version": "0.0.1",
  "network": "testnet",
  "height": 445
}
```

### Block routes
GET 'v1/blocks?from=100&offset=4&limit=2'
  ```
[
  {
    "header": {
      "version": 1,
      "prevHash": "0000000040a24e14497879bdd67db948cf30edc5d0a5833e8cb2736582157b49",
      "merkleRoot": "6749762ae220c10705556799dcec9bb6a54a7b881eb4b961323a3363b00db518",
      "time": 1296699408,
      "bits": 486604799,
      "nonce": 2783774724
    },
    "transactions": [
      {
        "version": 1,
        "inputs": [
          {
            "prevTxId": "0000000000000000000000000000000000000000000000000000000000000000",
            "outputIndex": 4294967295,
            "sequenceNumber": 4294967295,
            "script": "0410104a4d011e062f503253482f"
          }
        ],
        "outputs": [
          {
            "satoshis": 5000000000,
            "script": "33 0x02dd75eb56481a1be34cbea2dac1ed1b24c703fd42eb210fbc30112df5373ecc11 OP_CHECKSIG"
          }
        ],
        "nLockTime": 0
      }
    ]
  },
  {
    "header": {
      "version": 1,
      "prevHash": "00000000a04a30baed00999ad971f807b5e742f602e013519f89eb7248c7ddf5",
      "merkleRoot": "b52fcf0359ba4dae01fece4dbf9907f459396ff755fec3af4447a150b846658f",
      "time": 1296699475,
      "bits": 486604799,
      "nonce": 2389020417
    },
    "transactions": [
      {
        "version": 1,
        "inputs": [
          {
            "prevTxId": "0000000000000000000000000000000000000000000000000000000000000000",
            "outputIndex": 4294967295,
            "sequenceNumber": 4294967295,
            "script": "0453104a4d013e062f503253482f"
          }
        ],
        "outputs": [
          {
            "satoshis": 5000000000,
            "script": "33 0x032b388f00544d231a1c964db35142e8909eb079aa533c8b70f23947a8a3002a89 OP_CHECKSIG"
          }
        ],
        "nLockTime": 0
      }
    ]
  }
]
  ```

GET '/blocks/latest'
GET '/blocks/0000000040a24e14497879bdd67db948cf30edc5d0a5833e8cb2736582157b49'
GET '/blocks/:height([0-9]+)'

### Transaction routes
GET '/transactions/:txHash([A-Fa-f0-9]{64})'
POST '/transactions/send'

### Input routes
GET '/transactions/:txHash([A-Fa-f0-9]{64})/inputs'
GET '/transactions/:txHash([A-Fa-f0-9]{64})/inputs/:index([0-9]+)'

### Output routes
GET '/transactions/:txHash([A-Fa-f0-9]{64})/outputs'
GET '/transactions/:txHash([A-Fa-f0-9]{64})/outputs/:index([0-9]+)'

### Address routes
GET '/addresses/:address'
GET '/addresses/:address/transactions'
GET '/addresses/:addresses/utxos'



## License
(The MIT License)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
