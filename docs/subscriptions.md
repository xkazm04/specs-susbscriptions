# Subscriptions & Alerts
Endpoints to handle subscriptions to the Tatum Platform. Subscriptions allow users to enable some additional features or reports that are not enabled by default, like outgoing off-chain transaction scanning, accounts with balances above the limit, etc.

## Pricing
2 credits for API call. Every type of subscription has different credit pricing.

## Subscription types

Subscription type | Description | Additional credit cost
-----|----------|---------
 ADDRESS_TRANSACTION | Enable HTTP POST JSON notifications for any blockchain transactions at the specified address. | 5 credits / fired webhook
 ACCOUNT_INCOMING_BLOCKCHAIN_TRANSACTION | Enable HTTP POST JSON notifications on incoming blockchain transactions on virtual accounts. | 1 credit / monitored day
 ACCOUNT_PENDING_BLOCKCHAIN_TRANSACTION |   Enable HTTP POST JSON notifications on incoming blockchain transactions on virtual accounts. | | 1 credit / monitored day
 CUSTOMER_TRADE_MATCH| Enable HTTP POST JSON notifications on closed trade, which occurs on any customer account. | 10 credit / monitored day
CUSTOMER_PARTIAL_TRADE_MATCH| Enable HTTP POST JSON notifications on partialy filled trade, which occurs on any customer account. | 10 credit / monitored day
TRANSACTION_IN_THE_BLOCK | Enable HTTP POST JSON notifications on ledger => blockchain transaction, when transaction is included in the block. | 10 credits daily + 1 per notification
ACCOUNT_BALANCE_LIMIT | Report with all account balances above desired limit.
TRANSACTION_HISTORY_REPORT | Report with all ledger transactions for last X hours, where X is set by the subscription attribute as interval.
KMS_FAILED_TX | Enable HTTP POST JSON notifications on error during KMS signature process.| 10 credits daily
OFFCHAIN_WITHDRAWAL| Off-chain scanning of outgoing transactions for BTC, BCH, LTC, DOGE and ETH blockchains | 20 credits / address

### 1. ADDRESS_TRANSACTION
This webhook will be sent when a transaction arrives to or is sent from the blockchain address specified in the call - this applies to both incoming/outgoing transactions in the native currency of the blockchain as well as any incoming/outgoing transactions for any type of token.

Free community plans can monitor up to 10 addresses per plan.
Please refer to the availability and consumption table below.

#### Pricing & limitation
- **Free plans** - 10 addresses across all blockchains
- **Paid plans** - unlimited addresses across all blockchains


Blockchain | Asset support | Credit consumption
---------|----------|---------
**Bitcoin** | BTC | 20 credits / day / address
**Bitcoin Cash** | BCH, only incoming transactions | 20 credits / day / address
**Celo** | CELO, Internal transfers, ERC20, ERC721, ERC1155 | 25 credits / day / address
**Dogecoin** | DOGE, only incoming transactions | 20 credits / day / address
**Ethereum** | ETH, Internal transfers, ERC20, ERC721, ERC1155 | 25 credits / day / address
**Klaytn** | KLAY, ERC20, ERC721, ERC1155 | 25 credits / day / address
**Litecoin** | LTC | 20 credits / day / address
**Polygon** | MATIC, ERC20, ERC721, ERC1155 | 40 credits / day / address
**Solana** | SOL, SLP and NFTs | 50 credits / day / address
**Terra Luna** | LUNA, KRW, USD and all other assets | 30 credits / day / address


#### Request example

```json
{
  "address": "FykfMwA9WNShzPJbbb9DNXsfgDgS3XZzWiFgrVXfWoPJ", // address on which the transaction occurs. For EVM chains, this is recipient address
  "txId": "2rdy3YCZHSwvpWtuDom1d4Jjy5UU9STLxF3ffXau6GToReDkfw8wEgX541fvzvh6btVC5D8iNapcKTXfPsoDBk7A", // transaction id
  "blockNumber": 110827114, // block number
  "asset": "3gUeeR3BfVhukYJMwtHownRtRkGcf1bvwiV8TbKMZBVz", // Asset of transaction, for token assets it is address of that token, for native name of the native asset like SOL
  "amount": "1", // amount of the asset that was credited (+) or debited (-) from the address. In case of EVM chains, amount is always positive when counterAddress is present
  "tokenId": "1", // ID of the transferred token, if it is ERC721 / ERC1155
  "type": "token", // type of transaction, could be 'native' or 'token'
  "counterAddress": "FykfMwA9WNShzPJbbb9DNXsfgDgS3XZzWiFgrVXfWoPJ" // optional counter party address of the transaction. For EVM chains, this is recipient address
}
```


### 2. ACCOUNT_INCOMING_BLOCKCHAIN_TRANSACTION

This web hook will be invoked, when the transaction is credited to the virtual account. Transaction is credited, when it has sufficient amount of blockchain confirmations.

1 credit will be debited for every monitored account every day

For BTC, LTC, BCH, DOGE - 2 confirmations, others - 1 confirmation

#### Request example
```json 
{
  "date": 1619176527481,
  "amount": "0.005",
  "currency": "BTC",
  "id": "6082ab462936b4478117c6a0",
  "reference": "c9875708-4ba3-41c9-a4cd-271048b41b9a", // ledger transaction reference
  "txId": "45af182a0ffab58e5ba32fee57b297b2260c6e23a1de5ddc76c7ee22d72dea99",
  "blockHash": "45af182a0ffab58e5ba32fee57b297b2260c6e23a1de5ddc76c7ee22d72dea99", // hash of the block, might not be present all the time
  "blockHeight": 12345,
  "from": "SENDER_ADDRESS", // might not ebe present all the time, for UTXO based blockchains it's not there
  "to": "RECIPIENT_ADDRESS_CONNECTED_TO_LEDGER_ACCOUNT", // blockchain address of the recipient
  "index": 5 // for UTXO based chains (BTC,LTC,DOGE,BCH,ADA) this is the index of the output in the transaction
}
```

### 3. ACCOUNT_PENDING_BLOCKCHAIN_TRANSACTION
 
This web hook will be invoked, when the transaction appears for the first time in the block - it has 1 confirmation, but is still not credited to the ledger account, because it does not have enough confirmations.

This web hook is invoked only for BTC, LTC, DOGE and BCH accounts.

#### Request example
```json 
{
  "date": 1619176527481,
  "amount": "0.005",
  "currency": "BTC",
  "id": "6082ab462936b4478117c6a0",
  "reference": "c9875708-4ba3-41c9-a4cd-271048b41b9a", // ledger transaction reference
  "txId": "45af182a0ffab58e5ba32fee57b297b2260c6e23a1de5ddc76c7ee22d72dea99",
  "blockHash": "45af182a0ffab58e5ba32fee57b297b2260c6e23a1de5ddc76c7ee22d72dea99", // hash of the block, might not be present all the time
  "blockHeight": 12345,
  "from": "SENDER_ADDRESS", // might not ebe present all the time, for UTXO based blockchains it's not there
  "to": "RECIPIENT_ADDRESS_CONNECTED_TO_LEDGER_ACCOUNT", // blockchain address of the recipient
  "index": 5 // for UTXO based chains (BTC,LTC,DOGE,BCH,ADA) this is the index of the output in the transaction
}
```

### 4. CUSTOMER_TRADE_MATCH

This web hook will be invoked, when the open trade is filled and closed. Works also for the Trade Futures. If is triggered by the futures, bool field expiredWithoutMatch is present.

Costs 10 credits will be debited for every monitored customer every day.

#### Request example
```json 
{
  "created": 1619176527481,
  "amount": "0.005",
  "price": "0.02",
  "type": "SELL",
  "pair": "VC_CHF/VC_CHF3",
  "id": "6082ab462936b4478117c6a0", // id of the trade
  "currency1AccountId": "6082ab462936b4478117c6a0",
  "currency2AccountId": "6082ab512936b4478117c6a2",
  "fee": null,
  "feeAccountId": null,
  "isMaker": true,
  "expiredWithoutMatch": false
}
```

### 5. CUSTOMER_PARTIAL_TRADE_MATCH

This web hook will be invoked, when the open trade is partialy filled.

Costs 10 credits will be debited for every monitored customer every day.

#### Request example
```json 
{
  "created": 1619176527481,
  "amount": "0.005",
  "price": "0.02",
  "type": "SELL",
  "pair": "VC_CHF/VC_CHF3",
  "id": "6082ab462936b4478117c6a0", // id of the trade
  "currency1AccountId": "6082ab462936b4478117c6a0",
  "currency2AccountId": "6082ab512936b4478117c6a2",
  "fee": null,
  "feeAccountId": null,
  "isMaker": true,
  "expiredWithoutMatch": false
}
```

### 6. TRANSACTION_IN_THE_BLOCK

This web hook will be invoked, when the outgoing transaction is included in the block.

Costs 10 credits will be debited every day, 1 credit for every included transaction notified via web hook.

#### Request example
```json 
{
  "txId":"0x026f4f05b972c09279111da13dfd20d8df04eff436d7f604cd97b9ffaa690567",
  "reference": "90270634-5b07-4fad-b17b-f82899953533",
  "accountId": "6082ab462936b4478117c6a0",
  "currency": "BSC",
  "withdrawalId": "608fe5b73a893234ba379ab2",
  "address": "0x8ce4e40889a13971681391AAd29E88eFAF91f784",
  "amount": "0.1",
  "blockHeight": 8517664
}
```

### 6. KMS_FAILED_TX
This web hook will be invoked, when the Tatum KMS receives error during processing transactions.

Costs 10 credits daily.

#### Request example
```json 
{
  "signatureId": "6082ab462936b4478117c6a0",
  "error": "Error message from the KMS"
}
```


### 7. OFFCHAIN_WITHDRAWAL
Off-chain scanning of outgoing transactions for BTC, BCH, LTC, DOGE and ETH blockchains - by default addresses in registered for off-chain scanning are synchronized only against incoming transactions. 

By enabling this feature, off-chain accounts with connected blockchain addresses will be scanned also for outgoing transactions. This transaction wil be recorder to the ledger and account balance will be decreased - don't use it if you perform your own transactions rom the account to the blockchain.

20 credits will be debited for every address registered for scanning every day.

### 8. ACCOUNT_BALANCE_LIMIT
Report with all account balances above desired limit.
### 9. TRANSACTION_HISTORY_REPORT
Report with all ledger transactions for last X hours, where X is set by the subscription attribute as interval.

Maximum number of transactions returned by this report is 20000. Transactions are obtained from the time of the invocation of the `GET` method to obtain report - X hours.

In case of unsuccesful web hook response status - other then 2xx - web hook is repeated 9 more times with exponential backoff.

Parameters are T = 15 * 2.7925^9, where 15 is interval in s, backoff rate is 2.7925 and 9 is current number of retries. Last web hook is fired after 24 hours approximatelly. After last failed attempt, web hook is deleted from our system. The 2xx response must be returned in 10 seconds after web hook is fired.


