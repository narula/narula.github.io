---
layout: post
title:  "Technical Risks of Stablecoins"
date:   2021-09-23 13:52:14
categories:
comments: true
---


The term “stablecoin” typically refers to tokens issued on a blockchain which are designed to maintain a stable value with respect to an existing currency, for example the dollar. The largest stablecoins currently are Tether and USDC, with market capitalizations of $68B and $29B, respectively.

There is generally a reasonable understanding of the financial risks involved in stablecoins, particularly around whether the stablecoin is fully backed, and the liquidity of those assets backing it. What we have not heard discussed are the technical risks associated with stablecoins, which is what this post covers. We specifically focus on stablecoins issued by a centralized entity who custodies backing assets, not decentralized algorithmic stablecoins (such as Dai, issued by MakerDAO), though some of these issues apply to those stablecoins as well.

Even centralized stablecoins operate as software running on decentralized blockchains, and it’s important to keep in mind that stablecoins involve layers of software that are not used in traditional financial institution activity. If this software doesn’t operate safely and securely, or if the decentralized networks don’t operate as expected, there could be a loss of confidence in the stablecoin or even outright theft. As it stands now, regulators, issuers, and users of a stablecoin need to understand stablecoin software and the underlying blockchain platforms they run on in order to accurately measure technical and operational risk.

Three important parts of the system to consider are the underlying blockchain, the stablecoin smart contract, and the stablecoin provider’s minting and redemption process.

# The underlying blockchain

Stablecoins run as smart contracts on blockchains. In many cases what we call one stablecoin (for example, Tether or USDC) runs on multiple different blockchains, as entirely separate tokens sharing a single backing. Users can choose which token on which blockchain to obtain, but some might be easier to obtain or use than others, as the tokens are accepted in smart contracts or as payment individually. Each blockchain is used as an accounting of who currently controls a stablecoin token. For example, USDC runs on five different chains: [Ethereum](https://etherscan.io/token/0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48) (26.8B USDC), [Algorand](https://algoexplorer.io/asset/31566704) (220M USDC), [Solana](https://explorer.solana.com/address/EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v) (2.4B USDC), [Stellar](https://stellar.expert/explorer/public/asset/USDC-GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN-1) (11M USDC), and [Tron](https://tronscan.org/#/token20/TEkxiTehnzSmSe2XqrBj4w32RUN966rdz8) (205M USDC), with the intention of adding ten more.[^1] There is no “universal” USDC token—each USDC token is specific to one of these chains. This is often a point of confusion; for example CoinMarketcap mistakenly says that “All of the USDCs in circulation are actually ERC-20 tokens, which can be found on the Ethereum blockchain.”[^2] This could be because Coinbase, one of the largest US cryptocurrency exchanges, currently only sells USDC issued on the Ethereum blockchain.

[^1]: Centre Consortium. “Announcing USDC on Ten New Platforms.” June 29, 2021. https://www.centre.io/blog/announcing-usdc-on-ten-new-blockchain-platforms. Linked explorers accessed as of September 19, 2021.
[^2]: Coinmarketcap. “What is USD Coin (USDC)?” Accessed September 19,2021. https://coinmarketcap.com/currencies/usd-coin/

The choice of blockchain is not a mere implementation detail. These chains have vastly different architectures, languages, dependencies, cryptography, miners or validator sets, and fees. As we describe below, the operation of the blockchain can affect a user’s ability to use the stablecoin effectively. It is unclear whether users can distinguish between the individual features and risks these different blockchain technologies introduce as they use stablecoin tokens built on top of them, nor whether enough information is always conveyed to the user when they engage with the different tokens.

*Security guarantees*. Each blockchain makes different assumptions about the miners and/or validators. For example, Tron relies on 27 rotating validators who “are entrusted to become the next block validators and maintain the blockchain history.”[^3] Unfortunately, in some cases the chain’s correctness (there is always one chain and everyone sees it) and liveness (one can get a transaction processed in a reasonable amount of time) guarantees will no longer hold if even a minority (one third) of validators collude.

[^3]: https://www.circle.com/blog/usdc-on-tron-high-speed-on-chain-value-transfers

This, and other factors, can affect the ability to even get a stablecoin transaction onto the chain. 
Miners or validators might collude to actively censor or prevent certain transactions, for example, one that is issued to upgrade the software of the smart contract to fix bugs. Or, miners or validators could prevent specific issuance, transfer, or withdrawal transactions. This doesn’t necessarily affect the stablecoin reserves, or backing, but could cause disruption and confusion about token ownership and prevent the stablecoin from operating effectively.

*Bugs*. All software has bugs. As a recent example, the blockchain Solana was down for 17 hours due to exploit of a resource exhaustion bug,[^4] meaning that for that period of time no one could settle any transactions on Solana, including Solana-issued stablecoin transactions (such as USDC). We wrote recently about addressing bugs and vulnerabilities in blockchain ecosystems. It is unclear whether users understand how to evaluate different blockchain developer communities for development best practices (including testing, detecting bugs, and deploying fixes).

[^4]: https://www.bloomberg.com/news/articles/2021-09-16/solana-network-of-top-10-sol-token-applies-fixes-after-outage

*Fees*. Decentralized blockchain networks charge transaction fees, which float according to market prices for blockchain space. The fee on Ethereum for a USDT ERC-20 transfer is currently $9.75 and for a USDC ERC-20 transfer is $10.25. Fees will rise if Ethereum transaction demand grows without a corresponding increase in transaction capacity. This might even unexpectedly render some stablecoins token accounts as unusable “dust”—of a value where it is no longer economically viable to transfer them, given the fees on the blockchain to do so. Fees also can spike temporarily, which might make it hard for users to predict when they will be able to afford to transfer their tokens.

# The stablecoin smart contract

On Ethereum and some other blockchains, each stablecoin has its own set of smart contracts. These contracts control the issuance, redemption, transfer, and blacklisting of the stablecoin tokens issued on this blockchain. These smart contracts are critical to tracking stablecoin ownership. There have been many cases of funds being frozen or hacked in smart contracts; as such there needs to be processes in place for auditing this software and understanding the upgrade and deployment process (note that this is a completely different task from auditing the composition of the backing reserves). There should be a recovery process plan in place for if the contract is hacked. 

# Minting and redemption

Stablecoin providers need to have a process for minting and redeeming stablecoins, usually connected to functionality in the smart contract, but requiring sensitive private keys. There are several risks and questions to address around the minting and redemption procedures, and the threat model behind them:

* Where are keys stored and what is the access control and authorization needed to access them (e.g. physical access, remote with password, OTP, some combination)?
* How many keys need to be compromised at once to compromise the processes, and how likely is that to occur? 
* What safeguards are in place, for example in-contract limiting of minting or redemption amounts over time? 
* How is recovery handled if the stablecoin provider loses access to the minting and redemption keys?

With all of these components, has the developer team run practice scenarios responding to bugs in the blockchain, smart contract, or minting and redemption processes? Might trading on exchanges need to be halted if compromise occurs, and is there a communication plan in place? Is there an estimate of the impact to the users of other smart contracts and financial instruments which use the stablecoin?

# In stablecoins, blockchains are just accounting

Stablecoins rely on some backing and a stablecoin provider to redeem stablecoins. Usually, in the redemption process, the provider gives other funds (for example, an ACH, wire, or Bitcoin transfer) to the person who can show they control the stablecoin token on the blockchain after they transfer it back to the provider. But if the blockchain and the stablecoin provider disagree about token ownership, who wins?  What responsibility does the stablecoin backing-holder (which might even be different from the provider) actually have to honor the state of the blockchain for redemptions?

Users need to understand how the stablecoin provider will mediate conflicting state—if there is disagreement (for example, due to an attack), who gets restitution, who doesn’t, and how is this determined? 

# Risk to the underlying blockchain from large stablecoins

Interestingly, the blockchain itself might be at risk if a stablecoin which runs on it becomes too big, especially compared to the monetary base of the blockchain’s native token. Stablecoin providers can have influence on the blockchain’s governance by only honoring the version of the stablecoin on one side of a blockchain fork, for example, even if it is not the side that most blockchain users prefer. Or, a stablecoin issuer could choose to honor tokens on either side in different ways. For example, they could honor the first token to be redeemed on either side and cancel the corresponding token on the other side. They could reject all transfers and redemptions during a time of uncertainty until a clear “victor” emerges. They could split the backing between the two forks.

Additionally, It is unclear how a very large stablecoin might affect the blockchain’s underlying security incentives, especially if miners (or stakers) are processing large stablecoin transactions but are rewarded (or slashed) in amounts denominated in the smaller, less valuable blockchain token. This might cause more incentive for miners or validators to conduct double spend attacks.

There is also a regulatory risk. If stablecoins grow large and are deemed systemically important, an argument could be made that the blockchains they run upon are also systemically important. This might increase attempts to regulate decentralized blockchain networks. 
 