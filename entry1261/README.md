# [1268 by Slim Dodo]

## [Stake pool]

> To participate in the pool, the participant must transfer N + 1 gram to the given address (required for commission fees, the surplus will be returned to your account). After collecting all the necessary funds, the pool administrator can transfer the funds to the destination address. After getting a refund from the destination address (for example, a refund after validation), the pool administrator can re-transfer the funds to the destination address. After the end of the pool’s lifetime, each participant can submit a request from 1 gram to proportional balance distribution.

### Dictionary indexed by 256u address

Dictionaries `admins`, `members`, `whitelisted`, and `withdrawals` are indexed by 256u address only.

### Init

The only purpose of `recv_external` is initialization. Yet it will keep accepting signed messages incrementing `seqno`. During init `pool_open` is set to `now()`, this can be used as init flag. As result `seqno` and `pubkey` can be dropped.

```clojure
;; exceptions
throw_unless(33, msg_seqno == seqno);
throw_unless(34, check_signature(slice_hash(in_msg), signature, pubkey));

;; init
accept_message();
if (seqno == 0) {
  ;; ...
}
store_data(seqno + 1, pubkey, config, admins, members, whitelist, grams);
```

### Deposit processing fees

> To participate in the pool, the participant must transfer N + 1 gram to the given address (required for commission fees, _the surplus will be returned to your account_)

Here is the relevant part of `process_transfer` where fee is checked. I inlined some of the functions:

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

I'm not sure if `member_stake` wasn't added to `msg_value` on purpose during `min_stake` check. Only thanks to that `throw_unless` correctly checks that at least GR\$1 was sent to pay for processing fees. But that GR\$1 is immediately sent back.

Because GR\$1 is sent back and the rest can be withdrawn later it is possible to drain contract out of funds. That `accept_message` makes no sense, but removing it doesn't remove the threat.

Instead different techinque that is used by [Mellow Squid] and [Shiny Giraffe] can be implemented here:

```clojure
raw_reserve(get_balance().first() - 1000000000, 0);
send_message_back(s_addr, 0xf374484c, 0, 0, 0, 128);
```

---

It appears that they took that code from `elector-code.fc`, but used +3 instead of +2 for `send_raw_message`. I haven't checked standard contracts yet...

> _- Знаешь, что говорит уточка?_  
> _- ???_  
> _- Зря-зря_

Any `send_raw_message` +2 usage indicates that there is something wrong with how funds are handled. Carefully crafted message may leave contract with very small yet still positive balance. Because `elector-code.fc` also contains `accept_message` this can be used to destroy contract completely. Otherwise it boils down to if next inbound message value is added before or after deducting storage fees, and are storage fees deducted during reverts or not.

## Sending to destination address

Sending collected grams to destination address is performed by some admin address. Collected grams can be re-sent several times. Destination address can pre-set or chosen once by admin.

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

Also funds may be locked out, because `send_raw_message` mode 3 is used.

### Withdrawals

There were attempts to implement subsequent withdrawals. Tracking balance separately in `grams` variable, storing `amount` in `withdrawals` dictionary, but then simply rejecting if previous withdrawal was found:

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
[stake pool]: https://github.com/vanasprin/pool_ton
[mellow squid]: ../entry1265
[shiny giraffe]: ../entry1269
