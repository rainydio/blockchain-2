# [1247 by Small Kitten]

## [Ticket System]

Contract `seqno` is only 16 bit long, and there is attempt to loop it (great idea!). But it isn't stored in final data.

```clojure
if (query_seq == 65535) { ;; reset query number
  query_seq = 0;
}
saveData(owner, seq + 1, price, endOfSale, endOfRefunds, refundPercentage, store, eventInfo);
return ();
```

Incorrect understanding of inbound message value:

```clojure
int paid = msg_value - 1000000000; ;; makes sure that sender paid processing fee
if (paid < 0) {
  return (); ;; not enough funds for processing, we wont send msg back to prevent fund depletion
}
```

## [Software Validation]

[1247 by small kitten]: https://contest.com/blockchain-2/entry1247
[ticket system]: https://github.com/5eraph/ts
[software validation]: https://github.com/5eraph/sw
