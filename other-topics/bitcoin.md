---
title: Bitcoin
parent: Emerging Topics in Security
nav_order: 2
layout: page
header-includes:
  - \pagenumbering{gobble}
output:
  pdf_document:
    pandoc_args: ["--lua-filter=color-text-span.lua"]
---

# Bitcoin

## Motivation

Imagine you want to send money to a friend on the other side of the world. In today’s world, you would probably use a bank, a payment app, or a credit card company. These services work remarkably well, but they all have one thing in common: they rely on a _trusted central party_ to make the transaction happen.

What if we could build a digital currency that works _without_ needing to trust any single company or institution? This is the central idea behind Bitcoin.

In our everyday lives, physical cash (such as the Canadian or US dollar) and digital bank accounts allow us to store, send, and receive value. Bitcoin is a digital cryptocurrency, which means it should have all the same properties as physical currency. In our simplified model, a functioning currency should have the following properties:

- _Ownership_: Each person has a bank account, in which they can securely store the units of currency they own.
- _Authentication_: Only the rightful owner should be able to spend their money. No one else should be able to impersonate them and spend it instead.
- _Transferability_: Any two people should be able to engage in a _transaction_. For example, Alice should be able to send $$n$$ units of currency to Bob, decreasing her balance by $$n$$ and increasing Bob's balance by $$n$$.
- _Conservation (No Double-Spending)_: If Alice only has $$n$$ units, she must not be able to spend more than $$n$$ units (preventing double-spending).

In traditional financial systems, these properties are enforced by _trusted centralized intermediaries_, primarily banks, payment processors, and governments. When Alice sends money to Bob through her bank, both parties trust that the bank will correctly update their account balances, verify their identities, and prevent fraud. The bank maintains a private ledger that records all account balances and transactions.

While this model has worked for decades, it has significant limitations:

- _Single Point of Trust and Failure_: Users must fully trust the bank not to make mistakes, freeze accounts, or act maliciously. If the bank's systems are hacked or go offline, users lose access to their funds.
- _Censorship and Control_: Banks or governments can block transactions, freeze accounts, or deny service to individuals or groups.
- _Privacy Concerns_: Banks maintain detailed records of every transaction, often sharing data with authorities or third parties.
- _High Costs and Inefficiency_: Cross-border transfers can be slow (days) and expensive due to multiple intermediaries.
- _Financial Exclusion_: Billions of people worldwide do not have access to traditional banking services.
- _Systemic Risk_: The 2008 global financial crisis demonstrated the dangers of over-reliance on centralized financial institutions.

In 2008, an anonymous person or group using the pseudonym **Satoshi Nakamoto** published a whitepaper titled _Bitcoin: A Peer to Peer Electronic Cash System_. Bitcoin was the first practical solution that aimed to replicate the basic properties of a working currency system without relying on any centralized party. Instead of depending on trusted intermediaries, Bitcoin uses cryptography, a distributed ledger known as the _blockchain_, and a novel consensus mechanism called _Proof of Work_ to achieve these goals.

## 2. Cryptographic Building Blocks

Bitcoin is built on two cryptographic primitives that were introduced earlier in this book: cryptographic hash functions and digital signatures. We briefly review the properties that matter most for Bitcoin.

A cryptographic hash function \( H \) takes an input of arbitrary length and produces a fixed-length output \( H(x) \). The essential property is _collision resistance_: it is computationally infeasible to find two different inputs \( x \neq y \) such that \( H(x) = H(y) \). This lets us use a hash as a compact, tamper-evident fingerprint of data.

A digital signature scheme allows a user to prove they authorized a message. Each user holds a public verification key \( PK \) and a corresponding private signing key \( SK \). The user signs a message with \( SK \); anyone can verify the signature using \( PK \). The key security property is _unforgeability_: without knowledge of \( SK \), it is infeasible to produce a valid signature on a new message.

These two primitives let Bitcoin establish ownership and create verifiable, tamper-evident records without relying on any trusted central party. We now use them to construct identities and transactions.

## 3. Identities

In Bitcoin there is no central authority that issues or manages user accounts. Instead, every participant creates their own identity using public-key cryptography.

A user generates a key pair consisting of a public verification key \( PK \) and a corresponding private signing key \( SK \). The user’s identity on the Bitcoin network is simply their public key \( PK \). Anyone can generate as many such identities as they wish, with no registration or permission required.

To prove they control a particular identity, the user signs a message with their private key \( SK \). Anyone else can verify the signature using the corresponding public key \( PK \). Because digital signatures are unforgeable, an attacker who does not know \( SK \) cannot produce a valid signature and therefore cannot impersonate the owner of that identity.

This approach gives users full self-sovereignty over their identities while allowing the network to authenticate actions without any trusted intermediary.

## 4. Transactions

Without a central party to validate and record transfers, Bitcoin must let users prove cryptographically that they authorize a payment. This is achieved using digital signatures.

Suppose Alice wants to send \( n \) units of currency to Bob. She creates a transaction message that states her intent (for example, “\( PK_A \) sends \( n \) units to \( PK_B \)”) and signs it with her private key \( SK_A \). The signed transaction is then broadcast to the network.

Anyone can verify the signature using Alice’s public key \( PK_A \). A valid signature provides cryptographic proof that the holder of \( SK_A \) authorized the transfer. Because signatures are unforgeable, no one else can create a valid transaction on Alice’s behalf.

Note that the Bitcoin protocol itself does not check whether Bob wants to receive the funds. If Bob does not wish to accept the payment, he can simply create another transaction that sends the coins elsewhere.

## 5. Tracking Balances

So far we have a way for users to prove they authorize a transaction, but nothing yet prevents Alice from signing a transaction that spends more money than she actually owns. We need a mechanism to determine, for any proposed transaction, whether the sender has sufficient funds.

Bitcoin does not maintain explicit account balances like a traditional bank ledger. Instead, it records every transaction in a public, append-only history. Anyone can reconstruct the current state of ownership by scanning this history from the beginning. A user’s balance at any moment is simply the total value of all outputs they have received but not yet spent.

Consider a simple example. Assume Bob initially receives 10 BTC (we will explain where this initial amount comes from later). The following transactions are recorded in order:

- **TX₁**: Bob sends 5 BTC to Alice. (Signed by Bob)
- **TX₂**: Alice sends 3 BTC to Eve. (Signed by Alice)
- **TX₃**: Bob sends 2 BTC to Mallory. (Signed by Bob)

After these transactions, we can calculate everyone’s balance by tracing the flow of funds:

- Bob has spent 7 BTC total and still controls 3 BTC.
- Alice received 5 BTC and spent 3 BTC, so she controls 2 BTC.
- Eve controls 3 BTC.
- Mallory controls 2 BTC.

Now suppose Alice tries to create a new transaction sending 4 BTC to someone else. To validate this transaction, nodes perform three checks:

1. _Signature verification_: The transaction must be correctly signed by Alice’s private key (verifiable with her public key \( PK_A \)).
2. _Ownership of funds_: The funds Alice is trying to spend must have been previously sent to her in an earlier transaction (in this case, the 5 BTC from TX₁).
3. _No double-spending_: Those same funds must not have already been spent in a previous transaction. Here, Alice has only spent 3 BTC so far, so 2 BTC remain unspent.

If all three checks pass, the transaction is valid and can be added to the history.

At this point, we have created a functioning currency:

- Each person has a unique account, uniquely identified by public key.

- Users cannot impersonate other users, because each user can be validated by a secret signing key that only that user knows.

- Users can engage in a transaction by having the sender add their transaction to the ledger, with a signature on the transaction.

- Users cannot spend more than their current balance, because the trusted ledger is append-only, and everyone is able to calculate balances from the ledger.

However, this design still assumes the existence of a single, trusted, append-only ledger that everyone agrees on. Creating such a ledger in a fully decentralized network, where anyone can join and some participants may be malicious, is the remaining major challenge. We address it in the next section.

## 6. Hash Chains

We now have a way to record transactions and validate balances by scanning a public history. However, this only works if we can guarantee that the history itself cannot be altered after it is written. We need a public ledger that is _append-only_ and _immutable_: new entries can be added, but existing entries cannot be changed or deleted without detection.

Bitcoin achieves this using a structure called a **hash chain**.

Suppose we have a sequence of messages \( m_1, m_2, m_3, \ldots \) that we want to record. We organize them into blocks, where each block contains its data plus the cryptographic hash of the previous block:

| Block | Contents                                |
| ----- | --------------------------------------- |
| 1     | \( \text{Data}\_1 \)                    |
| 2     | \( \text{Data}\_2, H(\text{Block 1}) \) |
| 3     | \( \text{Data}\_3, H(\text{Block 2}) \) |
| 4     | \( \text{Data}\_4, H(\text{Block 3}) \) |

Because each block includes the hash of the block before it, the blocks become cryptographically linked. We can see this linkage clearly by expanding the hashes:

$$
\begin{aligned}
\text{Block 4} &= m_4,\ H(\text{Block 3}) \\
&= m_4,\ H(m_3,\ H(\text{Block 2})) \\
&= m_4,\ H(m_3,\ H(m_2,\ H(\text{Block 1}))) \\
&= m_4,\ H(m_3,\ H(m_2,\ H(m_1)))
\end{aligned}
$$

Block 4 now contains a compact cryptographic summary of every message that came before it (\( m_1 \) through \( m_4 \)).

This structure has a powerful security property: _tamper evidence_. If an attacker tries to modify any earlier message (for example, changing \( m_2 \)), the hash of Block 2 will change. This will cause the hash stored in Block 3 to no longer match, which will invalidate Block 4, and so on. Any change to the past breaks the chain and is immediately detectable by anyone who recomputes the hashes.

The hash chain therefore gives us a way to create a verifiable, append-only history. However, a remaining challenge remains: in a decentralized network, who is allowed to create and add new blocks? We address this next with Proof of Work.

## 7. Properties of Hash Chains

A key advantage of a hash chain is that it allows anyone to verify the integrity of the entire history while trusting only a small piece of information.

Suppose Alice receives the hash of the most recent block, \( H(\text{Block } i) \), from a trusted source. She can then download all previous blocks from any untrusted source (for example, a compromised server) and still verify that nothing has been altered.

She does this by recomputing the hash of each block and checking that it matches the hash stored in the next block, working forward until she reaches the final block. She then compares the computed hash of the last block against the trusted hash she received. If they match, the entire chain is valid.

**Example.** Alice receives \( H(\text{Block 4}) \) from a trusted source and downloads blocks 1 through 4 from an untrusted server. Suppose an attacker tries to give her a modified chain in which Block 2 has been changed to a different block \( 2' \).

Because cryptographic hashes are collision-resistant, \( H(\text{Block } 2') \neq H(\text{Block 2}) \). This causes Block 3 (which contains the hash of Block 2) to also be invalid, producing a different Block \( 3' \). The mismatch then propagates to Block 4, resulting in a different Block \( 4' \) whose hash does not match the trusted \( H(\text{Block 4}) \) that Alice received. Alice immediately detects the tampering.

In contrast, if the downloaded chain is unaltered, the final computed hash will match the trusted hash she was given.

Thus, the most important property of a hash chain is this: _if you obtain the hash of the latest block from a trusted source, you can independently verify the correctness of the entire preceding history, even if you downloaded it from completely untrusted sources_.

## 8. Consensus in Bitcoin

Because Bitcoin has no central server, every participant (node) independently stores and maintains a full copy of the blockchain. When a user creates a new transaction, they broadcast it to the network. Each node verifies the transaction according to the protocol rules and, if valid, adds it to its local copy of the blockchain.

This design creates a fundamental challenge: some participants may be malicious or faulty. A malicious node might ignore certain transactions, include invalid ones, or attempt to rewrite history. Bitcoin does not assume that all nodes are honest. Instead, **Bitcoin assumes that the majority of computational power in the network is controlled by honest participants**.

One of the most important problems that arises is _forks_. A fork occurs when two valid but conflicting blocks are created at roughly the same time, resulting in two different versions of the blockchain.

For example, say that Mallory bought a house from Bob for 500 $$B$$, and this transaction is appended to the ledger. Mallory can then try “go back in time” and start the blockchain from just before this transaction was added to it, and can start appending new transaction entries from there. If Mallory can get other users to accept this new forked chain, she can get her 500 $$B$$ back!

This raises a critical question: in a decentralized network where anyone can propose new blocks and some participants may be dishonest, how do all honest nodes agree on a single, consistent version of the blockchain? Bitcoin solves this problem using a mechanism called **Proof of Work**, which we examine next.

## 9. Consensus via Proof of Work

In Bitcoin, any node can validate transactions, but only special nodes called **miners** are allowed to create and append new blocks to the blockchain. To add a block, a miner must solve a computationally difficult puzzle known as **Proof of Work**.

### The Proof-of-Work Puzzle

To create a new block, a miner collects valid pending transactions and constructs a candidate block header. The miner then repeatedly tries different values of a field called the **nonce** until the hash of the block header satisfies a specific condition: it must begin with a required number of leading zeros.

Because cryptographic hashes are unpredictable, the only way to find a valid nonce is through brute-force trial and error. The number of leading zeros required (the _difficulty_) is adjusted automatically by the network approximately every two weeks to maintain an average block interval of 10 minutes.

Solving this puzzle is expensive in terms of computation and electricity, but once a valid solution is found, it is trivial for any other node to verify.

### Reaching Consensus: The Longest Chain Rule

When a miner finds a valid block, they broadcast it to the network. Other nodes verify the block (checking all transactions and the Proof of Work) and, if valid, append it to their local chain.

The key consensus rule is simple: **nodes always accept the valid chain with the most cumulative Proof of Work** (in practice, the longest chain). When temporary forks occur, because two miners happen to find valid blocks at roughly the same time, honest miners will extend whichever chain they see as longest. Over time, the chain that grows faster (supported by the majority of mining power) will become the accepted one, and shorter competing chains are abandoned.

**Example.** Suppose the current chain ends at block \( b_3 \). Miner \( M_1 \) and Miner \( M_2 \) both find valid blocks (\( b_4 \) and \( b_4' \)) at nearly the same time, creating a fork. The next miner to solve a block will extend one of these chains. Whichever chain receives the next block first becomes longer and is more likely to be extended further. Eventually, one chain will pull ahead, and honest nodes will switch to it, discarding the shorter fork.

Bitcoin’s security rests on the assumption that the majority of the network’s total computational power (hash rate) is controlled by honest miners. Under this assumption, honest miners will, on average, find blocks faster than any attacker. An attacker who wants to create a longer alternative chain (for example, to reverse a transaction) would need to control more than 50% of the total hash rate, a so-called _51% attack_, which is extremely expensive and difficult to sustain.

This economic reality protects the system: rewriting recent history or sustaining a malicious fork requires out-computing the rest of the honest network, which becomes prohibitively costly as the chain grows.

By combining Proof of Work with the longest-chain rule, Bitcoin achieves decentralized consensus on the order and validity of transactions without relying on any trusted central authority.

## 10. Mining and Economic Incentives

In the previous section, we saw how Proof-of-Work and the longest chain rule allow the Bitcoin network to reach consensus. However, for this system to function, someone must actually perform the work of creating new blocks. This role is played by **miners**. Mining is an extremely computationally intensive process that requires specialized hardware on a massive scale.

A typical modern personal computer (PC) or laptop can perform roughly _10 to 50 million hashes per second_ (MH/s) when trying to mine Bitcoin. Even a powerful gaming PC with a high-end graphics card might reach around _50–100 million hashes per second_. A single modern Bitcoin mining machine (called an _ASIC_) can do _200–500 terahashes per second_ (TH/s), which is already thousands of times more powerful than a regular PC.

As of 2026, the entire Bitcoin network performs roughly _900–980 exahashes per second_ (EH/s). One exahash per second equals _1,000,000,000,000,000,000_ (one quintillion) hashes per second. This means the Bitcoin network as a whole is performing roughly _20 billion times_ more hash calculations per second than a good personal computer. Controlling just over 50% of the network’s power (required for a successful 51% attack) would mean an attacker would need to command roughly _450 to 500 exahashes per second_, equivalent to running a pool of roughly _1 to 2.5 million_ high-end mining machines, plus the enormous electricity required to power them.

Miners perform several important tasks:

1. They listen for new transactions broadcast across the Bitcoin network and collect them into a pool of unconfirmed transactions (often called the _mempool_).

2. They verify that each transaction is valid. This includes checking that:

   - The transaction is properly signed.
   - The inputs reference unspent outputs.
   - The transaction does not attempt to spend more bitcoin than is available.

3. They assemble a candidate block containing a selection of valid transactions from the mempool.

4. They attempt to solve the Proof-of-Work puzzle for their candidate block by trying many different nonce values until they find one that produces a hash meeting the current difficulty target.

5. Once they find a valid solution, they broadcast the completed block to the rest of the network.

Other nodes then verify the block and, if it is valid, add it to their copy of the blockchain. The miner who created the block is rewarded for their work.

### Block Rewards and Transaction Fees

Miners are compensated in two ways:

**Block Reward**  
When a miner successfully creates a new block, they are allowed to include a special transaction called the _coinbase transaction_. This transaction creates a predetermined amount of new bitcoin and sends it to an address controlled by the miner. This is how new bitcoin enters circulation.

The block reward started at 50 BTC in 2009 and is halved every 210,000 blocks (approximately every four years). As of 2027, the block reward is 3.125 BTC. This halving process continues until the total supply reaches 21 million bitcoin, which is expected to occur around the year 2140. After that, no new bitcoin will be created, and miners will be compensated only through transaction fees.

**Transaction Fees**  
In addition to the block reward, miners also collect the difference between the total value of the inputs and outputs in the transactions they include in their block. This difference is known as the _transaction fee_.

Users can attach higher fees to their transactions to encourage miners to include them more quickly, especially during periods of high network activity. Over time, as the block reward decreases due to halving events, transaction fees are expected to become an increasingly important part of miners’ revenue.

### Why Miners Follow the Rules

Bitcoin’s security depends heavily on the assumption that miners will generally act in their own economic self-interest. Several factors encourage miners to follow the protocol rules:

- If a miner includes invalid transactions in their block, other nodes will reject the block. The miner will lose both the block reward and the transaction fees.

- If a miner attempts to create a block that extends a shorter chain (instead of the longest chain), their block is less likely to be accepted by the rest of the network. They risk wasting their mining effort.

- Creating an invalid block or attacking the network (for example, by trying to reverse recent transactions) would likely damage confidence in Bitcoin. Since miners have invested heavily in hardware and electricity, they have a strong financial incentive to maintain the value and stability of the currency.

In short, under normal conditions, it is more profitable for miners to behave honestly and extend the longest valid chain than to attack the system.

### Difficulty Adjustment

One of the most important features of Bitcoin’s mining system is its automatic _difficulty adjustment_ mechanism.

The network adjusts the difficulty of the Proof-of-Work puzzle every 2016 blocks (roughly every two weeks). The goal is to maintain an average block interval of approximately _10 minutes_, regardless of how much total computational power (hash rate) exists in the network.

- If miners are producing blocks faster than expected (because more hash rate has joined the network), the difficulty increases.
- If blocks are being produced too slowly (because some miners have left), the difficulty decreases.

This adjustment makes Bitcoin’s block production rate relatively stable and predictable, even as the total mining power in the network changes dramatically over time.

### Costs and Trade-offs of Proof-of-Work

While Proof-of-Work provides strong security guarantees, it comes with significant costs and trade-offs.

The most obvious cost is _energy consumption_. Because miners must perform large amounts of computation to find valid blocks, Bitcoin mining uses substantial amounts of electricity. Critics argue that this energy usage is wasteful and environmentally harmful.

On the other hand, the high cost of mining is also what makes Bitcoin secure. Attacking the network (such as attempting a 51% attack) requires controlling more computational power than the honest network, which is extremely expensive. In this sense, the energy expenditure can be viewed as the price paid for decentralized security.

Other important trade-offs include:

- _Centralization pressures_: Mining has become dominated by large operations with access to cheap electricity and specialized hardware (ASICs). This has led to concerns about mining pool centralization.
- _Hardware specialization_: General-purpose computers are no longer competitive for Bitcoin mining, raising the barrier to entry for new participants.
