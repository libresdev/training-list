_**What is Bitcoin Script?**_ Bitcoin Script is a stack-based Forth-like language that purposefully avoids loops and so is not Turing-complete. It's made up of individual opcodes. Every single transaction in Bitcoin is locked with a Bitcoin Script; when the locking transaction for a UTXO is run with the correct inputs, that UTXO can then be spent.

The fact that transactions are locked with scripts means that they can be locked in a variety of different ways, requiring a variety of different keys. In fact, we've met a number of different locking mechanisms to date, each of which used different opcodes:

### There are some types of scipts:
Here's a breakdown of common Bitcoin script types, focusing on the "sigScripts" (which are technically "scriptPubKeys") that define the conditions for spending UTXOs:

**1. Pay-to-Public-Key-Hash (P2PKH)**
* **ScriptPubKey:**  `OP_DUP OP_HASH160 <pubkeyhash> OP_EQUALVERIFY OP_CHECKSIG`
* **Description:**  This is the most common script type. It requires a signature using the public key corresponding to the provided `pubkeyhash`.
* **Spending:**
    * The transaction's input script (sigScript) must include the signature for the public key and the public key itself.
    * The `pubkeyhash` is then derived from the public key and compared to the `pubkeyhash` embedded within the `scriptPubKey`.

**2. Pay-to-Public-Key (P2PK)**
* **ScriptPubKey:** `<pubkey> OP_CHECKSIG` 
* **Description:**  This script directly uses a public key without a hash.  
* **Spending:**
    * The transaction's input script must include the signature that verifies against the public key in the `scriptPubKey`.
* **Considerations:**
    * P2PK is less common because it exposes the public key directly, potentially increasing the risk of key compromise.

**3. Pay-to-Script-Hash (P2SH)**
* **ScriptPubKey:** `OP_HASH160 <scripthash> OP_EQUAL` 
* **Description:** This script type uses a hash of a script (the `scripthash`) and allows for greater flexibility in the underlying script logic.
* **Spending:**
    * The transaction's input script must provide the actual script (the one whose hash is in the `scriptPubKey`).
    * The hash of this script is calculated and compared to the `scripthash` in the `scriptPubKey`.
* **Advantages:**
    * P2SH enables more complex scripts and multi-signature transactions while keeping the `scriptPubKey` concise.

**4. Multi-Signature (Multisig)**
* **ScriptPubKey:**  Various forms, typically involving `OP_CHECKMULTISIG` or `OP_CHECKMULTISIGVERIFY`.
* **Description:** Requires multiple signatures from different parties before a transaction can be spent.  
* **Example (2-of-3 Multisig):** `OP_2 <pubkey1> <pubkey2> <pubkey3> OP_3 OP_CHECKMULTISIG`
    * This example requires at least two signatures from the three provided public keys to spend.

**5. Pay-to-Witness-Public-Key-Hash (P2WPKH)**
* **ScriptPubKey:** `0x0014 <pubkeyhash>` 
* **Description:** A SegWit (Segregated Witness) script type that uses a witness stack to separate the signature data from the transaction input.  
* **Advantages:**
    * Improved transaction efficiency and lower fees.
    * Enhanced privacy due to the witness stack.

**6. Pay-to-Witness-Script-Hash (P2WSH)**
* **ScriptPubKey:**  `0x0020 <scripthash>`
* **Description:** A SegWit script type that uses a witness stack for more complex scripts.
* **Advantages:**
    * Similar benefits to P2WPKH, including increased efficiency and privacy.

**Beyond the Basics:**

* **Custom Scripts:**  Bitcoin's scripting language is flexible, allowing for custom scripts beyond these common types.
* **Opcodes:** The `scriptPubKey` is composed of opcodes (operation codes) and data (e.g., public keys, hashes).  You can find a comprehensive list of opcodes in the Bitcoin documentation.
* **Script Analysis:** Tools like `bci` (from `Learning-Bitcoin-from-the-Command-Line`) and blockchain explorers help you analyze and decode scriptPubKeys.

**Remember:** Understanding script types is fundamental to working with Bitcoin transactions and UTXOs. If you're building applications or interacting with Bitcoin at a low level, it's essential to be familiar with these different script structures.


###  How to Extract address from script
ChatGPT says:
```
Extracting a Bitcoin address directly from a UTXO (Unspent Transaction Output) isn't straightforward. The information stored within a UTXO doesn't explicitly contain the Bitcoin address. Instead, it holds the scriptPubKey, which is essentially the "spending rules" for that UTXO.  

Here's how you can obtain a Bitcoin address from a UTXO using the Learning-Bitcoin-from-the-Command-Line project as a guide:

1. Access the UTXO Data

* Using bitcoin-cli: If you have a Bitcoin Core node running, you can query for UTXOs using the bitcoin-cli command. For example, to get the UTXOs associated with a specific address:   bitcoin-cli listunspent 1 confirmations 1000000 "your_bitcoin_address"
   This will return a list of UTXOs, including their txid (transaction ID), vout (output index), and scriptPubKey.

* Blockchain Explorer APIs: Many blockchain explorers offer APIs that allow you to query for UTXOs. You can use libraries like requests in Python to make API calls and retrieve the necessary data.

2. Extract the scriptPubKey

The scriptPubKey is crucial for determining the Bitcoin address. It's a hexadecimal string encoded in the UTXO information.

3. Analyze the scriptPubKey

You need to decode the scriptPubKey to identify the type of script it represents.  The Learning-Bitcoin-from-the-Command-Line project provides tools and examples for script analysis:

* bitcoin-cli: You can use the decodescript command to decode the scriptPubKey and understand its structure.   bitcoin-cli decodescript "your_scriptPubKey_hex"
   * bci (Bitcoin Command-Line Interface):  The bci tool within the project offers script analysis capabilities.   bci script decode "your_scriptPubKey_hex"
   4. Decode the Public Key Hash

The scriptPubKey often contains a public key hash (P2PKH). Once you've decoded the script, you can extract the public key hash.

5. Calculate the Bitcoin Address

* Network Prefix:  You need the appropriate network prefix (e.g., 0x00 for Bitcoin Mainnet, 0x6f for Bitcoin Testnet).
* Base58Check Encoding:  The public key hash is encoded using Base58Check, resulting in a human-readable Bitcoin address.

Simplified Example using bci:# Assuming your UTXO data:
txid = "your_txid"
vout = "your_vout"

# Retrieve the scriptPubKey from your UTXO
scriptPubKey = bci tx getoutputscript $txid $vout

# Decode the scriptPubKey
script_details = bci script decode "$scriptPubKey"

# Extract the public key hash (if P2PKH)
pkhash = script_details.pkh  # Extract the public key hash from the decoded script

# Calculate Bitcoin Address 
address = bci address hashtopubkeyaddress "$pkhash" 
Important Considerations:

* Script Types: There are different types of scripts, each with its own structure (e.g., P2PKH, P2SH).  Make sure you correctly identify the script type to extract the address.
* Security: Avoid sharing your private keys or sensitive data.
* Libraries:  The Learning-Bitcoin-from-the-Command-Line project uses various libraries like bitcoinlib and bitcoin-cli. Ensure you have them installed.

Remember: While you can extract Bitcoin addresses from UTXOs, it's important to understand the underlying cryptography and scripting concepts. This process can be complex, and utilizing dedicated libraries and tools is recommended for accuracy and security.
```

#### TLDR: 
address if a function of public_key. sha256(public_key) is enougth to compute address. 
If public script contains public_key or it's hash, then we can compute address, otherwise we can't :(

#### golang sample
```go
package main

import (
	"encoding/hex"
	"fmt"

	"github.com/btcsuite/btcd/chaincfg"
	"github.com/btcsuite/btcd/txscript"
)

func main() {
	// Start with a standard pay-to-pubkey-hash script.
	scriptHex := "76a914128004ff2fcaf13b2b91eb654b1dc2b674f7ec6188ac"
	script, err := hex.DecodeString(scriptHex)
	if err != nil {
		fmt.Println(err)
		return
	}

	// Extract and print details from the script.
	scriptClass, addresses, reqSigs, err := txscript.ExtractPkScriptAddrs(
		script, &chaincfg.MainNetParams)
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println("Script Class:", scriptClass)
	fmt.Println("Addresses:", addresses)
	fmt.Println("Required Signatures:", reqSigs)

}

```

```
Output:

Script Class: pubkeyhash
Addresses: [12gpXQVcCL2qhTNQgyLVdCFG2Qs2px98nV]
Required Signatures: 1
```

### Разные транзакции
* [ссыль](https://www.blockchain.com/explorer/transactions/btc/a4bfa8ab6435ae5f25dae9d89e4eb67dfa94283ca751f393c1ddc5a837bbc31b) - транзакция-пазл. 
	sigscript не стандартный, получить из него адрес не возможно, потому что он создан без первичного ключа, да и формат не типовой. но ее все же потратили: [вот как](https://en.bitcoin.it/wiki/Script#Transaction_puzzle)
	т.е. возможно существование неформатных sogscript, которые могут быть потрачены - на то они на forh-подобном языке и пишутся
	
### References:
* [описание sigscript как языка](https://en.bitcoin.it/wiki/Protocol_documentation#Common_standards)
* [разборы некоторых sigscript](https://en.bitcoin.it/wiki/Script#Script_examples)
* [golang библиотека, достающая адрес](https://pkg.go.dev/github.com/btcsuite/btcd/txscript#example-ExtractPkScriptAddrs) 
* [детальное объяснение про разные форматы sigscript (см. следующие главы)](https://github.com/BlockchainCommons/Learning-Bitcoin-from-the-Command-Line/blob/master/09_1_Understanding_the_Foundation_of_Transactions.md)
* [тоже полезно :)](https://github.com/bitcoinbook/bitcoinbook/blob/develop/ch07_authorization-authentication.adoc)
