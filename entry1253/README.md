# [1253 by Dreamy Chicken]

## [Lottery 3 of 13]

> Rules:
>
> - Participants choose 3 arbitrary unique numbers 0-99 and "buys" a ticket.
> - Prize fund of current round is a sum of all "sold" tickets in this round, plus "unused" fund from previos round, minus round commision.
> - Sponsors can increase current round prize fund by any amount of GR\$.
> - Owner of lottery smc conducts a lottery round (say once per day):
>   - Generate 13 unique random numbers 0-99.
>   - Find winners and their prize among participants:
>     - Guess all 3 numbers - 60% of prize fund / winners.
>     - Guess 2 numbers - 22% of prize fund / winners.
>     - Guess 1 number - 15% of prize fund / winners.
>   - Charge smc commision - 3% of prize fund.
>   - Calculates round winners, creates new round.
> - Results can be fetched with lucky_nums, winners and is_winner methods.
> - Winners can withdraw their prizes until next round.
> - Owner of lottery smc can withdraw only commision, not the entire smc balance.
>
> Limitations:
>
> - Participant's accounts must reside in basechain (wc = 0).
> - Participant can buy only 1 ticker per round.
> - Winners must withdraw their prizes until next round.

Ok quality. Simple. Clean! Good readme! Clear about limitations!

### Sloppy probabilities

There are several issues:

> - Participants choose 3 arbitrary unique numbers 0-99 and "buys" a ticket.
> - Prize fund of current round is a sum of all "sold" tickets in this round, _plus "unused" fund from previos round_, minus round commision.
> - Guess all 3 numbers - 60% of prize fund / winners.
> - Guess 2 numbers - 22% of prize fund / winners.
> - Guess 1 number - 15% of prize fund / winners.

Prize fund isn't disributed fully after each round. If no-one guessed the number, then no-one gets the money. Because it's carried over into next round, it may even become profitable to play. _Rational gambler_ would wait until prize fund includes undistributed funds from previous round and then he will play. _Rational gambler_ will never buy ticket first.

Another problem is how prize funds are distributed. If there are two winners who guessed two numbers, and only one who guessed one number. Those who guessed two numbers will receive less! They will get 11% each, while the one who guessed only one number will get 15%.

### Deposit processing fees

Contract isn't careful in how it handles processing fees. Simply `msg_value` is added to `prize_fund`. So the value of prize fund plus commision always gonna be higher than actual balance. The error will slowly accumulate because prize fund (and commission) are carried over into next round. I'm not sure if commision of GR\$0.03 per message is enough to cover processing costs. If it is, then it is possible to recover by sending non-bounceable message to _fix_ balance.

[1253 by dreamy chicken]: https://contest.com/blockchain-2/entry1253
[lottery 3 of 13]: https://github.com/alunegov/ton-contest-2/blob/1b3cd40786452ea1b9ebee61f62dfe6c3032f21b/lottery/code.fc
[dreamy chicken]: https://contest.com/blockchain-2/entry1253
