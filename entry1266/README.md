# [1266 by Nice Llama]

## [ERC20]

> ERC20 tokens ported to TON blockchain
>
> Features:
>
> - token name and symbol
> - token decimals
> - multiple non-interchangeable networks
> - ready to use with lite-client

### Multiple non-interchangeable networks

Заявлено как фича! Контракт не дает пересылать токены между воркчейнами. Дикшинари с балансами вложен в дикшинари с воркчейнами. Функции обработки воркчейнов и ошибок идут "поверх" балансов. Еще и разным почерком написаны. В общем кто-то за кем-то дописывал. Можно было просто реджекнуть если не воркчейн контракта, раз уж так получилось.

### Создание токенов

В контракте нет механизма создания токенов. В примере значение supply присваивается на баланс одному адресу, который потом отдельными трансферами должен его раздавать.

```fsharp
// setup balances to give owner_address the entire supply
variable network_balances dictnew network_balances !
<b supply 256 u, b> <s owner_address network_balances @ 256 udict! drop network_balances !
<b network_balances @ ref, wc balances @ 8 b>idict! drop balances !
```

В итоге supply это просто константа типа name, symbol, decimal. В настоящем ERC20 есть механизмы создания токенов, а supply это количество созданых токенов.

Если создания токенов нет, тогда и симпл трансфер нужно реджектить.

```clojure
;; simple transfer with comment
if (op == ApiOps::None()) {
	return();
}
```

### Ready to use with lite-client

Методы баланса и апрувала просто весь дикшинари возвращают. Комментарий забыли обновить, там еще один дикшинари с воркчейнами.

```clojure
;; get balances dictionary (owner -> balance)
cell getbalances() method_id {
  (_, _, _, _, cell balances, _) = load_data();
  return balances;
}

;; get approvals dictionary (owner -> { sender -> approved })
cell getapprovals() method_id {
  (_, _, _, _, _, cell approvals) = load_data();
  return approvals;
}
```

Потом кто-то дописал `getbalance`, там нормально.

### Нет интернал операций

Что же это за ERC20 который нельзя спросить баланс из другого контракта?

## [Ring-signature based mixer]

> Ring-signature based mixer
>
> Find friends and use this contract to mix your outputs using ring-signature algorithm implemented with RSA keys. Allows to detach senders and recipients without using any centralized service.
>
> Features:
>
> - RSA on funC
> - Ring signatures on funC
> - Prepare contract offchain and process withdrawals onchain

## RSA on funC, Ring signatures on funC

Не шарю в криптографии на столько, чтобы проверить. Но ключи маленькие и использовать это нельзя. Вся суть ведь в RSA с большими ключами. В общем это не совсем _RSA on funC_.

## Prepare contract offchain

Не шарю в криптографии на столько, чтобы знать как маскировать транзакции. Но ringsig точно надо использовать не так. Там на самом деле есть `approve_addresses` по которому и проходит авторизация, а ringsig это какой-то лишний элемент. Если я видел любую подпись и мой адрес в `approve_addresses`, то я смогу вывести реплейнув ту подпись (ringsig же). Так что если бы там были просто ключи (хотя зачем?), то было бы безопасней.

```clojure
(int exist, int index) = if_exist(approve_addresses, number_of_participants, addr);
if(exist == 0) {
  return send_message(addr, Fail(), query_id, 1, 0, 0);
}
```

## ... and process withdrawals onchain

Контракт требует депозиты от всех участников перед тем как согласится отдавать. Если один из участников не отправит, то другие будут так и сидеть. Не отправляя депозит я ничем не рискую, а могу и заработать. Ребятам которые ждут теперь чтобы забрать свои деньги выгодно скинуться и отправить вместо меня.

```clojure
if(get_balance().first() > number_of_participants * payment_amount) {
  money_available = 1;
}
if(money_available == 0) {
  return send_message(addr, Fail(), query_id, 1, 0, 0);
}
```

## Мелочи всякие

Перменную эту `money_available` не понятно зачем в data пишут. Сообщения пытаются отправлять с 0 грам и без мод 64. Дикшинари `approve_addresses` как массив индексирован.

## [PR218 by blackyblack]

> body-cell slice bits are calculated before 1 i flag so there is a case when builder has exactly the body-cell length bits left and body-cell won't fit.

Все так, только вместо:

```fsharp
sbitrefs swap 1 + swap rot brembitrefs rot >= -rot <= and
```

Лучше:

```fsharp
sbitrefs rot brembitrefs rot >= -rot < and
```

[1266 by nice llama]: https://contest.com/blockchain-2/entry1266
[erc20]: https://github.com/adoriasoft/ton_fungible_token
[ring-signature based mixer]: https://github.com/adoriasoft/ton_payment_mixer
[pr#218 by blackyblack]: https://github.com/ton-blockchain/ton/pull/218
