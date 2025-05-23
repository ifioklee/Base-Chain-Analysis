# <p align="center" style="margin-top: 0px;">  BASE CHAIN ANALYSIS 

## <p align="center"> EXPLORING THE DATABASE

### 1. What is the total number of transactions, average daily transaction, unique number of wallet addresses, number of blocks created, average daily transactions and average transaction per wallet address

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


