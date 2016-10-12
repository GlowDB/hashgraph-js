# Hashgraph

This is a [hashgraph](https://en.wikipedia.org/wiki/Hashgraph) implementation written in javascript. It is currently in development and not yet ready to be used. This implementation uses [IPFS](http://ipfs.io) as storage and networking backend.

The goal of this project is a hashgraph impelementation that can be used in any javascript project that is run on several nodes that require some kind of consensus. For example for a replicated log, stock exchange, ledger, state machine, etc.

The interesting feature of this implementation is that you can configure your node to only include a defined set of remote nodes in your consensus group (like a trusted network) while at the same time have all transactions publicly available using IPFS as underlying storage backend.

## Specification

For a formal specification of the hashgraph and the hashgraph consensus, please refer to the [white paper](http://www.swirlds.com/downloads/SWIRLDS-TR-2016-01.pdf). The following specification just maps the abstract idea described in the whitepaper to an existing implementation.

The specification is lifted from an [experimental implementation in python](https://github.com/Lapin0t/py-swirld/blob/ipfs/swirld.py) which seemed workable to me. A hashgraph event is stored as a node of the merkel DAG in the IPFS network.

The `Data` of the node is a JSON string with the following format:

    Data: {"c": "event peerID", "t": unix time of event (in seconds, floating point), "d": "event payload"}
  
The `Links` of the node point to the parents of the event.

    Links: [{"Name": "0", "Hash": ownParentHash}, {"Name": "1", "Hash": otherParentHash}]

## Tests

Currently this code does not use a real testing framework. 

The following script sets up multiple hashgraph repositories in `~/.hashgraph_test_X` and runs multiple nodes to test the consensus mechanism. Be sure to run a local IPFS daemon while running the tests.

    coffee tests/test_hashgraph.coffee

You can safely delete the repositories if you're not using them anymore.


## Getting Started

### Installation

The javascript implementation of IPFS is not yet fully featured (IPNS is missing). For now, this project depends on the go-lang implementation to be installed. If you're using Ubuntu, you can use this script to install it:

    curl https://raw.githubusercontent.com/buhrmi/hashgraph/master/install.sh | sh

Then install NPM and the hashgraph package:

    npm install hashgraph --save
        
### Usage

In your node application, you can create a new hashgraph node like so:

    hashgraph = require('hashgraph')(options)
    hashgraph.init()

Supported options are:

* `path`: The path to the hashgraph repository. This is an IPFS repository that stores your private key pair and a local copy of the hashgraph data. Default: `~/.hashgraph`

You can access information about your own hashgraph peer:

    hashgraph.on('ready', function() {
      console.log(hashgraph.info())
    })

The `info()` method returns an object with the following properties:

* `peerID`: Your own peer ID.
* `head`: The Hash of the last event recorded by your peer.

It's a little bit boring to run the network only with one node. You can join another peer like so:

    hashgraph.join(remotePeerID)

After joining another node on the network you can submit transactions to the network like this:

    hashgraph.on('ready', function() {
      hashgraph.sendTransaction('somePayload');
    })
    
After the transaction has been sent to the network, it will try to achieve consensus over the question where to place this transaction in the global order of all transactions in the network. Once consensus has been achieved, the hashgraph will emit an event that you can listen to:

    hashgraph.on('consensus', function(transactions) {
      // Apply transactions to state machine
      // The transactions in the `transactions` parameter are now covered by the consensus.
      // The order of the transactions is guaranteed to be the same among all participating nodes.
    })


This is all you need to know to use hashgraph with your own projects. Read on for information on how to use the [hashgraph ledger](http://github.com/buhrmi/hashgraph-ledger).
