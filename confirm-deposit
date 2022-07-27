# Confirm a Deposit to the Exchange

Deposits are made in several stages. 
One of the stages is to go through the [Exchange server](https://docs.flare.network/exchange/architecture/#architecture-of-an-exchange), which monitors for deposits and updates the Balances database. 
Run the code below to have the Exchange server listen for deposit transactions and confirm the deposits.

On a blockchain, multiple [validators](h[ttps://docs.flare.network/tech/glossary/](https://docs.flare.network/tech/validators/)) examine newly created transactions and come to an agreement about whether they are valid. 
When they are valid, they are added to a new block on the chain and identified by a one-way cryptographic algorithm, called a _hash_. 
The one-way hash prevents transactions from being reverted. 
[Blocks](https://docs.flare.network/tech/glossary/) are added to the [blockchain](https://docs.flare.network/tech/glossary/) in chronological order and it may take a few blocks for the hash to be completed.
To prevent problems, we confirm deposits several blocks back (five blocks in our code), when the blocks are most likely to be irreversible.

Newly submitted deposits to our receiving address are labeled "pending."
After five blocks have been created, valid deposits are labeled "confirmed."

Run the javascript below to see pending and confirmed transactions and which block number they were added to.

## One-Time Setup

1. Install [node.js](https://nodejs.org/en/download/). 
On the command line, run `node -v`  to check your installation.

2. Clone the Flare repository: 

`git clone https://github.com/flare-foundation/flare.git
cd flare`

See [the Flare repository on GitHub](https://github.com/flare-foundation/flare).

3. Install the web3.js library: 

`npm install web3`

See [Adding web3.js](https://web3js.readthedocs.io).

4. Build Flare with this script:

`./scripts/build.sh`

## Run the Code

1. `cd` to `flare`

2. Run the code: 

The following code:

* Sets up the deposit by instantiating `web3`, assigning the endpoint URL, and assigning the `receivingAddress`
* Subscribes to 'pending transactions` to listen for deposit transactions and, if they are sent to the receiving address, labels them as "pending."
* Subscribes to 'newBlockHeaders` to get a block that is five blocks back from the block with the pending transactiions and labels it as "confirmed."

```javascript
// You must have web3 installed 
const Web3 = require('web3');

// Test this code on the Coston testnet. Use your own node URL for actual runtime.
// https://docs.flare.network/dev/reference/coston-testnet/
const web3 = new Web3("wss://coston-api.flare.network/ext/bc/C/ws");

// This is the testnet receiving address. Use your receiving wallet address for actual runtime.
const receivingAddress = "0x947c76694491d3fD67a73688003c4d36C8780A97";

// Monitor for newly submitted deposit transactions.
web3.eth.subscribe("pendingTransactions")
// When a new transaction hash is received...
.on("data", async (transactionHash) => {
    // retrieve the transaction.
    let tx = await web3.eth.getTransaction(transactionHash);
    // If it is directed to our address...
    if (tx.to === receivingAddress) {
        // mark it as pending.
        console.log("Transaction", tx.hash, "is pending");
    }
}).on("error", console.error);

// Retrieve blocks 5 blocks back for confirmation.
web3.eth.subscribe("newBlockHeaders")
// When a new block has been produced...
.on("data", async (blockHeader) => {
    // retrieve a block old enough to be considered confirmed.
    let block = await web3.eth.getBlock(blockHeader.number - 5);

    // Get all of its transactions.
    block.transactions.forEach(async (transactionHash) => {
        // Retrieve the transaction...
        let tx = await web3.eth.getTransaction(transactionHash);
        // If it is directed to our address...
        if (tx.to === receivingAddress) {
            // mark it as confirmed.
            console.log("Transaction", tx.hash,
                "is confirmed in block", block.number);
        }
    });
}).on("error", console.error);
```

### Outcome

You can see which transactions are pending and which are confirmed and the block number they are in.

For example,

```javascript
Transaction 0xc3f3836b9fbe867d460b70258793e6601e4ffcb7f44203e8f40aca995ec21feb is pending
Transaction 0xc3f3836b9fbe867d460b70258793e6601e4ffcb7f44203e8f40aca995ec21feb is confirmed in block 4305057
```

The server is ready to move on to the next step: Check the wallet address to find the user account it belongs to.

