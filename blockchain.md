### Hashing
hash function translates variable length string into fixed length string. act as unique identifier, address, transaction signature, checksum and other validations.

2 compound hash functions used for blockchain:
  - hash256(d) = SHA-256(SHA-256(d))     # creates a 256 bit hash (32 byte)
  - hash160(d) = RIPEMD-160(SHA-256(d))  # creates a 160 bit hash (20 byte)

* a block id is hash256
* a transaction id is hash256
* a address is a hash160

hash values are in little endian, but network byte order is big endian. therefore we often see reversed hashs on the wire.

###Serialization
* little endian is the default byte order for serialization
* there is no use of negative integers in blockchain, integers are always unsigned
* integers come in 8, 16, 32, 64 bits, or 1, 2, 4, 8 bytes
* fixed length strings are encoded in utf-8, padded with \0 characters up to desired length.
* openssl hash algorithms generate result with MSB being the first byte. hash openssl sha256 hash function generates result in big endian
* when serializing variable length data, the result is prefixed with the length information
* varint is used to conserve serialization length, a integer type with variable length, of up to 8 bytes
* varint has 4 modes, 8 bit mode, 16 bit mode starts with header 0xfd, 32 bit mode starts with 0xfe, 64 bit mode starts with 0xff
* a serialized variable length data is just a piece of data of any size start with a varint declaring its size

### Blockchain Authentication
A blockchain is a database where everyone can look through and submit transactions to. A transaction is a exchange of property, and property is strictly bound to keys. Bitcoins are bound to keys, but bitcoins can be shared. All co-owners of a bitcoin must agree to the transaction in order for it to happen.

Bitcoin uses public key cryptography **PKC**. Bitcoin owners uses private key to spend money, while using public key to receive money.

Bitcoin keys use Elliptic curve **EC** cryptography. Bitcoin needs PKC for:
  1. keypair generation
  2. signing
  3. signature verification

blockchain is not encrypted, therefore there is no requirement on encryption or decryption. PKC is only used for signatures.

private key is used to generate signature (spending), and public key is used to verify (receive).

Example: let C be a elliptic curve function, H be a cryptographic hash function

to send transaction
```
digest = H(message)
signature = ec_sign(C, digest, private_key)
send(signature, public_key)
```

to receive transaction
```
digest = H(message)
is_auth = ec_verify(C, digest, signature, public_key)
```

### Elliptic Curve
an elliptic curve is a plane algebraic curve defined as 

y^2 = x^3 + ax + b

that is non-singular, it has no cusps or self-intersectons

Elliptic Curve Cryptography **ECC** is an approach to public key cryptography based on algebraic structure of elliptic curves over finite fields. ECC requires smaller keys compared to other PKC methods, such as Galois fields.

The intractability property of ECC comes from finding the discrete logarithm of a random elliptic curve element  with respect to publically known base point being infeasible. or the **Elliptic curve discrete logarithm problem ECDLP**. The security depend on the ability to compute point multiplication and the inability to compute the multiplicand given the original and product points. The size of the elliptic curve determines the difficulty of the problem.

The primary benefit of ECC is the smaller key size while not compromising security level. A 256bit elliptic curve public key should provide comparable security as 3072 bit RSA public key.

Elliptic curve algorithms *elliptic curve diffe-hellman ECDH* for key exchange and *elliptic curve digital signature algorithm ECDSA* for digital signature.

### Elliptic Curve Keys
facts about Bitcoin keys:
  * private key is 32 bytes (256 bits) long
  * public keys are 64 bytes (uncompressed form) long with an additional 1 byte prefix
  * The elliptic curve is secp256k1 curve
  * EC cryptography are based on modulo arithmetic

The public key is uniquely derived from private key. It has uncompressed form and compressed form.

It is possible to generate EC keys using OpenSSL

The private key is a 32 byte number chosen at random, to generate one using openssl:

```
openssl ecparam -name secp256k1 -genkey -out ec-priv.pem
```

the file can be decoded by:

```
openssl ec -in ec-priv.pem -text -noout
```

the secp256k1 curve formula: y^2 = x^3 + 7

the public key contains they value of (x, y) coordinate. the point location is represented by the private key, but it is infeasible to infer point location from the point coordinate. due to dependent nature, y can be implied from x and the curve forumula. the compressed form save space by omitting the y value.

it is possible to take the public key out of the private key using the following command:

```
openssl ec -in ec-priv.pem -pubout -out ec-pub.pem
```

and the public key can be decoded from the file:

```
openssl ec -in ec-pub.pem -pubin -text -noout
```

the uncompressed version is 65 bytes, starting with constant prefix 0x04 followed by the x coordinate, then y coordinate each in 32 bytes

to compress the public key, we just need to remove the y coordinate and change the prefix according to its value. the first byte becomes 0x02 for even value of y and 0x03 for odd value of y.

note openssl display of the key is using big endian, the least byte is the last byte.

to use openssl to compress the public key:

```
openssl ec -in ec-pub.pem -pubin -text -noout -conv_form compressed
```

the compressed public key takes 33 bytes

### ECDSA
with EC keypairs, next step is to use them to sign and verify messages. in bitcoin, bitcoin clients produce signatures to authenticate transaction, miners verify the signature to authorize and broadcast valid transactions. messages in bitcoin are fixed standard messages that are program of the transaction.

EC signing algorithm is the Elliptic Curve Digital Signing algorithm **ECDSA**. In ECDSA, all parties must agree on a Hashing algorithm, because all signing are done on the hash instead of the real message. The agreed hashing algorithms are the two mentioned: hash256() (double SHA256) and hash160 (RIPEMD-SHA)

To sign a message using openssl:

```
openssl dgst -sha256 -sign ec-priv.pem ex-message.txt > ex-signature.der
```

where ex-signature.der file is the message in **DER format**, because openSSL uses DER encoding. it's like a simple pair of big numbers (r,s) 

the signature changes each time the program is run, and is not deterministic. The signature is also called the transaction id. It is a vulnerability called transaction malleability.

to display the signature:

```
openssl dgst -sha256 -hex -sign ec-priv.pem ex-message.txt
```

to verify the signature, we use command:

```
openssl dgst -sha256 -verify ec-pub.pem -signature ex-signature.der ex-message.txt
```

if signature is verified, then the message is authentic.

#### Transient Maleability
an attack on bitcoin, specifically someone changes the unique ID of a bitcoin transaction before it is confirmed on the network. the change makes it possible for someone to pretend transaction didn't happen.

problem happens because the format of the private key was not checked (i.e. whether to use compressed or uncompressed address), this means badly formatted key can be used to sign transaction, altering the hash value.

this can be used to reject valid transactions. additional information regarding the transaction can be used to reject the failure.

#### Double Spending
spending the coin twice, a second time right before the first transaction is confirmed. If second spending was confirmed on the network before the first, a double spend happens.

### BER and DER encoding
basic type-length-value encoding of the content, where a object is encoded in the order of its type, its length and finally its value, followed by end-of-content octets.

### Network Interoperability
bitcoin is on peer-to-peer network. There are 2 subnets: mainnet, and testnet3. mainnet is where real money is exchanged, testnet3 is used for testing and bitcoins on testnet3 are worthless. There is also regtest subnet, which has a specific mode of bitcoin core where local blocks are mined immediately, and miners must comply with certain set of consensus rules. It's possible to create custom block chain on regtest with negligible processing power.

Mainnet and testnet3 differ in the TCP port being used for p2p access and the prefix of the transmitted packets.

          | port  | packet prefix
----------------------------------
mainnet   | 8333  | f9 be b4 d9
testnet3  | 18333 | 0b 11 09 07

testnet3 validation rules are much more relaxed.

notice blocks and transactions are agnostic in network information and there is no way to tell the network based on transaction data alone. A transaction can be published to multiple networks.

to avoid ambiguity, user shared entities (mainly private key and address) were made network dependent by adding magic prefixes.

### Base58 encoding
in an attempt to make keys and addresses readable, bitcoin uses base58 encoding, which is a stripped down version of base64 encoding, to display binary data in ASCII text. 
  * base58 only has alphanumeric symbols, no + and /
  * exclude 0, O, I, l for visual ambiguity
  * strings are fully selectable with double click
  * symbol values follow ASCII ordering

contrary to the majority of blockchain, base58 strings are big endian.

base58 also includes a checksum suffix to avoid typos. the checksum is the first 4 byte of hash256 payload. the payload + checksum format is **Base58Check**.

algorithm:
  1. perform hash256 on payload
  2. append first 4 byte of hash to payload
  3. encode payload + checksum into base68 encoding

### Wallet Import Format (WIF)
A WIF is made of a key (private key) and a address (public key) encoded using base58. given

| network  | hex version for private key |
|==========|=============================|
|mainnet   | 80                          |
|testnet3  | ef                          |

WIF key representation:
1. repent 80 for mainnet or ef for testnet3
2. append 01 if key will correspond to a compressed public key
3. encode base58check (including adding a 4 byte checksum)

e.g.

```
/* 1 (Testnet3 version) */

ef
16 26 07 83 e4 0b 16 73
16 73 62 2a c8 a5 b0 45
fc 3e a4 af 70 f7 27 f3
f9 e9 2b dd 3a 1d dc 42

/* 2 (yes to compressed public keys) */

ef
16 26 07 83 e4 0b 16 73
16 73 62 2a c8 a5 b0 45
fc 3e a4 af 70 f7 27 f3
f9 e9 2b dd 3a 1d dc 42
01

/* 3.1 (hash256 of step 2) */

35 06 7f 25 1e 07 d0 2b
59 ca f4 cc 77 36 20 7d
73 0d 21 88 f9 62 8f 47
a9 2a 1a 92 7d 33 7b 2a

/* 3.2 (4-bytes checksum) */

35 06 7f 25

/* 3.3 (append checksum to step 2) */

ef
16 26 07 83 e4 0b 16 73
16 73 62 2a c8 a5 b0 45
fc 3e a4 af 70 f7 27 f3
f9 e9 2b dd 3a 1d dc 42
01
35 06 7f 25

/* 3.4 (encode Base58) */

cNKkmrwHuShs2mvkVEKfXULxXhxRo3yy1cK6sq62uBp2Pc8Lsa76
```

in WIF we must know exactly if we're going to use compressed public key or not, because it will affect the address generated from the private key

WIF key takes:
1. 1 byte version
2. 32 byte ECDSA private key
3. optional 01 for compressed public key, nothing otherwise
4. 4 byte checksum

#### Address
address are the public key of the PKC keypair. e.g. a **P2PKH** address (pay to public key hash) is a hash160 that is a short form for RIPEMD-160(SHA0-256(public_key)).

Other types of addresses are P2SH or multisignature.

#### Encoding of Address for P2PKH

given version table

|network   | hex version for public key |
|==========|============================|
|mainnet   | 00                         |
|testnet3  | 6f                         |

we generate P2PKH address this way
1. perform hash160 on public key
2. prepend 00 for mainnet or 6f for testnet3
3. encode in base58check (include 4 byte checksum)

e.g.

```
/* 1 (hash160 of public key) */

6b f1 9e 55 f9 4d 98 6b
46 40 c1 54 d8 64 69 93
41 91 95 11

/* 2 (Testnet3 version) */

6f
6b f1 9e 55 f9 4d 98 6b
46 40 c1 54 d8 64 69 93
41 91 95 11

/* 3.1 (hash256 of step 2) */

41 7c 62 6a 31 b5 9b 6e
1a 0b 7f 30 36 e6 d3 49
26 61 20 cf cc e6 9b 46
69 ac a8 7f ff a9 e1 21

/* 3.2 (4-bytes checksum) */

41 7c 62 6a

/* 3.3 (append checksum to step 2) */

6f
6b f1 9e 55 f9 4d 98 6b
46 40 c1 54 d8 64 69 93
41 91 95 11
41 7c 62 6a

/* 3.4 (encode Base58) */

mqMi3XYqsPvBWtrJTk8euPWDVmFTZ5jHuK
```

P2PKH address content:
1. 1 byte version
2. 20 byte hash160 of public key
3. 4 byte checksum

NOTE compressed and uncompressed public keys yield different hash values, or different addresses, and this may lead to undesirable consequences in transaction validation.

#### Magic Version Prefix
the side effect of version byte is that bitcoin encoded strings always begin with special letters:

| Key and Address  | hex version | prefix |
|==================|=============|========|
|mainnet key       | 80          | 5,K,L  |
|testnet3 key      | ef          | 9,c    |
|mainet address    | 00          | 1      |
|testnet3 address  | 6f          | m,n    |

the prefix helps us identify the meaning of the key and address in base58 string. this is intentional

### Transaction Processing
bitcoins move through **transaction outputs**. mining is the only case bitcoins are created out of thin air, outputs are the real endpoint holding bitcoin in the blockchain. When the outputs are re-used for another transaction, they become the input of the other transaction.

Unspent transaction output **UTXO** are output that is not used up by other transactions. and a transaction can take multiple outputs as input. During each transaction, there is also a fee (for mining).

If we track all UTXO on blockchain, we find all the spendable bitcoin in the world. This is a convinient index to maintain to prevent double spending. Transaction inputs must point to a global UTXO in order to be spendable.

**Scripts** are used to do transactions, i.e. generating inputs from output. It describes the process of authenticate use of UTXO using certain public/private key and the transaction transfer.

Scripts in bitcoin are standardized low level programming language that does not allow loop construct. A script runs, terminates and generate a boolean result (whether transaction successful with authentication).

A **transaction output** contains the amount of bitcoin to transfer, and a output script. (this is not UTXO, but the script that generates UTXO)
A **transaction input** contains an output reference to previous transaction output (as input here) and a input script.

In order to validate a transaction, all transaction input must validate. Following validation process:
  1. find transaction referenced by the output
  2. find the output through its index in the transaction
  3. take the output script
  4. append the output script to the input script
  5. execute the joint script using **Script Interpreter**
  6. if script succeeds, transaction is valid

### Bitcoin Script Language
low level asm-like langauge that computes value with the assumption of a stack. each operation has an opcode.

bitcoin core has virtual processor to interpret the script machine code. 

* bitcoin script does not loop
* always terminate
* memory access is stack based

[Reference](https://en.bitcoin.it/wiki/Script)

opcodes:
opcode  0x00 push 0 to stack
opcodes 0x01 - 0x04b pushes next L length value to stack, L is the value of the opcode
opcodes 0x4c, 0x4d, 0x4e pushes custom value to stack, 0x4c takes next byte as custom value length, 0x4d takes next 2 bytes as length, and 0x4e 4 bytes, followed by actual value, value is in little endian
opcodes 0x4f pushes -1 to stack
opcodes 0x51 - 0x60 - push constant value 1 to 16 to stack, 1 is specified by 0x51, and 16 is 0x60

opcode  0x93 add, pop top 2 value from stack, add them, and push value back onto top of stack
opcode  0x94 sub
opcode  0x8f negate
opcode  0x90 abs
opcode  0x91 not
opcode  0x87 cmp, compare equal, if true pop top 2 from stack and push 1, otherwise push 0
opcode  0x76 dup, duplicate top value on stack 
opcode  0xa9 hash160, compute hash160 value based on top value of stack, pops value, then pushes in hash160 value
opcode  0xac checksig, check signature, pop top 2 stack item, first being ECDSA public key, followed by DER-encoded ECDSA signature. push 1 if check success, 0 otherwise
opcode  0xaa hash256, compute the hash256 value of the value on top of the stack
opcode  0xad checksig then verify, check the sig as 0xac then execute verify
opcode  0xae check multi-sig
opcode  0xaf check multi-sig verify

opcode  0x61 nop
opcode  0x63 if condition, if top of stack value is not 0, statement is execute. top value is poped
opcode  0x64 not if condition
opcode  0x67 else clause, if previous if or not if is not executed, following statements will
opcode  0x68 endif
opcode  0x69 verify, marking transaction invalid if top stack is not true
opcode  0x6a return, marking transaction as invalid

### The Validation Script
to build a transaction from UTXO, 2 scripts are needed:
* the unspent output script, **OS**
* the spending input script, **IS**

validation script **VS** = IS + OS , after executed, returns non-zero value if valid. validation script decides if transaction input has the right to spend the UTXO, and there is enough to spend.

Because of the danger of programming, bitcoin only accept standard scripts, only testnet3 accepts non-standard scripts. this is to avoid degenerate scripts that invalidate valid bitcoin, or stealing bitcoins from others. standard script pass the **IsStandard** test

Standard Scripts:
1. **P2PKH** pay-to-public-key-hash
2. **P2SH** pay-to-script-hash
3. **P2PH** pay-to-public-key
4. multi-signature
5. return metadata

#### P2PKH Script
The output script structure:

dup
hash160
push (hash160(public_key))
euqalverify
checksig

The input script structure:
push [signature]
push [public_key]

input script also need to append a SIGHASH_ALL constant 0x01 at the end of input script. it is related to contracts

#### Contract
a **distributed contract** is a method using bitcoin to form agreement between people via block chain. contract don't make anything posible that was previously impossible. it solves the problem of minimizing trust.

contract in bitcoin can be used to create:
1. **smart property**, property that can automatically traded or loaned by block chain
2. **transferable virtual property** are digital items that can be traded but not duplicated
3. **agents** are autonomous program that maintain its own wallet, which agent uses to buy server time, obtain money by providing services, if demand exceeds supply the agent can spawn more children that either survive or die depending on whether they can get business.
4. **distributed markets** used to implement peer to peer bond and stock trading.

theory:
bitcoin transaction has one or more inputs and outputs. each input/output has a script that contains signatures over simplified form of transaction itself.

every transaction also have lock time, allowing transaction to be pending and replaceable until agreed future time, specified either as block index or as timestamp. if lock time is reached, transaction is final.

each transaction input script has a sequence number. in a normal transaction, it just moves value around. the sequence number are all UNIT_MAX and lock time is zero. if lock time is not reached, but all sequence numbers are UNIT_MAX, the transaction is also considered final. sequence number can be used to issue new versions of the transaction without invalidating other input signatures, e.g. where each input of the transaction comes from different party, each sequence number can be incremented from value 0 independently.

signature checking is flexible because the form of transaction that is signed can be controlled through use of SIGHASH flag, which is at the end of the signature. in this way contracts can be constructed where each party can only sign part of it, allowing other parts to change without their involvement. the SIGHASH flag has 2 parts:

1. SIGHASH_ALL. the default. it indicates everything about the transaction is signed, except for the input scripts (which if signed would make it impossible to create the transaction). Note other properties of the input such as connected output and sequence numbers are signed. 
2. SIGHASH_NONE, the outputs are not signed and can be anything. This mode allows others to change the input sequence numbers.
3. SIGHASH_SINGLE. the input is signed but the sequence number is left blank, so others can create new versions of the transaction. the only output that is signed is the one at the same position as the input.

SIGHASH_ANYONECANPAY modifier can be combined with the above 3. when input is signed, other inputs can be anything.

scripts can contain CHECKMULTISIG opcode. the opcode provides n-to-m checking. can provide multiple public keys, specify number of valid signatures that must be present, the number of signatures can be less than number of public keys. output can also require multiple signatures in the form (e.g. 2)

2 <pubkey1> <pubkey2> 2 CHECKMULTISIGVERIFY

there are 2 patterns of safely creating contracts:
1. transactions are passed output of the P2P network, in partially complete or invalid form
2. two transactions are used. one is created and signed, but not broadcasted away. instead the other transaction (they payment) is braodcasted after contract is agreed to lock the money, and then the contract is broadcasted

this is to ensure people know what they are agreeing to

[reference](https://en.bitcoin.it/wiki/Contract)

#### Example Input Script Execution

example stack of script execution:
()
(signature)
(signature, public_key)
(signature, public_key, public_key)
(signature, public_key, hash160(public_key))
(signature, public_key, hash160(public_key), hash160(public_key))
(signature, publick_key)
(1)

2 key points:
1. the public key from input script must hash to the bitcoin address of the output script
2. signature in the input script must be valid for the public key
3. the message must be verified (not mentioned here)
4. note whether the public key is compressed or not is important here in order to validate public_key in VS

### Inside Transactions
transaction data contains:
1. constant values
2. array of one or more input
3. array of one or more output


#### Transaction Output
transaction output are blockchain entities holding bitcoin value. it contains

1. value in satoshis
2. length of output script
3. pointer to output script containing instructions to redeem the output value.

within a transaction, each output has an index that is its offset in the outputs array.

#### Outpoint
any transaction that is not coinbase (freshedly mined) must refer to one or more previously mined transactions. an **outpoint** is a fixed structure that expresses a pointer to an existing transaction output.

an output contains:
1. txid, the hash256 identifier of the referred transaction
2. index, 0-based offset of the output in the transaction output array

#### Input
input is an entity able to redeem value from UTXO. it contains
1. outpoint pointing to an UTXO we want to use
2. length of input script
3. input script which can be prepented to the UTXO output script, is expected to validate and succeed.
4. sequence number. mainly used for contracts, often left with value 0xffffffff for maximum sequence number

#### Transaction Data Structure
transaction data structure contains:
1. version, established by network consensus, curently with value 1
2. input length, number of input data structures
3. pointer to input array
4. output length, number of output data structures
5. point to output array
6. locktime, used by contracts, indicate when the transaction will be considred valid. non-contracts will leave with value 0

when serialized, the input and output length are prefixed. finally during serialization, a flag indicating the type of SIGHASH is added to the end.

#### Coinbase Transaction
coinbase transactions are the only transactions without real inputs. each block in blockchain has one and only coinbase transaction that rewards the block miner with the famous ever-halving amount of bitcoin (currently 25BTC)

it has a single output that sends a 25BTC plus fee to an address using an standard P2PKH output script:
OP_DUP
OP_HASH160
push HASH160
OP_EUQALVERIFY
OP_CHECKSIG

the transaction has only 1 input. since coinbase transaction creates bitcoins, it doesn't point to an UTXO. Coinbase scripts are placeholders for block metadata, like block height and block chain, name of mining software, generic binary data etc.

#### Building a Transaction
to build a transaction, we need
1. ECDSA keypair
2. blockchain history of the keypair i.e. transactions that sent money to the address of the keypair
3. third party P2PKH address
4. amount to transfer in satoshis

if we want to send S satoshis to address A via keypair K:
1. scan blockchain for UTXOs to address of K
2. build transaction output from S and A
3. gather input values from UTXOs to each S
4. for each input, generate the subject of its signature
5. generate ECDSA signature for each input subject
6. pack the transaction

a UTXO is always spent once, if there is change, it will be a output of the transaction for change. UTXO can be found searching on the network, but it is faster if there is software tracking it locally, but privacy could be a concern as you don't want a untrusted server having your address information.

#### Building the Signature Subject
the subject of transaction signature is not the transaction itself, because the signature has to be part of the transaction (chicken and egg problem). we produce a different transaction for each input, which will be embedded into the P2PKH script.

for each input I, the message to be signed is a slightly modified version of the transaction where:
1. the I script is set to the output script of the UTXO it refers to
2. input scripts other than I are truncated to zero-length
3. a SIGHASH flag is appended

if there is a single input script, we don't have to truncate any other input scripts.

example subject of the input signature

```
/* version (32-bit) */
01 00 00 00

/* number of inputs (varint) */
01

/* input UTXO txid (hash256, little-endian) */
f3 a2 7f 48 5f 98 33 c8
31 8c 49 04 03 30 7f ef
13 97 12 1b 5d d8 fe 70
77 72 36 e7 37 1c 4e f3

/* input UTXO index (32-bit) */
00 00 00 00

/* input UTXO script (varint + data) */
19 76 a9 14 6b f1 9e 55
f9 4d 98 6b 46 40 c1 54
d8 64 69 93 41 91 95 11
88 ac

/* input sequence */
ff ff ff ff

/* number of outputs (varint) */
02

/* output value (64-bit) */
e0 fe 7e 01 00 00 00 00

/* output script (varint + data) */
19 76 a9 14 18 ba 14 b3
68 22 95 cb 05 23 0e 31
fe cb 00 08 92 40 66 08
88 ac

/* change output value (64-bit) */
e0 84 b0 03 00 00 00 00

/* change output script (varint + data) */
19 76 a9 14 6b f1 9e 55
f9 4d 98 6b 46 40 c1 54
d8 64 69 93 41 91 95 11
88 ac

/* locktime (32-bit) */
00 00 00 00

/* SIGHASH (32-bit) */
01 00 00 00
```

we sign the hash256 digest of the signature message instead of the message itself with the ECDSA private key. e.g.

use the hash256 digest of the message:

```
62 44 98 0f a0 75 2e 5b
46 43 ed b3 53 fd a5 23
8a 9a 3d 44 49 16 76 78
8e fd d2 5d d6 48 55 ba
```

with private key:

```
16 26 07 83 e4 0b 16 73
16 73 62 2a c8 a5 b0 45
fc 3e a4 af 70 f7 27 f3
f9 e9 2b dd 3a 1d dc 42
```

to produce the DER signature

```
30 44 02 20 11 1a 48 2a
ba 6a fb a1 2a 6f 27 de
76 7d d4 d0 64 17 de f6
65 bd 10 0b c6 8c 42 84
5c 75 2a 8f 02 20 5e 86
f5 e0 54 b2 c6 ca c5 d6
63 66 4e 35 77 9f b0 34
38 7c 07 84 8b c7 72 44
42 ca cf 65 93 24
```

the SIGHASH is then appended again to the signature (this time 8 bit, previously 32bit) together with the compressed ECDSA public key:

```
02
82 00 6e 93 98 a6 98 6e
da 61 fe 91 67 4c 3a 10
8c 39 94 75 bf 1e 73 8f
19 df c2 db 11 db 1d 28
```

the final script is

```
47 30 44 02 20 11 1a 48
2a ba 6a fb a1 2a 6f 27
de 76 7d d4 d0 64 17 de
f6 65 bd 10 0b c6 8c 42
84 5c 75 2a 8f 02 20 5e
86 f5 e0 54 b2 c6 ca c5
d6 63 66 4e 35 77 9f b0
34 38 7c 07 84 8b c7 72
44 42 ca cf 65 93 24 01

21 02 82 00 6e 93 98 a6
98 6e da 61 fe 91 67 4c
3a 10 8c 39 94 75 bf 1e
73 8f 19 df c2 db 11 db
1d 28
```

finally the serialized transaction is

```
/* version (32-bit) */
01 00 00 00

/* number of inputs (varint) */
01

/* UTXO txid (hash256, little-endian) */
f3 a2 7f 48 5f 98 33 c8
31 8c 49 04 03 30 7f ef
13 97 12 1b 5d d8 fe 70
77 72 36 e7 37 1c 4e f3

/* UTXO index (32-bit) */
00 00 00 00

/* input script (varint + data) */
6a 47 30 44 02 20 11 1a
48 2a ba 6a fb a1 2a 6f
27 de 76 7d d4 d0 64 17
de f6 65 bd 10 0b c6 8c
42 84 5c 75 2a 8f 02 20
5e 86 f5 e0 54 b2 c6 ca
c5 d6 63 66 4e 35 77 9f
b0 34 38 7c 07 84 8b c7
72 44 42 ca cf 65 93 24
01 21 02 82 00 6e 93 98
a6 98 6e da 61 fe 91 67
4c 3a 10 8c 39 94 75 bf
1e 73 8f 19 df c2 db 11
db 1d 28 

/* UTXO sequence */
ff ff ff ff

/* number of outputs (varint) */
02

/* output value (64-bit) */
e0 fe 7e 01 00 00 00 00

/* output script (varint + data) */
19 76 a9 14 18 ba 14 b3
68 22 95 cb 05 23 0e 31
fe cb 00 08 92 40 66 08
88 ac

/* change output value (64-bit) */
e0 84 b0 03 00 00 00 00

/* change output script (varint + data) */
19 76 a9 14 6b f1 9e 55
f9 4d 98 6b 46 40 c1 54
d8 64 69 93 41 91 95 11
88 ac

/* locktime (32-bit) */
00 00 00 00
```

the txid is obtained by performing a hash256 on the transaction:

```
99 96 e2 f6 4b 6a f0 23
2d d9 c8 97 39 5c e5 1f
dd 35 e6 35 9e dd 28 55
c6 0f f8 23 d8 d6 57 d1
```

once we have the transaction we can publish it using web services instead of using p2p network.

## Wallet
bitcoin users rely on wallet to create transactions and interact with the p2p network. even bitcoin core is a wallet. well known wallets out there are electrum, hive.

core service of the wallet is building transaction, broadcast it to bitcoin network. things like change address or encryption are convinient features but not fully mandatory.

#### Keypair

a bitcoin wallet is made of a WIF-encoded private key, base58check-encoded hash160 of the public key, the P2PKH address

#### Blockchain
the blockchain could be thin or heavy weight. heavy weights like bitcoin core keep track of the full blockchain. whereas thin wallet keeps tracks of only the most recent in order to be runnable on cell phone device or slow connection device.

at this moment a bitcoin core wallet is 40GB of disk space to allocate the full block chain locally, which include broadcasted block chain since the begining of time. thin client only keeps the transactiosn that's relevant to the user.

we can scan the blockchain for input scripts contain the user's public key (sending), and the output script for the hash of user's public key (receiving)

the relevant transactions form the history of a wallet.

#### UTXO
initially all transactions are considered UTXOs, if we found one of them being the input of another transaction, then it is removed from the UTXO set. we can also compute the wallet balance from UTXOs.

#### Architecture of Wallet
1. signing module
2. public address module
3. networking module

the signing module is the one holding the most sensitive data: the private key.

reference:
[developer's guide](http://davidederosa.com/basic-blockchain-programming/)

