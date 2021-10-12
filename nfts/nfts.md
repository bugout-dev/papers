# Analyzing the Ethereum NFT Market using on-chain data

## Authors

- [Sophia Arakelyan](https://github.com/SophiaAr)
- [Andrei Dolgolev](https://github.com/Andrei-Dolgolev)
- [Neeraj Kashyap](https://github.com/zomglings)
- Nana Landau
- Daria Navoloshnikova
- [Tim Pechersky](https://github.com/peersky)
- [Yhtyyar Sahatov](https://github.com/yhtiyar)
- [Sergei Sumarokov](https://github.com/kompotkot)

## Abstract

We present the Ethereum NFTs dataset, a representation of the activity on the Ethereum non-fungible
token (NFT) market between April 1, 2021 and September 25, 2021 constructed purely from on-chain data.

We then consider the following questions:
1. How decentralized is NFT ownership on the Ethereum blockchain?
2. How should one measure the utility of a non-fungible token?

We observe from the data that the distribution of the number of NFTs owned by Ethereum addresses is
exponential in nature. This indicates that the NFT market is indeed extremely decentralized, and not
concentrated in the accounts of a relatively small number of owners.

We also use the data to propose an entropy-based measure of utility for NFT contracts -- their
**ownership entropy**.

## Introduction

A non-fungible token, or NFT, is a blockchain asset which serves as a distinct representation on the blockchain
of some object or concept. It is non-fungible in the sense that it immutably and exclusively represents that object or concept.
It will never change what it represents and the represented object or concept admits no \emph{other} representation
on the blockchain.

The global market for NFTs has seen a massive boom over the past 3 months. Visual artists and other content
creators are digitizing their creations as NFTs to distribute their work to patrons. Game producers are
tokenizing assets in computer and mobile games as NFTs to create shared worlds with \emph{other} content
creators.

Conventional digital representations of physical works can be replicated aribtrarily and indefinitely.
For example, if you own a digital copy of a book, you could in principle make arbitrarily many clones
of that copy and distribute it to anyone who asked for it.

In contrast, NFTs naturally reflect the scarcity of the objects they represent. This essential scarcity
makes NFTs a perfect tool to globally conserve value when transferring ideas and assets from one digital reality
to another. NFTs allow people to create common representations of scarce resources across multiple realities.

Given the recent boom in the NFT market, there is a wild variance in utility across the NFTs that are being
released. Similarly, there is a growing number of first-time NFT buyers. This paper analyzes these variances
and builds profiles that can be used to classify NFTs and NFT purchasers.

## The Ethereum NFTs Dataset

The majority of recent NFT hype has been centered around the Ethereum blockchain. We will therefore
restrict our analysis to NFT operations on the Ethereum blockchain.

### Contracts, tokens, and events

A non-fungible token is a digital asset representing a distinct idea or physical object.

On the Ethereum blockchain, these tokens are created using [Ethereum smart contracts](https://ethereum.org/en/developers/docs/smart-contracts/)
which represent entire collections of non-fungible tokens. The most famous examples of such contracts
are [CryptoPunks](https://www.larvalabs.com/cryptopunks) and [CryptoKitties](https://www.cryptokitties.co/).

Every time an NFT is transferred on the Ethereum blockchain, it emits a `Transfer` event which is stored
on the blockchain. This dataset was constructed by crawling the emitted Transfer events that were emitted
in the period of time represented by the dataset.

### ERC721

Anyone interested in creating non-fungible tokens is free to implement their tokens in any
manner whatsoever \emph{as long as their implementation satisfies the non-fungibility condition}.
The most common implementation follows [the Ethereum ERC721 protocol](https://eips.ethereum.org/EIPS/eip-721).

There is real value in following a standard such as ERC721. There is a growing ecosystem of secondary protocols
that only allow NFTs which follow these standards to participate. Secondary markets for NFTs, such as
[Nifty Gateway](https://niftygateway.com) and [OpenSea](https://opensea.io), make the fundamental operational assumption that
the tokens they list follow these standards. As a consequence, ERC721-compliant tokens account for the overwhelming
majority of Ethereum NFTs.


### Technical details

Smart contracts that comply with the ERC721 standard store the following events into the global Ethereum
state every time a token is transferred from one party to another:

```
/// @dev This emits when ownership of any NFT changes by any mechanism.
///  This event emits when NFTs are created (`from` == 0) and destroyed
///  (`to` == 0). Exception: during contract creation, any number of NFTs
///  may be created and assigned without emitting Transfer. At the time of
///  any transfer, the approved address for that NFT (if any) is reset to none.
event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
```

These `Transfer` events can have one of the two following application binary interfaces (ABIs):
```json
[
    {
        "anonymous": false,
        "inputs": [
            {"indexed": false, "name": "from", "type": "address"},
            {"indexed": false, "name": "to", "type": "address"},
            {"indexed": false, "name": "tokenId", "type": "uint256"},
        ],
        "name": "Transfer",
        "type": "event"
    },
    {
        "anonymous": false,
        "inputs": [
            {"indexed": true, "name": "from", "type": "address"},
            {"indexed": true, "name": "to", "type": "address"},
            {"indexed": true, "name": "tokenId", "type": "uint256"},
        ],
        "name": "Transfer",
        "type": "event"
    }
]
```

These ABIs allow us to identify when we encounter an Ethereum transaction which emitted `Transfer` event.

The core of our dataset is all the `Transfer` events that we crawled from the Ethereum blockchain
between April 1, 2021 and September 25, 2021.

Our dataset is a single [SQLite](https://sqlite.org) database. The dataset was crawled using
[Moonstream](https://moonstream.to), an open source blockchain analytics tool. The code is available
on GitHub: https://github.com/bugout-dev/moonstream.

#### Core relations

We have partitioned the `Transfer` events we crawled into two relations -- `mints` and `transfers`.
A mint represents the creation of an NFT and is signified by a `Transfer` event with a zero `from` address.
A transfer represents the transfer of ownership of an NFT and is signified by a `Transfer` event with a
*non-zero* `from` address.

These tables share the same schema:

<!-- schema_mints -->
```
.schema mints
sqlite> .schema mints
CREATE TABLE mints
    (
        event_id TEXT NOT NULL UNIQUE ON CONFLICT FAIL,
        transaction_hash TEXT,
        block_number INTEGER,
        nft_address TEXT REFERENCES nfts(address),
        token_id TEXT,
        from_address TEXT,
        to_address TEXT,
        transaction_value INTEGER,
        timestamp INTEGER
    );
```

<!-- schema_transfers -->
```
.schema transfers
sqlite> .schema transfers
CREATE TABLE transfers
    (
        event_id TEXT NOT NULL UNIQUE ON CONFLICT FAIL,
        transaction_hash TEXT,
        block_number INTEGER,
        nft_address TEXT REFERENCES nfts(address),
        token_id TEXT,
        from_address TEXT,
        to_address TEXT,
        transaction_value INTEGER,
        timestamp INTEGER
    );
```

The columns are as follows:
- `event_id` - a unique ID associated with each event, generated when we created the dataset
- `transaction_hash` - the hash of the [Ethereum transaction](https://ethereum.org/en/developers/docs/transactions/) in which we observed the event
- `block_number` - the number of the [Ethereum transaction block](https://ethereum.org/en/developers/docs/blocks/) in which the transaction containing the event was mined
- `nft_address` - the address of the smart contract containing the NFT that the event describes
- `token_id` - the identifier that represents the NFT that the event describes within the context of the smart contract with address `nft_address`
- `from_address` - the address that owned the NFT *before* the `Transfer` event denoted by this row in the relation (it is **not** the address that initiated the transaction in which the NFT transfer took place)
- `to_address` - the address that owned the NFT *after* the `Transfer` event denoted by this row in the relation (it is **not** the address that was the recipient the transaction in which the NFT transfer took place)
- `transaction_value` - the amount of WEI that were sent with the transaction in which the `Transfer` event took place
- `timestamp` - the Unix timestamp at which the Ethereum transaction block with number `block_number` was mined into the Ethereum blockchain

There are 6,667,282 `mints` and 4,514,729 `transfers` in our dataset. Together, these events represent
the activity of 9,292 NFT contracts which were actively used on the Ethereum blockchain **between
April 1, 2021 and September 25, 2021**.

#### Derived relations

For ease of analysis, we have also included several derived relations in the Ethereum NFTs dataset:
- `nfts` - available metadata (from [Moonstream](https://moonstream.to)) about the NFT contracts represented in the dataset
- `current_market_values` - the current (estimated) market value of each NFT, in WEI
- `current_owners` - the current owner of each NFT
- `transfer_statistics_by_address` - transfers in and out of every address that was involved in an NFT transfer
- `transfer_values_quartile_10_distribution_per_address` and `transfer_values_quantile_25_distribution_per_address` - for each 10th or 4th quantile of each NFT collection, these relations give the proportion of the value of the most valuable token in that collection that the quantile represents

Here, "current" means "as of September 25, 2021".

The analysis in this paper will make use of `nfts`, `current_market_values`, `current_owners`, and `transfer_statistics_by_address`.

### Caveats

The Ethereum NFTs dataset is constructed purely from events on the Ethereum blockchain. It does not
include any data from Layer 2 networks like Polygon. Nor does it include any data from centralized
APIs like the OpenSea API. It does not account for events or data from any non-ERC721 smart
contracts associated with these platforms on the Ethereum blockchain.

### Accessing the dataset

The complete Ethereum NFTs dataset is available on Kaggle under a creative commons license: https://www.kaggle.com/simiotic/ethereum-nfts

If you would like access to this dataset and it is not convenient to access it from Kaggle, you can discuss
it with us on the [Moonstream community Discord](https://discord.gg/K56VNUQGvA) or email [Neeraj Kashyap](mailto:neeraj@moonstream.to).

If you use this dataset in your own research, please use the following citation:

```
Sophia Arakelyan et al., “Ethereum NFTs.” Kaggle, 2021, doi: 10.34740/KAGGLE/DSV/2698517.
```
