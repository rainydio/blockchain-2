# [1263 by Merry Ant]

## [Decentralized Exchange]

> - Supports grams, extra currencies, trc20.
> - Exchange is made once someone create directly opposite order.
> - No partial-exchange of orders.

Very low quality. Complex.

### Draining

There was discussion in the [comments]:

> TRC20: recv_external() security issue. Creator of the token may drain his balance and reinitialize contract effectively gaining his initial balance back.  
> ... by Nice Llama
>
> Thanks! This is an absurd mistake that made recv_external critically wrong. I fixed it in the repository.  
> ... by Merry Ant

This contract has even worse external message handler implementation:

```clojure
() recv_external(slice sender, slice in_msg) impure {
  accept_message();
}
```

### Contract is paying for queries

Instead of carrying inbound value contract is paying for queries that it makes:

```clojure
int gas_grams = 1 << 30; ;; ~1 gram of value to process
builder msg = begin_cell()
  .store_uint(0x18, 6)
  .store_uint(0x4, 3)
  .store_slice(addr)
  .store_grams(gas_grams)
  .store_dict(currencies)
  .store_uint(0, 4 + 4 + 64 + 32 + 1 + 1);
send_raw_message(msg.end_cell(), 0);
```

### Stealing grams

There is no whitelist of tokens. Contract is paying for queries and this can be used to steal funds. TRC20 address is taken from input, so contract will send query to it carrying 1 gram.

```clojure
() recv_internal(int msg_value, cell in_msg_cell, slice in_msg) impure {
  cs = in_msg;
  process_user_msg(action, msg_value, extra_currencies, sender, cs);
}

() process_user_msg(int action, int msg_value, cell extra_currencies, slice sender, slice cs) impure {
  slice from_currency = cs~load_currency();
  transfer_from(sender, from_value, from_currency, seqno + 1);
}

() transfer_from(slice addr, int value, slice currency, int order_id) impure {
  int gas_grams = 1 << 30; ;; ~1 gram of value to process and obtain answer
  builder msg = begin_cell()
    .store_uint(0x18, 6)
    .store_uint(0x4, 3)
    .store_slice(currency)
    .store_grams(gas_grams)
    .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1)
    .store_uint(2, 32) ;; transfer from
    .store_uint(order_id, 64) ;; transfer from
    .store_ref(ref.end_cell());
  send_raw_message(msg.end_cell(), 0);
}
```

## [TRC20]

> - Usual constants `name`, `symbol`, `decimals`.
> - Total supply is also constant assigned to single address.
> - Three ususal operations `transfer`, `transfer_from`, `approve`.

Low quality. Simple. No minting. No balance or allowance queries. Clean!

### Initialization front-running

Initial balance is read from initial message allowing to front-run initialization:

```clojure
() recv_external(slice in_msg) impure {
  slice address = in_msg~load_bits(264);
  (..., int total_supply, cell balances, cell allowed) = load_data(get_data());
  throw_unless(60, balances.dict_empty?());
  accept_message();
  balances~set_balance(address, total_supply);
  store_data(..., total_supply, balances, allowed);
}
```

### Incorrectly handles simple transfer

Simple trasnfer is handled incorrectly. First reading operation and the checking if body is empty:

```clojure
() recv_internal(int msg_value, cell in_msg_cell, slice in_msg) impure {
  cs = in_msg;
  int action = cs~load_uint(32);
  if ((action == 0) & (cs.slice_bits() == 0)) { ;; just accept grams
    return ();
  }
}
```

### That subtle parse_msg

There is subtle `parse_msg` that is concatenating body slice with reference. But also it contains code that re-encodes simple transfer comment that isn't needed here.

[1263 by merry ant]: https://contest.com/blockchain-2/entry1263
[decentralized exchange]: https://github.com/cod1ng-studio/ton-exchange/blob/e20872afd907fa260d80cadc25046d66bc9ab12f/dex.fc
[trc20]: https://github.com/cod1ng-studio/TRC20/blob/28ac5d39a88284c9ada4685f07d317ccd07551d3/token.fc
[comments]: https://contest.com/blockchain-2/entry1263
