# [1269 by Shiny Giraffe]

## [Conditional transfer]

> - **Minimum amount of collected grams** - this allows to set how much grams have to be on the balance of this smart contract in order to pass > collected grams to beneficiaries
> - **Maximum amount of collected grams** - this lets ceiling the amount of grams to be collected, any amount that overflows this will be > immediately returned to sender
> - **Minimum accepted amount** - combats dust by enforcing minimum amount per each message
> - **Collection deadline** - enforces that **minimum amount** shall be collected by this time, otherwise all collected grams will become > reclaimable by senders
> - **Release locktime** - makes it that neither releasing or reclaiming collected grams is not possible until this specified moment (if > collection was successful, of course)
> - **Release deadline** - by this moment money should be released or refunded, in case it is not done by this moment, next incoming message will > trigger the action or change
> - **Continuous collection flag** - it is possible to permit (or deny) continuing of collection of grams even after **collection deadline** have > passed (if target was reached successfully)
> - **Automatic release** - it is possible to specify whether money should be released to beneficiaries or refunded to senders if decision is not > made by **release deadline**
> - **Controller support** - an external entity may control this contract, either a user with signed external messages, or another smart contract > via internal messages. The control is limited to releasing money to beneficiaries or refunding them to senders, if allowed by deadlines and > locktime.
> - **Multiple beneficiaries support** - it is possible to specify multiple beneficiaries, and what they should receive upon releasing grams: a > fixed amount of grams or a percentage against the total balance of contract. There always must be ultimate beneficiary which will receive all > remaining grams from the balance of the contract after distributing it to another beneficiaries.
> - **Ultimate simplicity for users** - putting money into the contract and reclaiming it does not require any special programs or constructing > special messages: contract will automatically receive and account for received funds, and if the contract is in collect phase, will > automatically attempt to return any amount that is due to the user, no any commands and special queries required, simple transfer from the > wallet should suffice.
> - **Getter methods** - there are several getter methods implemented that allows to easily check some important status information about the > contract.

Ok quality. Complex. Tuples as structure element!

### Mixing controller address and public key

There is controller role that can release funds and initiate reclaim. It can be either public key or address. That creates several difficulties.

#### Destroyed check

Internal message handler begins with check of destroyed flag, that will immediately send everything to beneficiary:

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

I don't really know if this function may raise exception. But if it does, then some beneficiary would not see their money. They all gonna be taken by main beneficiary through internal handler.

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

I was under assumption, that some _client contract_ will store information in this user's data storage proxy. Later user client software may proxy call that information into _client contract_. But if box name isn't sent then there is no way to authenticate which box _client contract_ received. For all purposes this may be information filled by someone else:

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

[1269 by shiny giraffe]: https://contest.com/blockchain-2/entry1269
[conditional transfer]: https://github.com/Skydev0h/ton-freestyle/blob/45b5c9df9fb193dab824b75a37c8b810f7f05bef/src/ct/ctsc.fc
[data storage proxy]: https://github.com/Skydev0h/ton-freestyle/blob/45b5c9df9fb193dab824b75a37c8b810f7f05bef/src/dsp/dspsc.fc
