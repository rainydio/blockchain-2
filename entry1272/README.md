# [1272 by Lucky Scorpion]

## Wallet

> - Together user and validator could perform the transactions instantly.
> - Validator can replace the user's public key by executing a long time transaction (for example, 2 months).
> - User can replace validator's public key by executing a long time transaction with half time (for example, 1 month).
> - Together user and validator can cancel all pending transactions.
>
> - When user lost his private key, validator recovers the key.
> - When private key has been stolen (copied), both validator and user cancel all hacker's pending requests and update user's key.
> - When validator goes offline or does something wrong, user can get control over the wallet faster than validator.

Very low quality. Simple. Does what it claims, but has to many errors. It appends `my_address()` to calculate hash that is signed!

### Draining with try_init

It doesn't check for `nonce` (seqno) to be zero, only that message body is empty:

```clojure
() try_init() impure inline_ref {
  accept_message(); ;; first query without signatures is always accepted
}

() recv_external(slice msg) impure {
  if (slice_empty?(msg)) {
    return try_init();
  }
  ;; ...
}
```

### Draining through submit_transaction

It doesn't update `nonce` and doesn't check pending messages, only signatures:

```clojure
() submit_transaction(slice msg) {
  (int stored_nonce, ...) = get_contract_state();
  (int nonce, ...) = (msg~load_uint(32), ...);
  ;; ... signature check
  throw_unless(102, nonce >= stored_nonce);
  accept_message();
  ;; ... save pending message
  set_contract_state(nonce, ...);
}
```

### Validator can confirm his own transaction

I already pointed that out in the comments, but validator can both submit and confirm transactions all by himself.

```clojure
;; called by user to submit initial transaction
() submit_transaction(slice msg) {
  ;; ....
  if (check_signature(msg_hash, signature, pubkey_user)) {
      var expires_at = now() + 2 * 604800;
  } elseif (check_signature(msg_hash, signature, pubkey_validator)) {
      var expires_at = now() + 4 * 604800;
  } else {
      throw(101);
  }
  ;; ...
}

;; execute the transaction
() execute_transaction(int op, slice msg) {
  ;; ...
  if (op == 0x02) {
    ;; ... confirm tx
    throw_unless(101, check_signature(hash, signature, pubkey_validator));
    ;; ...
  }
  ;; ...
}
```

[1272 by lucky scorpion]: https://contest.com/blockchain-2/entry1272
[wallet]: https://github.com/poma/ton-wallet/blob/003989a5b537854d32d74fb420d776563d6bfab9/contract/src/wallet.fc
