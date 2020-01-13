# [1264 by Dreamy Crab]

## [Lottery]

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

[1264 by dreamy crab]: https://contest.com/blockchain-2/entry1264
[lottery]: https://github.com/Arseny271/tonGame
