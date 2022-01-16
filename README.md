# Whitepaper

ParaLedger: A peer to peer plugable ledger

## Abstract
Blockchain, the foundation of Bitcoin, has received extensive attentions recently. Blockchain serves as an immutable ledger which allows transactions take place in a decentralized manner. Blockchain-based applications are springing up, covering numerous fields including financial services, reputation system and Internet of Things (IoT), and so on. However, there are still many challenges of blockchain technology such as scalability and security problems waiting to be overcome. This paper provides a pluggable blockchain architecture which can be deployed using git.

## Introduction
ParaLedger /Paraˈlɛʤə/ is derived from two words, parasitee and ledger. Parasite is an organism that lives in or on an organism of another species (its host) and benefits by the host. Similarly ParaLedger relies on the functionality of git for reliability of the relay chain. Ledger, as the name suggests is a book or other collection of financial accounts. Hence ParaLedger is a peer to peer trustless blockchain which utilises git to keep an immutable record of transactions of any kind of digital data.

## Basics


## Voting
Situations might arise where the ParaLedger's relay chain has branched out. In this situation, how is it decided on which chain is the valid chain? This is where voting comes in. We are going to assume that one or the other is the valid commit and we are going to build on top of that. Ultimately, one chain is going to be longer than the other. The longer chain is the more trusted list of transactions. So that means the winner is the longest chain. 
Now, what are we going to do with the commits on the  other chain, they have to either be cherry-picked into the later transactions or the transactions get cancelled, hence the transactions have to be made again.

## Digital Signatures
Signatures are important to not have suspicious transactions on the ledger. Since we are leveraging git to function as a blockchain, suspicious activities include faking an author while committing your transaction. The tool is to sign transactions. 

Generating a digital signature, A digital signature relies on hashing and public key cryptography. When you sign data, you hash the data and encrypt the results with your private key. The encrypted hash value is called a digital signature. If you change the original data, a different digital signature will be generated.

Verifying a digital signature, Verifying a digital signature is the opposite of signing data. Verifying a signature will tell you if the signed data has changed or not. When a digital signature is verified, the signature is decrypted using the public key to produce the original hash value. The data that was signed is hashed. If the two hash values match, then the signature has been verified. 
To make it work, we generate a public key - private key pair.
The public key looks like below: 
``` bash
---- BEGIN SSH2 PUBLIC KEY ---- AAAAB3NzaC1yc2EAAAABJQAAAQB/nAmOjTmezNUDKYvEeIRf2YnwM9/uUG1d0BYs c8/tRtx+RGi7N2lUbp728MXGwdnL9od4cItzky/zVdLZE2cycOa18xBK9cOWmcKS 0A8FYBxEQWJ/q9YVUgZbFKfYGaGQxsER+A0w/fX8ALuk78ktP31K69LcQgxIsl7r NzxsoOQKJ/CIxOGMMxczYTiEoLvQhapFQMs3FL96didKr/QbrfB1WT6s3838SEaX fgZvLef1YB2xmfhbT9OXFE3FXvh2UPBfN+ffE7iiayQf/2XR+8j4N4bW30DiPtOQ LGUrH1y5X/rpNZNlWW2+jGIxqZtgWg7lTy3mXy5x836Sj/6L 
---- END SSH2 PUBLIC KEY ----
```
This is an awful lot of information. So we want to identify this public key with something shorter. A fingerprint is created by taking the SHA256 digest of that public key. Since the digest is itself long, the first 7 characters of the digest will be your final fingerprint.

``` bash
openssl dgst  -sha256 ~/.ssh/paraledger_rsa.pub
e50cf320dfd3b91ab38e8a8dc37e2cfd909ab306006357aef754105293a9bc31

cp ~/.ssh/paraledger_rsa.pub e50cf32.pub
```
This works because it is a one way function. Given a fingerprint, the public key cannot be generated. This fingerprint is an indication for your public key. Verifying fingerprints can be easy because everyone can take your fingerprint and by using openssl they can get the digest of the public key present in the .pub file and can verify the first 7 characters. 

``` bash 
openssl dgst  -sha256 e50cf32.pub
e50cf320dfd3b91ab38e8a8dc37e2cfd909ab306006357aef754105293a9bc3
```
Using open ssl, the transaction can be signed as follows,
``` bash
openssl dgst -sha256 -sign ~/.ssh/paraledger_rsa -out 20221601-transaction.sign 20221601-transaction.json
```
Commit all three files (transaction json, public key fingerprint and the signed transaction),  as a part of the transaction. 

## Wallets
Fingerprint as folders. Every user has a fingerprint and this fingerprint folder (wallet folder) is what represents that user’s wallet. Everytime a user makes a translation, it has to be committed in the wallet folder for it to be validated by the validators and to be added to the ledger while a new block is being mined. These folders can be split using git subtree. Git subtree split will take one of the folders from your working set and then go through all the commits that changes a file in that folder. It then creates another commit which has only the changes to those files. So we have a parallel history that is going on which are only the changes that changed the subtree. At the end of this, we end up with a SHA which represents the final commit. If we were to checkout to that SHA, your working set would not be the entire directory. It would just be that one subfolder.
``` bash
git subtree split -P e50cf32/ --rejoin
a734832ef897b87a434d9688a98698ed9868de98698f668fd68

git checkout a734832ef897b87a434d9688a98698ed9868de98698f668fd68 -b wallet
```
Wallet can be seen as the history of changes that only affect you and the private key pair that is stored. Wallet holders don't need to have the entire chain transactions unless they are also validators. Only validators need to have the information of the entire chain.

## Validation
Validation is a script which executes a validation program when new commits from users is pulls from remote into the node validator’s local repository.
Validating signatures:
Validators can verify the signatures by taking the sha256 digest of the transaction and verify if the public key matches the signature of the digest of the transaction with the signature file.
``` bash
openssl dgst -sha256 -verify e50cf32.pub -signature 20221601-transaction.sign 20221601-transaction.json
```
Validating balances:
Node validators also validate the balances of every user throughout the blockchain.Making sure that no one is defaulting. If everything passes, then the commit passes.

User file deletion and modification validation:

## Incentive
Validator incentive is paid in the form of gas fee. A small percentage of everyone’s transaction that the validator has validates will go to the node validator. This small percentage is deducted from the transaction which is being made on the chain and then a new transaction would be created at the end of the block transferring a percentage of the money to the node validator. This validator incentive would not be part of the main transaction, instead it is a new file that has information of all the node validators’ rewards. This new transaction is not signed by anyone. It is just signed by the person who has actually validated the main transaction file. 

## Proof of Work:
Proof-of-work is the mechanism that allows the decentralized ParaLedger to come to consensus, or agree on things like account balances and the order of transactions. This prevents users from "double spending" their coins and ensures that the ParaLedger's relay chain is tremendously difficult to attack or manipulate.

The proof-of-work protocol, ParaHash, requires miners to go through an intense race of trial and error to find the hash for a block. Only commits with a valid hash can be added to the chain.

When racing to create a hash, a miner will repeatedly put a dataset, that you can only get from downloading and running the full chain (as a miner does), through a function. The dataset gets used to generate a mixHash below a target, as dictated by the block difficulty. The best way to do this is through trial and error.

The difficulty determines the target for the hash. The lower the target, the smaller the set of valid hashes. Once generated, this is incredibly easy for other miners and clients to verify. Even if one transaction were to change, the hash would be completely different, signalling fraud.

Hashing makes fraud easy to spot. But proof-of-work as a process is also a big deterrent to attacking the relay chain.

Miners are incentivised to do this work on the ParaLedger's relay chain. There is little incentive for a subset of miners to start their own chain – it undermines the system. Blockchains rely on having a single state as a source of truth. And users will always choose the longest or "heaviest" chain.


