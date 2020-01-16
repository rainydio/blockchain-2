# [1259 by Groovy Fly]

## [Dota2 autochess]

> - Owner configures figures shop.
> - Players buy figures.
> - Players choose previously bought figures, provides expiration, and enter matchmaking queue.
> - Match starts once 8 players are in queue.
> - Chosen figures are combined to create game board.

Lowish quality. Complex. Original claims describe game, not contract.

> After player death in match his/her figures are not going back to pool.

Because contract isn't checking game rules.

### Players loose figures entering queue

Previously bought figures are removed from player's inventory when he enters matchmaking queue. Because player also sets expiration, he might be removed from queue without entering match. Only loosing figures because there was no-one else to play with:

```clojure
() line_up_to_match(slice owner_address, int valid_until, ...) impure {
  throw_if(55, valid_until < now() + 30); ;; interval limit
  throw_if(56, valid_until > now() + 7200); ;; interval limit
  var (..., units) = get_user(owner_address, users);
  while (level_units.slice_bits()) {
    units = update_units_counter(units, unit_id, level, counter, false); ;; decrease
  }
  users = create_user(owner_address, users, ..., units);
}
```

### In-game flag

Players's `in_game` is set to true when he enters matchmaking queue. It prevents player from entering queue again. But it isn't set back to false anywhere.

### Expiration race

Players and figures they bought may be removed due to inactivity. This is questionable decision on its own. Player's expiration isn't updated when he enters matchmaking queue. So it's possible that player will be removed while he is in queue. Suprisingly this is handled by adding player back into dictionary with expiration set to zero:

```clojure
(cell, cell) remove_from_queue(cell queue, cell users) impure {
  var (s_user, f) = users.udict_get?(256, player_hash);
  var (paid_until, in_game, bans_counter, units) = (0, 0, 0, new_dict());
  if (f) {
    (paid_until, in_game, bans_counter, units) = (
      s_user~load_uint(32),
      s_user~load_int(1),
      s_user~load_uint(32),
      s_user.preload_dict()
    );
  }
  slice user = begin_cell()
    .store_uint(paid_until, 32)
    .store_int(in_game, 1)
    .store_uint(bans_counter, 32)
    .store_dict(units)
    .end_cell()
    .begin_parse();
  users~udict_set(256, player_hash, user);
}
```

He will be removed right after at the end of `run_ticktock`.

### Bans

Unused `ban_requests` and `bans_counter` variables.

## [Ton-Hack]

> This is set of contest-like and/or education-like smart contracts for junior-middle ton smart-contract developers.

Lowish quality. Simple.

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

There are no usual scripts that may help user create and test his ideas. Contracts and tasks aren't well thought:

- First contract has unnecessary `set_c3`.
- Second task asks to withdraw money from contract, but that contract can only be drained out to funds.
- Although amount is already hardcoded to 0, third task asks to withdraw in single attempt, and contract has attempts counter.

[1259 by groovy fly]: https://contest.com/blockchain-2/entry1259
[dota2 autochess]: https://github.com/copperbasin/dota2_autochess_but_bring_your_own_figures/blob/f3da3540f9b30c41c4d9879d98ef9456db599056/contracts/game-code.fc
[ton-hack]: https://github.com/KStasi/Ton-Hack/tree/d0b19f7b1b94688f6569dc4831e6aa27ea2e1014/contracts
