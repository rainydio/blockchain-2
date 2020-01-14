## Returning change from processing fee

Several submissions account processing fees wrong by simply sending whole fee back. Usually it looks like this:

```clojure
int value = msg_value - fee;
if (value < 0) {
  return send_message_back(s_addr, 0xee6f454c, 0, 0, 0, 64);
}
;; ... buy something for `value`
return send_message_back(s_addr, 0xf374484c, 0, 0, fee, 2);
```

[Mellow Squid] and [Shiny Giraffe] use simple technique to return change from processing fee:

```clojure
raw_reserve(get_balance().first() - fee, 0);
send_message_back(s_addr, 0xf374484c, 0, 0, 0, 128);
```

### Send raw message +2 flag

It appears that wrong code to return change was taken from `elector-code.fc`. I haven't checked standard contracts yet...

> _- Знаешь, что говорит уточка?_  
> _- ???_  
> _- Зря-зря_

But generally any `send_raw_message` +2 usage indicates that there is something wrong with how funds are handled. Otherwise why there is error at all? Carefully crafted message may leave contract with very small yet still positive balance, instead of being skipped by +2.

Because `elector-code.fc` also contains `accept_message` this can be used to destroy contract completely. I'm not sure if it is possible to destroy contract if it has very small balance, but does not contain `accept_message`.

## Reviews

Most submission pages contain aggregate claims that were combined by me. So please verify them if uncertain. Progress notation in the list below:

- no review
- shallow review
- incomplete review
- almost complete review
- complete review

### Submissions

- [1246 Multisig by Suave Beetle] (shallow review).  
  Nothing to see. Previous submission.

- [1247/1 Ticket System by Small Kitten] (almost complete review).  
  Very low quality. Complex. 16 bit seqno. Incorrectly handles the fees.

- [1247/2 Software Validation by Small Kitten] (incomplete review).  
  Very low quality. Simple. 16 bit seqno. Unnecessary `query_id` instead of `valid_until`.

- [1254 ICO by Shiny Shrimp] (shallow review).  
  Ok quality. Complex. Currency has to be registered in masterchain.

- [1255 Self Regulated Community by Cuddly Hamster] (incomplete review).  
  Ok quality. Complex. Something ported to funC. Doesn't do what it claims. Contract owner is the only user.

- [1259 Dota2 autochess by Groovy Fly] (shallow review).  
  Low quality. Complex. Some claims aren't about contract. It's only figure shop.

- [1259 Ton-Hack by Groovy Fly] (almost complete review).  
  Low quality. Simple. Only 4 example contracts to hack and they aren't well thought.

- [1260 TON Delegation Pool Platform by Night Turkey] (no review).  
  Very high quality. Complex. Something real.

- [1261 Stake pool by Slim Dodo] (complete review).  
  Ok quality. Complex. Almost does what it claims. Pool controlled by administrator. Potential funds lock-out. Can be drained.

- [1262 Cheques by Grim Duck] (incomplete review).  
  Very low quality. Simple. Original had major errors. Attempts to set gas limit in grams.

- [1264 Lottery by Dreamy Crab] (incomplete review).  
  Low quality. Simple. Can be drained.

- [1265 The Chat Game by Mellow Squid] (incomplete review).  
  Ok quality. Complex. Carefully handles balance and fees.

- [1265 OracleHub by Mellow Squid] (incomplete review).  
  Ok quality. Complex. Can be drained.

- [1266/1 ERC20 by Nice Llama] (incomplete review).  
  Low quality. Simple. Balances within single workchain.

- [1266/2 Ring-signature based mixer by Nice Llama] (almost complete review).  
  Very low quality. Simple. Doesn't do what it claims.

- [1267 MLM by Funny Owl] (shallow review).  
  Lowest quality. I don't even understand how it supposed to be used.

- [1268 TON Gate by Big Elk] (no review).  
  Primitive C++ client-server. Server is discovered through TON DHT.

- [1269/1 Conditional transfer by Shiny Giraffe] (shallow review).  
  Ok quality. Complex. Mixes controller address and public key.

- [1269/2 Data storage proxy by Shiny Giraffe] (shallow review).  
  Ok quality. Complex. Almost does what it claims. I don't understand the purpose.

- [1270 Charity Foundation by Dreamy Snail] (incomplete review).  
  Low quality. Simple. Doesn't account processing fees.

- [1271 Apportion group expense by Dreamy Pug] (shallow review).  
  Low quality. Complex. Has vulnerabilities.

- [1272 Wallet by Lucky Scorpion] (almost complete review).  
  Very low quality. Simple. Wallet with two keys. Has several vulnerabilities.

[mellow squid]: entry1265
[shiny giraffe]: entry1269
[1246 multisig by suave beetle]: entry1246#multisig
[1247/1 ticket system by small kitten]: entry1247#ticket-system
[1247/2 software validation by small kitten]: entry1247#software-validation
[1254 ico by shiny shrimp]: entry1254#ico
[1255 self regulated community by cuddly hamster]: entry1255#self-regulated-community
[1259 dota2 autochess by groovy fly]: entry1259#dota2-autochess
[1259 ton-hack by groovy fly]: entry1259#ton-hack
[1260 ton delegation pool platform by night turkey]: entry1260#ton-delegation-pool-platform
[1261 stake pool by slim dodo]: entry1261#stake-pool
[1262 cheques by grim duck]: entry1262#cheques
[1264 lottery by dreamy crab]: entry1264#lottery
[1265 the chat game by mellow squid]: entry1265#the-chat-game
[1265 oraclehub by mellow squid]: entry1265#oraclehub
[1266/1 erc20 by nice llama]: entry1266#erc20
[1266/2 ring-signature based mixer by nice llama]: entry1266#ring-signature-based-mixer
[1267 mlm by funny owl]: entry1267#mlm
[1268 ton gate by big elk]: entry1268#ton-gate
[1269/1 conditional transfer by shiny giraffe]: entry1269#conditional-transfer
[1269/2 data storage proxy by shiny giraffe]: entry1269#data-storage-proxy
[1270 charity foundation by dreamy snail]: entry1270#charity-foundation
[1271 apportion group expense by dreamy pug]: entry1271#apportion-group-expense
[1272 wallet by lucky scorpion]: entry1272#wallet
