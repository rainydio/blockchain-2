# [1262 by Grim Duck]

## [Cheques]

> - Owner can create cheque for specific amount that can be withdrawn by associated key.

Very low quality. Simple. Original had major errors. Front-running.

### Front-running

This has to be commit-reveal scheme. It isn't:

```clojure
() recv_external(slice in_msg) impure {
  var signature = in_msg~load_bits(512);
  var cs = in_msg;
  var cheque_activation_info = cs~load_ref().begin_parse();
  var wc = cheque_activation_info~load_int(8);
  var addr = cheque_activation_info~load_uint(256);
  var cheque_id = cheque_activation_info~load_uint(256);

  var (cheque_msg, found) = cheques.udict_get?(256, cheque_id);
  throw_unless(36, found);
  throw_unless(37, check_signature(slice_hash(in_msg), signature, cheque_id));

  var (grams) = (cheque_msg~load_grams());
  cheques~udict_delete?(256, cheque_id);
  accept_message();
  send_raw_message(msg, 3);
}
```

### Poorly written

Attempts to set gas limit in grams:

```clojure
set_gas_limit(msg_value - grams);
```

[1262 by grim duck]: https://contest.com/blockchain-2/entry1262
[cheques]: https://github.com/catchain/cheques/blob/8a8de1dcad3e362697e3bdb92f963592ce333aad/cheque/cheque-code.fc
