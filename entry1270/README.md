# [1270 by Dreamy Snail]

## [Charity Foundation]

> - Accepts simple transfers.
> - Payments to campaign are multiplied by campaign factor and sent.
> - Owner registers campaigns.

Low quality. Simple. It is _forwarding_ contract. Multiplier is an integer value. Here is the essential part:

```clojure
int balance = get_balance().first();
int value = max(msg_value * multiplier, min_amount);
value = min(value, balance);
value = max(min(value, goal - transferred), msg_value);
sendValue(address, value, 3);
```

### Draining

Contract will always forward at least `msg_value`, it doesn't account processing fees. So it can be drained by campaign beneficiary. But there is likely no reason to do that.

### Underfunding

If contract balance is lower than processing fees sending `msg_value` can't be achieved. Possible error is suppressed by mode 3, but campaign will be updated anyway.

[1270 by dreamy snail]: https://contest.com/blockchain-2/entry1270
[charity foundation]: https://github.com/dblokhin/ton-charity-foundation/blob/8b1f603dd2407bd3f9f9827be24e35060411c6e3/charity-code.fc
