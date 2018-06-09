# Backgrounds

It is necessary to buy ram for creating new accounts in EOSIO. To buy, it costs some EOS. To boot up the mainnet based on EOSIO, some of block producer candidates proposed method to create user accounts in `genesis.json`.

Two methods emerged. 1) Print some EOS more and gift ram costs to genesis users. 2) Transfer (or Workout) and make supply to 1B exactly.

But there are hot debates about results of methods proposed. Before **resignation of system accounts**, states below are ideal and desirable because this is a blockchain. See three conditions below.

## Three conditions

1. **supply of EOS token** : 1,000,000,000.0000 EOS exactly.
   * Initial EOS distribution is 1B.
   * If larger than 1B, inflation is slightly **more**.
2. **connector balance of EOS in Bancor system** : (1,000,000,000.0000 / 1000) exactly.
    * If larger than value above, RAM price starts slightly **higher** and grows slightly **steeper**. In consequence, users pay **more**.
3. **Initial token balance of system accounts** : 0.0000 EOS exactly.
   * sum of all accounts including user accounts and system accounts should be same with supply of EOS token above.
   * system accounts such as `eosio`, `eosio.ram`, `eosio.ramfee`, `eosio.stake`

It is quite easy to check whether three conditions are met before resignation of system accounts using commands below:
```
cleos get currency stats eosio.token EOS
cleos get currency balance eosio.token eosio EOS
# .... and many other system accounts
cleos get table eosio eosio rammarket
```

In either way, in order to be in desirable states, three conditions above should be satisfied. But there were no viable options to do that although hot debates.

To fill the gap between proposed methods and to make adjustments easy, `eos-ramcost` is implemented, tested and presented to community before mainnet launch for review and inclusion.

# `eos-ramcost` contracts

Two contracts, `eosio.token.ramcost` and `eosio.system.ramcost` for adjusting during boot. **And only three simple actions are just added to be audited easily.** It is tested on master branch of eos-mainnet repo only. And **`wasm` included.**

# Codes

Original `eosio.token` and `eosio.system` is from master branch of eos-mainnet repo.
* commit id : `464b687e684290beee6a37c3521f3181216700e8`

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

Or you can use `wasm` files included.

# Usage

* You can use `eos-ramcost` contracts from the beginning or when necessary.
```
cleos set contract eosio.token /path/to/eosio.token.ramcost
cleos set contract eosio /path/to/eosio.system.ramcost
```

It's time to adjust for the common good. **Calculate the amount to adjust before.**

* Check the initial state of 1) token supply, 2) balance of system accounts, and 3) bancor state.
```
cleos get currency stats eosio.token EOS
cleos get currency balance eosio.token eosio EOS
# .... and many other system accounts
cleos get table eosio eosio rammarket
```
* **Adjust token supply of EOS.** Say subtrating 101019.2000 EOS for example.
```
cleos push action eosio.token unissue '["101019.2000 EOS", "Adjust supply to correct initial supply and inflation"]' -p eosio
```
* Check result.
```
cleos get currency stats eosio.token EOS
```

* **Zero** balances of system accounts of interest.
```
cleos push action eosio.token zerobalance '["eosio.ram", "0.0000 EOS"]' -p eosio
# .... and many other system accounts
```

* Check results.
```
cleos get currency balance eosio.token eosio EOS
# .... and many other system accounts
```

* **Adjust balance of EOS in bancor state.** Say subtracting 249.0751 EOS for example.
```
cleos push action eosio biosramcost '["eosio", "249.0751 EOS"]' -p eosio
```

* Check again 1) token supply, 2) balance of system accounts, and 3) bancor state to satisfy three conditions.
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
