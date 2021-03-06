---
uti: com.xamarin.workbook
id: 7c6a7c28-9e2c-4536-8cb3-841b6173cacb
title: nethereum-managing-nonces
platforms:
- Console
packages:
- id: Nethereum.Web3
  version: 2.0.1
---

# Managing nonces with Nethereum

To prevent replay attacks (submitting the same transaction several times) Ethereum provides a transaction counter: the `nonce` parameter. Nonce keeps track of the number of times a transaction has been run by an account.

When making a transaction in Ethereum, a consecutive number should be attached to each transaction on the same account. Each node will process transactions from a specific account in a strict order according to the value of its nonce.

Therefore, failing to increment this value correctly can result in different kinds of errors. For instance, let’s say the latest transaction nonce was 121:

Reusing nonce: if we send a new transaction for the same account with a nonce of either 121 or below, the node will reject it.
“Gaps”: if we send a new transaction with a nonce of either 123 or higher, the transaction will not be processed until this gap is closed, i.e. until a transaction with nonce 122 has been processed.

## How Nethereum helps managing nonces

Each time a transaction is sent via clients like Geth or Parity, the clients confirm the transaction with a delay, this can generate duplicate nonces when several transactions are sent in a short timeframe.

To keep nonce count in sync between a Nethereum Dapp and the client it communicates with, Nethereum provides with `NonceServices`.
`NonceServices` is assigned to Nethereum's `TransactionManager` and keeps track of the nonce assigned to every transaction sent to your Ethereum client.

This workbook shows how to leverage `NonceServices` 

## prerequisites:

first, let's download the test chain matching your environment from <https://github.com/nethereum/testchains>

start a geth chain (geth-clique-linux\\, geth-clique-windows\\ or geth-clique-mac\\) using **startgeth.bat** (windows) or **startgeth.sh** (mac/linux). the chain is setup with the proof of authority consensus and will start the mining process immediately.

we then need to add nethereum's nuget package:

```csharp
#r "nethereum.web3"
```

after that, we will need to add `using` statements:

```csharp
using Nethereum.Web3;
using Nethereum.Web3.Accounts;
using Nethereum.Web3.Accounts.Managed;
using Nethereum.Signer;
using Nethereum.Hex.HexConvertors.Extensions;
using Nethereum.KeyStore;
using Nethereum.Hex.HexConvertors;
using Nethereum.Hex.HexTypes;
using System;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System.Threading.Tasks;
using Nethereum.RPC.NonceServices;
using Nethereum.RPC.TransactionReceipts;
```

## Nonce management with Nethereum `accounts` objects

In most cases, Nethereum takes care of incrementing the `nonce` automatically (unless you need to sign a raw transaction manually, we'll explain that in the next chapter).

Once you have loaded your private keys into your account, if Web3 is instantiated with that account all the transactions made using the TransactionManager, Contract deployment or Functions will be signed offline using the latest nonce.

Nonce management is insured whether you are using an `account` or a `managed account` object.

Example:
This example shows what happens to the `nonce` value when we send a transaction with a Nethereum account.
We first need to create an instance of an account, then use it to
instantiate a `web3` object.

- `web3` is the Web3 instance using the new `account` as constructor
```csharp
var privateKey = "0xb5b1870957d373ef0eeffecc6e4812c0fd08f554b37b233526acc331bf1544f7";
var account = new Nethereum.Web3.Accounts.Account(privateKey);
var web3 = new Web3(account);
```

Let's now examine what happens to the `nonce` value before and after we send a transaction:

#### Before a transaction is sent:
`txCount` represents the `nonce` object:
````csharp
var txCount = await web3.Eth.Transactions.GetTransactionCount.SendRequestAsync(account.Address);
var nonce = txCount.Value;
```

Now, let's send a simple transaction:

```csharp
var recipientAddress = "0xde0b295669a9fd93d5f28d9ec85e40f4cb697bae";
var transaction = await web3.TransactionManager.SendTransactionAsync(account.Address, recipientAddress, new HexBigInteger(1));
```

#### After a transaction has been sent

```csharp
var txCount = await web3.Eth.Transactions.GetTransactionCount.SendRequestAsync(account.Address);
var nonce = txCount.Value;

```
As the above code demonstrates, the `nonce` was automatically incremented, thanks to the use of `TransactionManager`.
