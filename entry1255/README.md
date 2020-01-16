# [1255 by Cuddly Hamster]

## [Self Regulated Community]

> - We provide the opportunity for all users to participate in self-regulation (moderation) of communities.
> - Users' votes are not equal, but weighted according to their relative influence in each community.
> - User’s voting power is calculated periodically (once a day).
> - Donations that are distributed 70% to author, 20% to top 15 users, 10% commissions.

Highish quality. Complex. Doesn't do what it claims. Looks like some rating system ported to funC.

### Contract owner is the only user

All of the operations are performed by owner through external message handler. There are no other users.

### Recalculations

Recalculation are performed by calling special operation. Contract doesn't even use `now()` to calculate time difference. Recalculation has to be performed exactly once a day by owner.

### Accepting future seqno

It accepts future `seqno` and doesn't have `valid_until` checks, and it throws on many invalid operations. So owner has to be sure that previous seqno was included in block. He can't simply send _add_user_ followed by _like_, he has to wait for _add_user_ to be included in block.

### Replaying likes/dislikes

The main difficulty with rating systems is that they are easy to game. This particular implementation makes no attempt to diminish power of user who is rating someone. Note that only `usr_to` is modified, `usr_from` can repeat his like/dislike as many times as he want.

```clojure
(cell, cell, cell) rate_func(
  cell users_top,
  cell current_users,
  cell banned_users,
  int id_from,
  int id_to,
  int increase
) impure {
  var usr_from = get_user(current_users, id_from, err_user_was_not_found());
  var usr_to = get_user(current_users, id_to, err_user_was_not_found());
  ;; ...
  current_users = set_user(current_users, id_to,
    save_user(prev_TPi, to_days, to_bans_count,balance)
  );
  return (users_top, current_users, banned_users);
}
```

[1255 by cuddly hamster]: https://contest.com/blockchain-2/entry1255
[self regulated community]: https://github.com/qpIlIpp/blocklab-ton_contest_2/tree/e97255f25f4081887323f9add92d45064fa82d6c/blockchain/smartcont
