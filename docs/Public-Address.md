# Public Address

Monero public address is what you publish to get paid.

An address can be generated offline and for free. It boils down to generating a large random number representing your private spending key.

Publishing your Monero address does __not__ endanger your privacy. That's because in Monero transactions go to stealth addresses which are decoupled from your public address.

There are a few types of public addresses in Monero:

* Standard address - the basic type of the address, also refered to as raw address
* Integrated address - embeds payment ID so you can learn for what you are being paid
* Subaddress - allows for organizing your funds in subaccounts within a single wallet

## Standard address

Monero standard ("raw") address is composed of two public keys:

* public spend key
* public view key

It also contains a checksum and a "network byte" which actually identifies both the network and the address type.

Data structure ([src](https://github.com/monero-project/monero/blob/f7b9f44c1b0d53170fd7f53d37fc67648f3247a2/src/cryptonote_basic/cryptonote_basic_impl.cpp#L159)):

Index       | Size in bytes    | Description
------------|------------------|-------------------------------------------------------------
0           | 1                | identifies the network and address type; [18](https://github.com/monero-project/monero/blob/793bc973746a10883adb2f89827e223f562b9651/src/cryptonote_config.h#L149) - main chain; [53](https://github.com/monero-project/monero/blob/793bc973746a10883adb2f89827e223f562b9651/src/cryptonote_config.h#L161) - test chain
1           | 32               | public spend key
33          | 32               | public view key
65          | 4                | checksum ([Keccak-f[1600] hash](https://github.com/monero-project/monero/blob/8f1f43163a221153403a46902d026e3b72f1b3e3/src/common/base58.cpp#L261) of the previous 65 bytes, trimmed to first [4](https://github.com/monero-project/monero/blob/8f1f43163a221153403a46902d026e3b72f1b3e3/src/common/base58.cpp#L53) bytes)

It totals to 69 bytes. The bytes are then encoded ([src](https://github.com/monero-project/monero/blob/8f1f43163a221153403a46902d026e3b72f1b3e3/src/common/base58.cpp#L240)) in [Monero specific Base58](Monero-Base58) format, resulting in a 95 chars long string. Example standard address:

`4AdUndXHHZ6cfufTMvppY6JwXNouMBzSkbLYfpAV5Usx3skxNgYeYTRj5UzqtReoS44qo9mtmXCqY45DJ852K5Jv2684Rge`

Reference:

* [https://xmr.llcoins.net/addresstests.html](https://xmr.llcoins.net/addresstests.html)

## Integrated address

Monero integrated address embeds a compact payment ID.

Monero integrated address obsoletes the former practice of using full 32-bytes payment ID in a transaction extra field (where it was not encrypted).

Data structure ([src](https://github.com/monero-project/monero/blob/f7b9f44c1b0d53170fd7f53d37fc67648f3247a2/src/cryptonote_basic/cryptonote_basic_impl.cpp#L172)):

Index       | Size in bytes    | Description
------------|------------------|-------------------------------------------------------------
0           | 1                | identifies the network and address type; [19](https://github.com/monero-project/monero/blob/793bc973746a10883adb2f89827e223f562b9651/src/cryptonote_config.h#L150) - main chain; [54](https://github.com/monero-project/monero/blob/793bc973746a10883adb2f89827e223f562b9651/src/cryptonote_config.h#L162) - test chain
1           | 32               | public spend key
33          | 32               | public view key
65          | 8                | compact payment ID -  8 bytes randomly generated by the recipient; note that it does not need encryption in the address itself but it is hidden in a transaction paying to integrated address to prevent linking payment with the address by external observers
73          | 4                | checksum ([Keccak-f[1600] hash](https://github.com/monero-project/monero/blob/8f1f43163a221153403a46902d026e3b72f1b3e3/src/common/base58.cpp#L261) of the previous 73 bytes, trimmed to first [4](https://github.com/monero-project/monero/blob/8f1f43163a221153403a46902d026e3b72f1b3e3/src/common/base58.cpp#L53) bytes)

It totals to 78 bytes. The bytes are then encoded ([src](https://github.com/monero-project/monero/blob/8f1f43163a221153403a46902d026e3b72f1b3e3/src/common/base58.cpp#L240)) in [Monero specific Base58](Monero-Base58) format, resulting in a 106 chars long string. Example integrated address:

`4LL9oSLmtpccfufTMvppY6JwXNouMBzSkbLYfpAV5Usx3skxNgYeYTRj5UzqtReoS44qo9mtmXCqY45DJ852K5Jv2bYXZKKQePHES9khPK`

Reference:

* question on [StackExchenge](https://monero.stackexchange.com/questions/3179/what-is-an-integrated-address) 

## Subaddress

Subaddresses serve two purposes.

### Subadresses group incoming payments

Think income streams.

Subaddresses allow you to group your incoming transactions within a single wallet.

You may want to organize your incoming funds into a streams like "donations", "work", "speculation", etc.

At first glance this is similar to subaccounts in your bank account. There is a very important difference though.

In Monero funds don't really sit on public addresses. Public addresses are conceptually a gateway or a routing mechanism. Funds sit on the unspent outputs. Thus, a single transaction can aggregate and spent outputs from multiple addresses. This means you can spend more than any individual address balance.

### Subadresses prevent payer from linking your payouts together

To prevent the payer from linking your payouts together simply generate a new subaddress for each payout. This way services like [Shapeshift](https://shapeshift.io) wouldn't know it is you again receving Monero.

The advantage over creating multiple wallets is that you only have a single seed and wallet cache to manage. All subaddresses can be derived from the wallet seed.

Each subaddress has an index (with 0 being the base standard address). User interface allows to assign convenience labels to subaddresses. However, labels are not preserved when recreating from seed.

## Caveates

* Subaddress cannot be used to receive transactions having multiple destinations (e.g. pool payouts). Only the standard address (the one with index == 0) can receive such transactions.
* It is not recommended to sweep all the balances of subaddress to main address in a single transaction. That links the subaddresses together on the blockchain. However, this only concerns privacy against specific sender and the situation will never get worse than not using subaddresses in the first place. If you need to join funds while preserving maximum privacy do it with individual transactions (one per subaddress).

Index       | Size in bytes    | Description
------------|------------------|-------------------------------------------------------------
0           | 1                | identifies the network and address type; [42](https://github.com/monero-project/monero/blob/784f7b07f05a645d43f62ed3a9cefda4b0c57825/src/cryptonote_config.h#L153) - main chain; [63](https://github.com/monero-project/monero/blob/784f7b07f05a645d43f62ed3a9cefda4b0c57825/src/cryptonote_config.h#L167) - test chain

Reference:

* [https://github.com/monero-project/monero/pull/2056](https://github.com/monero-project/monero/pull/2056)
* [https://www.reddit.com/r/Monero/comments/5vgjs2/subaddresses_and_disposable_addresses/](https://www.reddit.com/r/Monero/comments/5vgjs2/subaddresses_and_disposable_addresses/)