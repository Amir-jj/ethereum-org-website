---
title: Транзакции
description: "Обзор транзакций Ethereum: как они работают, их структура данных и как их отправлять через приложение."
lang: ru
---

Транзакции — это криптографически подписанные инструкции от аккаунтов. Аккаунт инициирует транзакцию для обновления состояния сети Ethereum. Самая простая транзакция — перевод ETH с одного аккаунта на другой.

## Прежде чем начать {#prerequisites}

Чтобы помочь вам лучше понять эту страницу, мы рекомендуем сначала прочитать разделы [Аккаунты](/developers/docs/accounts/) и наше [Введение в Ethereum](/developers/docs/intro-to-ethereum/).

## Что такое транзакция? {#whats-a-transaction}

Транзакция Ethereum относится к действию, инициированному внешним аккаунтом, то есть аккаунтом, управляемым человеком, а не контрактом. Например, если Боб отправляет Алисе 1 ETH, аккаунт Боба должен быть дебетован, а счет Алисы — кредитован. Это действие по изменению состояния происходит внутри транзакции.

![Диаграмма, показывающая изменение состояния из-за транзакции](./tx.png) _Диаграмма адаптирована к [Ethereum EVM иллюстрирована](https://takenobu-hs.github.io/downloads/ethereum_evm_illustrated.pdf)_

Транзакции, которые изменяют состояние EVM, должны транслироваться по всей сети. Любой узел может транслировать запрос на выполнение транзакции в EVM; после этого майнер выполнит транзакцию и распространит полученное изменение состояния на остальную часть сети.

Транзакции требуют комиссии и должны быть добыты, чтобы стать действительными. Чтобы упростить этот обзор, мы рассмотрим газовые комисии и майнинг в другом месте.

Отправленная транзакция включает следующую информацию:

- `recipient` — адрес получателя (если аккаунт внешний, транзакция будет передавать стоимость. Если это аккаунт контракта, транзакция выполнит код контракта).
- `signature` — идентификатор отправителя. Он генерируется, когда закрытый ключ отправителя подписывает транзакцию и подтверждает, что отправитель авторизовал эту транзакцию.
- `value` — количество ETH для перевода от отправителя к получателю (в WEI, номинал ETH).
- `data` — необязательное поле для включения произвольных данных.
- `gasLimit` — максимальное количество единиц газа, которое может быть использовано транзакцией. Единицы измерения газа представляют собой вычислительные шаги.
- `maxPriorityFeePerGas` — максимальное количество газа, которое можно использовать в качестве награды для майнера.
- `maxFeePerGas` — максимальное количество газа, которое будет выплачено за транзакцию (включая `baseFeePerGas` и `maxPriorityFeePerGas`).

Газ — это ссылка на вычисления, необходимые для обработки транзакции майнером. Пользователи должны платить за это вычисление. `GasLimit` и `maxPriorityFeePerGas` определяют максимальную комиссию за транзакцию, выплачиваемую майнеру. [Подробнее о газе](/developers/docs/gas/).

Объект транзакции будет выглядеть примерно так:

```js
{
  from: "0xEA674fdDe714fd979de3EdF0F56AA9716B898ec8",
  to: "0xac03bb73b6a9e108530aff4df5077c2b3d481e5a",
  gasLimit: "21000",
  maxFeePerGas: "300",
  maxPriorityFeePerGas: "10",
  nonce: "0",
  value: "10000000000"
}
```

Но объект транзакции должен быть подписан с использованием приватного ключа отправителя. Это доказывает, что транзакция могла исходить только от отправителя и не была отправлена ​​обманным путем.

Клиент Ethereum, такой как Geth, будет обрабатывать этот процесс подписи.

Пример вызова [JSON-RPC](https://eth.wiki/json-rpc/API):

```json
{
  "id": 2,
  "jsonrpc": "2.0",
  "method": "account_signTransaction",
  "params": [
    {
      "from": "0x1923f626bb8dc025849e00f99c25fe2b2f7fb0db",
      "gas": "0x55555",
      "maxFeePerGas": "0x1234",
      "maxPriorityFeePerGas": "0x1234",
      "input": "0xabcd",
      "nonce": "0x0",
      "to": "0x07a565b7ed7d7a678680a4c162885bedbb695fe0",
      "value": "0x1234"
    }
  ]
}
```

Пример ответа:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "raw": "0xf88380018203339407a565b7ed7d7a678680a4c162885bedbb695fe080a44401a6e4000000000000000000000000000000000000000000000000000000000000001226a0223a7c9bcf5531c99be5ea7082183816eb20cfe0bbc322e97cc5c7f71ab8b20ea02aadee6b34b45bb15bc42d9c09de4a6754e7000908da72d48cc7704971491663",
    "tx": {
      "nonce": "0x0",
      "maxFeePerGas": "0x1234",
      "maxPriorityFeePerGas": "0x1234",
      "gas": "0x55555",
      "to": "0x07a565b7ed7d7a678680a4c162885bedbb695fe0",
      "value": "0x1234",
      "input": "0xabcd",
      "v": "0x26",
      "r": "0x223a7c9bcf5531c99be5ea7082183816eb20cfe0bbc322e97cc5c7f71ab8b20e",
      "s": "0x2aadee6b34b45bb15bc42d9c09de4a6754e7000908da72d48cc7704971491663",
      "hash": "0xeba2df809e7a612a0a0d444ccfa5c839624bdc00dd29e3340d46df3870f8a30e"
    }
  }
}
```

- `raw` — это подписанная транзакция в кодированной форме Recursive Length Prefix (RLP)
- `tx` — это подписанная транзакция в форме JSON

С помощью хэша подписи можно криптографически доказать, что транзакция пришла от отправителя и была отправлена ​​в сеть.

### Поле данных {#the-data-field}

В подавляющем большинстве операций доступ к контракту осуществляется с внешнего аккаунта. Большинство контрактов написаны на Solidity и интерпретируют свое поле данных в соответствии с [бинарным интерфейсом приложения (ABI)](/glossary/#abi/).

Первые четыре байта указывают, какую функцию следует вызвать, используя хэш имени функции и ее аргументов. Иногда можно определить функцию по селектору, используя [эту базу данных](https://www.4byte.directory/signatures/).

Остальная часть calldata — это аргументы, [закодированные в соответствии со спецификациями ABI](https://docs.soliditylang.org/en/latest/abi-spec.html#formal-specification-of-the-encoding).

Например, рассмотрим [эту транзакцию](https://etherscan.io/tx/0xd0dcbe007569fcfa1902dae0ab8b4e078efe42e231786312289b1eee5590f6a1). Чтобы увидеть calldata, используйте **Нажмите, чтобы увидеть больше**.

Селектор функции — `0xa9059cbb`. Существует несколько [известных функций с такой сигнатурой](https://www.4byte.directory/signatures/?bytes4_signature=0xa9059cbb). В этом случае [исходный код контракта](https://etherscan.io/address/0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48#code) был загружен в Etherscan, поэтому мы функцию: `transfer(address, uint256)`.

Остальные данные таковы:

```
0000000000000000000000004f6742badb049791cd9a37ea913f2bac38d01279
000000000000000000000000000000000000000000000000000000003b0559f4
```

Согласно спецификациям ABI целочисленные значения (такие как адреса, которые являются 20-байтовыми целыми числами) отображаются в ABI как 32-байтовые слова, заполненные нулями спереди. Итак, мы знаем адрес `to`: [`4f6742badb049791cd9a37ea913f2bac38d01279`](https://etherscan.io/address/0x4f6742badb049791cd9a37ea913f2bac38d01279). Значение `value` равно 0x3b0559f4 = 990206452.

## Типы транзакций {#types-of-transactions}

В Ethereum существует несколько различных типов транзакций:

- Обычные транзакции: транзакция с одного кошелька на другой.
- Транзакции развертывания контракта: транзакция без адреса «to», где поле данных используется для кода контракта.
- Исполнение контракта: транзакция, которая взаимодействует с развернутым умным контрактом. В этом случае адрес «to» — это адрес умного контракта.

### О газе {#on-gas}

Как уже упоминалось, выполнение транзакций требует затрат [газа](/developers/docs/gas/). Для простых транзакций перевода требуется 21 000 единиц газа.

Таким образом, чтобы Боб отправил Алисе 1 ETH с `baseFeePerGas` 190 gwei и `maxPriorityFeePerGas` 10 gwei, Бобу необходимо будет заплатить следующую комиссию:

```
(190 + 10) * 21 000 = 4 200 000 gwei
--или--
0,0042 ETH
```

Со счета Боба будет списано **-1,0042 ETH**

На счет Алисы будет зачислено **+1,0 ETH**

Сжигаемая базовая комиссия составит **-0,00399 ЕТН**

Майнер удержит **+0,000210 ETH**

Газ также необходим для любого взаимодействия с умными контрактами.

![Схема, показывающая, как возвращается неиспользованный газ](./gas-tx.png) _Источник адаптированной диаграммы: [Ethereum EVM illustrated](https://takenobu-hs.github.io/downloads/ethereum_evm_illustrated.pdf)_

Любой газ, не использованный в транзакции, возвращается в аккаунт пользователя.

## Жизненный цикл транзакции {#transaction-lifecycle}

После отправки транзакции происходит следующее:

1. После отправки транзакции криптография генерирует хэш транзакции: `0x97d99bc7729211111a21b12c933c949d4f31684f1d6954ff477d0477538ff017`
2. Затем транзакция транслируется в сеть и включается в пул с множеством других транзакций.
3. Чтобы подтвердить транзакцию и считать ее «успешной», майнер должен выбрать вашу транзакцию и включить ее в блок.
   - На этом этапе вам, возможно, придется подождать, если сеть занята, а майнеры не запаздывают.
4. Ваша транзакция получает «подтверждение». Количество подтверждений — это количество блоков, созданных с момента блока, в который была включена ваша транзакция. Чем выше это число, тем больше вероятность того, что сеть обработала и распознала транзакцию.
   - Недавние блоки могут быть реорганизованы, создавая впечатление, что транзакция не была удачной; однако транзакция может быть действительной, но включенной в другой блок.
   - Вероятность реорганизации уменьшается с каждым последующим добытым блоком, т. е. чем больше количество подтверждений, тем более неизменной становится транзакция.

## Визуализация {#a-visual-demo}

Посмотрите, как Остин рассказывает о транзакциях, газе и майнинге.

<YouTube id="er-0ihqFQB0" />

## Типизированная оболочка транзакций {#typed-transaction-envelope}

Ethereum изначально имел один формат транзакций. Каждая транзакция содержала значение nonce, цену на газ, лимит газа, адрес, значение, данные, v, r и s. Эти поля закодированы RLP и выглядели примерно так:

`RLP([nonce, gasPrice, gasLimit, to, value, data, v, r, s])`

Ethereum эволюционировал для поддержки нескольких типов транзакций, чтобы реализовать новые функции, такие как списки доступа и [EIP-1559](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md), без влияния на старые форматы транзакций.

[EIP-2718: типизированная оболочка транзакции](https://eips.ethereum.org/EIPS/eip-2718) определяет тип транзакции, который является основой для будущих типов транзакций.

EIP-2718 — это новая универсальная оболочка для типизированных транзакций. В новом стандарте транзакции интерпретируются как:

`TransactionType || TransactionPayload`

Где поля определяются как:

- `TransactionType`: число между 0 и 0x7f, в общей сложности 128 возможных типов транзакций.
- `TransactionPayload` — произвольный байтовый массив, определяемый типом транзакции.

## Дополнительные ресурсы {#further-reading}

- [EIP-2718: типизированная оболочка транзакции](https://eips.ethereum.org/EIPS/eip-2718)

_Знаете ресурс сообщества, который вам пригодился? Измените эту страницу и добавьте его!_

## Похожие темы {#related-topics}

- [Аккаунты](/developers/docs/accounts/)
- [Виртуальная машина Ethereum (EVM)](/developers/docs/evm/)
- [Газ](/developers/docs/gas/)
- [Майнинг](/developers/docs/consensus-mechanisms/pow/mining/)
