# [1268 by Slim Dodo]

## [Stake pool]

> - Collect funds and then subsequently redistribute profits.
> - Optional whitelist of participants.
> - Participant must transfer N + 1 gram to the given address.
> - After collecting funds, pool administrator transfers then to destination address.
> - After receiving transfer from destination address, pool administrator may re-transfer it back to destination address.
> - After the end of pool’s lifetime, each participant can withdraw from 1 gram to proportional balance distribution.

Highish quality. Complex. Almost does what it claims.

### Dictionary indexed by 256u address

Dictionaries `admins`, `members`, `whitelisted`, and `withdrawals` are indexed by 256u address only.

### Init

The only purpose of `recv_external` is initialization. Yet it will keep accepting signed messages incrementing `seqno`.

```clojure
throw_unless(33, msg_seqno == seqno);
throw_unless(34, check_signature(slice_hash(in_msg), signature, pubkey));
accept_message();
if (seqno == 0) {
  ;; ... init
}
store_data(seqno + 1, pubkey, config, admins, members, whitelist, grams);
```

During init `pool_open` is set to `now()`, this can be used as init flag. This way `seqno` and `pubkey` can be dropped completely.

### Deposit processing fees

> To participate in the pool, the participant must transfer N + 1 gram to the given address (required for commission fees, _the surplus will be returned to your account_)

It handles processing fees wrong. It's likely was copied from `elector-code.fc`. I inlined some of the functions:

```clojure
if (member_stake + msg_value - 1000000000 > max_stake) {
  return send_message_back(s_addr, 0xee6f454c, 0, 0, 0, 64);
}
if (msg_value - 1000000000 < min_stake) {
  return send_message_back(s_addr, 0xee6f454c, 0, 0, 0, 64);
}
msg_value -= 1000000000;
throw_unless(44, msg_value);
accept_message();
;; ... member_stake += msg_value, save_data, etc
return send_message_back(s_addr, 0xf374484c, 0, 0, 1000000000, 3);
```

I'm not sure if `member_stake` wasn't added to `msg_value` on purpose during `min_stake` check. Only thanks to that `throw_unless` correctly checks that at least GR\$1 was sent to pay for processing fees. But that GR\$1 is immediately sent back. Because GR\$1 is sent back and the rest can be withdrawn later it is possible to drain contract out of funds. That `accept_message` makes no sense, but removing it doesn't remove the threat.

## Sending to destination address

Sending collected grams to destination address is performed by admin. Collected grams can be re-sent several times. Destination address can be pre-set or chosen once by admin.

```clojure
if (
  (destination_addr != 0) &
  (msg_destination_addr != destination_addr) &
  (msg_destination_wc != destination_wc)
) {
  return return_stake(s_addr, query_id, op);
}
```

_Why use contract at all, if you trust admin to choose destination address?_

Message sent is hard-coded and isn't simple transfer. Instead `recover_stake` operation `0xf96f7324` and `query_id` from inbound message are sent:

```clojure
send_raw_message(begin_cell()
  .store_uint(0x18, 6)
  .store_slice(msg_addr)
  .store_grams(grams)
  .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1)
  .store_uint(0xf96f7324, 32)
  .store_uint(query_id, 64)
.end_cell(), 3);
```

Also funds may be locked out, because `send_raw_message` mode 3 is used. Contract will think that it had sent the money, when in fact it didn't.

### Withdrawals

> - After the end of pool’s lifetime, each participant can withdraw from 1 gram to proportional balance distribution.

This isn't implemented, but there were attempts. Contract tracks balance separately in `grams` variable, stores `amount` in `withdrawals` dictionary, but then simply rejects if previous withdrawal was found:

```clojure
(_, int found) = withdrawals.udict_get?(256, src_addr);
if (found) {
  return return_stake(s_addr, query_id, op);
}
;; ... calculate amount
withdrawals~udict_set_builder(256, src_addr, begin_cell().store_grams(amount));
;; ... save, send
```

### Transfer from destionation address

Destination address has to return money back through simple transfer, there is no way to send withdrawal query to destination address. Contract does not accept simple transfer unless it has empty body.

```clojure
var s_addr = cs~load_msg_addr();
if (in_msg.slice_empty?()) {
  return process_transfer(s_addr, msg_value);
}
```

[1268 by slim dodo]: https://contest.com/blockchain-2/entry1261
[stake pool]: https://github.com/vanasprin/pool_ton/blob/bbe4a6bb5cf27c1b4d80a9636c8a309a868014a5/smartcont/pool-code.fc
