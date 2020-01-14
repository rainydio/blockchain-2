# [1262 by Grim Duck]

## [Cheques]

> - Owner can create cheque for specific amount that can be withdrawn by associated key.

Very low quality. Simple. Does what it claims. Original submission had major errors.

### Poorly written

Attempts to set gas limit in grams:

```clojure
set_gas_limit(msg_value - grams);
```

[1262 by grim duck]: https://contest.com/blockchain-2/entry1262
[cheques]: https://github.com/catchain/cheques/blob/8a8de1dcad3e362697e3bdb92f963592ce333aad/cheque/cheque-code.fc
