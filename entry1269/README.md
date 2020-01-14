# [1269 by Shiny Giraffe]

## [Conditional transfer]

> - _Multiple beneficiaries_ receiving percentage or fixed amount.
> - Optional _controller_ that can either _release_ money to beneficiaries or enable _reclaims_.
> - _Controller_ can be either public key or address.
> - _Deposit_ and _reclaim_ through simple transfer depending on contract mode.
> - _Minimum amount_ that has to be collected or _reclaims_ will be enabled.
> - _Maximum amount_ that has to be collected. Futhure transfers will be returned back to sender.
> - _Minimum accepted amount_ that will not be returned back to sender.
> - _Collection deadline_. Aftewards _reclaims_ are enabled if _minimum amount_ isn't collected.
> - _Continuous collection flag_. Disables _collection deadline_.
> - _Realease locktime_ until neither _reclaims_ nor _release_ is possible.
> - _Release deadline_. Next message after deadline triggers _reclaim_ or _release_ (specified by whom???).
> - _Automatic release_. Default (_reclaim_ or _release_) when no decision was made by _release deadline_.

_There are contradictory claims! Reviewing..._

Ok quality. Complex. Tuples as structure element!

#### Destroyed check

Mixing public key and address creates difficulties. Internal message handler begins with check of destroyed flag, that will immediately send everything to beneficiary:

```clojure
() recv_internal(int msg_value, cell in_msg_cell, slice in_msg) impure {
	var data = Storage:Load();
	int Flags = data.[flags_pack]();
	if (Flags.Destroyed?()) {
		accept_message();
		Message:Send(data.[beneficiary_wc](), data.[beneficiary_addr](), 0, MF:AllBalance());
		return();
	}
```

Controller might have sent message as external, which may have failed, and now internal message will finish the job. I think that `accept_message` is artifact of refactoring.

#### Failing to release funds

It appears that multiple beneficiaries were added later. There is main beneficiary and then there are additional ones. There is loop first, and then main beneficiary gets the rest of the balance:

```clojure
() Logic:DoRelease(tuple data) impure {
  accept_message();
	Flags |= Flag:Destroyed();
  data~[flags_pack=](Flags);
  Storage:Save(data);
	commit();
  ;; ... loading beneficiaries linked list
  while (val.slice_bits()) {
    ;; ... calculating amount, building message cell
    send_raw_message(msg.end_cell(), MF:IgnoreErrors());
    ;; Message:Send(bwc, baddr, bvalue, 0);
  }
  Message:Send(data.[beneficiary_wc](), data.[beneficiary_addr](), 0, MF:AllBalance());
}
```

Commented out line indicates attempts to send message properly. I guess that destroyed flag check was added to internal handler when message was failing. Then Shiny Giraffe finally settled on suppressing errors with +2 flag. There is also comment about disabled functionaly and high precision percentages. Maybe it's somehow related.

I don't really know if this function may raise exception. But if it does, then some beneficiaries would not see their money. They all gonna be taken by main beneficiary through internal handler.

## [Data storage proxy]

> - Contract owner defines named boxes and security configuration of them.
> - Box may be filled with external message by owner of the contract.
> - Box may be filled with external message by owner of the box.
> - Box may be filled with internal message by owner of the box.
> - Box data can be obtained via getter methods.
> - Box data can be injected into internal message and forwarded to provided address.
> - Box data can be queried with internal message.

Ok quality. Complex. Almost does what it claims. But I don't understand the purpose.

### External messages only by contract owner

> Box may be filled with external message by owner of the box.

This isn't true. External message handler checks only signature of contract owner.

```clojure
() recv_external(slice in_msg) impure {
  var data = Storage:Load();
  slice signature = in_msg~load_bits(512);
  int shash = slice_hash(in_msg);
  throw_unless(42, check_signature(shash, signature, data.[owner_key]()));
  ;; ...
}
```

### Proxying does not include box name

If box name isn't sent there is no way to authenticate which box _target contract_ received. For all purposes this may be information filled by someone else:

```clojure
() recv_internal(int msg_value, cell in_msg_cell, slice in_msg) impure {
  ;; ...
  if (op == Op:Proxy()) {
    ;; ...
    slice prefix = in_msg~load_ref().begin_parse();
    slice suffix = in_msg~load_ref().begin_parse();
    Message:Send(targ_wcid, targ_addr, 0, MF:MsgBalance() | (box_mode << 8), prefix, box.[data](), suffix);
    return ();
  }
}
```

Usages proposed by Shiny Giraffe are to synthetic. I was trying to invent one where _target contract_ stores information in this user's data storage proxy. Later user's client software may proxy call that information into _target contract_. But without name that can be anyones information.

[1269 by shiny giraffe]: https://contest.com/blockchain-2/entry1269
[conditional transfer]: https://github.com/Skydev0h/ton-freestyle/blob/45b5c9df9fb193dab824b75a37c8b810f7f05bef/src/ct/ctsc.fc
[data storage proxy]: https://github.com/Skydev0h/ton-freestyle/blob/45b5c9df9fb193dab824b75a37c8b810f7f05bef/src/dsp/dspsc.fc
