# Analyzing the Ethereum NFT Market using on-chain data

## Authors

- [Andrei Dolgolev](https://github.com/Andrei-Dolgolev)
- [Neeraj Kashyap](https://github.com/zomglings)
- [Tim Pechersky](https://github.com/peersky)
- [Yhtyyar Sahatov](https://github.com/yhtiyar)
- [Sergei Sumarokov](https://github.com/kompotkot)

## Abstract
We present an analysis of the Ethereum non-fungible token (NFT) market between April 1, 2021 and
September 25, 2021 based purely on on-chain data. The analysis focuses on: (1) classifying NFT contracts
based on the distribution of value over their tokens, (2) classifying NFT owners based on their
purchasing and holding characteristics.

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

## Dataset

The majority of recent NFT hype has been centered around the Ethereum blockchain. We will therefore
restrict our analysis to NFT operations on the Ethereum blockchain.

Anyone interested in creating non-fungible tokens is free to implement their tokens in any
manner whatsoever \emph{as long as their implementation satisfies the non-fungibility condition}.
The most common implementation follows [the Ethereum ERC721 protocol](https://eips.ethereum.org/EIPS/eip-721).

There is real value in following a standard such as ERC721. There is a growing ecosystem of secondary protocols
that only allow NFTs which follow these standards to participate. Secondary markets for NFTs, such as
[Nifty Gateway](https://niftygateway.com) and [OpenSea](https://opensea.io), make the fundamental operational assumption that
the tokens they list follow these standards. As a consequence, ERC721-compliant tokens account for the overwhelming
majority of Ethereum NFTs.

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

These ABIs allow us to identify whenever we encounter an Ethereum transaction which resulted in a
`Transfer` event being emitted.

Our dataset is a single [SQLite](https://sqlite.org) database.

The core of our dataset is all the `Transfer` events that we crawled from the Ethereum blockchain
between April 1, 2021 and September 25, 2021. We have partitioned these events into two tables -- `mints` and `transfers`.
A mint represents the creation of an NFT and is signified by a `Transfer` event with a zero `from` address.
A transfer represents the transfer of ownership of an NFT and is signified by a `Transfer` event with a
*non-zero* `from` address.

These tables have the following schemas:

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

There are 6,687,719 `mints` and 4,533,252 `transfers` in our dataset.

<!-- Smart contracts that comply with ERC721 write a

Date range: April 1, 2021 to September 25, 2021.

NFT means ERC721.

\section{Distribution of transfers}

Exponential v.s. Zipfian

blah blah

\bibliographystyle{amsplain}
\bibliography{nfts}

\end{document} -->
