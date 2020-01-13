# [1259 by Groovy Fly]

## [Dota2 autochess]

> This is our freaky custom variation of dota2 autochess. We tried to make it works somehow on TON smart-contracts. Key differences from original autochess:
>
> - Match can't be free (so we need some way users pay)
> - Figures' pool is dependent on what figures was in "players hand" they came with
> - Figures can be bought in shop
> - In-game rolls can't be requested from server frequently, so we split pool beyond players > each round evenly for each level but random.
> - Player can't lock good roll
> - Player can see all figures he have access. Access to "next line" costs 2 gold same as original. But now you have "spy eye"
> - After player death in match his/her figures are not going back to pool.

Contract doesn't know game rules. Players can buy figures and contract builds board out of figures players choose once there are 8 of them.

> In-game rolls can't be requested from server frequently, so we split pool beyond players > each round evenly for each level but random.

They aren't talking about contract here.

> After player death in match his/her figures are not going back to pool.

Because contract doesn't know game rules.

## [Ton-Hack]

> This is set of contest-like and/or education-like smart contracts for junior-middle ton smart-contract developers.

Only 4 simple example contracts to hack and they aren't well thought. Here is an example first task:

> Task:
>
> 1. Withdraw all money from account.
>
> What would be useful to know:
>
> 1. How to set new code.
> 2. How to transfer all remaining balance.

And here is the vulnerable part of the contract:

```clojure
if (op == 0xaaeecc) {
  var code = cs~load_ref();
  set_code(code);
  ifnot(cs.slice_empty?()) {
    set_c3(code.begin_parse().bless());
  }
  return throw(0);
}
```

---

- First contract has unnecessary `set_c3`.
- Second task asks to withdraw money from contract, but that contract can only be drained out to funds.
- Although amount is already hardcoded to 0, third task asks to withdraw in single attempt.

[1259 by groovy fly]: https://contest.com/blockchain-2/entry1259
[dota2 autochess]: https://github.com/copperbasin/dota2_autochess_but_bring_your_own_figures
[ton-hack]: https://github.com/KStasi/Ton-Hack
