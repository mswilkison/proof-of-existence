# proof-of-existence
An overview of proof of existence on the blockchain.

###Overview
The blockchain can be used for "trusted timestamping". [1] The process, commonly referred to as proof of existence (PoE), involves taking the hash of a document, timestamping it, and putting it into the blockchain.

The reliability of PoE depends on the security of the underlying blockchain. Assuming a competitive mining process that generates publicly visible, authenticated timestamps (such as in Bitcoin), the PoE certification process can be relied upon with a high degree of confidence. However, timestamps sourced from the blockchain *do not* offer a high degree of precision or accuracy, due to the lag time in block creation as well as transaction/block propagation time across the peer-to-peer network. [2] Thus, blockchain PoE may not be appropriate for applications that require highly precise/accurate timestamps.

###Advantages
The benefit of PoE is that it removes the need for a trusted third party (a notary) to validate the existence of a document at a certain time. It guarantees timestamps and hashes of documents cannot be tampered with retroactively and ensures they will not be lost due to database failure.

###Disadvantages
As discussed above, blockchain timestamps are not especially precise or accurate.

If a PoE is contained in a block which is later deprecated by a hard fork, then it *cannot be verified* and enjoys none of the benefits listed above.

If the hashing algorithm used is vulnerable, the validity of a PoE scheme is compromised. For example, if the hashing algorithm has many collisions (different input values that hash to the same output value), an author could retroactively assert he wrote "foo" instead of "bar," or vice versa.

Using the blockchain for non-financial payment purposes is politically sensitive and has been criticized for causing blockchain bloat. Eligius/luke-jr censors `OP_RETURN` and Counterparty transactions, arguing that using the blockchain as a data store is abusive.

Finally, the legal enforceability of PoE is uncertain and needs to be tested in court.

###Protocol
Bitcoin has several mechanisms for timestamping at the protocol level, namely `OP_RETURN` (the official, preferred way) and encoding hex values into addresses.

`OP_RETURN` is an opcode that renders the transaction output provably unspendable and allows a small amount of data to be inserted. It currently allows 40 bytes of data per transaction, although an increase to 80 bytes was recently merged to master and will likely ship with 0.11. [3] 

Typically, the data under consideration is hashed prior to being recorded on the blockchain. This compresses data of arbitrary length into a fixed length that can fit into the space constraints imposed by `OP_RETURN`.

###Extensions
Basic PoE can be extended in several ways.

Non-repudiable contracts can be created and certified by requiring the counterparties to sign with their respective private keys.

Public commitments can be made to a certain value (for example, predicting the score of a football game). This is distinct from basic PoE in that it requires the committer to publicly announce the hashed message in advance. Otherwise, he could simply write an infinite number of predictions to the blockchain and reveal the correct one retroactively, while ignoring his many false predictions.

Finally, PoE pairs nicely with distributed storage protocols like Storj, IPFS, and Blockstore. For example, a PoE hash can be used to verify the integrity of data stored in one of these protocols, making it easy to detect any changes.

###Meta-Protocol Layers
There are several protocol layers that extend the blockchain's functionality as a timestamping service and/or leverage that functionality for various purposes.

**Factom** (http://factom.org/)
Factom is designed to be a data layer for the blockchain. In Factom, an application has its own Entry Chain, in which it records application-specific data. New Entry Blocks are hashed and consolidated into Factom's Directory Chain. New Directory Blocks are hashed and written to the Bitcoin blockchain every ten minutes. In this way, Factom enables massive amounts of data to be certified by the blockchain without causing excessive bloat.

![Factom diagram](https://s3.amazonaws.com/maclanewilkison/factom-diagram.png)

**Blockstore** (https://github.com/openname/blockstore)
Blockstore is a distributed hash table (DHT) on top of the Bitcoin blockchain. Its key-value pairs correspond, unsurprisingly, to the hash of the stored data and the stored data itself. The keys/data hashes are embedded into the blockchain and can be transferred between addresses. Think Namecoin, but on the Bitcoin blockchain.

![Blockstore diagram](https://s3.amazonaws.com/maclanewilkison/openname-bitcoin-dht-diagram-4.png)

**Storj** (http://storj.io/)
Storj is a decentralized storage protocol. As part of its proof of storage scheme, it publishes PoE audits to the blockchain. This serves as proof that files are appropriately stored somewhere in the network.

###Startups
There is a long list of websites offering timestamping/proof of existence/proof of publication services, most all of which use `OP_RETURN`. For the most part, they position themselves as consumer apps but do not outline clear use cases. A large portion of their users may be experimenting or writing things to the blockchain as a gimmick. [4] Some of the more notable or unique ones are described below.

**Proof of Existence** (http://www.proofofexistence.com/)
This is the original Bitcoin timestamping service. Its drag-and-drop interface calculates the `SHA256` hash of the target file (plus a `DOCPROOF` flag) client-side and writes the result to the blockchain using `OP_RETURN`. It is anonymous and does not upload or store files. It also exposes its functionality via an API (http://www.proofofexistence.com/developers). Proof of Existence charges a 5mBTC fee per proof.

**BlockSign** (https://blocksign.com/)
Extends basic PoE to support the non-repudiable contracts described previously. Supports multiple signatories and provides verification of previously signed documents.

**bitproof** (https://bitproof.io/)
Requires registration and stores your files for future viewing/retrieval via their web app. Currently free for early users. Does a better job with messaging and outlining potential consumer use cases than most.

**BtStamp** (https://itunes.apple.com/us/app/btstamp-timestamp-documents/id955250858?mt=8)
The first Proof of Existence mobile app. Currently only available on iOS.

**CryptoGraffiti** (http://www.cryptograffiti.info/)
Enables users to read/write plain text messages to the blockchain. Uses a similar mechanism as Satoshi's genesis quote, "The Times 03/Jan/2009 Chancellor on brink of second bailout for banks". [5]

**Contractual** (http://registry.contractual.io/)
Provides white label collateral registries that can be used by financial institutions to register new claims and view prior liens against moveable assets. Writes a hash of each registration to the blockchain and stores associated documents/contracts for future viewing/retrieval. Currently running on testnet.

**Others**

- Stampd (http://stampd.io/)
- Bitstamped (https://bitstamped.io/)
- Blockai (https://www.blockai.com/)
- BTProof (https://www.btproof.com/)
- DCC Blockchain Recordation Tool (http://www.digitalcurrencycouncil.com/)
  - Used to record *The Age of Cryptocurrency*

###Recommendation
Coinbase should consider extending the Wallet API to support `OP_RETURN` transactions. Possible endpoints include:

	GET /addresses/op_returns // returns array of OP_RETURN values associated with an account
    GET /transactions/:transaction_id/op_return // returns OP_RETURN value of :transaction
	POST /transactions/op_return/:value // broadcasts transaction with OP_RETURN output containing :value

Chain and/or Proof of Existence's APIs may be used as references:

- https://chain.com/docs/v1#bitcoin-address-op-returns
- http://www.proofofexistence.com/developers


######Footnotes
[1] http://en.wikipedia.org/wiki/Trusted_timestamping

[2] http://bitcoin.stackexchange.com/questions/20479/how-accurate-is-bitcoin-network-time

[3] https://github.com/Flavien/bitcoin/commit/a9306587a42eac7fb889b9c8d03140980fdf1398

[4] http://www.righto.com/2014/02/ascii-bernanke-wikileaks-photographs.html

[5] https://en.bitcoin.it/wiki/Genesis_block
