# BinaryMerkleTree
Python3 implementation of a *binary* Merkle tree. This code is pedagogical - this is not the most efficient implementation. This implementation is simple and readable and may help the reader to better understand binary Merkle trees. Note: Ethereum blockchains use a variant of Merkle trees named [Merkle Patricia trees](https://blog.ethereum.org/2015/11/15/merkling-in-ethereum/).

### Structure
**main.py:**
* a block with 4 transactions is taken as input
* the merkle tree of the block is computed from the block
* integrity of the tree is verified
* a single transaction is verified
* the subtree of a single transaction is retrieved
* a single transaction is verified with a subtree
* a single transaction is tampered
* the integrity of the tree is verified again, failing

**transaction.py:** class for transaction, fields:
* sender
* receiver
* amount
* hash;

**block.py:** class for block, fields:
* nonce (dummy)
* timestamp (dummy)
* hash of the previous block (dummy)
* list of transactions;
* hash of the Merkle root

**node.py:** implementation of the Merkle tree, fields:
* pointer to right child (null if leaf)
* pointer to left child (null if leaf)
* hash (of the transaction if leaf)
* exposed APIs (look next section) 

### Exposed APIs for Merkle tree:
* getRightChild(node);
* getLeftChild(node);
* isLeaf(node);
* getHash(node);
* computeHash(node);
* verifyIntegrity(rootNode);
* getRelevantSubTree(transaction);
* checkTransaction(transaction, rootNode); #it's also ok a root for a subtree.
* printTree(rootNode);

### Attention
* This is not the most efficient implementation. It's a pedagogical, readable, implementation.
* A real proof-of-work is not computed: here, values `nonce`, `timestamp` and `previous hash` of a block are basically random.
* A block can contain up to thousands of transaction, but in this example only 4 transactions will be present.
* Syntax of a transaction of a block (i.e., class `transaction`) does not respect the syntax of a real Eth transaction, it's pedagogical.

## Short theory
Merkle trees are wodely used to verify any kind of data which are transferred or archived through the use of cryptographic hashes. Briefly, "verifying" means assuring that a certain data block has not been tampered/it really comes from who claims to be the sender of it. Variants of Merkle trees are mainly used in Ethereum, Bitcoin or in any other blockchain peer-to-peer network.

Merkle tree composition:
* Leaves contains the hash of actual data (e.g., transactions in Ethereum).
* Intermediate nodes (named _inodes_) contains the cryptographic hash of its two children.
* A single root node, the top of the tree, that has the same property of an inode.

The point is being able to demonstrate that a child node is really part of a certain Merkle tree. This is computational expensive because you should compute the hash of all its ancestors, which in `O` notation is `O(L)`, where `L` is the number of leaves in the tree.

Let's translate this data structure in cryptocurrency, like Ethereum (actually, there are slight differences among each of them. This short explanation is _generic enough_ to understand the concept). We assume you already know what a block is and the meaning of each component (hash, nonce, proof-of-work, and so on). When we talk about the hash of a block, we actually talk about the hash of the block header. A block header does not contain data actually (i.e., the ethereum transactions, such as "Alice->Bob;1 eth"), but only timestamp, nonce, hash of the previous block in the blockchain and the hash of the root of the Merkle tree. Leaves of a Merkle tree stores all the transactions of a block (up to thousands). As we already told, not the complete files are stored, but only the hash (on 256 bit for Bitcoin/Eth). Why? Because storing real data is too expensive. According to [this source](https://ethereum.org/en/developers/tutorials/merkle-proofs-for-offline-data-integrity/):
> Storing a 32-byte word typically costs 20,000 gas, around 6.60$ (12/2021).

Hashing the data is an option, but it is still too expensive. The hash of the root node of the tree is the only part that needs to be stored on the blockchain (hash algorithm in Ethereum blockchain: _Keccak-256_).

Why does this data structure protect from tampering transactions? Why can't an attacker simply change a transaction in a leaf, for instance from "Alice->Bob;1 Eth" to "Alice->Bob;9999 eth"? Because **the hash is propagated upward**. This means that the root of the tree has an hash which is generated by combining a bunch of information (nonce of a block, timestamp, etc.) plus the has of its children (inodes). The has of each inode depends, in turn, on the hash of the two children (two because it is a binary tree), until arriving to the leaves which stores the hash of the transaction.

Let's imagine to tamper a transaction:
1. the hash of the transaction changes;
2. the hash of the parent inode changes;
3. the hash of each inode changes as well;
4. the hash of the Merkle tree root changes;
5. the hash of the block changes, the proof-of-work for the block will fail.

The upward propagation of the hash guarantees the integrity of a block and it is the reason for which tampering a blockchain transaction has an almost null probability to be successful.

The distributed nature of the blockchain allows a node of a blockchain (a peer) to download the header of a block from a source, the tree (or a portion of the tree, from the transaction to the root) from another source and still verifying the integrity of the transaction.

## Useful links
1. [Ethereum whitepaper](https://ethereum.org/en/whitepaper/)
2. [Merkling in ethereum](https://blog.ethereum.org/2015/11/15/merkling-in-ethereum/)
3. [Blockchain demo](https://andersbrownworth.com/blockchain/blockchain)
4. [Solidity bootcamp](https://www.youtube.com/watch?v=M576WGiDBdQ)
5. [Rinkeby Faucet](https://faucets.chain.link/)
