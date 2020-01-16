# [1254 by Shiny Shrimp]

## [ICO]

> - Soft cap. Users can return their funds back if it isn't reached.
> - Bounty steaks for enthusiasts before ICO. Afterwards tokens percentage are transferred to bounty wallets.
> - Multiple stages. Each stage has its own discounts and limited number of tokens.

Lowish quality. Complex.

It requires currency to be registered in masterchain. Buying tokens doesn't account processing fees, so during refund someone will be left without money.

### Denying refunds

Owner can deny refunds when soft-cap wasn't reached by precisely timing his transaction.

```clojure
throw_if (47, (now() < date_end) & (tokens > 0));
if (now() > date_end) & (collected_funds < softcup) {
  ;; ...
}
```

[1254 by shiny shrimp]: https://contest.com/blockchain-2/entry1254
[ico]: https://github.com/plexar88/ico/blob/300d6fb96f297087e29de11d9e4a3387775f2630/simple-ico.fc
