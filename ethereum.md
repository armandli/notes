## Ethereum
blockchain distributed computing platform, allowing application to run on a global computer without any downtime, censorship, fraud or third-party interference; allowing transfer of money between 2 parties without central authority.

all ethereum computers are fully connected and have a fully copy of the code and data. when code is deployed to ethereum blockchain, code is replicated across all nodes in the network. if the application stores data, the data is also replicated across all nodes.

#### Dapp
a decentralized application have the complete copy of the blockchain locally and the application only needs local resources for client interaction. however, it is not always possible to have a full blockchain, so alternative solutions such as blockchain servers and metamasks help mitigate the need to have the entire blockchain without compromising the system.

there are 2 components in ethereum blockchain:
* database - every transaction is stored in a block in the blockchain. a transaction is an action of the application. all transactions are open to public and verifiable. the transaction data cannot be tampered with, **proof of work** algorithm is used to ensure the validity of the transaction
* code - the contract written by programmers in a language **Solidity**. the solidity compiler compiles it into ethereum byte code and deploy the byte code to blockchain. the ethereum blockchain stores the contract code

 - the contract written by programmers in a language **Solidity**. the solidity compiler compiles it into ethereum byte code and deploy the byte code to blockchain. the ethereum blockchain stores the contract code - the contract written by programmers in a language **Solidity**. the solidity compiler compiles it into ethereum byte code and deploy the byte code to blockchain. the ethereum blockchain stores the contract code - the contract written by programmers in a language **Solidity**. the solidity compiler compiles it into ethereum byte code and deploy the byte code to blockchain. the ethereum blockchain stores the contract code

#### Smart Contract
function of code is to produce smart contract. a contract is agreement between multiple parties that's intended to be enforced by law. a smart contract is code written that automatically enforce the outcome of the said contract. because code is replicated in the blockchain. a smart contract cannot be stopped nor modified.

example smart contract: crowd funding. anyone can send money to contribute to a project. if total goal of project is met, money is sent to creator, otherwise money is returned to original contributor.

#### Ether
the currency of ethereum is called **ether**. ether has many denominations. the lowest denomination is wei. 1 ether = 10^18 wei. wei is the denomination to be used in smart contracts.

#### Address
address is the public key that can be shared with the public to identify an entity. private key that pairs with the address should never be shared. the address and private keys are not stored anywhere except locally for the end entity.

process of generating a key-pair:
1. generate a private key
2. derive the public key from the private key
3. compute the hash of the public key using keccak256 algorithm
4. take the last 20 bytes of the hash and use it as the addreess

#### Account
combination of a public key + the private key creates an account. an account can hold balance. there are 2 types of accounts:
1. externally owned account (EOA) - normal account with a public-private key pair. can be used to send/recv ether from/to another account, send transaction to smart contracts
2. contract account - don't have a corresponding private key. these are generated when a contract is deployed to the blockchain. can also be referred to as contracts. key features are
    * can send and recv ether the same as EOA
    * can have code associated with them unlike EOA
    * transaction have to be triggered by an EOA or another contract

#### Wallet
software that keep track of balance for an account. there are 2 types of wallet:
* non-deterministic wallet - use random private key and generate a public key for it. can generate multiple accounts but each private key has no relation to each other.
* deterministic wallet - key is derived from a single starting point called **seed**. allowing user to easily backup and restore a wallet using the seed without additional information. in some cases can create public key without the knowledge of the private key. seeds are typically serialized into human-readable words. e.g. metamask asks for a 12 word seed.

#### Gas
there is cost associated with each transaction. you have to pay ether to miners in the network to execute a transaction on the blockchain.

there is specification on how many units of work a contract has. e.g. a contract adds 2 numbers together, that's 3 unit of cost; a contract that multiplies 2 numbers cost 5 unit of work. the unit of work is called gas.

#### Gas Price
this is set by the user. gas price is the price per gas user is willing to pay. the higher the price, the faster the transaction is mined.

#### Gas Limit
to avoid having a transaction costing more ether than expected, a gas limit can be set to indicate the maximum amount of gas the user is willing to buy for the transaction. this is because sometimes it is difficult to know the exact amount of gas needed for a transaction.

#### Ethereum Virtual Machine (EVM)
a truing complete 256bit VM that allows anyone to execute EVM byte code. 

#### Geth
official client software. Written in Go. it connect to the network and always try to be up to date with the blockchain. it can mine blocks and add transaction to the blockchain, validate the transaction, and execute the transaction. it also expose a server that can interact with through RPC.

it also has a javascript client (geth console) that can be used to communicate with blockchain.

#### Parity
implementation of ethereum network in Rust.

#### Web3JS
javascript library that can interact with ethereum blockchain. can use the library to build a front end Dapp.

#### Truffle
library used to develop Dapp. abstract away a lot of complexities of compiling and deploying contract in blockchain. also have testing framework. 

#### Embark
another library used to develop Dapp.

#### Ganache
in-memory blockchain for development process. the blockchain is for testing only and have currency of its own for testing purpsoes. both command line client `genache-cli` and GUI version available

#### Etherscan
free frontend service to explore ethereum blockchain. can display the address, transactions, block details. 

#### Remix
IDE for ethereum development

#### Reference
Zastrin
