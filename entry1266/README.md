# [1266 by Nice Llama]

## [ERC20]

> - Token name, symbol, decimals
> - Multiple non-interchangeable networks
> - Ready to use with lite-client

Low quality. Simple. Almost does what it claims, but what it claims!

### Multiple non-interchangeable networks

Contract doesn't allow to send tokens between workchains. Dictionary with balances is stored within dictionary with workchains. Functions that handle workchains and errors are wrapped around balance functions. They were clearly written afterwards by a different person. Could have just rejected any message from workchain different than used by contract.

### Isn't ready to be used with lite-client

Couple of methods simply return whole dictionary with balances. And they forgot to update comment there, it's now wrapped into workchain dictionary.

```clojure
;; get balances dictionary (owner -> balance)
cell getbalances() method_id {
  (_, _, _, _, cell balances, _) = load_data();
  return balances;
}

;; get approvals dictionary (owner -> { sender -> approved })
cell getapprovals() method_id {
  (_, _, _, _, _, cell approvals) = load_data();
  return approvals;
}
```

At the same time `getbalance` is more advanced, and walks through dictionaries to get balance.

## [Ring-signature based mixer]

> - Find friends and use this contract to mix your outputs.
> - Allows to detach senders and recipients without using any centralized service.
> - RSA on funC
> - Ring signatures on funC
> - Prepare contract offchain and process withdrawals onchain

Very low quality. Simple. Doesn't do what it claims.

> Find friends and use this contract to mix your outputs

With friends???

### RSA on funC

Keys are small, so this implementation cannot be used. Whole difficulty of implementing _RSA on funC_ has to do with key size.

### Ring signatures serve no purpose

Withdrawal is authorized by `approve_addresses` dictionary, and then additionally signature is checked. Nothing really changes if ring signatures are completely dropped.

```clojure
(int exist, int index) = if_exist(approve_addresses, number_of_participants, addr);
if(exist == 0) {
  return send_message(addr, Fail(), query_id, 1, 0, 0);
}
```

### Funds lock-out

Contract refuses to process withdrawal until all deposits were made. There is no way to bail out. If I can lure my ~~victims~~ _friends_ into contract and then refuse to send my deposit they are likely to divide losses providing profit for me.

```clojure
if(get_balance().first() > number_of_participants * payment_amount) {
  money_available = 1;
}
if(money_available == 0) {
  return send_message(addr, Fail(), query_id, 1, 0, 0);
}
```

## [PR218 by blackyblack]

> body-cell slice bits are calculated before 1 i flag so there is a case when builder has exactly the body-cell length bits left and body-cell won't fit.

Correct, but instead:

```fsharp
sbitrefs swap 1 + swap rot brembitrefs rot >= -rot <= and
```

better:

```fsharp
sbitrefs rot brembitrefs rot >= -rot < and
```

[1266 by nice llama]: https://contest.com/blockchain-2/entry1266
[erc20]: https://github.com/adoriasoft/ton_fungible_token/blob/feb80f612dd7b6cd68f21d7526444c97ab4a513f/token-contract.fc
[ring-signature based mixer]: https://github.com/adoriasoft/ton_payment_mixer/blob/6e87a6d4452db30148ae733dfca75881054348f5/src/laundromat.fc
[pr218 by blackyblack]: https://github.com/ton-blockchain/ton/pull/218
