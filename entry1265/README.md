# [1265 by Mellow Squid]

## [The Chat Game]

> - Allows item manipulation both for game admins and external users.
> - Game admins may create new items, place them into inner user inventories, put specifications of the items on-chain
> - Game admins can transfer and burn items on behalf on inner users.
> - Game admins have no power to change inventories of outer users.
> - Inner and outer users are equal with only differense that inner users has first 196 bits of their 'addresses' zeroed.

Ok quality. Complex. Does what it claims.

It isn't simple or clean, but written carefuly. By using `raw_reserve` combined with `send_raw_message` mode 128 it handles fees correctly. External message handler used by admin loads user address with `in_msg~load_uint(64)`. This is how those 196 zeroes got there.

> Since it is expected that there may be period of high rate of item operations, and since game engine may be runned independently in many processes (on many servers) we chose to implement replay protection using 'remember recent queries mechanism'. That way using of blockchain is not only 'not obstructive', but otherwise allows to guarantee atomicity.

They got that from `highload-wallet-v2-code.fc` and `multisig-code.fc`. Encoding `valid_until` into `query_id` in my opinion this is the most controversial part. What's the point of maintaining smc guidelines if there are additional rules to what `query_id` should be?

## [OracleHub]

> - Client sends `get_data` internal message to OracleHub.
> - OracleHub emits log message to notify oracle.
> - Oracle sends response in external message to OracleHub.
> - Client gets response, oracle gets fees.

Ok quality. Complex. Does what it claims.

### Drainining contract out of funds

Because oracles send external messages to contract `oracle_seqno` has to be written back into data immediately after `accept`. This isn't done, and so by simply sending message without required `internal_query_id` contract can be drained out of funds.

```clojure
oracle_seqno += 1;
accept_message();
if (op == (128 + 1)) {
  int internal_query_id = cs~load_uint(96);
  ;; ...
}
```

### Oracle punishment fees

There is an attempt to punish oracle for sending invalid message, but that again doesn't account exceptions. Only non-existing `query_id`:

```clojure
int min_fee_per_operation = 25000000;
int fail_fee = min_fee_per_operation * 5;
throw_if(36, balance < min_fee_per_operation);
balance -= min_fee_per_operation;
;; ...
ifnot(request_found?) {
  balance = max(0, balance - fail_fee);
  ;; ...
}
```

Correct way is to first deduct fail fees, commit that, and then add difference with normal fee at the end.

[1265 by mellow squid]: https://contest.com/blockchain-2/entry1265
[the chat game]: entry1265/TheChatGame/collectibles.fc
[oraclehub]: entry1265/OracleHub/oracle_hub.fc
