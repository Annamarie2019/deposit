# Confirm Deposits to an Exchange

Deposits are made in several stages. 
After the user deposits funds to their assigned reception wallet (step 1 in [Detecting Deposits](https://docs.flare.network/exchange/architecture/#detecting-deposits)), an [exchange server](https://docs.flare.network/exchange/architecture/#architecture-of-an-exchange) monitors for deposits and updates the Balances database. 
Run the code below to have an exchange server listen for deposit transactions and confirm the deposits.

On a [blockchain](https://docs.flare.network/tech/glossary/), multiple [validators](https://docs.flare.network/tech/validators/) examine newly created transactions and decide on whether they are valid and which [block](https://docs.flare.network/tech/glossary/) to add to the chain next. 
Since it takes time for all validators to agree, there is a period of time in which they might add or revert transactions to a block.
After the validation process is complete, reversing a transaction is very unlikely.
To avoid the risk of accepting a deposit from a block that could be later reverted, only transactions that are a few blocks old are considered (for example, five blocks back as in the code below). 

Newly submitted deposits to the receiving address are labeled "pending."
After five more blocks have been created, valid deposits are then labeled "confirmed."

Run the javascript below to see pending and confirmed transactions and which block number they were added to.

## One-Time Setup

1. Install [node.js](https://nodejs.org/en/download/). 

   On the command line, run `node -v`  to check your installation.

2. Clone the Flare repository: 

   ```
   git clone https://github.com/flare-foundation/flare.git
   cd flare
   ```

   See [the Flare repository on GitHub](https://github.com/flare-foundation/flare).

3. Install the web3.js library: 

   ```
   npm install web3
   ```

   Run `npm web3 -v`  to check your installation.

   See [Adding web3.js](https://web3js.readthedocs.io).

4. Build Flare with this script:

   ```
   ./scripts/build.sh
   ```

## Run the Code

The following code:

* Sets up the deposit by instantiating `web3`, assigning the endpoint URL, and assigning the `receivingAddress`.
* Subscribes to `pendingTransactions` to listen for deposit transactions and, if they are sent to the receiving address, labels them as "pending."
* Subscribes to `newBlockHeaders` to get a block that is five blocks back from the block with the pending transactions. 
If they are sent to the receiving address, it labels each transaction as "confirmed" and provides the block number.

To run the code,

1. Save the code to a .js file in your flare folder. 

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
       // If it is directed to the receiving address...
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
           // Retrieve each transaction...
           let tx = await web3.eth.getTransaction(transactionHash);
           // If it is directed to the receiving address...
           if (tx.to === receivingAddress) {
               // mark it as confirmed and display the block number.
               console.log("Transaction", tx.hash,
                   "is confirmed in block", block.number);
           }
       });
   }).on("error", console.error);
   ```

2. `cd` to `flare`

3. Run: 

   ```
   node <name of file>.js
   ```

## Outcome

You can see which transactions are pending and which are confirmed and the block number they are in.

For example,

```javascript
Transaction 0x57b7c9e13836b32ac8a65e23f140fe5ab022894a7546393757043dd3c3b8e20c is pending
Transaction 0x5819ad1bc4f5837c436599589e1ca2ddeac55c7a6c61908c1f64984887af43a3 is confirmed in block 4452518
```

The server is ready to move on to the next step: Check the wallet address to find the user account it belongs to (step 3 of [Detecting Deposits](https://docs.flare.network/exchange/architecture/#detecting-deposits)). 

