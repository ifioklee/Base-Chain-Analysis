# <p align="center" style="margin-top: 0px;">  BASE CHAIN ANALYSIS 

## <p align="center"> EXPLORING THE DATABASE

#### 1. What is the total number of transactions, average daily transaction, unique number of wallet addresses, number of blocks created, average daily transactions and average transaction per wallet address

### Step Taken:

* A CTE table using the WITH function to query both the _base.core.fact_transactions_ and _base.core.fact_blocks_ table

```sql
WITH txn_counts AS (
  SELECT
    COUNT(DISTINCT DATE_TRUNC('day', block_timestamp)) AS days,
    COUNT(DISTINCT tx_hash) AS transactions,
    COUNT(DISTINCT from_address) AS addresses,
    COUNT(DISTINCT to_address) AS receivers
  FROM
    base.core.fact_transactions
),
base AS (
  SELECT
    days,
    transactions,
    -- average transactions per day
    (transactions :: DECIMAL / days) AS avg_daily_txn,
    addresses,
    -- average unique wallets per hour
    (addresses :: DECIMAL / (days * 24)) AS avg_hourly_wallet,
    -- average transactions per wallet (over the whole period)
    (transactions :: DECIMAL / NULLIF(addresses, 0)) AS avg_txn_per_wallet,
    receivers
  FROM
    txn_counts
),
blocks AS (
  SELECT
    COUNT(DISTINCT block_number) AS produced_blocks
  FROM
    base.core.fact_blocks
)
SELECT
  b.days,
  b.transactions,
  b.avg_daily_txn,
  b.addresses,
  b.avg_hourly_wallet,
  b.avg_txn_per_wallet AS "Average Transactions Per Wallet",
  b.receivers,
  blk.produced_blocks AS "Produced Blocks"
FROM
  base b
  CROSS JOIN blocks blk;
```

### Output:
| DAYS | TRANSACTIONS |  AVG_DAILY_TXN | ADDRESSES | AVG_HOURLY_WALLET | Average Transactions Per Wallet | RECEIVERS | Produced Blocks |   |
|:----:|:------------:|:--------------:|:---------:|:-----------------:|:-------------------------------:|:---------:|:---------------:|---|
| 709  | 2615307871   | 3688727.603667 | 178097073 | 10466.447638      | 14.684732                       | 94177268  | 30605644        |   |



#### 2. What is the share of transactions( number of wallet addresses with successfull transactions and number of wallet addresses with failed transactions)

### Steps Taken:

```sql
SELECT
CASE WHEN (TX_SUCCEEDED = 'TRUE' or TX_SUCCEEDED is null) THEN 'SUCCESSFUL TXNS' else 'FAILED TXNS' end as type,
COUNT(DISTINCT from_address) AS address,
COUNT(DISTINCT tx_hash) AS txns
FROM base.core.fact_transactions
GROUP BY 1
```

#### Output:

|   |       TYPE      |  ADDRESS  |    TXNS    |  
|:-:|:---------------:|:---------:|:----------:|
| 1 | SUCCESSFUL TXNS | 177252776 | 2269375022 |  
| 2 | FAILED TXNS     | 7173006   | 345932849  |  
|   |                 |           |            |   

#### 3. What is daily number of wallet addresses and daily number of wallet addresses on Base?

### Steps Taken:

* Run the sql query below 

```sql
WITH base AS (
    SELECT  
        block_timestamp::date AS date,
        COUNT(DISTINCT tx_hash) AS transactions,
        COUNT(DISTINCT from_address) AS addresses,
        COUNT(DISTINCT tx_hash) * 1.0 / COUNT(DISTINCT from_address) AS average_transaction_per_wallet,
        COUNT(DISTINCT to_address) AS receivers
    FROM 
        base.core.fact_transactions
    GROUP BY 
        block_timestamp::date
),
blocks AS (
    SELECT 
        block_timestamp::date AS date,
        COUNT(DISTINCT block_number) AS produced_blocks
    FROM 
        base.core.fact_blocks
    GROUP BY 
        block_timestamp::date
)
SELECT 
    base.date,
    base.transactions,
    base.addresses,
    base.average_transaction_per_wallet,
    base.receivers,
    blocks.produced_blocks
FROM 
    base
JOIN 
    blocks 
ON 
    base.date = blocks.date;
```

#### Output:

|   |           DATE          | TRANSACTIONS | ADDRESSES | AVERAGE_TRANSACTION_PER_WALLET | RECEIVERS | PRODUCED_BLOCKS |  
|:-:|:-----------------------:|:------------:|:---------:|:------------------------------:|:---------:|:---------------:|
| 1 | 2025-03-11 00:00:00.000 | 7512152      | 810385    | 9.269856                       | 162361    | 43200           | 
| 2 | 2025-02-28 00:00:00.000 | 7465480      | 711795    | 10.488245                      | 173169    | 43200           |   
| 3 | 2024-08-05 00:00:00.000 | 3568321      | 406075    | 8.787345                       | 172993    | 43200           |   
| 4 | 2024-08-14 00:00:00.000 | 4101846      | 744758    | 5.507623                       | 382090    | 43200           |   
| 5 | 2024-09-10 00:00:00.000 | 4455066      | 1161252   | 3.836433                       | 555864    | 43200           |   
| 6 | 2024-09-27 00:00:00.000 | 5382655      | 1318809   | 4.081452                       | 782729    | 43200           |   
| 7 | 2024-10-02 00:00:00.000 | 4814011      | 1001487   | 4.806863                       | 480840    | 43200           |   
| 8 | 2024-10-12 00:00:00.000 | 6099543      | 1424650   | 4.281433                       | 619305    | 43200           |  
| 9 | 2025-02-01 00:00:00.000 | 8594775      | 1093815   | 7.857613                       | 225043    | 43200       
---
* This isn't the entire result of the table
---

#### 4. Daily number of new wallet addresses on Base

### Steps Taken:

```sql
WITH new AS (SELECT min(block_timestamp) AS date,
(from_address) as users
FROM base.core.fact_transactions
WHERE TX_SUCCEEDED = 'TRUE'
GROUP BY 2)

SELECT date::date as "Date",
COUNT(DISTINCT users) AS "New Address",
sum("New Address") over (ORDER BY "Date" asc) AS "Cumulative New Addresses"
FROM new
WHERE date >= '2023-06-15'
GROUP BY 1
```

#### OutPut 

|    |           Date          | New Address | Cumulative New Addresses |   
|:--:|:-----------------------:|:-----------:|:------------------------:|
|  1 | 2025-03-10 00:00:00.000 | 345081      | 140470087                |  
|  2 | 2024-07-20 00:00:00.000 | 489586      | 21300275                 |   
|  3 | 2024-12-15 00:00:00.000 | 665133      | 96283257                 |  
|  4 | 2024-01-26 00:00:00.000 | 8963        | 3161544                  |   
|  5 | 2025-03-28 00:00:00.000 | 328050      | 148293557                |  
|  6 | 2025-04-22 00:00:00.000 | 315827      | 157295217                |   
|  7 | 2024-04-04 00:00:00.000 | 77930       | 6098423                  |   
|  8 | 2023-10-01 00:00:00.000 | 14432       | 1497498                  |   
|  9 | 2024-01-03 00:00:00.000 | 8797        | 2954236                  |   
| 10 | 2024-04-08 00:00:00.000 | 125572      | 6556016                  |   

---
* This isn't the entire output of the table
---

#### 5. What is the average block time on Base

#### Steps Taken:

```sql
SELECT 
avg(datediff(second,a.block_timestamp, b.block_timestamp)) AS "Avg Time (Sec)"
FROM base.core.fact_transactions a, base.core.fact_blocks b
WHERE a.block_number = b.block_number-1
```

### Output:

|   | Avg Time (Sec) |  
|---|----------------|
| 1 | 2              | 

* The average block creation time on Base is 2 seconds

#### 6. What is the total transaction fee, maximum ETH spent in fee and average transaction fee on Base

### Steps Taken:

* Run the sql query below

  ```sql
  SELECT 
sum(tx_fee) AS "Total Fee (ETH)",
avg(tx_fee) AS "Avg Fee (ETH)",
max(tx_fee) AS "Max Fee (ETH)"
FROM base.core.fact_transactions
WHERE block_timestamp::date >= '2023-06-15'```

### Output:

|   | Total Fee (ETH) |   Avg Fee (ETH)  | Max Fee (ETH) |  
|:-:|:---------------:|:----------------:|:-------------:|
| 1 | 48125.782852643 | 0.00001839662013 | 7.286356889   | 


#### 7. Daily total transaction fee and daily average transaction fee

#### Steps Taken:

* Run the sql query below

```sql
SELECT block_timestamp::date as date,
sum(tx_fee) AS "Total Fee (ETH)",
avg(tx_fee) AS "Avg Fee (ETH)",
max(tx_fee) AS "Max Fee (ETH)"
FROM base.core.fact_transactions
WHERE block_timestamp >= '2023-06-15'
GROUP BY 1
```

#### Output:

|   |           DATE          | Total Fee (ETH) |   Avg Fee (ETH)   | Max Fee (ETH) |   |   |   |
|---|:-----------------------:|:---------------:|:-----------------:|:-------------:|---|---|---|
| 1 | 2025-04-03 00:00:00.000 | 54.161970889    | 0.000007163568349 | 0.9688898851  |   |   |   |
| 2 | 2024-04-24 00:00:00.000 | 150.34274875    | 0.00005299315576  | 0.9675286433  |   |   |   |
| 3 | 2025-04-18 00:00:00.000 | 56.960639523    | 0.00001026073605  | 0.9667143531  |   |   |   |
| 4 | 2025-04-29 00:00:00.000 | 71.435984174    | 0.00001019346471  | 2.19047981    |   |   |   |
| 5 | 2024-05-25 00:00:00.000 | 55.750442008    | 0.0000326237745   | 0.3506594444  |   |   |   |

* This isn't the complete output of the table

#### 8. What is the Total Value Locked (TVL) on Base?

#### Steps Taken:

* Run the sql query below

```sql
SELECT 
    tvl_usd as tvl
FROM external.defillama.fact_chain_tvl
WHERE chain = 'Base'
    AND date = (SELECT MAX(date) FROM external.defillama.fact_chain_tvl WHERE chain = 'Base');
```

#### OUtput

|   | TVL        |   
|---|------------|
| 1 | 5202477014 |

#### 9. Total number of smart contracts deployed on Base

#### Steps Taken:

* Run the sql query below

```sql
SELECT COUNT(ADDRESS) AS number_of_smart_contracts_deployed
FROM base.core.dim_contracts
```

#### Output:

|   | NUMBER_OF_SMART_CONTRACTS_DEPLOYED |   |   |   |   |   |   |
|---|------------------------------------|---|---|---|---|---|---|
| 1 | 276525773                          |   |   |   |   |   |   |


#### 10. What is the number of smart contract deployers?

### Steps Taken:

* Run the sql query below

```sql
SELECT COUNT(DISTINCT CREATOR_ADDRESS) AS number_of_smart_contracts_creators
FROM base.core.dim_contracts
```

#### Output

|   | NUMBER_OF_SMART_CONTRACTS_CREATORS |   |   |   |   |   |   |
|---|:----------------------------------:|---|---|---|---|---|---|
| 1 | 1692610                            |   |   |   |   |   |   |
