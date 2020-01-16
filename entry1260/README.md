# [1260 by Night Turkey]

## [TON Delegation Pool Platform]

> - Distinct state transitions from Init setting _deadline_.
> - ... into Delegation collecting deposits.
> - ... after _deadline_ into NeedToStakeToElect to send _stake to elect_:
>   - ... after _elect callback_ WaitingForNextVset.
>   - ... after _elected time_ WaitingToWithdrawFundsFromElector.
>   - ... after _vset deadline_ NeedToWithdrawFundsFromElector to _withdraw from elect_.
>   - ... after _elect callback_ CanWithdrawalFromDelegationPool to _withdraw_.
>   - ... after 2 months ValidatorCanWithdrawRemainingFunds2 to _withdraw remaining_ by owner.
> - Or if _stake to elect_ wasn't sent:
>   - ... after questionable time NotStakedToElectOnTime to _recover stake_.
>   - ... after 2 months ValidatorCanWithdrawRemainingFunds1 to _withdraw remaining_ by owner.
> - Owner (validator) is handling processing fees.

Very high quality. Complex.

### Front-running initialization

During initialization deadline to collect deposits is taken from initial message, but there are no signature checks:

```clojure
() recv_external(slice in_msg) impure {
  ;; recv_external allowed only during initial state
  throw_unless(101, contract_status == Status::Init());
  var delegation_deadline = in_msg~load_uint(32);
  accept_message();
  store_data(Status::Delegation(), now() + delegation_deadline, ...);
}
```

### Front-running contract owner

Withdrawing remaining funds is authorized by `validator_public_key`. Funds are sent to internal message source address. Surprisingly contract address is signed (can be implemented better), but internal message source address and amount aren't. Attacker may discover correct signature from yet unprocessed message and send identical one from his address:

```clojure
() withdraw_remaining_funds(ds, cs, s_addr, contract_status) impure {
  var amount = cs~load_grams();
  var public_key = cs~load_uint(256);

  var (_, my_addr) = my_address().parse_std_addr();
  var contract_address = cs~load_uint(256);
  throw_unless(204, contract_address == my_addr);

  var signature = cs~load_ref().begin_parse().preload_bits(512);
  throw_unless(205, check_data_signature(begin_cell()
    .store_uint(public_key, 256)
    .store_uint(contract_address, 256)
  .end_cell().begin_parse(), signature, validator_public_key));

  ;; ... send amount to s_addr
}
```

A better approach would be to implement standard technique of checking signature over whole message body instead picking individual components.

### Unnecessary delegator public key

Delegators are required to send public key, but it isn't used for signaure checks. The only two signature checks are both using `validator_public_key`. Instead public key is used to index `delegators` dictionary that also contains address. Address is correctly checked and it's impossible to override it.

It's possible to have two delegate stakes from same address but with different keys. As result there is separate `credits` dictionary indexed by address (only last 256u part) that contains amount that can be withdrawn. When it's built additional checks are implemented to handle that one address with two keys case.

### Recovering stake

Deposits are not accounting processing fees. Stakes are simply sums of `msg_value` sent. Moreover it appears that contract deliberately calls `accept_message` when it authenticates delegate. If stake wasn't sent, then delegators are allowed to recover their stakes. There won't be enough money to recover all of them.

[1260 by night turkey]: https://contest.com/blockchain-2/entry1260
[ton delegation pool platform]: https://github.com/button-tech/ton-delegation-pool/blob/e970e738746f04a73a95d4bbde87461fd7d93bf4/contracts/v3/delegation_pool_v3.fc
