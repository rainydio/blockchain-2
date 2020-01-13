# [1266 by Nice Llama]

## [ERC20]

> ERC20 tokens ported to TON blockchain
>
> Features:
>
> - token name and symbol
> - token decimals
> - multiple non-interchangeable networks
> - ready to use with lite-client

### Multiple non-interchangeable networks

It's a feature! Contract doesn't allow to send tokens between workchains. Dictionary with balances is stored within dictionary with workchains. Functions that handle workchains and errors are wrapped around balance functions. They were clearly written afterwards by a different person. Could have just rejected any message from workchain different than used by contract.

### Minting tokens

There is no mechanism for minting tokens. In example `deploy.fif` whole supply is assigned to owner address. And owner has to send those out through separate transfers.

```fsharp
// setup balances to give owner_address the entire supply
variable network_balances dictnew network_balances !
<b supply 256 u, b> <s owner_address network_balances @ 256 udict! drop network_balances !
<b network_balances @ ref, wc balances @ 8 b>idict! drop balances !
```

As result `supply` is just another constant like `name`, `symbol`, and `decimal`. In real ERC20 supply sum of tokens minted so far.

### Ready to use with lite-client

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

### Contract doesn't handle queries

What kind of ERC20 is it, if it cannot be asked about balance from another contract?

## [Ring-signature based mixer]

> Ring-signature based mixer
>
> Find friends and use this contract to mix your outputs using ring-signature algorithm implemented with RSA keys. Allows to detach senders and recipients without using any centralized service.
>
> Features:
>
> - RSA on funC
> - Ring signatures on funC
> - Prepare contract offchain and process withdrawals onchain

### RSA on funC

Keys are small, so this implementation cannot be used. Whole difficulty of implementing _RSA on funC_ has to with key size.

### Ring signatures serve no purpose

Withdrawal is authorized by `approve_addresses` dictionary, and then additionally signature is checked. Nothing really changes if ring signatures are completely dropped.

```clojure
(int exist, int index) = if_exist(approve_addresses, number_of_participants, addr);
if(exist == 0) {
  return send_message(addr, Fail(), query_id, 1, 0, 0);
}
```

### Funds lock-out

Contract refuses to process withdrawal until all deposits were made. There is no way to bail out. If attacker can lure victims into contract and then refuse to send his deposit they are likely to divide losses providing profit for attacker.

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
[erc20]: https://github.com/adoriasoft/ton_fungible_token
[ring-signature based mixer]: https://github.com/adoriasoft/ton_payment_mixer
[pr218 by blackyblack]: https://github.com/ton-blockchain/ton/pull/218
