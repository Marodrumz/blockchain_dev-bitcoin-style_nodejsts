# Blockchain-bitcoin by Oghenemaro Bekaren, method in Node.js

A simple blockchain and cryptocurrency wallet illustration written in Node.js and TypeScript.

This project is meant for learning. It shows the basic ideas behind a bitcoin-style system in a small file, so new engineers, interns, and beginners can see how wallets, transactions, digital signatures, blocks, hashes, and mining fit together.

It is not a real cryptocurrency, not a real Bitcoin clone, and not production-ready software. It is a teaching demo.

Watch out for python upcoming illustration "blockchain_dev_Python".

## What This System Is

This system is a tiny blockchain simulator.

In normal language, it does this:

1. It creates people with wallets.
2. Each wallet gets a private key and a public key.
3. A wallet can send money to another wallet.
4. The sender signs the transaction with their private key.
5. The blockchain checks the signature with the sender's public key.
6. If the signature is valid, the system creates a new block.
7. Before the block is added, the system performs a simple mining task.
8. After mining succeeds, the block is added to the chain.
9. At the end, the whole chain is printed to the console.

Think of the blockchain as a notebook where every page records one transfer. Once a new page is added, it points back to the page before it. This makes a chain of records.

## The Main Parts

The full system lives in `index.ts`. It has four main classes:

### 1. `Transaction`

A transaction means "somebody is sending money to somebody else."

Each transaction stores three things:

- `amount`: how much money is being sent.
- `payer`: the public key of the person sending the money.
- `payee`: the public key of the person receiving the money.

Example idea:

```text
Satoshi sends 50 coins to Bob
```

In the code, the real names are not stored inside the transaction. Instead, the sender and receiver are identified by public keys.

The `toString()` method turns the transaction into text using `JSON.stringify(this)`. That text is important because it is what gets signed and verified.

### 2. `Block`

A block is one record in the blockchain.

Each block stores:

- `prevHash`: the hash of the block before it.
- `transaction`: the transaction stored inside this block.
- `ts`: the timestamp showing when the block was created.
- `nonce`: a random number used during mining.

The block also has a `hash` getter. This creates a SHA-256 hash from the block data.

A hash is like a digital fingerprint. If the block data changes, the fingerprint changes too.

The `prevHash` field is what links one block to the previous block. That is why it is called a blockchain: each block points backward to the block before it.

### 3. `Chain`

The chain is the blockchain itself.

It stores an array of blocks:

```ts
chain: Block[];
```

When the chain starts, it creates the first block automatically. This first block is called the genesis block.

The genesis block is special because it has no previous block before it.

In this project, the genesis block contains this starter transaction:

```text
100 coins from "genesis" to "satoshi"
```

The `Chain` class also uses a singleton pattern:

```ts
public static instance = new Chain();
```

That means the program uses one shared blockchain called `Chain.instance`.

#### `lastBlock`

This returns the most recent block in the chain.

When a new block is added, it needs to know the hash of the latest block so it can point back to it.

#### `mine(nonce)`

Mining is the system's proof-of-work step.

In this demo, mining means:

1. Start with a solution number.
2. Combine the block's nonce with the solution number.
3. Hash the result using MD5.
4. Check whether the hash starts with `0000`.
5. If it does not, increase the solution number and try again.
6. Keep trying until a valid hash is found.

This is intentionally simple. It demonstrates the idea that mining requires repeated work before a block can be added.

Important note: this demo finds a mining solution but does not store that solution inside the block afterward. So this is a learning version of proof-of-work, not a complete blockchain mining design.

#### `addBlock(transaction, senderPublicKey, signature)`

This method adds a new block to the chain.

Before adding the block, it checks that the transaction was really signed by the sender.

The process is:

1. Take the transaction.
2. Take the sender's public key.
3. Take the signature.
4. Use `crypto.createVerify('SHA256')` to verify the signature.
5. If the signature is valid, create a new block.
6. Mine the block.
7. Push the block into the chain.

If the signature is not valid, the block is not added.

### 4. `Wallet`

A wallet represents a person or account.

When a new wallet is created, the system generates an RSA key pair:

- `privateKey`: secret key used to sign transactions.
- `publicKey`: public key used by others to verify the signature.

The private key is like a password or stamp that proves ownership. It should be kept secret.

The public key is like an account number. Other people can see it and use it to verify that a transaction really came from the correct wallet.

#### `sendMoney(amount, payeePublicKey)`

This method sends money from one wallet to another.

It does the following:

1. Creates a new transaction.
2. Sets the payer to the sender's public key.
3. Sets the payee to the receiver's public key.
4. Signs the transaction using the sender's private key.
5. Sends the transaction, sender public key, and signature to the blockchain.
6. The blockchain verifies the signature and adds the block if everything is valid.

## What Happens When You Run The App

At the bottom of `index.ts`, the program creates three wallets:

```ts
const satoshi = new Wallet();
const bob = new Wallet();
const alice = new Wallet();
```

Then it sends three transactions:

```ts
satoshi.sendMoney(50, bob.publicKey);
bob.sendMoney(23, alice.publicKey);
alice.sendMoney(5, bob.publicKey);
```

That means:

1. Satoshi sends 50 coins to Bob.
2. Bob sends 23 coins to Alice.
3. Alice sends 5 coins to Bob.

For each transfer:

1. A transaction is created.
2. The sender signs it.
3. The blockchain verifies the signature.
4. A new block is created.
5. Mining starts.
6. When mining finds a valid solution, the block is added to the chain.

Finally, this line prints the blockchain:

```ts
console.log(Chain.instance);
```

The printed result shows the blockchain object and all blocks inside it.

## Simple Real-Life Explanation

Imagine three people: Satoshi, Bob, and Alice.

Each person has:

- A private stamp that only they can use.
- A public stamp checker that everyone can use.

When Bob sends money to Alice, Bob stamps the transaction with his private stamp.

The blockchain uses Bob's public stamp checker to confirm:

```text
Did Bob really approve this transaction?
```

If the answer is yes, the blockchain lets the transaction continue.

Then the computer must solve a small puzzle before writing the transaction into the notebook.

After the puzzle is solved, the transaction is added as a new page in the notebook.

That notebook is the blockchain.

## Why Hashing Matters

A hash is a fixed-looking text created from data.

In this project, block hashes are made with SHA-256.

If a block contains this:

```text
Alice sends 5 coins to Bob
```

it gets one hash.

If somebody changes the amount from 5 to 500, the hash changes.

That is why hashes are useful. They help show whether data has been changed.

Each block stores the previous block's hash. So if an old block changes, the blocks after it no longer match properly.

## Why Signatures Matter

Without signatures, anyone could pretend to be someone else.

For example, Alice could try to create a fake transaction saying:

```text
Bob sends 1000 coins to Alice
```

Digital signatures help prevent that.

Bob's wallet signs Bob's real transactions using Bob's private key. The blockchain verifies the signature using Bob's public key.

If the signature does not match, the blockchain rejects the transaction.

## Why Mining Matters

Mining adds work before a block can be accepted.

In this project, the mining puzzle is:

```text
Find a number that creates an MD5 hash starting with 0000
```

The computer keeps trying numbers until it finds one that works.

This shows the idea behind proof-of-work: adding a block should require effort.

Real blockchains use more advanced mining rules, difficulty adjustment, network consensus, and validation.

## What This Demo Does Not Do

This project is intentionally small, so it does not include many things a real cryptocurrency needs.

It does not include:

- Real account balances.
- Protection from spending more coins than a wallet owns.
- Transaction fees.
- A peer-to-peer network.
- Multiple computers agreeing on the same chain.
- Persistent storage or a database.
- A mempool for pending transactions.
- Real Bitcoin block structure.
- Difficulty adjustment.
- Full chain validation.
- Storing the mining solution in the block.
- Wallet recovery or saved private keys.

This is normal for a beginner demo. The goal is to understand the basic building blocks first.

## Project Flow In One Line

```text
Wallet creates transaction -> wallet signs transaction -> chain verifies signature -> block is mined -> block is added to blockchain
```

## Usage

Install dependencies:

```bash
npm install
```

Run the project:

```bash
npm start
```

Clone and run from GitHub:

```bash
git clone https://github.com/Marodrumz/blockchain_dev-bitcoin-style_nodejsts.git
cd blockchain_dev-bitcoin-style_nodejsts
npm install
npm start
```

## Files

- `index.ts`: main TypeScript source code.
- `index.js`: compiled JavaScript output.
- `package.json`: project scripts and dependencies.
- `tsconfig.json`: TypeScript compiler settings.

## Author

```text
Oghenemaro Paul Bekaren
Senior Software Engineer | IT Consultant
Bemcol
```
