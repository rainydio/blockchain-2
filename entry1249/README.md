# [1249 by Desert Falcon]

## [Contract manager]

> - Contract is simply interface to dictionary.
> - App claims to be sort-of keyring.

Very low quality. Simple.

_... do I have to review it, really?_

## [Messenger]

> - Each user creates his own contract.
> - Internal message handler is inbox for incoming messages.
> - Each message is encrypted with owner public key and includes sender's public key.
> - To reply back message is encrypted with provided key and sent back.

Low quality. Simple.

### Main in the middle

Actually front-running and key distribution problem. Contract keeps contact list in dictionary indexed by public key. When message arrives for the first time sender's address is written into dictionary and public key is taken from message itself. Attacker can discover messenger instance contracts, public keys used, see unconfirmed transactions and attempt to write his own address into that dictionary.

When that dictionary is used by `MessengerContract.cs` to locate public key address it will send message unknowingly to attacker address instead. Although message itself cannot be read, grams that were sent can be collected. Even better strategy would be to precisely calculate fees required and relay message to intended recipient. Both parties are unlikely to notice anything when messages are delivered.

```clojure
() addMessage(slice sender_addr, int msgTime, slice in_msg) impure {
  (slice cs, int found) = memberDict.udict_get?(256, pk);
  ifnot (found) {
    cell data = begin_cell().store_uint(src_wc, 8).store_uint(src_addr, 256).end_cell();
    memberDict~udict_set_builder(256, pk, begin_cell().store_ref(data));
  }
}
```

[1249 by desert falcon]: https://contest.com/blockchain-2/entry1249
[contract manager]: https://github.com/AlexGor-dev/contract_manager/blob/99f885a40e29ac2b5b6792dcd225fdaca6567963/bin/Manager/manager_code.fc
[messenger]: https://github.com/AlexGor-dev/messenger/blob/4e24e1b91b775725267f1b2fe59dbb3f225757ad/bin/Messenger/messenger_code.fc
