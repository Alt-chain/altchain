# Whitepaper

ParaLedger: A peer to peer plugable ledger

## Abstract
Blockchain, the foundation of Bitcoin, has received extensive attentions recently. Blockchain serves as an immutable ledger which allows transactions take place in a decentralized manner. Blockchain-based applications are springing up, covering numerous fields including financial services, reputation system and Internet of Things (IoT), and so on. However, there are still many challenges of blockchain technology such as scalability and security problems waiting to be overcome. This paper provides a pluggable blockchain architecture which can be deployed using git.

## Introduction


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

Validation:
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
