# [1269 by Shiny Giraffe]

## [Conditional transfer]

> - Multiple _beneficiaries_ receiving percentage or fixed amount.
> - Two possible contract state transitions after collecting _deposits_:
>   - _Releasing_ money to _beneficiaries_.
>   - Enabling _reclaims_.
> - _Deposit_ and _reclaim_ through simple transfer depending on contract state.
> - Required _minimum amount_ that has to be collected.
> - Optional _maximum amount_ that has to be collected, anything above it is returned to sender.
> - Optional _minimum accepted amount_, _deposits_ lower than that are returned to sender.
> - Optional _collection deadline_:
>   - Aftewards _reclaims_ are enabled unless _minimum amount_ was collected.
>   - Aftewards _deposits_ are disabled unless _continuous collection_ is enabled.
> - Optional _release deadline_, aftewards required _action_ will transition contract state to:
>   - _Releasing_ money to _beneficiaries_.
>   - Enabling _reclaims_.
> - Optional _controller_:
>   - Public key or address.
>   - Can enable _reclaims_.
>   - Can _release_ money to _beneficiaries_.
>   - Cannot act until required _release locktime_.
>   - Cannot act until required _collection deadline_.
>   - Cannot act after optional _release deadline_.

Ok quality. Spaghetti-level complex. Does what it claims. Correctly handles deposits! No exploits found!

Although most of the logic is within internal message handler, it's still extremely complex and inefficient. Original description doesn't specify which of the settings are optional.

### Controller and release locktime

It doesn't make sense to have such restriction for controller. Restricting controller actions between collection deadline (if set) and release deadline (if set) is enough and eliminates possible controller lock-out due to release locktime being past release deadline.

### Controller without collection deadline

Surprisingly collection deadline isn't optional when restricting controller (release deadline is optional). There is no way to retrieve funds if contract is created without collection deadline and without release deadline. This is realistic scenario if intent was to collect any amount until controller decides what to do.

```clojure
() Controller:ReleaseFunds(tuple data, int now) impure {
  throw_if(31, now < data.[collect_deadline]());
  throw_if(32, now < data.[release_locktime]());
  throw_if(33, (data.[release_deadline]() != 0) & (now >= data.[release_deadline]()));
  ;; ...
}

() Controller:InitiateReclaim(tuple data, int now) impure {
  throw_if(31, now < data.[collect_deadline]());
  throw_if(32, now < data.[release_locktime]());
  throw_if(33, (data.[release_deadline]() != 0) & (now >= data.[release_deadline]()));
  ;; ...
}
```

### Controller and external message handler

Controller optionally can execute action through external message handler. This is questionable decision given that controller can execute action only once.

Sometimes there are additional checks to call `accept_message`:

```clojure
() Controller:ReleaseFunds(tuple data, int now) impure {
  if (data.[ext_controller]()) {
    accept_message();
  }
  Logic:DoRelease(data);
}
```

But sometimes there aren't:

```clojure
() Logic:DoRelease(tuple data) impure {
  accept_message();
}
```

_Also note that `Logic:DoRelease` is called from `Controller:ReleaseFunds` that already called `accept_message`._

And sometimes `accept_message` doesn't make much sense:

```clojure
() recv_internal(int msg_value, cell in_msg_cell, slice in_msg) impure {
  if (Flags.Destroyed?()) {
    accept_message();
    Message:Send(beneficiary_wc, beneficiary_addr, 0, MF:AllBalance());
    return();
  }
}
```

### Releasing funds

There is loop over additional beneficiaries first, and then main beneficiary gets the rest of the balance:

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

I don't see reason to `commit` and ignore errors. Total amount sent is tracked, validity of additional beneficiary cell can be checked. If there was no external message handler then amount that has to be sent can be calculated precisely. In any case uncertainty due to it can be contained by subtracting small amount from balance (for processing) that will be sent to main beneficiary at the end.

If function that releases funds is safe and correct then there is no reason to keep destroyed flag and questionable backups:

```clojure
() recv_internal(int msg_value, cell in_msg_cell, slice in_msg) impure {
  if (Flags.Destroyed?()) {
    accept_message();
    Message:Send(beneficiary_wc, beneficiary_addr, 0, MF:AllBalance());
    return();
  }
}

() recv_external(slice in_msg) impure {
  if (Flags.Destroyed?()) {
    accept_message();
    Message:Send(data.[beneficiary_wc](), data.[beneficiary_addr](), 0, MF:AllBalance());
    return();
  }
}
```

_Unnecessary `accept_message` in internal message handler is clearly result of copy-paste._

Main beneficiary can still receive all funds if something goes wrong. This may seem like a rational choice. But on the other hand main beneficiary now has the incentive to intetionally crash release funds function. It's better to be implemented without requiring such backups.

### Multiple data serialization

Data cell is serialized, saved, and sometimes committed (in internal message handler) several times. I see no reason why it could not be saved once at the very end right before return.

### No exloits!

Although contract is complex and has many options that can potentially interact in unexpected ways. I wasn't able to find a single realistic exploit. To give an idea what I was able to find:

```clojure
if ((max_grams > 0) & (bal + deposit > max_grams)) {
  deposit = max_grams - bal;
}
if (deposit == 0) {
  Message:Send(src_wc, src_addr, 0, MF:MsgBalance());
  return ();
}
```

This limits deposit so not to surpass `max_grams`. It doesn't assume that `bal` is already higher than `max_grams`. Balance can be increased past `max_grams` with non-bounceable message. As result `deposit` of next message will be negative. Two messages have to be processed immediately after each other. So it only allows validator to reclaim his funds once `max_grams` was reached. Yeah....

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
