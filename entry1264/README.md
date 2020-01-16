# [1264 by Dreamy Crab]

## [Lottery]

> - Players buy any of 64 squares (or several).
> - Later contract sets a win amount for each square.
> - Contract will send money back on its own.

Low quality. Simple.

### Address is taken from message

Nothing particulary exploitable, but it's strange that address is taken from message body slice, instead of using source address:

```clojure
slice address = msg_body.preload_bits(264);
```

### Draining contract funds

It is possible to drain contract out of funds, because it will return full `msg_value` back when buying zero squares.

```clojure
() recv_internal(int msg_value, cell msg_cell, slice msg_body) impure {
  ;; ...
  slice address = msg_body.preload_bits(264);
  ;; ...
  int total_price = squares_buyed_count * square_price;
  int balance = msg_value - total_price;
  ;; ...
  throw_if(35, balance < 0);
  send_grams(address, balance);
```

### Implicit floating point arithmetic

I didn't calculated consequences:

```clojure
tuple create_multipliers() impure {
	var arr = new_tuple();
	int i = 0;
	repeat (16) {
		arr~tpush(i * i * i / 13);
		i += 1;
	}
	return arr;
}
```

[1264 by dreamy crab]: https://contest.com/blockchain-2/entry1264
[lottery]: https://github.com/Arseny271/tonGame/blob/3aebfcd9b21c1861c1670e882baecbea41f52884/casino.fc
