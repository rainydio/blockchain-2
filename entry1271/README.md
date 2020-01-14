# [1271 by Dreamy Pug]

## [Apportion group expense]

> - Owner add member addresses and public keys.
> - Member sends his expenses.
> - Owner approves expenses.
> - Contract recalculates who should pay whom.
> - Member can query his balance.
> - Member sends transfer that will be redirected to other members.
> - Member can withdraw manually.

Low quality. Complex.

Suprisingly although contract stores both member addresses and member public keys. Addresses are used only to authenticate transfer sender.

### Any member can withdraw to any address

Contract allows any member to withdraw any amount to any address:

```clojure
;; withdraw grams from smc balance
var amount = cs~load_grams();
var msg = new_dict();
ifnot (slice_refs_empty?(cs)) {
  msg = cs~load_ref();
}
throw_if(33, (get_balance().first() - amount) < tolerance);
members~withdraw(tolerance, member_id, amount, msg);
```

Note that `msg` can be loaded from input message, and `amount` claimed doesn't have to be equal to amount set in `msg`

[1271 by dreamy pug]: https://contest.com/blockchain-2/entry1271
[apportion group expense]: https://github.com/Vasylll/apportion-ton-smc/blob/ca4c844f5d860fca460a3d40443955c9dbcaf6f4/apportion-code.fc
