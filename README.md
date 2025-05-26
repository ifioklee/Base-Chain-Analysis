# <p align="center" style="margin-top: 0px;">  Base Chain Analysis 
---
---
## Use Case
Base is an Ethereum [Layer 2](https://x.com/TheRealIfiokLee/status/1532788242508128258)(L2) network that is incubated within [Coinbase](www.coinbase.com), it is designed with the aim to make onchain application fast, cheap and developer-friendly. Fundamentally, Ethereum Layer 2 networks work by taking most transaction execution off-chain and only committing minimal cryptographic proofs or summaries back to the Ethereum chain which serves as the Layer 1 network. Base leverages [Optimism’s open-source OP Stack](https://cointelegraph.com/learn/articles/what-is-base-coinbase-l2-network), using optimistic rollups to batch transactions off-chain and submit succinct proofs to Ethereum Mainnet. This retains Ethereum’s security guarantees while dramatically cutting gas costs and increasing throughput.

The Base mainnet officially went live to the public on August 9, 2023 as part of Coinbase’s “Onchain Summer” initiative. From that time, various data have been generated from the use of the chain across various onchain applications developed accross various usecases.

The goal is to analyze these data and extract insight from various metrics relating to Base.

---

## The Database

For this work [Flipsidecrypto](https://flipsidecrypto.xyz/) Base core database was used to derive the tables used for the analysis. Within Base Core database on Flipsidecrypto, there are 10 tables but for this analysis just 3 tables were used, they are:

- base.core.dim_contracts
- base.core.fact_transactions
- base.core.fact_blocks

Other data source that was used was Defillama Total Value Locked(TVL) database

-external.defillama.fact_chain_tvl

---

## The Analysis
[Exploring the database](https://github.com/ifioklee/Base-Chain-Analysis/blob/main/Exploring%20the%20Database.md)

[Visualization](https://github.com/ifioklee/Base-Chain-Analysis/tree/main/Visulization)

[Dashboard](https://flipsidecrypto.xyz/IfiokLee/base-chain-Zi0L9h)


## Visualization 

Visualizations was created for every metric from the query of Base database using Flipsidecrypto visual studio. The following visulizations were creted:

* [Total number of transactions](https://github.com/ifioklee/Base-Chain-Analysis/blob/main/Visulization/Screenshot%20(46).png)
* [Unique wallet addresses](https://github.com/ifioklee/Base-Chain-Analysis/blob/main/Visulization/Screenshot%20(47).png)
* [Created blocks](https://github.com/ifioklee/Base-Chain-Analysis/blob/main/Visulization/Screenshot%20(48).png)
* [Average daily transactions](https://github.com/ifioklee/Base-Chain-Analysis/blob/main/Visulization/Screenshot%20(49).png)
* [Average transactions per wallet address](https://github.com/ifioklee/Base-Chain-Analysis/blob/main/Visulization/Screenshot%20(50).png)
* [Number of smart contract creators](https://github.com/ifioklee/Base-Chain-Analysis/blob/main/Visulization/Screenshot%20(51).png)


## The Dashboard

The dashboard was created created using [Flipsidecrypto](https://flipsidecrypto.xyz/IfiokLee/base-chain-Zi0L9h) by combinig the individual visualization that were created from the result of the various query done on the Base database on Flipsidecrypto.

The dashboard is a dynamic dashboard which refreshes data every 12 hours. An up-to-date version can be found [here](https://flipsidecrypto.xyz/IfiokLee/base-chain-Zi0L9h)


![Screenshot (65)](https://github.com/user-attachments/assets/a51ee202-36fe-4bc3-bae4-18c24e1a7f40)


# <p align="center" style="margin-top: 0px;">THANKS FOR READING!!
