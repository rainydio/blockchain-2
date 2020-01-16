# [1247 by Small Kitten]

## [Ticket System]

> - Point of sale for single ticket/product as 1 contract per ticket type.
> - Each contract holds its event info.
> - Price cannot be changed during the contract lifetime.
> - End of sale can not be changed.
> - End of refunds and refunds percentage are configurable.
> - Verifier gives one time code to the owner to sign. Signature is checked through get method.

Lowish quality. Complex.

### Buying tickets

Simple transfers are just eaten by contract. Buying tickets is separate operation that requires processing fees and just sends that fee back:

```clojure
send_message(s_addr, exitCode, query_id, op, remainer + 1000000000, 0);
```

### Verification

Verification boils down to a simple `check_signature` call:

```clojure
int proof_ownership(int owner, cell data, cell signatureC) method_id {
  var signature = signatureC.begin_parse();
  return check_signature(cell_hash(data), signature, owner);
}
```

I would rather expect tickets to have unique numbers to help verifier with replay protection.

### Refunds

Refunds are processed through internal message handler, but authenticated by public key. Ticket owner has to carefully check _one time code_ that he signs. It might be refund request crafted by verifier.

### 16 bit seqno

Contract `seqno` is only 16 bit long, and there is attempt to loop it. But it isn't stored in final data.

```clojure
if (query_seq == 65535) { ;; reset query number
  query_seq = 0;
}
saveData(owner, seq + 1, price, endOfSale, endOfRefunds, refundPercentage, store, eventInfo);
return ();
```

## [Software Validation]

> - Registry to provide hash validation of obtained binaries.
> - External messages allows contract owner to manipulate hash registry.
> - It is possible to set or remove hash record.
> - Records might have expiration.

Low quality. Simple.

### Expiration

Expiration affects `validate` call result. Expired records are not removed. But why there is expiration at all in such registry? How file integrity hash can expire?

### Purposeless

Records can be changed and removed. Public key cannot be changed, so it has to be protected forever. Simply signing the hash itself provides same level of security.

### Unecessary query_id

Instead of `valid_until` expiration is done through `query_id` and that is the only instance where `query_id` is used.

```clojure
var query_id = in_msg~load_uint(64);
var bound = (now() << 32);
throw_unless(35, query_id > bound);
```

### 16 bit seqno

Contract is using 16 bit `seqno` yet again and this time _correctly_ looping it.

```clojure
if (query_seq == 65535) { ;; reset query number
  query_seq = 0;
}
saveData(owner, query_seq + 1, storesInfo, dict);
return ();
```

[1247 by small kitten]: https://contest.com/blockchain-2/entry1247
[ticket system]: https://github.com/5eraph/ts/blob/01d1ce013523701ea1a158778ac5158ba8cfb45d/ts/ts.fc
[software validation]: https://github.com/5eraph/sw/blob/1b1e72d585b478c045add1849d39d5b3c6a49f93/sw/sw.fc
