# EOS hello world

> An [EOS](https://eos.io/) hello world contract tutorial

## Tutorial

Install EOS node

```bash
# pull eos image
docker pull eosio/eos-dev

# run eos node
docker run --rm --name eosio -d -p 8888:8888 -p 9876:9876 -v /tmp/work:/work -v /tmp/eosio/data:/mnt/dev/data -v /tmp/eosio/config:/mnt/dev/config eosio/eos-dev  /bin/bash -c "nodeos -e -p eosio --plugin eosio::producer_plugin --plugin eosio::history_plugin --plugin eosio::chain_api_plugin --plugin eosio::history_api_plugin --plugin eosio::http_plugin -d /mnt/dev/data --config-dir /mnt/dev/config --http-server-address=0.0.0.0:8888 --access-control-allow-origin=* --contracts-console --http-validate-host=false"

# tail node logs
docker logs --follow eosio
```

Verify it's running `http://localhost:8888/v1/chain/get_info`

In a seperate tab, SSH into container

```bash
docker exec -it eosio bash
```

Creating a wallet

```bash
root@080f8f30a32b:/# cleos wallet create --to-console
"/opt/eosio/bin/keosd" launched
Creating wallet: default
Save password to use in the future to unlock this wallet.
Without password imported keys will not be retrievable.
"PW5JgTJAcf9JTeHCrecqntBeZnjePoRWSpCxspfiA386EN5zyncQr"
```

Unlock wallet to allow importing keys

```bash
root@080f8f30a32b:/hello# cleos wallet unlock
password: Unlocked: default
```

Create a key

```bash
root@080f8f30a32b:/# cleos create key --to-console
Private key: 5HuQGCKyW1cGHDVuHdfLfDHx3QqBhhudHSGFu9NCMSGkg3e6dnk
Public key: EOS5wpScdMKwBnmZ95AGrAqFUfLUL8pg2V35eQb6GKyfu7wvAHGs8
```

Import key

```bash
root@080f8f30a32b:/# cleos wallet import --private-key 5HuQGCKyW1cGHDVuHdfLfDHx3QqBhhudHSGFu9NCMSGkg3e6dnk
imported private key for: EOS5wpScdMKwBnmZ95AGrAqFUfLUL8pg2V35eQb6GKyfu7wvAHGs8
```

Set the eosio master contract and sign it with the private key we imported ([?](https://github.com/EOSIO/eos/issues/4154#issuecomment-397820824))

```bash
root@080f8f30a32b:/# cleos set contract eosio contracts/eosio.bios -p eosio@active
Publishing contract...
```

Create an account

```bash
root@080f8f30a32b:/# cleos create account eosio bob EOS5wpScdMKwBnmZ95AGrAqFUfLUL8pg2V35eQb6GKyfu7wvAHGs8
executed transaction: 2ee5250dc8bcaf8c23fef2e9adc2abc50e93687249aab0c8908175f87e6ca6b5  200 bytes  392 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"bob","owner":{"threshold":1,"keys":[{"key":"EOS5wpScdMKwBnmZ95AGrAqFUfLU...
warning: transaction executed locally, but may not be confirmed by the network yet    ]
```

List accounts

```bash
root@080f8f30a32b:/# cleos get accounts EOS5wpScdMKwBnmZ95AGrAqFUfLUL8pg2V35eQb6GKyfu7wvAHGs8
{
  "account_names": [
    "bob"
  ]
}
```

Create a directory for the contract and cd into it

```bash
root@080f8f30a32b:/# mkdir hello
root@080f8f30a32b:/# cd hello/
```

Copy the contents of the hello world contract into `hello.cpp`

```bash
root@080f8f30a32b:/hello# vim hello.cpp
```

Compile the contract into WASM

```bash
root@080f8f30a32b:/hello# eosiocpp -o hello.wast hello.cpp
```

Generate the ABI

```bash
root@080f8f30a32b:/hello# eosiocpp -g hello.abi hello.cpp
2018-08-17T18:05:53.631 thread-0   abi_generator.hpp:68          ricardian_contracts  ] Warning, no ricardian clauses found for hello

2018-08-17T18:05:53.632 thread-0   abi_generator.hpp:75          ricardian_contracts  ] Warning, no ricardian contract found for hi

Generated hello.abi ...
```

Create an account to upload the contract

```bash
root@080f8f30a32b:/# cleos create account eosio hello.code EOS5wpScdMKwBnmZ95AGrAqFUfLUL8pg2V35eQb6GKyfu7wvAHGs8 EOS5wpScdMKwBnmZ95AGrAqFUfLUL8pg2V35eQb6GKyfu7wvAHGs8
executed transaction: 1dd7ca1707bacbf02a3ad1b529b7b0072162d1dde3576674bd70143a7c573d27  200 bytes  359 us
#         eosio <= eosio::newaccount            {"creator":"eosio","name":"hello.code","owner":{"threshold":1,"keys":[{"key":"EOS5wpScdMKwBnmZ95AGr...
warning: transaction executed locally, but may not be confirmed by the network yet    ]
```

Upload the hello world contract

```bash
root@080f8f30a32b:/hello# cleos set contract hello.code ../hello -p hello.code@active
Reading WASM from ../hello/hello.wasm...
Publishing contract...
executed transaction: c29c81b8314d9666ab9bb5a58874e22f8945761dac4673a573cccaab0175e339  1800 bytes  522 us
#         eosio <= eosio::setcode               {"account":"hello.code","vmtype":0,"vmversion":0,"code":"0061736d01000000013b0c60027f7e006000017e60...
#         eosio <= eosio::setabi                {"account":"hello.code","abi":"0e656f73696f3a3a6162692f312e30000102686900010475736572046e616d650100...
warning: transaction executed locally, but may not be confirmed by the network yet    ]
```

Send a test transaction without broadcasting and tx info as json

```bash
root@080f8f30a32b:/hello# cleos push action hello.code hi '["bob"]' -d -j
{
  "expiration": "2018-08-17T18:32:36",
  "ref_block_num": 45110,
  "ref_block_prefix": 2386479167,
  "max_net_usage_words": 0,
  "max_cpu_usage_ms": 0,
  "delay_sec": 0,
  "context_free_actions": [],
  "actions": [{
      "account": "hello.code",
      "name": "hi",
      "authorization": [],
      "data": "0000000000000e3d"
    }
  ],
  "transaction_extensions": [],
  "signatures": [],
  "context_free_data": []
}
```

Sign transaction with the user key and broadcast it

```bash
root@080f8f30a32b:/hello# cleos push action hello.code hi '["bob"]' -p bob@active
executed transaction: 70dfa363049d8abc073d01568b9953569abe1e8a1c5da3be735f047a5f6dab19  104 bytes  435 us
#    hello.code <= hello.code::hi               {"user":"bob"}
>> Hello, bob
warning: transaction executed locally, but may not be confirmed by the network yet    ]
```

Stop the EOS node

```bash
root@080f8f30a32b:/hello# exit

$ docker stop eosio
```

## Resources

- [EOSIO Developer Portal](https://developers.eos.io/)

## License

MIT
