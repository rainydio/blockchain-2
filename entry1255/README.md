# [1255 by Cuddly Hamster]

## [Self Regulated Community]

> To create a community self-regulation mechanics that stimulates the creation of useful content and allows Telegram community members to deal with flood and spam without attracting of dedicated administrators and moderators.

> We provide the opportunity for all users to participate in self-regulation (moderation) of communities using a mechanism for a weighted rating of messages from other users.

Looks like rating system ported to func.

### Contract owner is the only user

All of the operations are performed by owner through external message handler. Recalculation is performed daily by calling special operation. It doesn't even use `now()` to calculate time difference. Recalculation has to be performed exactly once a day.

### Accepting future seqno

It accepts future `seqno` and doesn't have `valid_until` checks, but it throws on many invalid operations. So owner has to be absolutely sure that previous seqno was included in block. He can't simply send _add_user_ followed by _like_, he has to wait for _add_user_ to be included in block.

[1255 by cuddly hamster]: https://contest.com/blockchain-2/entry1255
[self regulated community]: https://github.com/qpIlIpp/blocklab-ton_contest_2
