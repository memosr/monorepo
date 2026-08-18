---
title: "Hush Money"
description: "Every private payment scheme ever built pays for privacy in one of three currencies: trusted parties, small anonymity sets, or scanning and state that grow with the chain. We map more than four decades of the literature into one DAG and trace the arcs bending toward the dream."
date: "August 18th, 2026"
published-time: "2026-08-18T00:00:00Z"
modified-time: "2026-08-18T00:00:00Z"
author: "Guru Vamsi Policharla"
author_twitter: "https://x.com/guruvamsip"
url: "https://commonware.xyz/blogs/private-payments"
image: "https://commonware.xyz/imgs/private-payments.png"
katex: true
---

In [earlier](/blogs/bte) [posts](/blogs/pick-your-poisons) we covered encrypted mempools, which protect a transaction only until it is finalized. Private payments are the harder problem: the origin, destination, and amount of a transfer must stay hidden *forever* -- from the public, from the validators executing it, and from counterparties beyond what their own view already reveals -- while everyone can still verify that no value was created and nothing was spent twice. [Zerocash](https://eprint.iacr.org/2014/349) named the three properties that a decentralized anonymous payment (DAP) scheme must provide: *ledger indistinguishability*, *transaction non-malleability*, and *balance*.

More than four decades of constructions -- Chaumian e-cash, accumulator coins, shielded pools, ring signatures, encrypted account balances, regulated CBDCs -- form a single family tree, and every branch pays for privacy in one of three currencies. We map the tree below, then walk the bill.

```{=html}
<style>
  .cw-magic-move {
    --cw-magic-track-height: 1200vh;
    height: var(--cw-magic-track-height);
    margin: 44px 0 20px;
    position: relative;
  }

  .cw-magic-final {
    aspect-ratio: 1024 / 700;
    margin: 20px 0;
  }

  .cw-magic-story-source,
  .cw-magic-dream-source {
    display: none;
  }

  @media (max-width: 600px) {
    .cw-magic-move {
      --cw-magic-track-height: 1150vh;
    }
  }
</style>
<div id="private-payments-magic-move" class="cw-magic-move" aria-label="Animated tour through the private payments landscape, from Chaumian e-cash to the three poisons every scheme picks from." role="region">
  <noscript>
    <style>
      .cw-magic-move {
        height: auto;
        margin: 0;
      }

      .cw-magic-final {
        aspect-ratio: auto;
      }

      .cw-magic-story-source,
      .cw-magic-dream-source {
        display: block;
      }
    </style>
    This section contains an animated tour through the private payments landscape, from Chaumian e-cash to the three poisons every scheme picks from.
  </noscript>
</div>
<script type="module" src="private-payments.magic-move.js"></script>
```

::: {#private-payments-story-source .cw-magic-story-source}
Private payments are as old as the field. [Cha82](https://link.springer.com/chapter/10.1007/978-1-4757-0602-4_18) introduced blind signatures so a bank can sign a coin it never sees, making withdrawals unlinkable from deposits at the cost of an online double-spending check. [CFN88](https://chaum.com/wp-content/uploads/2021/12/Untraceable_Electronic_Cash.pdf) took spending offline by secret-sharing the payer's identity into every coin: one spend reveals nothing, two spends reveal the double-spender. [CHL05](https://eprint.iacr.org/2005/060) compressed a wallet of $2^{\ell}$ coins into a single CL signature -- coin $i$ is spent by revealing a serial number $F_s(i)$ and a double-spending tag, both PRF outputs proven correct in zero knowledge. The PRF-derived serial number is the ancestor of every nullifier that follows.

[MGGR13](https://isi.jhu.edu/~mgreen/ZerocoinOakland.pdf) removed the bank. A zerocoin is a Pedersen commitment to a serial number, minted into a public RSA accumulator and spent by revealing the serial number with a zero-knowledge proof of accumulator membership. The membership proof hides which coin is being redeemed; the revealed serial number replaces the bank's list of deposited coins. But spend proofs exceeded 45 KB, denominations were fixed, and recipients and amounts stayed public -- a decentralized mix, not yet a payment system.

[BCGGMTV14](https://eprint.iacr.org/2014/349) formalized *decentralized anonymous payment (DAP) schemes* and gave the construction that still defines the space. Coins are commitments in a Merkle tree; a *pour* transaction consumes up to two coins and creates up to two new ones, revealing only serial numbers $\mathsf{sn} = \mathsf{PRF}_{a_{\mathsf{sk}}}(\rho)$ and a zk-SNARK proving membership, correct derivation, and value conservation. Pours are under 1 kB and verify in under 6 ms, and the anonymity set is every coin ever created. Three costs are baked in: the SNARK needs a trusted setup, recipients must trial-decrypt the entire chain, and the nullifier set grows forever.

Zcash deployed the DAP scheme and spent a decade paying down its debts. Sprout's BCTV14 parameters harbored a counterfeiting vulnerability (CVE-2019-7167) found by [Gab19](https://eprint.iacr.org/2019/119); [Sapling](https://zips.z.cash/protocol/protocol.pdf) moved to Groth16 with diversified payment addresses and viewing keys; Orchard eliminated the ceremony entirely by moving to Halo 2 -- descended from [Halo](https://eprint.iacr.org/2019/1021)'s recursive proof composition -- whose inner-product-argument polynomial commitment needs no trusted setup. The deployed anonymity set proved smaller than the definitional one: round-trip heuristics linked 31.5% of coins entering the pool ([Que17](https://arxiv.org/abs/1712.01210), [KYMM18](https://www.usenix.org/conference/usenixsecurity18/presentation/kappos)), and the PING and REJECT attacks ([TBP20](https://eprint.iacr.org/2020/220)) identified payees remotely from trial-decryption timing.

A parallel thread hid only amounts. Confidential Transactions ([Max15](https://www.weusecoins.com/confidential-transactions/)) replace values with Pedersen commitments plus range proofs, and [Bulletproofs](https://eprint.iacr.org/2017/1066) shrank a 64-bit range proof to 672 bytes ($2\log_2{n} + 9$ group and field elements) with no trusted setup. Mimblewimble ([Jed16](https://docs.beam.mw/Mimblewimble.pdf), formalized in [Poe16](https://download.wpsoftware.net/bitcoin/wizardry/mimblewimble.pdf)) pushed CT to its logical end -- a scriptless ledger of commitments and transaction kernels where *cut-through* prunes spent outputs -- and [FOS19](https://eprint.iacr.org/2018/1039) formalized it as an *aggregate cash system* with inflation resistance and resistance to coin theft. Amounts vanish; the transaction graph does not.

[CryptoNote](https://bytecoin.org/old/whitepaper.pdf) fixed Bitcoin's two leaks with one-time addresses for recipients and one-time ring signatures for senders, publishing a *key image* to catch double spends -- the [LSAG](https://eprint.iacr.org/2004/027) tag in disguise. RingCT ([Noe16](https://eprint.iacr.org/2015/1098)) layered Pedersen commitments and commitments-to-zero into the MLSAG signature so amounts, origins, and destinations are all hidden; [CLSAG](https://eprint.iacr.org/2019/654) halved it. But signatures are linear in ring size, so Monero's rings froze at 11, then 16 -- and zero-mixin chain reactions ([MSH+18](https://petsymposium.org/popets/2018/popets-2018-0025.php)), guess-newest age heuristics, and the March 2024 black-marble flood ([Ruc24](https://rucknium.me/html/monero-black-marble-flood.pdf)) showed how cheaply a fixed decoy set is degraded, or simply bought.

The sub-linear line descends from one-out-of-many proofs ([GK15](https://eprint.iacr.org/2014/764)): a $\Sigma$-protocol showing that one of $N$ commitments opens to zero, with $O(\log{N})$ proof size. Lelantus ([Jiv19](https://eprint.iacr.org/2019/373)) built burn-and-redeem payments on it; Omniring ([LRRSTW19](https://eprint.iacr.org/2019/580)) gave RingCT formal foundations with log-size proofs and a single ring shared by all inputs; Triptych ([NG20](https://eprint.iacr.org/2020/018)) made linkable ring signatures logarithmic. Every descendant hit the same wall: verification linear in the anonymity set, capping deployed sets near $2^{16}$.

Veksel ([CH22](https://eprint.iacr.org/2021/327)) showed the way around the wall: build payments from a proof that a homomorphic commitment is a *rerandomization* of a commitment from a set, without revealing which one. Curve trees ([CHK23](https://eprint.iacr.org/2022/756)) scaled the idea into a transparent zero-knowledge accumulator -- shallow Merkle trees of Pedersen commitments over a 2-cycle of elliptic curves, proved with *select-and-rerandomize* relations in Bulletproofs-style circuits -- giving membership proofs of a few KB for sets beyond $2^{32}$. [FCMP++](https://gist.github.com/kayabaNerve/0e1f7719e5797c826b87249f21ab6f86) retrofits this onto Monero's existing outputs using Generalized Bulletproofs and Eagen's divisor technique ([Eag22](https://eprint.iacr.org/2022/596)) while preserving legacy key images: the anonymity set becomes the entire chain, ending the decoy era.

The account model answered differently. Quisquis ([FMMO19](https://eprint.iacr.org/2018/990)) gives accounts *updatable public keys*: a transaction consumes sender, receiver, and decoys and re-emits all of them re-randomized, so each address appears on the blockchain at most twice and there is no ever-growing set of unspent outputs -- at the price of proofs linear in the decoy set and races on overlapping decoys. Zether ([BAZB20](https://eprint.iacr.org/2019/191)) keeps balances as exponent-ElGamal ciphertexts inside a smart contract and tames concurrency with *epochs*, *pending transfers*, and *rollover*, proving statements over ciphertexts with $\Sigma$-Bullets; many-out-of-many proofs ([Dia21](https://eprint.iacr.org/2020/293)) made its anonymous variant logarithmic in the ring of accounts. The same twisted ElGamal machinery ([CMTA19](https://eprint.iacr.org/2019/319)) now runs Solana's confidential transfers -- whose 2025 Fiat-Shamir "phantom challenge" bug is a reminder of where $\Sigma$-protocol stacks go to die.

Permissioned designs keep the trust topology of existing finance and make it verifiable. Solidus ([CZJKJS17](https://eprint.iacr.org/2017/317)) hides the transaction graph inside a publicly-verifiable oblivious RAM machine (PVORM) per bank; zkLedger ([NVV18](https://eprint.iacr.org/2018/241)) writes a commitment entry for *every* bank into every row of the ledger so no bank can omit transactions from an audit, and audit tokens let an auditor extract provably complete answers.

The CBDC wave modernizes Chaum with threshold trust. UTT ([TBAAGPY22](https://eprint.iacr.org/2022/452)) thresholdizes the bank, auditor, and registrar over BFT infrastructure and prices anonymity itself: *anonymity budgets* cap the value a user can send privately per month. Platypus ([WKDC22](https://eprint.iacr.org/2021/1443)) combines e-cash's transaction processing with account commitments so holding and receiving limits become extra ZK conjuncts on each state transition; PEReDi ([SKK22](https://eprint.iacr.org/2022/974)) distributes the operator into a quorum of maintainers with threshold tracing and privacy revocation, proven in the UC framework. The same machinery extends outward: PARScoin ([SKK23](https://eprint.iacr.org/2023/1908)) to fiat-backed stablecoins, and DART ([SKKK25](https://eprint.iacr.org/2025/239)) to anonymous multi-asset tokenization with asset-specific auditors whose identities themselves stay hidden.

Compliance for permissionless pools split on one question: does anyone hold a de-anonymization trapdoor? [GGM16](https://eprint.iacr.org/2016/061) put policies -- coin tracing, user tracing, spending limits, tax payments -- inside the Zerocash pour circuit, decryptable by an authority. After Tornado Cash was sanctioned, the trapdoor-free line won deployment: proofs of innocence, then Privacy Pools ([BINSS23](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4563364)), where each withdrawal carries a membership or exclusion proof against a curated *association set*, and Derecho's proof-carrying disclosures ([BF23](https://eprint.iacr.org/2023/273)) propagate such attestations through the pool's internal transaction graph.

Someone still has to notice they got paid. Trial decryption costs $O(\mathsf{chain})$ per wallet and leaks timing; view tags shave constants. Fuzzy message detection ([BLMG21](https://eprint.iacr.org/2021/089)) outsources detection to an untrusted server at a tunable false-positive rate $p$ -- statistically brittle under intersection attacks ([SPB21](https://eprint.iacr.org/2021/1180)). Oblivious message retrieval ([LT22](https://eprint.iacr.org/2021/1256)) gets full privacy from a single untrusted detector that homomorphically packs pertinent messages into an encrypted digest, at $\Theta(N)$ FHE cost. Tachyon ([Bow25](https://seanbowe.com/blog/tachyon-scaling-zcash-oblivious-synchronization/), formalized in [BM25](https://eprint.iacr.org/2025/2031)) exits the game entirely: out-of-band payments, wallet state carried as proof-carrying data, *evolving nullifiers*, and *oblivious synchronization* make wallet cost proportional to the wallet's own activity rather than global chain activity.
:::

::: {#private-payments-dream-source .cw-magic-dream-source}
Every corner of the map pays in at least one of three currencies: trusted parties, small anonymity sets, or scanning and state that grow with the chain.

[The dream]{#dream-goal} is:

::: {data-align="center"}
> *A private payment scheme with an anonymity set equal to the whole chain, no trusted issuer or setup, recipient cost proportional to a wallet's own activity, bounded validator state, concurrent account-style usage, and accountability without a de-anonymization trapdoor -- ideally against a quantum adversary*
:::

No construction gets all of it. But three arcs are bending toward it.
:::

## Poison One: Trusted Parties

The Chaumian line trusts a mint. Blindness means the bank *cannot* trace -- but blind issuance also means over-issuance is not publicly auditable, which is exactly the failure Zerocoin's public validity proofs were built to remove. The modern permissioned answer is to thresholdize the authority rather than delete it: [UTT](https://eprint.iacr.org/2022/452) splits bank, auditor, and registration across a BFT committee, [PEReDi](https://eprint.iacr.org/2022/974) requires a quorum of maintainers for tracing or revocation, and [RP23](https://petsymposium.org/popets/2023/popets-2023-0116.pdf)-style threshold issuance decentralizes the mint itself. The governance of those key shares -- not the cryptography -- becomes the real policy question.

Permissionless designs smuggle the trusted party back in through the proof system. Sprout's ceremony ran with six participants; its BCTV14 CRS carried redundant elements that enabled proofs "for any public input" -- silent, unbounded counterfeiting ([Gab19](https://eprint.iacr.org/2019/119)), fixed by new Groth16 parameters at Sapling, with a turnstile bounding any counterfeit already in the old pool. The [Powers of Tau](https://eprint.iacr.org/2017/1050) MPC scaled trust to 1-of-$n$ over ~90 contributors, and [Halo](https://eprint.iacr.org/2019/1021) removed the ceremony altogether. The 2026 rhyme: a previously undisclosed bug in the Orchard circuit forced the Ironwood upgrade to seal the old pool behind a supply-verifying turnstile. The lesson generalizes -- in any shielded pool, soundness cannot be audited from the ledger, so deployments need turnstiles between pools regardless of how the parameters were made.

## Poison Two: Small Anonymity Sets

Decoy sets have a statistical half-life. Roughly two-thirds of pre-2017 Monero inputs were spent with zero mixins, and chain-reaction elimination resolved most of the rest ([KFTS17](https://eprint.iacr.org/2017/338), [MSH+18](https://petsymposium.org/popets/2018/popets-2018-0025.php)); the fix -- sampling decoys from an estimated spend-age distribution -- still leaks a multiple-fold advantage over the nominal $1/16$ guessing baseline ([OSPEAD](https://www.getmonero.org/2025/04/05/ospead-optimal-ring-signature-research.html)). Worse, a fixed-size decoy set is *purchasable*: during the March 2024 flood, transaction volume jumped ~6x and the flooder's own outputs became known-not-decoys ("black marbles") inside everyone else's rings ([Ruc24](https://rucknium.me/html/monero-black-marble-flood.pdf)). Interactive adversaries do better still, intersecting rings across known endpoints (the EAE/EABE pattern). [FCMP++](https://gist.github.com/kayabaNerve/0e1f7719e5797c826b87249f21ab6f86) ends this arms race by making the anonymity set the chain -- an attacker cannot own a meaningful fraction of all historical outputs -- though endpoint metadata (exchange records, timing, amounts) survives any on-chain set.

Account-model schemes rent the same poison in different clothes. Anonymous Zether's ring of accounts costs the verifier linear work in $N$, capping practical rings; [PriDe CT](https://eprint.iacr.org/2023/1948) simplifies the proofs and batches multiple receivers but keeps the explicit ring; Quisquis-style decoy redistribution avoids growing state but invites races on overlapping decoy accounts, a friction [Nopenena](https://eprint.iacr.org/2024/903) attacks with small decoy sets engineered against graph analysis. The escape hatch here is expensive: [PriFHEte](https://eprint.iacr.org/2023/710) shows account-model *full privacy* -- anonymity set equal to all accounts -- is possible by homomorphically updating every account per transaction, a feasibility result whose validator cost is linear in the user population.

## Poison Three: Scanning and State

A global anonymity set sends two bills. The recipient's bill is detection: every wallet trial-decrypts every output, which is $O(\mathsf{chain})$ bandwidth and compute, and the detection loop itself is a side channel -- [TBP20](https://eprint.iacr.org/2020/220) identified the payee of any shielded transaction from ping timing. The outsourcing ladder trades trust against leakage against cost: [FMD](https://eprint.iacr.org/2021/089)'s detection keys leak a tunable-but-brittle false-positive haze; [Private Signaling](https://www.usenix.org/conference/usenixsecurity22/presentation/madathil) gets exact detection from a TEE or two non-colluding servers; [OMR](https://eprint.iacr.org/2021/1256) needs only a single untrusted detector under LWE, with [PerfOMR](https://eprint.iacr.org/2024/204) cutting its constants and snake-eye-resistant encryption ([LSTW25](https://link.springer.com/chapter/10.1007/978-3-031-91131-6_5)) closing clue-collision DoS. No scheme escapes total server work linear in the board -- the frontier is constants and trust, not asymptotics. OCash ([HNS24](https://eprint.iacr.org/2024/246)) attacks the reader's side of the ledger instead: payments are shuffled on chain so that a light client knows a few positions among which to find its payment, staying anonymous even against the full nodes it reads through.

The validator's bill is state: commitments and nullifiers can never be deleted, because the design hides exactly which outputs died. Mimblewimble's cut-through, Quisquis's non-increasing state, and randomized-binning proposals like [Toxic Decoys](https://eprint.iacr.org/2025/1124) each bound state by weakening or restructuring the hidden-spend property. [Tachyon](https://seanbowe.com/blog/tachyon-scaling-zcash-oblivious-synchronization/) is the most radical settlement of both bills at once: notes are delivered out-of-band, wallets carry proofs of their own state, nullifiers evolve unlinkably so untrusted services can prove non-spentness obliviously, and validators permanently prune the nullifier set ([BM25](https://eprint.iacr.org/2025/2031)) -- wallet and validator costs track the wallet's own activity, and the trial-decryption side channel disappears structurally.

## The Fourth Axis: Accountability

Compliance mechanisms divide on whether anyone holds a de-anonymization capability. Authority-held: [GGM16](https://eprint.iacr.org/2016/061)'s in-circuit policies with tracing keys, [PEReDi](https://eprint.iacr.org/2022/974)'s threshold tracing, escrowed selective de-anonymization, and -- the quantitative middle ground -- [AuditPay](https://eprint.iacr.org/2026/1118)'s *auditing budget*, which rate-limits how many addresses an auditor can monitor per epoch without users learning which. User-held: viewing keys and payment disclosures, [UTT](https://eprint.iacr.org/2022/452)'s anonymity budgets, and the provenance line -- proofs of innocence, [Privacy Pools](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4563364)' association sets (deployed on mainnet in 2025), [Derecho](https://eprint.iacr.org/2023/273)'s proof-carrying disclosures, and [Proof of Source of Funds](https://arxiv.org/abs/2606.10172), which proves a payload is funded by compliant value flowing from committed origins while hiding the entire intermediate graph.

The provenance line holds no trapdoor at all, which is why it won permissionless deployment after the Tornado Cash sanction. Its price is paid in the second poison: your effective anonymity set is not the pool but the association set you can prove membership in, and intersections across withdrawals leak. Compliance overlays do not remove a poison; they choose which one to deepen.

## Three Arcs, One Missing

Three arcs are converging on the dream. Anonymity sets are going full-chain: FCMP++ cleared its proof-system audits, ran a public testnet from October 2025 and a stressnet from May 2026, with the mainnet hard fork expected to follow (no activation height set as of this writing). Proving is going client-side and synchronization oblivious: Tachyon targets Zcash's next upgrade, Aztec proves private execution client-side, and the OMR line keeps cutting the detector's constants. And compliance is going zero-knowledge, from association sets to budgeted auditors.

The fourth arc is conspicuously unfinished: post-quantum. [MatRiCT](https://eprint.iacr.org/2019/1287) and its successor brought lattice RingCT from megabytes to tens of KB but kept ring-bounded anonymity; [SMILE](https://eprint.iacr.org/2021/564) showed logarithmic-size lattice set membership -- a 22 KB ring signature over a $2^{25}$-element set -- but was never built into a payment system; FCMP++ is explicitly *not* post-quantum sound. And unlike soundness, privacy cannot wait for the threat to materialize: everything encrypted onto a ledger today is a harvest-now-decrypt-later plaintext. This is the gap our recent work [Qurrency](https://eprint.iacr.org/2026/015) targets -- a UTXO-based token system that is simultaneously private against quantum adversaries and auditable, practical enough to run on EVM chains.

```{=html}
<div id="private-payments-frontier" class="cw-magic-final" role="region" aria-label="The private payments landscape extended with post-quantum constructions.">
  <noscript>
    This section contains a figure extending the private payments landscape with post-quantum constructions.
  </noscript>
</div>
```

As of this writing, no construction achieves [the dream](#dream-goal). Every deployment picks its poisons -- and the most interesting work in the field is whichever arc is bending yours.
