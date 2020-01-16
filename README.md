## Returning change from processing fee

Many contract that accept deposits do not account processing fees. Usually whole `msg_value` is added to deposit. [Mellow Squid] uses simple technique to return change from processing fee:

```clojure
raw_reserve(get_balance().first() - processing_grams_max, 0);
send_message_back(s_addr, 0xf374484c, 0, 0, 0, 128);
```

[Shiny Giraffe] also uses exact same technique:

```clojure
raw_reserve(get_balance().first() - msg_value + deposit, 0);
send_message_back(s_addr, 0xf374484c, 0, 0, 0, 128);
```

### Suppressing `send_raw_message` errors

Generally any `send_raw_message` +2 usage indicates that contract balance is lower than expected. Otherwise why there is error at all? At some point during processing withdrawal ignoring the error will lock someone's funds. As result real balance will become higher than expected, and contract will continue operating normally repeating the cycle.

It will look like random glitch and those who are unlucky will complain on the internet.

### Contract paying for processing

Many contracts contain `accept_message`, `set_gas_limit`, or `buy_gas` calls within internal message handler. It's often combined with `send_raw_message` +2 because gas consumed creates severe discrepancy between real and expected balance. Due to fatal misfortune or deliberate act of sabotage, processing withdrawal may leave contract with small yet still positive balance. It will run out of funds during next `accept_message`.

I haven't reviewed standard contracts yet, but [Slim Dodo] copied erroneous code from `elector-code.fc`

## Reviews

Most submission pages contain aggregate claims that were combined by me. So please verify them if uncertain. Progress notation in the list below:

- no review
- shallow review
- incomplete review
- almost complete review
- complete review

Not in the list, will add them later except:

- [Classy Bee]. It's good, we all know that.
- [Eager Boar]. It's meh. Reviewed it in comments.
- [Hip Hyena]. So many errors. Reviewed it in comments.
- [Bold Wolf]. It's terrible. Reviewd it in comments.

In the list, but need some additional work:

- [Big Elk]. Appears to be simplest client-server C++ app.
- [Mad Crow]. So many errors. Difficult to untangle.

### Submissions

- [1246 Multisig by Suave Beetle] (shallow review).  
  Nothing to see. Previous submission.

- [1247/1 Ticket System by Small Kitten] (almost complete review).  
  Lowish quality. Complex. 16 bit seqno. Incorrectly handles deposits.

- [1247/2 Software Validation by Small Kitten] (incomplete review).  
  Low quality. Simple. 16 bit seqno.

- [1249/1 Contract manager by Desert Falcon] (shallow review).  
  Very low quality. Simple.

- [1249/2 Messenger by Desert Falcon] (shallow review).  
  Low quality. Simple.

- [1250 American Roulette by Funny Deer] (complete review).  
  Very high quality. Simple. Stateless. Shining!

- [1253 Lottery 3 of 13 by Dreamy Chicken] (incomplete review).  
  Ok quality. Simple. Sloppy probabilities. Incorrectly handles deposits.

- [1254 ICO by Shiny Shrimp] (shallow review).  
  Lowish quality. Complex. Currency has to be registered in masterchain.

- [1255 Self Regulated Community by Cuddly Hamster] (incomplete review).  
  Highish quality. Complex. Something ported to funC. Doesn't do what it claims. Contract owner is the only user.

- [1256 Staking pool by Mad Crow] (shallow review).  
  Low quality. Complex. Vulnerable.

- [1259 Dota2 autochess by Groovy Fly] (incomplete review).  
  Lowish quality. Complex. Shop and simple matchmaking.

- [1259 Ton-Hack by Groovy Fly] (almost complete review).  
  Low quality. Simple. Only 4 example contracts to hack and they aren't well thought.

- [1260 TON Delegation Pool Platform by Night Turkey] (almost complete review).  
  Very high quality. Complex. Real. Vulnerable.

- [1261 Stake pool by Slim Dodo] (complete review).  
  Highish quality. Complex. Almost does what it claims. Incorrectly handles deposits. Vulnerable.

- [1262 Cheques by Grim Duck] (incomplete review).  
  Very low quality. Simple. Original had major errors. Front-running.

- [1263/1 Decentralized Exchange by Merry Ant] (incomplete review).  
  Very low quality. Complex. Extremely vulnerable.

- [1263/2 TRC20 by Merry Ant] (complete review).  
  Lowish quality. Simple. Vulnerable. Clean!

- [1264 Lottery by Dreamy Crab] (incomplete review).  
  Low quality. Simple. Can be drained.

- [1265 The Chat Game by Mellow Squid] (incomplete review).  
  High quality. Complex. Correctly handles deposits!

- [1265 OracleHub by Mellow Squid] (incomplete review).  
  High quality. Complex. Can be drained.

- [1266/1 ERC20 by Nice Llama] (incomplete review).  
  Low quality. Simple. Balances within single workchain.

- [1266/2 Ring-signature based mixer by Nice Llama] (almost complete review).  
  Very low quality. Simple. Doesn't do what it claims.

- [1267 MLM by Funny Owl] (shallow review).  
  Extremely low quality. I don't understand how it supposed to be used.

- [1268 TON Gate by Big Elk] (no review).  
  Primitive C++ client-server. Server is discovered through TON DHT.

- [1269/1 Conditional transfer by Shiny Giraffe] (complete review).  
  Lowish quality. Spaghetti-level complex. Inefficient. Correctly handles deposits! No exploits found!

- [1269/2 Data storage proxy by Shiny Giraffe] (shallow review).  
  Ok quality. Complex. Almost does what it claims. I don't understand the purpose.

- [1270 Charity Foundation by Dreamy Snail] (incomplete review).  
  Low quality. Simple. Doesn't account processing fees.

- [1271 Apportion group expense by Dreamy Pug] (incomplete review).  
  Low quality. Complex. Vulnerable.

- [1272 Wallet by Lucky Scorpion] (almost complete review).  
  Very low quality. Simple. Wallet with two keys. Vulnerable.

[classy bee]: https://contest.com/blockchain-2/entry1258
[eager boar]: https://contest.com/blockchain-2/entry1248
[hip hyena]: https://contest.com/blockchain-2/entry1251
[bold wolf]: https://contest.com/blockchain-2/entry1257
[mellow squid]: entry1265
[shiny giraffe]: entry1269
[slim dodo]: entry1261
[big elk]: entry1268
[mad crow]: entry1256
[1246 multisig by suave beetle]: entry1246#multisig
[1247/1 ticket system by small kitten]: entry1247#ticket-system
[1247/2 software validation by small kitten]: entry1247#software-validation
[1249/1 contract manager by desert falcon]: entry1249#contract-manager
[1249/2 messenger by desert falcon]: entry1249#messenger
[1250 american roulette by funny deer]: entry1250#american-roulette
[1253 lottery 3 of 13 by dreamy chicken]: entry1253#lottery-3-of-13
[1254 ico by shiny shrimp]: entry1254#ico
[1255 self regulated community by cuddly hamster]: entry1255#self-regulated-community
[1256 staking pool by mad crow]: entry1256#staking-pool
[1259 dota2 autochess by groovy fly]: entry1259#dota2-autochess
[1259 ton-hack by groovy fly]: entry1259#ton-hack
[1260 ton delegation pool platform by night turkey]: entry1260#ton-delegation-pool-platform
[1261 stake pool by slim dodo]: entry1261#stake-pool
[1262 cheques by grim duck]: entry1262#cheques
[1263/1 decentralized exchange by merry ant]: entry1263#decentralized-exchange
[1263/2 trc20 by merry ant]: entry1263#trc20
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
