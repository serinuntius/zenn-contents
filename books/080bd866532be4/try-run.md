---
title: "動かしてみる"
---


## storedを動かす
```bash
$ stored -vvv
stored: storage microservice
[2022-10-08T07:51:43Z DEBUG stored] CTL RPC socket 0.0.0.0:60960
[2022-10-08T07:51:43Z DEBUG stored] Starting runtime ...
[2022-10-08T07:51:43Z DEBUG stored::service] Opening RPC API socket 0.0.0.0:60960
[2022-10-08T07:51:43Z DEBUG stored::service] Opening database at /Users/serinuntius/Library/Application Support/Storm Node/sled.db
[2022-10-08T07:51:43Z DEBUG sled::pagecache::snapshot] no previous snapshot found
[2022-10-08T07:51:43Z DEBUG sled::pagecache::iterator] ordering before clearing tears: {}, max_header_stable_lsn: 0
[2022-10-08T07:51:43Z DEBUG sled::pagecache::iterator] in clean_tail_tears, found missing item in tail: None and we'll scan segments {} above lowest lsn 0
[2022-10-08T07:51:43Z DEBUG sled::pagecache::iterator] unable to load new segment: Io(Custom { kind: Other, error: "no segments remaining to iterate over" })
[2022-10-08T07:51:43Z DEBUG sled::pagecache::iterator] filtering out segments after detected tear at (lsn, lid) -1
[2022-10-08T07:51:43Z DEBUG sled::pagecache::iterator] unable to load new segment: Io(Custom { kind: Other, error: "no segments remaining to iterate over" })
[2022-10-08T07:51:43Z DEBUG sled::pagecache::segment] SA starting with tip 0 stable -1 free {}
[2022-10-08T07:51:43Z DEBUG sled::pagecache::iobuf] starting log for a totally fresh system
[2022-10-08T07:51:43Z DEBUG sled::pagecache::segment] segment accountant returning offset: 0 for lsn 0 on deck: {}
[2022-10-08T07:51:43Z DEBUG sled::pagecache::iobuf] starting IoBufs with next_lsn: 0 next_lid: 0
[2022-10-08T07:51:43Z DEBUG sled::pagecache::iobuf] storing lsn 0 in beginning of buffer
```


## rgbdを動かす
別窓で
```bash
$ rgbd -vvv --network bitcoin
```



## 初期化する
初期の発行でトークンを割り当てる場合はBitcoinのTXを指定できます。

https://www.blockchain.com/btc/tx/8bfe1d990e48e013be14d98ca0cd6c1cbcc22ebd8ccf576801a895ad21f578c4
構文は次の通りです。

`rgb20 issue <TICKER> <NAME> <amount>@<txid>:<vout>`

ここではTickerをETH、NameをEthereumとしました。
```bash

$ rgb20 --network bitcoin issue ETH Ethereum 1000@8bfe1d990e48e013be14d98ca0cd6c1cbcc22ebd8ccf576801a895ad21f578c4:0
Contract ID: rgb1ldpa3kxjel3zx50r223wux8lav4cdhxgpnwnfk44qqkgxyntcrnswp5lrv

Contract YAML:
---
schema_id: rgbsh18kp34t5nn5zu4hz6g7lqjdjskw8aaf84ecdntrtrdvzs7gn3rnzskscfq8
chain: mainnet
metadata:
  0:
    - AsciiString: ETH
  1:
    - AsciiString: Ethereum
  3:
    - U8: 8
  4:
    - I64: 1665218277
  160:
    - U64: 1000
owned_rights:
  161:
    value:
      - revealed:
          seal:
            method: TapretFirst
            txid: 8bfe1d990e48e013be14d98ca0cd6c1cbcc22ebd8ccf576801a895ad21f578c4
            vout: 0
            blinding: 14446519280031393988
          state:
            value: 1000
            blinding: "0000000000000000000000000000000000000000000000000000000000000001"
public_rights: []

Contract JSON:
{"schema_id":"rgbsh18kp34t5nn5zu4hz6g7lqjdjskw8aaf84ecdntrtrdvzs7gn3rnzskscfq8","chain":"mainnet","metadata":{"0":[{"AsciiString":"ETH"}],"1":[{"AsciiString":"Ethereum"}],"3":[{"U8":8}],"4":[{"I64":1665218277}],"160":[{"U64":1000}]},"owned_rights":{"161":{"value":[{"revealed":{"seal":{"method":"TapretFirst","txid":"8bfe1d990e48e013be14d98ca0cd6c1cbcc22ebd8ccf576801a895ad21f578c4","vout":0,"blinding":14446519280031393988},"state":{"value":1000,"blinding":"0000000000000000000000000000000000000000000000000000000000000001"}}}]}},"public_rights":[]}

Contract source:
rgbc1qxz49netq3g3el7vhjvhvwpstgtn7kl7qqmkurkg3udcwttjvdkvmex8segsurk2c8q7ygq8ghxj9j2xan9gjtdz53c9xjjz9xjefsexzl3m6denhdkkmnvmmmnhmhlhl07he0m0hm0jllyhpqza65mxfl2psv69pzgpp83ks65vzyc9gqxarw7zhn9lupqxsny39h2p3vprjersgsvmj5zfagrfn25gg5s5aac7jpsrfxw87clvy50zdpjktuqmvrdhrrnmtnp4dq9rz0gmdkwfan6gj2n3958fher97x6gpft60xwd8djfc9x8z90uehxylp9rfkutvn24cgsmw883ppyuun9e2yfan5l3mefx2darrarhfl72vd3pd0yysk4erc9jn7wh0wre4mexqe2rheu74m6vzu6ltfykn7mnxa643k9a4vzutgl4ulf9vqyard6c78yhw6hz03nm8c6ql054a2va3w70v2ltlfpc2klcg8pag6rt0lfu0rcjgcrccvg3s0dxy2qk8f8qdgr7zcqltg3n2vz2mcdkqpdp6ssqjvpkvazjxlcf5cwd7ax52hl0pny3wstmhz9etejtgxa6j3exyff6xmrz5zczjzd7x76k3nyajkthcy8yv6phyp8pwga98dalhg47300hkxecdz7na2g4k2lktz0qa0n5jkkdtlpdxrh5h0tj8mwnwuldk97edyrll5rsm7l75c
```

`Contract ID` は **rgb1** から始まります。

まだこのコントラクトはネットワークにデプロイされていません。初期の状態を作って、コントラクトのコンパイルが終わったぐらいの感覚でいいと思います。


## コントラクトをrgbdに登録する

先程の初期化で最後に出てきた、`Contract source` をコピーしてrgbdに登録します。
```bash
$ rgb-cli --chain bitcoin contract register rgbc1qxz49netq3g3el7vhjvhvwpstgtn7kl7qqmkurkg3udcwttjvdkvmex8segsurk2c8q7ygq8ghxj9j2xan9gjtdz53c9xjjz9xjefsexzl3m6denhdkkmnvmmmnhmhlhl07he0m0hm0jllyhpqza65mxfl2psv69pzgpp83ks65vzyc9gqxarw7zhn9lupqxsny39h2p3vprjersgsvmj5zfagrfn25gg5s5aac7jpsrfxw87clvy50zdpjktuqmvrdhrrnmtnp4dq9rz0gmdkwfan6gj2n3958fher97x6gpft60xwd8djfc9x8z90uehxylp9rfkutvn24cgsmw883ppyuun9e2yfan5l3mefx2darrarhfl72vd3pd0yysk4erc9jn7wh0wre4mexqe2rheu74m6vzu6ltfykn7mnxa643k9a4vzutgl4ulf9vqyard6c78yhw6hz03nm8c6ql054a2va3w70v2ltlfpc2klcg8pag6rt0lfu0rcjgcrccvg3s0dxy2qk8f8qdgr7zcqltg3n2vz2mcdkqpdp6ssqjvpkvazjxlcf5cwd7ax52hl0pny3wstmhz9etejtgxa6j3exyff6xmrz5zczjzd7x76k3nyajkthcy8yv6phyp8pwga98dalhg47300hkxecdz7na2g4k2lktz0qa0n5jkkdtlpdxrh5h0tj8mwnwuldk97edyrll5rsm7l75c

Registering contract rgb1ldpa3kxjel3zx50r223wux8lav4cdhxgpnwnfk44qqkgxyntcrnswp5lrv...
A new bucket daemon instance is started
Success: contract is valid and imported
```

**Success: contract is valid and imported** と表示されれば成功です。

## コントラクトが登録されているか確認する
次のコマンドで確認することができます。
```bash
$ rgb-cli --network bitcoin contract list
Listing contracts...
rgb1ldpa3kxjel3zx50r223wux8lav4cdhxgpnwnfk44qqkgxyntcrnswp5lrv
```

## コントラクトの状態(State)を確認する

```bash
$ rgb-cli --network bitcoin contract state rgb1ldpa3kxjel3zx50r223wux8lav4cdhxgpnwnfk44qqkgxyntcrnswp5lrv
Querying state of rgb1ldpa3kxjel3zx50r223wux8lav4cdhxgpnwnfk44qqkgxyntcrnswp5lrv...
---
schema_id: rgbsh18kp34t5nn5zu4hz6g7lqjdjskw8aaf84ecdntrtrdvzs7gn3rnzskscfq8
root_schema_id: ~
contract_id: rgb1ldpa3kxjel3zx50r223wux8lav4cdhxgpnwnfk44qqkgxyntcrnswp5lrv
metadata:
  0:
    - ETH
  1:
    - Ethereum
  3:
    - "8"
  4:
    - "1665218277"
  160:
    - "1000"
owned_rights: []
owned_values:
  - "1000#0000000000000000000000000000000000000000000000000000000000000001@8bfe1d990e48e013be14d98ca0cd6c1cbcc22ebd8ccf576801a895ad21f578c4:0"
owned_data: []
owned_attachments: []

```


## transferで送ってみる
トークンを受け取る側でblind factorを設定する必要がある（？）
１回だけ使えるアドレス的なもの？

`txob1` から始まる文字列

構文: `$ rgb blind <受け取る側のUTXO>:<vout>`

今回はテストなので、エクスプローラーから適当に拝借します。
https://www.blockchain.com/btc/tx/249defab7eb8e359d83cef07c0ebe433ed6938a4a931aae56c08b67994f5a844

```bash
$ rgb blind 249defab7eb8e359d83cef07c0ebe433ed6938a4a931aae56c08b67994f5a844:0
txob1qwfjr3u7d7rmkrmkx38e4tcgddc586l77acf2j5xjds0tw0ntmfsmmeeak
Blinding factor: 16577889476263249116
```

成功するとblind factorが出力されます。



Transferの状態遷移の下書きを作成します。

```bash
$ rgb-cli --chain bitcoin transfer compose rgb1ldpa3kxjel3zx50r223wux8lav4cdhxgpnwnfk44qqkgxyntcrnswp5lrv 8bfe1d990e48e013be14d98ca0cd6c1cbcc22ebd8ccf576801a895ad21f578c4:0 transfer.rgbc
Composing consignment for state transfer for contract rgb1ldpa3kxjel3zx50r223wux8lav4cdhxgpnwnfk44qqkgxyntcrnswp5lrv...
Task forwarded to bucket daemon
Saving consignment to transfer.rgbc
Success


```

いよいよTransferの状態遷移を作ります（？）

構文: `$ rgb20 -n bitcoin transfer --utxo <COINを持っているUTXO>:<VOUT> <さっき作った状態遷移の下書きのパス> <送る量>@<Blindの時にに出てくるtxobから始まるやつ> <状態遷移ファイルの名前>`


inputで1000持ってるのであれば、1000使い切る必要があるようです。
複数のアドレスを指定してお釣りを渡すスタイルっぽい？

```bash

$ rgb20 -n bitcoin transfer --utxo 8bfe1d990e48e013be14d98ca0cd6c1cbcc22ebd8ccf576801a895ad21f578c4:0 transfer.rgbc 1000@txob1qwfjr3u7d7rmkrmkx38e4tcgddc586l77acf2j5xjds0tw0ntmfsmmeeak transition.rgbt
---
transition_type: 0
metadata: {}
parent_owned_rights:
  e7c06b12832c00b5da34dd0cc8dc862bebff18eea252e35123e2cfd2d8d843fb:
    161:
      - 0
owned_rights:
  161:
    value:
      - confidential_seal:
          seal: txob1qwfjr3u7d7rmkrmkx38e4tcgddc586l77acf2j5xjds0tw0ntmfsmmeeak
          state:
            value: 1000
            blinding: "0000000000000000000000000000000000000000000000000000000000000001"
parent_public_rights: {}
public_rights: []

Success

```

あとはこのファイルをPSBTとしてBitcoinネットワークにコミットすれば、状態遷移のコミットが完了する（？）

なぜかYouTubeのデモがここで終わっていたので一旦はよしとする。
