# `eos-ramcost` contracts

Two contracts, `eosio.token.ramcost` and `eosio.system.ramcost` for adjusting during boot. It is tested on master branch of eos-mainnet repo only. **`wasm` included.**

# Codes

## `eosio.token.ramcost` contract

Two actions are added to original `eosio.token` contract.

* `zerobalance` for zeroing balance of target account.
```
cleos push action eosio.token zerobalance '["eosio.ram", "0.0000 EOS"]' -p eosio
```

* `unissue` for adjusting(subtracting) excess supply of a token.
```
cleos push action eosio.token unissue '["101019.2000 EOS", "Adjust supply to correct initial supply and inflation"]' -p eosio
```

## `eosio.system.ramcost` contract

One action is added to original `eosio.system` contract.

* `biosramcost` for adjusting(subtracting) excess balance of EOS inside Bancor.
```
cleos push action eosio biosramcost '["eosio", "249.0751 EOS"]' -p eosio
```

# Build

It is tested on master branch of eos-mainnet repo only.

* Clone this repo and copy two directorys into under `contracts/`.
* Make some changes into `contracts/CMakeLists.txt`
```
add_subdirectory(eosio.token.ramcost)
add_subdirectory(eosio.system.ramcost)
```
* Then, launch `./eosio_build.sh`

Sorry for not being neat.

# Usage

* You can use `eos-ramcost` contracts from the beginning or when necessary.
```
cleos set contract eosio.token /path/to/eosio.token.ramcost
cleos set contract eosio /path/to/eosio.system.ramcost
```

It's time to adjust for the common good. Calculate the amount to adjust before.

* Check the initial state of 1) token supply, 2) balance of system accounts, and 3) bancor state.
```
cleos get currency stats eosio.token EOS
cleos get currency balance eosio.token eosio EOS

# .... and many other system accounts

cleos get table eosio eosio rammarket
```
* Adjust token supply of EOS. Say subtrating 101019.2000 EOS for example.
```
cleos push action eosio.token unissue '["101019.2000 EOS", "Adjust supply to correct initial supply and inflation"]' -p eosio
```
* Check result.
```
cleos get currency stats eosio.token EOS
```

* Zero balances of system accounts of interest.
```
cleos push action eosio.token zerobalance '["eosio.ram", "0.0000 EOS"]' -p eosio
# .... and many other system accounts
```

* Check results.
```
cleos get currency balance eosio.token eosio EOS
# .... and many other system accounts
```

* Adjust balance of EOS in bancor state.
```
cleos push action eosio biosramcost '["eosio", "249.0751 EOS"]' -p eosio
```

* Check again 1) token supply, 2) balance of system accounts, and 3) bancor state.
```
cleos get currency stats eosio.token EOS
cleos get currency balance 
# .... and many other system accounts
cleos get table eosio eosio rammarket
```

* When finished, setcode again to original contracts.
```
cleos set contract eosio.token /path/to/eosio.token
cleos set contract eosio /path/to/eosio.system
```

Good. We're done. Proceed to the next step.

# Reference

* https://gist.github.com/redjade/bdbcb10708e165b29d0301c5defa1d83
* [EOSeoul Testnet Builder](https://github.com/eoseoul/testnetbuilder)

**EOSeoul**
