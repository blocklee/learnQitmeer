
## Qitmeer RPC Usage

## evm-to-utxo


```bash
Usage: qx tx-encode [-i tx-input] [-l tx-lock-time] [-o tx-output] [-v tx-version] 
  -i value
    	The set of transaction input points encoded as TXHASH:INDEX:SEQUENCE:TXTYPE. 
    	TXHASH is a Base16 transaction hash. INDEX is the 32 bit input index
    	in the context of the transaction. SEQUENCE is the optional 32 bit 
    	input sequence and defaults to the maximum value.
    	TXTYPE is type type
    	TxTypeRegular the standard tx
    	TxTypeGenesisLock the tx try to lock the genesis output to the stake pool
    	TxTypeCrossChainExport Cross chain by import tx
    	TxTypeCrossChainImport Cross chain by vm tx
    	
  -l value
    	the transaction lock time
  -o value
    	The set of transaction output data encoded as TARGET:MEER:COINID:TXTYPE. 
    	TARGET is an address (pay-to-pubkey-hash or pay-to-script-hash).
    	MEER is the 64 bit spend amount in qitmeer.COINID enum {0 => MEER,1=>ETHID}
  -v value
    	the transaction version (default 1)
```

#### nSequence 

nSequence 字段最初旨在（但从未正确实现）允许修改mempool中的事务。在这种使用中，包含n序列值低于2^32-1（0xFFFFFFFF）的输入的事务表示尚未“最终确定”的事务。此类事务将保存在mempool中，直到它被另一个花费相同输入并具有更高nSequence值的事务所取代。一旦收到输入n序列值为0xFFFFFFFFFF的事务，它将被视为“最终”并已挖掘。当 SEQUENCE 序列号设置为 最大值 4294967295 时，表示交易被视为"最终"。

nSequence的原始含义从未正确实现，在不使用时间锁的事务中，nSequence的值通常设置为0xFFFFFFFFFFFF。对于使用nLocktime或CHECKLOCKTIMEVERIFY的事务，nSequence值必须设置为小于2^31，时间锁保护才能产生效果，如下所述

**n序列作为共识强制的相对时间锁**

自激活BIP-68以来，新的共识规则适用于任何包含nSequence值小于2^31的输入的事务（未设置bit 1<<31）。从编程上讲，这意味着如果没有设置最重要的位（位1<<31），它是一个意味着“相对锁定时间”的标志。否则（位1<<31集），nSequence值将保留用于其他用途，例如启用CHECKLOCKTIMEVERIFY、nLocktime、Opt-In-Replace-By-Fee和其他未来开发。

n序列值小于2^31的事务输入被解释为具有相对时间锁。只有当输入按相对时间锁定金额计时，此类交易才有效。例如，一个输入n序列相对时间锁为30个块的事务只有在从挖掘输入中引用的UTXO开始至少过了30个块时才有效。由于nSequence是每个输入字段，事务可能包含任意数量的锁定输入，所有这些输入都必须有足够的老化才能使事务有效。事务可以包括时间锁定的输入（nSequence < 2^31）和没有相对时间锁定的输入（nSequence >= 2^31）。

nSequence值以块或秒为单位指定，但格式与我们在nLocktime中看到的略微不同。类型标志用于区分计数块的值和以秒为单位的值计数时间。类型标志设置在第23个最不重要的位（即值1<<22）。如果设置了类型标志，则nSequence值被解释为512秒的倍数。如果没有设置类型标志，则nSequence值将解释为多个块。

当将n序列解释为相对时间锁时，只考虑16个最不重要的位。一旦计算了标志（位32和23），nSequence值通常使用16位掩码（例如nSequence和0x0000FFFF）“掩码”。

n序列编码的BIP-68定义（来源：BIP-68）显示了BIP-68定义的n序列值的二进制布局。


```bash
./qx tx-encode -v 1 -i 4fff8c7827fedcea5fc60152a94edeee8c391073a109beb3dbdd72929faebb01:4294967294:258:TxTypeCrossChainImport -l 0 -o Mk6q66Yh54zeX7U4RSgUNSSoxyhekBDtUKDCCUzG3WPdYZFWFYuVe:0.1:0:TxTypeCrossChainImport
```
INDEX=4294967294

----

evm-tx-hash:0x066eebf7ddb077f1d3e96e529ab42beb962c96db71dc589db24e6c30b20fcd58




### TxTypeCrossChainImport

mnemonic = "plug family sugar pistol expire canyon rug conduct road sausage weapon crack"

ec-new = e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14
addr = MmNtNZpXD5XLhLY7wknzBu5oVGrdmiCT1DT
text-addr = TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw
pkaddr = Mk6qZQSSAvHhrzZKVpdXSg1btFkkuNckAtWhfrQkwNgFab17zJE47
evm-prv = 1c0f7568db440571712adf2e15e5c483eb939d8f509d0c00c9fd17da5beccd06
evm = 0xc422793dA8bd1c3eeE72023bC3488e7A4F444f12
text-child-meer = TnGRomDEyhLByhTcq9MYnvjDyU9ThyyQrNf

test-pkaddr = Tk2ccA1wxfrXEseUCYqss7N7RbhHAprVwmZrDvodcE8qcqYxTbDTD

./qx mnemonic-to-seed "plug family sugar pistol expire canyon rug conduct road sausage weapon crack"|./qx hd-new -v bip32|./qx hd-derive -v bip32 -p "m/44'/60'/0'/0/0"|./qx hd-to-ec -v bip32|./qx ec-to-public|./qx ec-to-pkaddr


划转交易是特殊交易，不需要源头hash，自己构造，不重复就行。

步骤：tx-encode => tx-sign => sendRawTransaction

```bash
./qx tx-encode -v 1 -i 0000000000000000000000000000000000000000000000000000000012000000:4294967294:258:TxTypeCrossChainImport -l 0 -o Tk2ccA1wxfrXEseUCYqss7N7RbhHAprVwmZrDvodcE8qcqYxTbDTD:100:0:TxTypeCrossChainImport

01000000010000001200000000000000000000000000000000000000000000000000000000feffffff0201000001000000e40b54020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000339760630100-7b22696e707574223a7b2230223a3235387d2c226f7574707574223a7b2230223a3235387d7d
```

```bash
./qx tx-decode 01000000010000001200000000000000000000000000000000000000000000000000000000feffffff0201000001000000e40b54020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000339760630100-7b22696e707574223a7b2230223a3235387d2c226f7574707574223a7b2230223a3235387d7d|jq
{
  "txid": "fb399c913f1f56b27190965dcd8a7611de538597127619097cd8bc9a3b94704f",
  "txhash": "368cc454176647a234a5edb94ea46e252b02e2529c347a26c8e12770388bded1",
  "version": 1,
  "locktime": 0,
  "expire": 0,
  "vin": [
    {
      "type": "TxTypeCrossChainImport",
      "scriptSig": {
        "asm": "",
        "hex": ""
      }
    }
  ],
  "vout": [
    {
      "coin": "MEER Asset",
      "amount": 10000000000,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 34acc0c42b2f04887302180eea42acb11d18e55a OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a91434acc0c42b2f04887302180eea42acb11d18e55a88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "TnGRomDEyhLByhTcq9MYnvjDyU9ThyyQrNf"
        ]
      }
    }
  ]
}
```

```bash
./qx tx-sign -k 1c0f7568db440571712adf2e15e5c483eb939d8f509d0c00c9fd17da5beccd06 -n testnet 01000000010000001200000000000000000000000000000000000000000000000000000000feffffff0201000001000000e40b54020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000339760630100-7b22696e707574223a7b2230223a3235387d2c226f7574707574223a7b2230223a3235387d7d
01000000010000001200000000000000000000000000000000000000000000000000000000feffffff0201000001000000e40b54020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000339760630180354d6b36715a51535341764868727a5a4b567064585367316274466b6b754e636b417457686672516b774e6746616231377a4a45343749483045022100a2e9e382ec14f0550d0db14483a25af9f7a102fe9f36be78f8a89626cdf4c76b022029946f1164186ed3c4fb5ff2c52d1f198bed1609eae053cc8b7f8366a2f8a22c01
```

```bash
curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["01000000010000001200000000000000000000000000000000000000000000000000000000feffffff0201000001000000e40b54020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000339760630180354d6b36715a51535341764868727a5a4b567064585367316274466b6b754e636b417457686672516b774e6746616231377a4a45343749483045022100a2e9e382ec14f0550d0db14483a25af9f7a102fe9f36be78f8a89626cdf4c76b022029946f1164186ed3c4fb5ff2c52d1f198bed1609eae053cc8b7f8366a2f8a22c01", true],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rule Error : Rejected transaction fb399c913f1f56b27190965dcd8a7611de538597127619097cd8bc9a3b94704f: transaction fb399c913f1f56b27190965dcd8a7611de538597127619097cd8bc9a3b94704f has 0 fees which is under the required amount of 2240, tx size is 224 bytes, policy-rate is 10000/KB."
  }
}
没有留fee
```

- 重来

```bash
./qx tx-encode -v 1 -i 0000000000000000000000000000000000000000000000000000000012100000:4294967294:258:TxTypeCrossChainImport -l 0 -o Tk2ccA1wxfrXEseUCYqss7N7RbhHAprVwmZrDvodcE8qcqYxTbDTD:99.99:0:TxTypeCrossChainImport
01000000010000101200000000000000000000000000000000000000000000000000000000feffffff02010000010000c0a1fc53020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000069c60630100-7b22696e707574223a7b2230223a3235387d2c226f7574707574223a7b2230223a3235387d7d


./qx tx-sign -k 1c0f7568db440571712adf2e15e5c483eb939d8f509d0c00c9fd17da5beccd06 -n testnet 01000000010000101200000000000000000000000000000000000000000000000000000000feffffff02010000010000c0a1fc53020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000069c60630100-7b22696e707574223a7b2230223a3235387d2c226f7574707574223a7b2230223a3235387d7d
01000000010000101200000000000000000000000000000000000000000000000000000000feffffff02010000010000c0a1fc53020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000069c6063017f354d6b36715a51535341764868727a5a4b567064585367316274466b6b754e636b417457686672516b774e6746616231377a4a4534374847304402205c6aab3fa0ac536e597fa4c595e4706a488f67b8fa26705fa6774a4a0c3926aa022061466d5fe87521aa0171addb567248d423d89f50eb608e19a0cc6cc0452ce53d01


curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["01000000010000101200000000000000000000000000000000000000000000000000000000feffffff02010000010000c0a1fc53020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000069c6063017f354d6b36715a51535341764868727a5a4b567064585367316274466b6b754e636b417457686672516b774e6746616231377a4a4534374847304402205c6aab3fa0ac536e597fa4c595e4706a488f67b8fa26705fa6774a4a0c3926aa022061466d5fe87521aa0171addb567248d423d89f50eb608e19a0cc6cc0452ce53d01", false],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "a6f43bb66823053fb67bd3d42bfeb5a74e344985a8defe99fb9b12316ff0bcef"
}


{
  "hex": "01000000010000101200000000000000000000000000000000000000000000000000000000feffffff02010000010000c0a1fc53020000001976a91434acc0c42b2f04887302180eea42acb11d18e55a88ac0000000000000000069c6063017f354d6b36715a51535341764868727a5a4b567064585367316274466b6b754e636b417457686672516b774e6746616231377a4a4534374847304402205c6aab3fa0ac536e597fa4c595e4706a488f67b8fa26705fa6774a4a0c3926aa022061466d5fe87521aa0171addb567248d423d89f50eb608e19a0cc6cc0452ce53d01",
  "txid": "a6f43bb66823053fb67bd3d42bfeb5a74e344985a8defe99fb9b12316ff0bcef",
  "txhash": "b112334f0781845c362b06e3bbd4ebab2d5bb5ad883523624620ddeeaa3887ac",
  "size": 223,
  "version": 1,
  "locktime": 0,
  "timestamp": "2022-11-01T12:09:42+08:00",
  "expire": 0,
  "vin": [
    {
      "type": "TxTypeCrossChainImport",
      "scriptSig": {
        "asm": "",
        "hex": "354d6b36715a51535341764868727a5a4b567064585367316274466b6b754e636b417457686672516b774e6746616231377a4a4534374847304402205c6aab3fa0ac536e597fa4c595e4706a488f67b8fa26705fa6774a4a0c3926aa022061466d5fe87521aa0171addb567248d423d89f50eb608e19a0cc6cc0452ce53d01"
      }
    }
  ],
  "vout": [
    {
      "coin": "MEER Asset",
      "amount": 9999000000,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 34acc0c42b2f04887302180eea42acb11d18e55a OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a91434acc0c42b2f04887302180eea42acb11d18e55a88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "TnGRomDEyhLByhTcq9MYnvjDyU9ThyyQrNf"
        ]
      }
    }
  ],
  "blockhash": "a0db8fb753a2e15a19300349e0194695d3f9a6180487c22f3695ec080a5db50f",
  "confirmations": 2,
  "txsvalid": true
}
```
至此，将99.99 evm-meer转到了 pkaddr 对应的 meer-utxo 地址 TnGRomDEyhLByhTcq9MYnvjDyU9ThyyQrNf。该地址为主密钥下地四代孙密钥地址。



### 将meer转到主密钥地址


```bash
./qx tx-encode -i a6f43bb66823053fb67bd3d42bfeb5a74e344985a8defe99fb9b12316ff0bcef:2:258:TxTypeRegular -l 0 -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:0.1:0:TxTypeRegular
0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a6020000000201000001000080969800000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000007ba660630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a307d7d

{
  "txid": "72020aea5d7a30d2f281b60b67b4c30b7ce118ccb429df8d809970459eba117e",
  "txhash": "4df4ec563c877ecaf38efd05a0e1cc9c497d1b98f57fd33ff14353424491f015",
  "version": 1,
  "locktime": 0,
  "expire": 0,
  "vin": [
    {
      "type": "TxTypeRegular",
      "scriptSig": {
        "asm": "",
        "hex": ""
      }
    }
  ],
  "vout": [
    {
      "coin": "MEER Asset",
      "amount": 10000000,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 2421972d46cedd2283d244665a05bde2aec3446f OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9142421972d46cedd2283d244665a05bde2aec3446f88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw"
        ]
      }
    }
  ]
}

./qx tx-sign -k 1c0f7568db440571712adf2e15e5c483eb939d8f509d0c00c9fd17da5beccd06 -n testnet 0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a6020000000201000001000080969800000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000007ba660630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a307d7d
0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a6020000000201000001000080969800000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000007ba66063016b483045022100c57ba6de65b8cfbd1cfc4c8a795fa48c59fdabe301eecc8a80b8e0291e91700802200dcada045b2e3fd08b8fb018dd0f91a75714a52543eeaad3be758ff3fce92df2012102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cd

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a6020000000201000001000080969800000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000007ba66063016b483045022100c57ba6de65b8cfbd1cfc4c8a795fa48c59fdabe301eecc8a80b8e0291e91700802200dcada045b2e3fd08b8fb018dd0f91a75714a52543eeaad3be758ff3fce92df2012102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cd", false],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rule Error : Rejected transaction 72020aea5d7a30d2f281b60b67b4c30b7ce118ccb429df8d809970459eba117e: orphan transaction 72020aea5d7a30d2f281b60b67b4c30b7ce118ccb429df8d809970459eba117e references outputs of unknown or fully-spent transaction a6f43bb66823053fb67bd3d42bfeb5a74e344985a8defe99fb9b12316ff0bcef"
  }
}

可能index错误
---
./qx tx-encode -i a6f43bb66823053fb67bd3d42bfeb5a74e344985a8defe99fb9b12316ff0bcef:0:258:TxTypeRegular -l 0 -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:0.1:0:TxTypeRegular
0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a6000000000201000001000080969800000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000006aa860630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a307d7d

./qx tx-sign -k 1c0f7568db440571712adf2e15e5c483eb939d8f509d0c00c9fd17da5beccd06 -n testnet 0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a6000000000201000001000080969800000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000006aa860630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a307d7d
0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a6000000000201000001000080969800000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000006aa86063016a4730440220467f7f03bc6c1e1679718678dd664cdb2edd9593f86755cf0b884b77d6d3fa4a02207b50c28c495643fa14e4fd2de7d9b381a0bcf3f09cc73608e7da341f5595f3fb012102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cd

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a6000000000201000001000080969800000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000006aa86063016a4730440220467f7f03bc6c1e1679718678dd664cdb2edd9593f86755cf0b884b77d6d3fa4a02207b50c28c495643fa14e4fd2de7d9b381a0bcf3f09cc73608e7da341f5595f3fb012102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cd", false],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Deserialization Error : rejected: failed to process transaction aeb2142487dd7db9bf74e88d8fdb2ff95b1f83d57df6bf39b546d7719c81f40d: transaction aeb2142487dd7db9bf74e88d8fdb2ff95b1f83d57df6bf39b546d7719c81f40d has 9989000000 fee which is above the allowHighFee check threshold amount of 20200000 (= 202 byte * 10000 atomMEER Asset/kB * 10000)"
  }
}

普通交易 需要自行分配金额以确定转出和找零等 不然差额就是手续费 怎么分开取决于你自己 | evm=>utxo 划转交易是特殊交易 不可拆分 utxo=>evm 待测

vin - vout = fee

-----

./qx tx-encode -i a6f43bb66823053fb67bd3d42bfeb5a74e344985a8defe99fb9b12316ff0bcef:0:258:TxTypeRegular -l 0 -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:99.989:0:TxTypeRegular
0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a60000000002010000010000201bfb53020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000005da960630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a307d7d

./qx tx-sign -k 1c0f7568db440571712adf2e15e5c483eb939d8f509d0c00c9fd17da5beccd06 -n testnet 0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a60000000002010000010000201bfb53020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000005da960630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a307d7d

0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a60000000002010000010000201bfb53020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000005da96063016a473044022019be645ed058c087a2a21f58ba67c25367a071dbd1a69a73dcac72cd0751318f0220758527bfba99481c36c440a1f83dd08c16058fe6e92f3306337cf717c6ce28c4012102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cd

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["0100000001efbcf06f31129bfb99fedea88549344ea7b5fe2bd4d37bb63f052368b63bf4a60000000002010000010000201bfb53020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000005da96063016a473044022019be645ed058c087a2a21f58ba67c25367a071dbd1a69a73dcac72cd0751318f0220758527bfba99481c36c440a1f83dd08c16058fe6e92f3306337cf717c6ce28c4012102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cd", false],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "04f6d4055fac1e17975f30e345ac8602c9fd4c54351c99af615273950fe43ee8"
}

```

至此，将 99.989 meer 从 evm地址转回到了主密钥地址 TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw。


 

## TxTypeCrossChainExport（TX from MEER to EVM)

ec-new = e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14.  
text-addr = TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw.  
pkaddr = Mk6qZQSSAvHhrzZKVpdXSg1btFkkuNckAtWhfrQkwNgFab17zJE47.  
evm-prv = 1c0f7568db440571712adf2e15e5c483eb939d8f509d0c00c9fd17da5beccd06.  
text-child-meer = TnGRomDEyhLByhTcq9MYnvjDyU9ThyyQrNf. 

继续测试划转

TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw 总余额 99.989 meer，

From UTXO: 04f6d4055fac1e17975f30e345ac8602c9fd4c54351c99af615273950fe43ee8 to MEER PKAddress Mk6qZQSSAvHhrzZKVpdXSg1btFkkuNckAtWhfrQkwNgFab17zJE47 9.9999 MEER coinID : 1 ETH

Meer to EVM，coinID 为 1，类似把 meer 变成 eth。

- 拆分金额测试

```bash
./qx tx-encode -v 1 -i 04f6d4055fac1e17975f30e345ac8602c9fd4c54351c99af615273950fe43ee8:0:4294967295:TxTypeCrossChainExport -l 0 -o Mk6qZQSSAvHhrzZKVpdXSg1btFkkuNckAtWhfrQkwNgFab17zJE47:9.9889:1:TxTypeCrossChainExport -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:90:0:TxTypeRegular

0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff02010010da893b00000000232102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cdac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000004fc060630100-7b22696e707574223a7b2230223a3235377d2c226f7574707574223a7b2230223a3235372c2231223a307d7d

./qx tx-sign -k (master-prv)e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14 -n testnet 0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff02010010da893b00000000232102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cdac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000004fc060630100-7b22696e707574223a7b2230223a3235377d2c226f7574707574223a7b2230223a3235372c2231223a307d7d
0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff02010010da893b00000000232102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cdac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000004fc06063014847304402205e5bdcc645215357c86ad5b89b78c2036706efb28a873ae09c5ca994654f1fac02202c5b78ac80baf50f32d257275a5bea30197ca53820503d4b7c19c7433af1eae701

./qx tx-sign -k (child-prv)1c0f7568db440571712adf2e15e5c483eb939d8f509d0c00c9fd17da5beccd06 -n testnet 0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff02010010da893b00000000232102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cdac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000004fc060630100-7b22696e707574223a7b2230223a3235377d2c226f7574707574223a7b2230223a3235372c2231223a307d7d
0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff02010010da893b00000000232102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cdac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000004fc060630149483045022100aeb4017c615271f45c4cd8207491217494c72f84a815abf15cfe7e94ec80c67302201309fcb1eb5f11a4cbcd4c078fbced96d87c295efa50e93b8913071cc186e2d501

先发子私钥签名的 测试是否成功 预测结果应该为失败

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff02010010da893b00000000232102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cdac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000004fc060630149483045022100aeb4017c615271f45c4cd8207491217494c72f84a815abf15cfe7e94ec80c67302201309fcb1eb5f11a4cbcd4c078fbced96d87c295efa50e93b8913071cc186e2d501", false],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rule Error : Rejected transaction fb33616908402280cd607be72ca519275f467d5590f0ed2f758c9f526f885d20: failed to validate input fb33616908402280cd607be72ca519275f467d5590f0ed2f758c9f526f885d20:0 which references output {04f6d4055fac1e17975f30e345ac8602c9fd4c54351c99af615273950fe43ee8 0} - verify failed (input script bytes 483045022100aeb4017c615271f45c4cd8207491217494c72f84a815abf15cfe7e94ec80c67302201309fcb1eb5f11a4cbcd4c078fbced96d87c295efa50e93b8913071cc186e2d501, prev output script bytes 76a9142421972d46cedd2283d244665a05bde2aec3446f88ac)"
  }
}

发送主私钥签名的

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff02010010da893b00000000232102ad6ca4cbfbce01693f759e304bbb520fd316429deb4f4a81527d8b136901e5cdac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000004fc06063014847304402205e5bdcc645215357c86ad5b89b78c2036706efb28a873ae09c5ca994654f1fac02202c5b78ac80baf50f32d257275a5bea30197ca53820503d4b7c19c7433af1eae701", false],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rule Error : Rejected transaction fb33616908402280cd607be72ca519275f467d5590f0ed2f758c9f526f885d20: failed to validate input fb33616908402280cd607be72ca519275f467d5590f0ed2f758c9f526f885d20:0 which references output {04f6d4055fac1e17975f30e345ac8602c9fd4c54351c99af615273950fe43ee8 0} - verify failed (input script bytes 47304402205e5bdcc645215357c86ad5b89b78c2036706efb28a873ae09c5ca994654f1fac02202c5b78ac80baf50f32d257275a5bea30197ca53820503d4b7c19c7433af1eae701, prev output script bytes 76a9142421972d46cedd2283d244665a05bde2aec3446f88ac)"
  }
} 

```
划转拆分测试失败

- 全额划转测试




### tx-encode to many

TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw has 99.989 meer.

to TnHqTCHGYPfFyNtLQfYKVf5jd4KiQkropmV 9.988 meer, to TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw 90 meer, fee = 0.001

```bash
~ ./qx tx-encode -i 04f6d4055fac1e17975f30e345ac8602c9fd4c54351c99af615273950fe43ee8:0:4294967295:TxTypeRegular -l 0 -o TnHqTCHGYPfFyNtLQfYKVf5jd4KiQkropmV:9.988:0:TxTypeRegular -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:90:0:TxTypeRegular
0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff020000807a883b000000001976a914441db1582a69878dfa83042b529ba918b74aaf1088ac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac0000000000000000c8d360630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a307d7d

~ ./qx tx-decode -n testnet 0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff020000807a883b000000001976a914441db1582a69878dfa83042b529ba918b74aaf1088ac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac0000000000000000c8d360630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a307d7d | jq
{
  "txid": "2532595f10645b5550f90c60a6cedac1fd947aba26ff880f609e7c686f7e0401",
  "txhash": "cf6be2dfccf7fc24d0b448fdb92cb7be934505f15d411bce2f4faf4f95d6e79b",
  "version": 1,
  "locktime": 0,
  "expire": 0,
  "vin": [
    {
      "type": "TxTypeRegular",
      "scriptSig": {
        "asm": "",
        "hex": ""
      }
    }
  ],
  "vout": [
    {
      "coin": "MEER Asset",
      "amount": 998800000,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 441db1582a69878dfa83042b529ba918b74aaf10 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914441db1582a69878dfa83042b529ba918b74aaf1088ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "TnHqTCHGYPfFyNtLQfYKVf5jd4KiQkropmV"
        ]
      }
    },
    {
      "coin": "MEER Asset",
      "amount": 9000000000,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 2421972d46cedd2283d244665a05bde2aec3446f OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9142421972d46cedd2283d244665a05bde2aec3446f88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw"
        ]
      }
    }
  ]
}
```
```bash
~ ./qx tx-sign -k e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14 -n testnet 0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff020000807a883b000000001976a914441db1582a69878dfa83042b529ba918b74aaf1088ac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac0000000000000000c8d360630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a307d7d
0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff020000807a883b000000001976a914441db1582a69878dfa83042b529ba918b74aaf1088ac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac0000000000000000c8d36063016b483045022100981bc66c021d792f53ecb6d2b1fc001ec91f11df055b8fc78f3cf7df88e51c0102200744540c34b4c0a5ab47341f4db48668a2003c85ee8647bf7495747aff6bb5f10121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8
```

```bash
curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000ffffffff020000807a883b000000001976a914441db1582a69878dfa83042b529ba918b74aaf1088ac0000001a7118020000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac0000000000000000c8d36063016b483045022100981bc66c021d792f53ecb6d2b1fc001ec91f11df055b8fc78f3cf7df88e51c0102200744540c34b4c0a5ab47341f4db48668a2003c85ee8647bf7495747aff6bb5f10121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8", false],"id":1}' https://127.0.0.1:28131 | jq
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "2532595f10645b5550f90c60a6cedac1fd947aba26ff880f609e7c686f7e0401"
}
```

```bash
./qx tx-decode -n testnet 0100000001e83ee40f95735261af991c35544cfdc90286ac45e3305f97171eac5f05d4f60400000000feffffff020000807a883b000000002004c20f6a63b17576a914441db1582a69878dfa83042b529ba918b74aaf1088ac0000001a7118020000002004c20f6a63b17576a9142421972d46cedd2283d244665a05bde2aec3446f88acc20f6a6300000000b5d860630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a307d7d
{
  "txid": "40c8a8fcd173ad0ec1ad75a06c3dfe1fd623821345f5417cb7562e4bf7a64aee",
  "txhash": "67acbab1d4c65ca2c1f0b6449d4d3374ff670004cf252087beaa474344e1465a",
  "version": 1,
  "locktime": 1667895234,
  "expire": 0,
  "vin": [
    {
      "type": "TxTypeRegular",
      "scriptSig": {
        "asm": "",
        "hex": ""
      }
    }
  ],
  "vout": [
    {
      "coin": "MEER Asset",
      "amount": 998800000,
      "scriptPubKey": {
        "asm": "c20f6a63 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 441db1582a69878dfa83042b529ba918b74aaf10 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "04c20f6a63b17576a914441db1582a69878dfa83042b529ba918b74aaf1088ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnHqTCHGYPfFyNtLQfYKVf5jd4KiQkropmV"
        ]
      }
    },
    {
      "coin": "MEER Asset",
      "amount": 9000000000,
      "scriptPubKey": {
        "asm": "c20f6a63 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 2421972d46cedd2283d244665a05bde2aec3446f OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "04c20f6a63b17576a9142421972d46cedd2283d244665a05bde2aec3446f88ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw"
        ]
      }
    }
  ]
}
```

- to many
  
 lock time = 1667298670

from TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw 90meer

to  TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL:5  TnVmhArErwnRSLZQiNWguQeq9w4xiHyB86M:5   TnHCAw3gfuZYuwkwFoo48FBpwPMWRMZWaRD:4.999  TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:75

fee = 0.001


```bash
./qx tx-encode -i 2532595f10645b5550f90c60a6cedac1fd947aba26ff880f609e7c686f7e0401:1:4294967295:TxTypeRegular -l 1667298670 -o TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL:5:0:TxTypeRegular -o TnVmhArErwnRSLZQiNWguQeq9w4xiHyB86M:5:0:TxTypeRegular -o TnHCAw3gfuZYuwkwFoo48FBpwPMWRMZWaRD:4.999:0:TxTypeRegular -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:75:0:TxTypeRegular
010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000feffffff0400000065cd1d0000000020046ef56063b17576a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d0000000020046ef56063b17576a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d0000000020046ef56063b17576a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf0100000020046ef56063b17576a9142421972d46cedd2283d244665a05bde2aec3446f88ac6ef5606300000000c9f660630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a302c2232223a302c2233223a307d7d

{
  "txid": "7c3e320bca2d9fb52963c317bbd029ae4a6dfcaf404f7ed24394ede008dd5055",
  "txhash": "dcd97f7814e7c0b5eb46dd5fb1d266bef8559559245dc50a315423e545d20d03",
  "version": 1,
  "locktime": 1667298670,
  "expire": 0,
  "vin": [
    {
      "type": "TxTypeRegular",
      "scriptSig": {
        "asm": "",
        "hex": ""
      }
    }
  ],
  "vout": [
    {
      "coin": "MEER Asset",
      "amount": 500000000,
      "scriptPubKey": {
        "asm": "6ef56063 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 f9e7e2a5afc7278bdd6f7354e729b8b49bc082f8 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "046ef56063b17576a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL"
        ]
      }
    },
    {
      "coin": "MEER Asset",
      "amount": 500000000,
      "scriptPubKey": {
        "asm": "6ef56063 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 c709344e2f73b022a22974acee264d8e8ad37b5a OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "046ef56063b17576a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnVmhArErwnRSLZQiNWguQeq9w4xiHyB86M"
        ]
      }
    },
    {
      "coin": "MEER Asset",
      "amount": 499900000,
      "scriptPubKey": {
        "asm": "6ef56063 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 3d10ba0e5a0ecf10205da56e2e6287d95041e8ec OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "046ef56063b17576a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnHCAw3gfuZYuwkwFoo48FBpwPMWRMZWaRD"
        ]
      }
    },
    {
      "coin": "MEER Asset",
      "amount": 7500000000,
      "scriptPubKey": {
        "asm": "6ef56063 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 2421972d46cedd2283d244665a05bde2aec3446f OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "046ef56063b17576a9142421972d46cedd2283d244665a05bde2aec3446f88ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw"
        ]
      }
    }
  ]
}
```


```bash
./qx tx-sign -k e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14 -n testnet 010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000feffffff0400000065cd1d0000000020046ef56063b17576a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d0000000020046ef56063b17576a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d0000000020046ef56063b17576a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf0100000020046ef56063b17576a9142421972d46cedd2283d244665a05bde2aec3446f88ac6ef5606300000000c9f660630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a302c2232223a302c2233223a307d7d
010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000feffffff0400000065cd1d0000000020046ef56063b17576a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d0000000020046ef56063b17576a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d0000000020046ef56063b17576a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf0100000020046ef56063b17576a9142421972d46cedd2283d244665a05bde2aec3446f88ac6ef5606300000000c9f66063016b483045022100c3a5d14d280f9addf7412cfb6a49a0ed066983963659ff284337293684339c860220084e119463c0545cfe8dd4def924201b03d9ea915af2697ee00455445df0f9680121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8
```

```bash
curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000feffffff0400000065cd1d0000000020046ef56063b17576a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d0000000020046ef56063b17576a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d0000000020046ef56063b17576a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf0100000020046ef56063b17576a9142421972d46cedd2283d244665a05bde2aec3446f88ac6ef5606300000000c9f66063016b483045022100c3a5d14d280f9addf7412cfb6a49a0ed066983963659ff284337293684339c860220084e119463c0545cfe8dd4def924201b03d9ea915af2697ee00455445df0f9680121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8", false],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "7c3e320bca2d9fb52963c317bbd029ae4a6dfcaf404f7ed24394ede008dd5055"
}


{
  "hex": "010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000feffffff0400000065cd1d0000000020046ef56063b17576a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d0000000020046ef56063b17576a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d0000000020046ef56063b17576a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf0100000020046ef56063b17576a9142421972d46cedd2283d244665a05bde2aec3446f88ac6ef5606300000000c9f66063016b483045022100c3a5d14d280f9addf7412cfb6a49a0ed066983963659ff284337293684339c860220084e119463c0545cfe8dd4def924201b03d9ea915af2697ee00455445df0f9680121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8",
  "txid": "7c3e320bca2d9fb52963c317bbd029ae4a6dfcaf404f7ed24394ede008dd5055",
  "txhash": "fa40f51ae181376d3dc016c3b16f3e0824f3be9d4b492fd8f5a6561806d3a92b",
  "size": 339,
  "version": 1,
  "locktime": 1667298670,
  "timestamp": "2022-11-01T18:36:57+08:00",
  "expire": 0,
  "vin": [
    {
      "txid": "2532595f10645b5550f90c60a6cedac1fd947aba26ff880f609e7c686f7e0401",
      "vout": 1,
      "sequence": 4294967294,
      "scriptSig": {
        "asm": "3045022100c3a5d14d280f9addf7412cfb6a49a0ed066983963659ff284337293684339c860220084e119463c0545cfe8dd4def924201b03d9ea915af2697ee00455445df0f96801 036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8",
        "hex": "483045022100c3a5d14d280f9addf7412cfb6a49a0ed066983963659ff284337293684339c860220084e119463c0545cfe8dd4def924201b03d9ea915af2697ee00455445df0f9680121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8"
      }
    }
  ],
  "vout": [
    {
      "coin": "MEER Asset",
      "amount": 500000000,
      "scriptPubKey": {
        "asm": "6ef56063 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 f9e7e2a5afc7278bdd6f7354e729b8b49bc082f8 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "046ef56063b17576a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL"
        ]
      }
    },
    {
      "coin": "MEER Asset",
      "amount": 500000000,
      "scriptPubKey": {
        "asm": "6ef56063 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 c709344e2f73b022a22974acee264d8e8ad37b5a OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "046ef56063b17576a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnVmhArErwnRSLZQiNWguQeq9w4xiHyB86M"
        ]
      }
    },
    {
      "coin": "MEER Asset",
      "amount": 499900000,
      "scriptPubKey": {
        "asm": "6ef56063 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 3d10ba0e5a0ecf10205da56e2e6287d95041e8ec OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "046ef56063b17576a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnHCAw3gfuZYuwkwFoo48FBpwPMWRMZWaRD"
        ]
      }
    },
    {
      "coin": "MEER Asset",
      "amount": 7500000000,
      "scriptPubKey": {
        "asm": "6ef56063 OP_CHECKLOCKTIMEVERIFY OP_DROP OP_DUP OP_HASH160 2421972d46cedd2283d244665a05bde2aec3446f OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "046ef56063b17576a9142421972d46cedd2283d244665a05bde2aec3446f88ac",
        "reqSigs": 1,
        "type": "cltvpubkeyhash",
        "addresses": [
          "TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw"
        ]
      }
    }
  ],
  "blockhash": "48664ea3716a32f60dc347c0513d4de89bae24ddd43476a83f841ec2f10079ec",
  "confirmations": 5,
  "txsvalid": true
}
```

结果描述：如果按照时间戳来看待 locktime=1667298670， 发送交易的时候实际上已经过了这个时间，但是最后的结果是所有交易都还是锁定状态。并且浏览器上查看，显示锁定高度为1667298670。

**继续造，去掉锁仓时间重新发送一遍上边的交易：**

```bash
./qx tx-encode -i 2532595f10645b5550f90c60a6cedac1fd947aba26ff880f609e7c686f7e0401:1:4294967295:TxTypeRegular -l 0 -o TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL:5:0:TxTypeRegular -o TnVmhArErwnRSLZQiNWguQeq9w4xiHyB86M:5:0:TxTypeRegular -o TnHCAw3gfuZYuwkwFoo48FBpwPMWRMZWaRD:4.999:0:TxTypeRegular -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:75:0:TxTypeRegular
010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000ffffffff0400000065cd1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000008b0761630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a302c2232223a302c2233223a307d7d

./qx tx-sign -k e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14 -n testnet 010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000ffffffff0400000065cd1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000008b0761630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a302c2232223a302c2233223a307d7d
010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000ffffffff0400000065cd1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000008b076163016a47304402205042ac78f873ffcb5b50deef7002685a4e2ece38e47cd3881c0ccfcd9893dd100220381d9a22f94d817483b1e8716f10c8208051e125f9fbb09ee5c8f2f6581902940121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000ffffffff0400000065cd1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000008b076163016a47304402205042ac78f873ffcb5b50deef7002685a4e2ece38e47cd3881c0ccfcd9893dd100220381d9a22f94d817483b1e8716f10c8208051e125f9fbb09ee5c8f2f6581902940121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8", false],"id":1}' https://127.0.0.1:28131 | jq
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rule Error : Rejected transaction 4db1b40f23cdc1166f1f6966df7754f2ee63488786f53cacd34a840cdfa64529: orphan transaction 4db1b40f23cdc1166f1f6966df7754f2ee63488786f53cacd34a840cdfa64529 references outputs of unknown or fully-spent transaction 2532595f10645b5550f90c60a6cedac1fd947aba26ff880f609e7c686f7e0401"
  }
}
```

再干：

```bash
./qx tx-encode -i 2532595f10645b5550f90c60a6cedac1fd947aba26ff880f609e7c686f7e0401:1:4294967295:TxTypeRegular -l 0 -o TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL:4.999:0:TxTypeRegular -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:85:0:TxTypeRegular
010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000ffffffff02000060decb1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000b5a3fa010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000002d2761630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a307d7d

./qx tx-sign -k e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14 -n testnet  010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000ffffffff02000060decb1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000b5a3fa010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000002d2761630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a307d7d
010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000ffffffff02000060decb1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000b5a3fa010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000002d276163016a473044022063b169873da5ad8b7a92a410133725bde1d1d9b8ddbaa6fc47191e80708279b8022055e7d7e87b5afdf9e2ba18347a1b2e1a613d90f3cefa968bbe4b88c28bbebbb30121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000ffffffff02000060decb1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000b5a3fa010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000002d276163016a473044022063b169873da5ad8b7a92a410133725bde1d1d9b8ddbaa6fc47191e80708279b8022055e7d7e87b5afdf9e2ba18347a1b2e1a613d90f3cefa968bbe4b88c28bbebbb30121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8", false],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rule Error : Rejected transaction d1766164f0723d1ed2ed71dfecad6786a070fe10fe817d871a117b1af3fc0773: orphan transaction d1766164f0723d1ed2ed71dfecad6786a070fe10fe817d871a117b1af3fc0773 references outputs of unknown or fully-spent transaction 2532595f10645b5550f90c60a6cedac1fd947aba26ff880f609e7c686f7e0401"
  }
}
```

7c3e320bca2d9fb52963c317bbd029ae4a6dfcaf404f7ed24394ede008dd5055

```bash
./qx tx-encode -i 7c3e320bca2d9fb52963c317bbd029ae4a6dfcaf404f7ed24394ede008dd5055:4:4294967295:TxTypeRegular -o TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL:4.999:0:TxTypeRegular -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:85:0:TxTypeRegular
01000000015550dd08e0ed9443d27e4f40affc6d4aae29d0bb17c36329b59f2dca0b323e7c04000000ffffffff02000060decb1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000863ba1010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac000000000000000001e661630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a307d7d

./qx tx-sign -k e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14 -n testnet 01000000015550dd08e0ed9443d27e4f40affc6d4aae29d0bb17c36329b59f2dca0b323e7c04000000ffffffff02000060decb1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000863ba1010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac000000000000000001e661630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a307d7d
01000000015550dd08e0ed9443d27e4f40affc6d4aae29d0bb17c36329b59f2dca0b323e7c04000000ffffffff02000060decb1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000863ba1010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac000000000000000001e66163016b483045022100abcfa2660f7c027f9676b949445803048cb4d0f4dba7fde744e957ffbfd6a704022023ae2d69efc1e7b736b612ede74a7d620fb2370daf1deacaf79b09f6ac1d18660121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["01000000015550dd08e0ed9443d27e4f40affc6d4aae29d0bb17c36329b59f2dca0b323e7c04000000ffffffff02000060decb1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000863ba1010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac000000000000000001e66163016b483045022100abcfa2660f7c027f9676b949445803048cb4d0f4dba7fde744e957ffbfd6a704022023ae2d69efc1e7b736b612ede74a7d620fb2370daf1deacaf79b09f6ac1d18660121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8", false],"id":1}' https://127.0.0.1:28131 | jq

{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rule Error : Rejected transaction fbedaec63a0468650a53758fe9607bfac96a5fbe7598c9095ba119ae80970104: orphan transaction fbedaec63a0468650a53758fe9607bfac96a5fbe7598c9095ba119ae80970104 references outputs of unknown or fully-spent transaction 7c3e320bca2d9fb52963c317bbd029ae4a6dfcaf404f7ed24394ede008dd5055"
  }
}
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rule Error : Rejected transaction 9f261a4051c825798ac0a3c602d0d34598c97753be961ba1700edf70bf02364f: orphan transaction 9f261a4051c825798ac0a3c602d0d34598c97753be961ba1700edf70bf02364f references outputs of unknown or fully-spent transaction 7c3e320bca2d9fb52963c317bbd029ae4a6dfcaf404f7ed24394ede008dd5055"
  }
}
```

./qx tx-encode -i 2532595f10645b5550f90c60a6cedac1fd947aba26ff880f609e7c686f7e0401:1:4294967294:TxTypeRegular -l 0 -o TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL:5:0:TxTypeRegular -o TnVmhArErwnRSLZQiNWguQeq9w4xiHyB86M:5:0:TxTypeRegular -o TnHCAw3gfuZYuwkwFoo48FBpwPMWRMZWaRD:4.999:0:TxTypeRegular -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:75:0:TxTypeRegular

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["010000000101047e6f687c9e600f88ff26ba7a94fdc1dacea6600cf950555b64105f59322501000000feffffff0400000065cd1d000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac00000065cd1d000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000060decb1d000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000000eb08bf010000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000009ce96163016b483045022100ca6c2512e735e7fb8bf9ec2279a487ccc46ddc9a08aed733882a03c47ea162f702206d79c8f92f3b2bd1e46ebfab719e923ee109b3ce60e3a11ca7b1f85b7364367d0121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8", false],"id":1}' https://127.0.0.1:28131 | jq

尝试各种方法，好像都无法让解锁失效。唉，一个大于 5亿的数怎么会被当作区块高度呢？

=====================

弄点新币继续造

7e30d6cb01ae393ff40e29b8c2fe469daeeccdd8cf34bc6489b4b21c09eeb015  9.9875meer

```bash
./qx tx-encode -i 7e30d6cb01ae393ff40e29b8c2fe469daeeccdd8cf34bc6489b4b21c09eeb015:1:4294967295:TxTypeRegular -l 0 -o TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL:1:0:TxTypeRegular -o TnVmhArErwnRSLZQiNWguQeq9w4xiHyB86M:1:0:TxTypeRegular -o TnHCAw3gfuZYuwkwFoo48FBpwPMWRMZWaRD:1:0:TxTypeRegular -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:6.9874:0:TxTypeRegular
010000000115b0ee091cb2b48964bc34cfd8cdecae9d46fec2b8290ef43f39ae01cbd6307e01000000ffffffff04000000e1f505000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000e1f505000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000000e1f505000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000020eda529000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac000000000000000087fa61630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a302c2232223a302c2233223a307d7d

./qx tx-sign -k e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14 -n testnet 010000000115b0ee091cb2b48964bc34cfd8cdecae9d46fec2b8290ef43f39ae01cbd6307e01000000ffffffff04000000e1f505000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000e1f505000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000000e1f505000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000020eda529000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac000000000000000087fa61630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a302c2232223a302c2233223a307d7d
010000000115b0ee091cb2b48964bc34cfd8cdecae9d46fec2b8290ef43f39ae01cbd6307e01000000ffffffff04000000e1f505000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000e1f505000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000000e1f505000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000020eda529000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac000000000000000087fa6163016a47304402203ce2e8e7420f3c6d1ab6257ee428385e5a72841af026ddf5c869566231b6d3d902206a0a9d873b85a0066229f43be0d8b8ed3dc488670f4b4358716b9125dc03bcae0121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["010000000115b0ee091cb2b48964bc34cfd8cdecae9d46fec2b8290ef43f39ae01cbd6307e01000000ffffffff04000000e1f505000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000e1f505000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000000e1f505000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000020eda529000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac000000000000000087fa6163016a47304402203ce2e8e7420f3c6d1ab6257ee428385e5a72841af026ddf5c869566231b6d3d902206a0a9d873b85a0066229f43be0d8b8ed3dc488670f4b4358716b9125dc03bcae0121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8", false],"id":1}' https://127.0.0.1:28131 | jq
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32000,
    "message": "Rule Error : Rejected transaction 8b9fd21a4e642174c6a18c91855e5312a29170549633a59f5018584b3839f0af: total MEER Asset value of all transaction inputs for transaction 8b9fd21a4e642174c6a18c91855e5312a29170549633a59f5018584b3839f0af is 44000 which is less than the amount spent of 998740000"
  }
} index=1 的 utxo=44000
````

```bash
./qx tx-encode -i 7e30d6cb01ae393ff40e29b8c2fe469daeeccdd8cf34bc6489b4b21c09eeb015:0:4294967295:TxTypeRegular -l 0 -o TnaQfibAxsZ2xfB8ehE18UaySC9DRyhTwqL:1:0:TxTypeRegular -o TnVmhArErwnRSLZQiNWguQeq9w4xiHyB86M:1:0:TxTypeRegular -o TnHCAw3gfuZYuwkwFoo48FBpwPMWRMZWaRD:1:0:TxTypeRegular -o TnEvLExwzew6LPL13yXmWKnnZxD1c5Lr8Tw:6.9874:0:TxTypeRegular
010000000115b0ee091cb2b48964bc34cfd8cdecae9d46fec2b8290ef43f39ae01cbd6307e00000000ffffffff04000000e1f505000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000e1f505000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000000e1f505000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000020eda529000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000009cfb61630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a302c2232223a302c2233223a307d7d

./qx tx-sign -k e2ec07936723d6b8c054f1f6bfe2cf1c439733303e5a6f0062d54168d9265b14 -n testnet 010000000115b0ee091cb2b48964bc34cfd8cdecae9d46fec2b8290ef43f39ae01cbd6307e00000000ffffffff04000000e1f505000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000e1f505000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000000e1f505000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000020eda529000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000009cfb61630100-7b22696e707574223a7b2230223a307d2c226f7574707574223a7b2230223a302c2231223a302c2232223a302c2233223a307d7d
010000000115b0ee091cb2b48964bc34cfd8cdecae9d46fec2b8290ef43f39ae01cbd6307e00000000ffffffff04000000e1f505000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000e1f505000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000000e1f505000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000020eda529000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000009cfb6163016a47304402204ecb17a9d070430baf09a12f71a700a5dfc98ff182917475314eb91ae64b749f02202ddeece5a119c464f674127bd6fa48ced35ec770991f348cddcc804dfb94f2bc0121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8

curl -sku "qitmeer:qitmeer123" -X POST -H 'Content-Type: application/json' --data '{"jsonrpc":"1.0","method":"sendRawTransaction","params":["010000000115b0ee091cb2b48964bc34cfd8cdecae9d46fec2b8290ef43f39ae01cbd6307e00000000ffffffff04000000e1f505000000001976a914f9e7e2a5afc7278bdd6f7354e729b8b49bc082f888ac000000e1f505000000001976a914c709344e2f73b022a22974acee264d8e8ad37b5a88ac000000e1f505000000001976a9143d10ba0e5a0ecf10205da56e2e6287d95041e8ec88ac000020eda529000000001976a9142421972d46cedd2283d244665a05bde2aec3446f88ac00000000000000009cfb6163016a47304402204ecb17a9d070430baf09a12f71a700a5dfc98ff182917475314eb91ae64b749f02202ddeece5a119c464f674127bd6fa48ced35ec770991f348cddcc804dfb94f2bc0121036d16522df27ff0b534fc853e19e1f77fefb6c7341819574b5c83fa1e968f54e8", false],"id":1}' https://127.0.0.1:28131 | jq
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": "1e4b29e6583987696cfe15cc69ffdaa060009e8b159c1ecc76ecc6f560e322aa"
}
```





