# [1250 by Funny Deer]

## [American Roulette]

> - American Roulette.
> - Immediate.
> - Stateless. Relies on probabilities.

Very high quality. Simple. Shining!

Expected 5% profit may not be enough to cover processing fees if `minBet` is to low. Simple transfer should probably be returned back, because it's likely mistake:

```clojure
int op = in_msg~load_uint(32);
if(op == 0) {
  ;; simple transfer with comment - just fill balance for prizes
  return ();
}
```

_And it uses camel-case instead of underscores._

### Simple. Shining!

Many other gambling submissions are afraid of relying on probabilities. This one side-steps all of the difficulties of selling tickets and conducting withdrawals. It completely eliminates any state transitions. So there is no _contract_ really.

But in any case it was made by someone who knows how to write code. Let me show you:

```clojure
;; for n > 0
int isOdd(int n) inline {
  return (n % 2) == 1;
}

;; for n > 0
int isEven(int n) inline {
  return (n % 2) == 0;
}

;; for n > 0
int isRed(int n) {
  if( ((n >= 1) & (n <= 10)) | ((n >= 19) & (n <= 28)) ) {
    return isOdd(n);
  } else { ;; from 11 to 18 and 29 to 36
    return isEven(n);
  }
}

;; for n > 0
int isBlack(int n) inline {
  return ~ isRed(n);
}

() recv_internal(int msg_value, cell in_msg_cell, slice in_msg) impure {
  if(isNotZero(wonNumber) & isOdd(wonNumber)) {
    return processWin(query_id, s_addr, msg_value);
  }
  if(isNotZero(wonNumber) & isEven(wonNumber)) {
    return processWin(query_id, s_addr, msg_value);
  }
  if(isNotZero(wonNumber) & isRed(wonNumber)) {
    return processWin(query_id, s_addr, msg_value);
  }
  if(isNotZero(wonNumber) & isBlack(wonNumber)) {
    return processWin(query_id, s_addr, msg_value);
  }
}
```

It wasn't immediately obvious why `isNotZero` wasn't included into `isOdd` and `isEven`. The problem is that `isBlack` will incorrectly return true when `n <= 0`. Most people would have added additional `isNotZero` to `isBlack`, but he didn't. It creates unnecessary second call to `isNotZero` and so he moved that out.

---

Commented out lines are optional features:

```clojure
() processWin(int query_id, slice s_addr, int prize) impure {
  send_raw_message(
    begin_cell()
      .store_uint(0x18, 6)
      .store_slice(s_addr)
      .store_grams(prize)
      .store_uint(0, 1 + 4 + 4 + 64 + 32 + 1 + 1)
      .store_uint(0xff000001, 32)
      .store_uint(query_id, 64)
    .end_cell(),

    ;; msg mode
    ;; +64 - return received amount + add prize
    ;; +2 - ignore errors (?)
    64 + 2
  );

  ;; updateStats(true, prize);
}

;; () processLoss(int value) impure {
;;   updateStats(false, value);
;; }

() recv_internal(int msg_value, cell in_msg_cell, slice in_msg) impure {
  throw_if(104, betAtNumber > 36);
  throw_if(104, betAtNumber < -1); ;; American
  ;; throw_if(104, betAtNumber < 0); ;; European

  if(wonNumber == betAtNumber) {
    return processWin(query_id, s_addr, msg_value * 35);
  } else {
    ;; return processLoss(msg_value);
    return ();
  }
}
```

_It shouldn't happen, but don't ignore errors. At least bounce remaining value back, [Funny Deer]._

[1250 by funny deer]: https://contest.com/blockchain-2/entry1250
[american roulette]: https://github.com/gr8den/ton-roulette/blob/25112dd31fa098c1e44ddbb531895213fa107a90/contract.fc
[funny deer]: https://contest.com/blockchain-2/entry1250
