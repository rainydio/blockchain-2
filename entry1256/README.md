# [1256 by Mad Crow]

## [Staking pool]

### Initialization front-running

There are no signature checks during initialization and `nominators` are loaded from initial message. So it's possible to front-run initialization.

```clojure
() try_init(slice in_msg) impure inline_ref {
  var cs = in_msg;
  (int current_period, cell pool_config) = (ds~load_uint(16), ds~load_ref());
  throw_unless(30, current_period == 0 );
  throw_if(37, current_period);
  accept_message();
  var nominators = cs~load_ref();
  pool_config = pack_pool_config(owner, owner_fee, min_amount, nominators);
  set_data(pack_state(current_period, pool_config, performance, subscriptions, subscribers));
}
```

_Also unnecessary second throw..._

### Draining

Contract doesn't account for deposit processing fees simply adding `msg_value`, and allows to withdraw immediately. This can be used to drain it out of funds:

```clojure
() process_new_subscription(slice s_addr, int query_id, int msg_value) impure {
  subscriptions~udict_set_builder(32, subscription_idx, pack_subscription(
    src_addr, msg_value, current_period, 0, 1
  ));
}

() process_redeem(slice s_addr, int query_id, int subscription_id) impure {
  var (subscription, found) = subscriptions.udict_get?(32, subscription_id);
  var (
    int subscriber,
    int grams,
    int start_period,
    int end_period,
    int status
  ) = unpack_subscription( subscription );

  if (start_period == current_period) { ;; redeem in current period
    send_message(s_addr, op, query_id, 0, grams, 64);
  }
}

() recv_internal(int msg_value, cell in_msg_cell, slice in_msg) impure {
  var s_addr = cs~load_msg_addr();
  if (in_msg.slice_empty?()) {
    return process_new_subscription(s_addr, 0, msg_value);
  } else {
    int subscription_id = decode_text(in_msg);
    return process_redeem(s_addr, 0, subscription_id);
  }
}
```

### Small things

_`int last_idx = ((1 << 32) - 1);`_

```clojure
int last_idx = (1 << 31) + ((1 << 31) - 1);
```

---

_`subscriber.udict_get?(32, current_period);`_

```clojure
() process_new_subscription(slice s_addr, int query_id, int msg_value) impure {
  (int current_period, cell subscriptions, cell subscribers, ...) = unpack_state();
  int last_idx = (1 << 31) + ((1 << 31) - 1);
  var (
    subscription_period,
    last_subscription,
    user_subscription_found
  ) = subscriber.udict_get_prev?(32, last_idx);

  ;; subscription is the current one, add grams
  if( user_subscription_found & ( subscription_period == current_period )  ){
    ;; ...
  }
}
```

---

Suprisingly `current_period` is 16u, so `subscriber` dictionary should also be indexed by 16u instead of 32u, and it is sometimes...

```clojure
() process_new_subscription(slice s_addr, int query_id, int msg_value) impure {
  var (
    subscription_period,
    last_subscription,
    user_subscription_found
  ) = subscriber.udict_get_prev?(32, last_idx);

  ;; create new subscription for subscriber
  subscriber~udict_set_builder(16, current_period, pack_subscribers_entry(last_subscription_idx));
}
```

Within same function `subscriber` may also be int (last address part):

```clojure
() process_new_subscription(slice s_addr, int query_id, int msg_value) impure {
  (
    int subscriber,
    int grams,
    int start_period,
    int end_period,
    int status
  ) = unpack_subscription(last_subscription);
}
```

---

_`subscription_id` from text_

```clojure
if( in_msg.slice_empty?() ) {
  return process_new_subscription(s_addr, 0, msg_value);
} else {
  int subscription_id = decode_text(in_msg);
  return process_redeem(s_addr, 0, subscription_id);
}
```

---

> _вход — рубль, выход — два_

Subtle hardcoded workchain check, easy deposit, difficult withdraw:

```clojure
() process_redeem(slice s_addr, int query_id, int subscription_id) impure {
  throw_unless(43, (src_wc == 0) & (src_addr == subscriber) ); ;; subscription doesn't belong to sender
}
```

[1256 by mad crow]: https://contest.com/blockchain-2/entry1256
[staking pool]: https://github.com/koinov/ton-staking/tree/535b658bdbd7f86a70710ca31e43cb4acac2554f/staking/smartcont
