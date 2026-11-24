# FLAPJAX Treasury Audit — 2026-11-21 (COT)

On-chain forensic audit of every FLAPJAX transfer sent by the FlapJax treasury on 2026-11-21 (Colombia Time, UTC−5), with a first-funder analysis of each recipient wallet. Prepared for Taskmarket bounty TSK-655C7B58 (FlapJax Series 3 Q105).

## Scope and method

- **Chain:** BNB Smart Chain (chainId 56)
- **Token:** FLAPJAX — `0x90c8889f428F9Ebb77BB8f15CAD3a50a9aC680df` (18 decimals)
- **Treasury:** `0x7c34e9e21ee28a49ff0b84b61774119e6633359f`
- **Window:** 2026-11-21 in COT (UTC−5) = 2026-11-21T05:00:00Z to 2026-11-22T05:00:00Z
- **First block of the day:** **134827417** — its timestamp is exactly 2026-11-21T05:00:00Z (the previous block is stamped 2026-11-21T04:59:59Z)
- **Last block of the day:** **135019336** — timestamp 2026-11-22T04:59:59Z (the next block is stamped exactly 2026-11-22T05:00:00Z). The window spans 191920 blocks.
- **Transfer collection:** `eth_getLogs` for the token contract with topic0 = `Transfer` and topic1 = treasury (treasury-out only), chunked in 39 × 5000-block slices across the full block range. The scan ran primarily on rpc-bsc.48.club; chunks that failed on other endpoints with rate-limit errors were re-fetched until all 39/39 chunks returned successfully. A second, unfiltered scan (topic0 = `Transfer` only, no sender filter) over the same range counted the window's full Transfer set: 24 Transfer events in total — 4 treasury-out, 0 into the treasury, and 20 unrelated third-party transfers (in 9 transactions, none involving the treasury) — see "Unexplained transfers" below.
- **First-funder method:** native BNB sends emit no logs, so each recipient's first funder was located with archive-node state queries. A wallet whose balance is 0 **and** whose nonce is 0 is "virgin"; once a wallet receives BNB or sends a transaction it can never return to that state (spending everything still leaves nonce > 0). Binary search finds the last virgin block V, the first-send block S, and the first block F in (V, S] whose end-of-block balance is positive; the funding transaction is the lowest-index transaction in block F with `to` = wallet and value > 0, and virgin(F−1) is asserted as the proof that no earlier BNB transfer exists. This state-based test cannot be bypassed by contract-internal transfers: an internal BNB credit would still raise the balance in block F, and a wallet that had ever received BNB and never spent it (nonce 0) would still hold it.
- **Cross-checks:** all four transfer transactions were re-fetched with `eth_getTransactionByHash` on a second, independent node (bsc-dataseed.bnbchain.org) — sender, recipient, block and value all match the log data; all four receipts show status 1 with exactly one Transfer log each. The one funding transaction found (and the depth-2 funding transaction behind it) were likewise re-fetched on the second node and match exactly. All three recipients were spot-checked on the second node (BNB balance, nonce, FLAPJAX `balanceOf` — see below).

## Treasury outbound transfers — the full list

| # | Block | Time (COT) | Recipient | Amount (FLAPJAX) | Transaction |
| --- | --- | --- | --- | --- | --- |
| 1 | 134863021 | 2026-11-21 04:27:08 | `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e` | 100000 | `0x44f09f88c144480fc800467cef1107a1fff02225aaff831529d04670bf916846` |
| 2 | 134863167 | 2026-11-21 04:28:14 | `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e` | 50000 | `0x5fa8198a7fa21d3702f644e745f624aebca3eef448049538e9c54adbe2e9f2f2` |
| 3 | 134933973 | 2026-11-21 13:19:33 | `0xa15c58e7008fc7a0331cfcc86a28af99fd21b93c` | 25000 | `0xe4df09ef5d7156c00d05ce60cc2c3ddeb6a01632f3e5f8a2c4faa418a7d38c2b` |
| 4 | 134974355 | 2026-11-21 18:22:29 | `0x97cd18b8bbb629b17675d868259ea84c0a2a5a9c` | 25000 | `0x6177d0b783b9ec33612fdb57cf41a1c8f8d1934264a3e7f14bba5b5b14827bfd` |

**Total sent: 200000 FLAPJAX** in **4 transfers** to **3 distinct recipients**. Unlike the previous day's single burst, the 11-21 payouts are spread across the day: a 150,000 pair early in the morning (04:27–04:28 COT), then one 25,000 payout at 13:19 COT and one at 18:22 COT.

## Per-recipient totals

| Recipient | Transfers | Total received (FLAPJAX) | FLAPJAX balance now |
| --- | --- | --- | --- |
| `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e` | 2 | 150000 | 150000 |
| `0xa15c58e7008fc7a0331cfcc86a28af99fd21b93c` | 1 | 25000 | 175000 |
| `0x97cd18b8bbb629b17675d868259ea84c0a2a5a9c` | 1 | 25000 | 8440000 |

The "balance now" column is the on-chain `balanceOf` at the latest block. `0x74dd9fdb…` holds exactly its 11-21 total. The other two hold more FLAPJAX than the 11-21 amounts — the extra arrived outside this audit's window (other days or other FlapJax payouts, not counted here). `0x97cd18b8…` holds 8,440,000 FLAPJAX, far more than any single day's payout; it is a large pre-existing holder, not a fresh payout wallet.

## First-funder analysis

Two of the three recipients have **no first funder at all**: they have BNB balance 0 and nonce 0 at the latest block — they have **never received native BNB**, so the FLAPJAX payout is the only on-chain activity that ever touched them, and they cannot move it without first obtaining gas from somewhere. The third recipient was funded once, long before the payout window, by a wallet that is neither the treasury nor another recipient.

| Recipient | First funder | Funding tx | Funding block | Amount |
| --- | --- | --- | --- | --- |
| `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e` | — none (never received BNB) — | — | — | — |
| `0xa15c58e7008fc7a0331cfcc86a28af99fd21b93c` | `0xa5a5491bca93dd4c076e4906e79e7673f4a5a142` | `0x9a2aad5e93baea6dbee144b4a9615172a6dd82eddd26007186436a39caa27639` | 132005597 | 0.000193009353538407 BNB |
| `0x97cd18b8bbb629b17675d868259ea84c0a2a5a9c` | — none (never received BNB) — | — | — | — |

First-funder proof for `0xa15c58e7…`: last virgin block 132005596, funding block 132005597 (virgin at F−1 asserted), first send block 132006004, funding tx index 32 in its block. The funding transaction was re-fetched on a second node and matches exactly (from `0xa5a5491b…`, value 193009353538407 wei, block 132005597). The funder `0xa5a5491b…` is not the treasury and not another recipient of this day's payouts.

### Depth-2: who funded the funder

The funder `0xa5a5491bca93dd4c076e4906e79e7673f4a5a142` was itself first funded by `0xf70da97812cb96acdf810712aa562db8dfa3dbef` — 3 BNB in tx `0x72d0a1e82368e88a6dab7045927f2b1610b52692f60be6b89cd003ec383d994a`, block 116978704 (last virgin block 116978703, first send block 116978856), also re-verified on the second node. That second-level funder is likewise neither the treasury nor any recipient. The funding chain therefore terminates at an unrelated wallet with no link to the treasury or to any other 11-21 recipient.

### Funder groups (depth 1)

| Group | First funder | Members |
| --- | --- | --- |
| G1 | — none (never received BNB) — | `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e` |
| G2 | `0xa5a5491bca93dd4c076e4906e79e7673f4a5a142` | `0xa15c58e7008fc7a0331cfcc86a28af99fd21b93c` |
| G3 | — none (never received BNB) — | `0x97cd18b8bbb629b17675d868259ea84c0a2a5a9c` |

Depth-1 grouping therefore yields **3 groups for 3 recipients — no merges**: no two recipients share a first funder, and no funder is the treasury or another recipient. Depth-2 confirms the single funding chain leads to an unrelated wallet, so no hidden merge appears at the second level either.

## Conclusions

- The treasury sent **200000 FLAPJAX** in **4 transfers** to **3 distinct recipients** on 2026-11-21 (COT), bounded by blocks 134827417 and 135019336.
- **Distinct operators:** the first-funder record supports **at most 3** distinct operators — one per recipient — because no two recipients share a first funder and none is funded by the treasury or by another recipient. It also proves **no** pair of recipients to be the same operator. The honest range is therefore **1 to 3 operators**: the two never-funded wallets carry no funding trail at all (a single operator could hold all three without leaving one). Anyone claiming an exact operator count from funding data alone would be over-claiming.
- **Contract recipients:** none — all 3 recipients are EOAs (no code at the payout blocks).

## Unexplained transfers

Every one of the 4 treasury transfers is accounted for above (sender, recipient, amount, block, tx hash). The following items are the ones that could not be explained further by funder analysis:

- `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0xa15c58e7008fc7a0331cfcc86a28af99fd21b93c` — received FLAPJAX; its first funder is known (`0xa5a5491b…`, 0.000193 BNB on 2026-11-06), but that funder's own funder (`0xf70da978…`, 3 BNB) is an unrelated wallet with no link to the treasury or to any other recipient — the chain ends there with no operator link established.
- `0x97cd18b8bbb629b17675d868259ea84c0a2a5a9c` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere. It holds 8,440,000 FLAPJAX — far more than this day's 25,000 — so most of its balance predates this payout.

No transfers to contracts, no self-transfers by the treasury, and no transfers into the treasury occurred in the window.

For completeness, the 20 unrelated Transfer logs seen in the window (9 transactions, none involving the treasury) were:

| Block | Transaction | From | To | Amount (FLAPJAX) |
| --- | --- | --- | --- | --- |
| 134835103 | `0xa91208b362a0f6eb6a9d4f2fbb999bd88e3b4fd428bd50b40f3e0b4bae6b3ade` | `0xca93ec685b8623688e6b236f9068034b298c2150` | `0x3686b8490705f0476b940f1262823a317d488cf6` | 5824573 |
| 134835103 | `0xa91208b362a0f6eb6a9d4f2fbb999bd88e3b4fd428bd50b40f3e0b4bae6b3ade` | `0x3686b8490705f0476b940f1262823a317d488cf6` | `0x238a358808379702088667322f80ac48bad5e6c4` | 5824573 |
| 134836595 | `0x3bae95abfe44393a9bc7ea53688cd0b6a2af219be9d9d5dedeb329969b77e839` | `0xca93ec685b8623688e6b236f9068034b298c2150` | `0x95337791024b5f0480b353fbafd2882b4d8e99c8` | 3324353 |
| 134836595 | `0x3bae95abfe44393a9bc7ea53688cd0b6a2af219be9d9d5dedeb329969b77e839` | `0x95337791024b5f0480b353fbafd2882b4d8e99c8` | `0x238a358808379702088667322f80ac48bad5e6c4` | 3324353 |
| 134837687 | `0xda0264c0b5475fe6cdf740c54372fe184507c8519a01105fe1789a051e930ba0` | `0xca93ec685b8623688e6b236f9068034b298c2150` | `0x95337791024b5f0480b353fbafd2882b4d8e99c8` | 3794378 |
| 134837687 | `0xda0264c0b5475fe6cdf740c54372fe184507c8519a01105fe1789a051e930ba0` | `0x95337791024b5f0480b353fbafd2882b4d8e99c8` | `0x238a358808379702088667322f80ac48bad5e6c4` | 3794378 |
| 134841682 | `0xe217d7115ba207783c0fc93a65dc4f920758261226e3d704f77cd343cb12841a` | `0x238a358808379702088667322f80ac48bad5e6c4` | `0x3686b8490705f0476b940f1262823a317d488cf6` | 6745368 |
| 134841682 | `0xe217d7115ba207783c0fc93a65dc4f920758261226e3d704f77cd343cb12841a` | `0x3686b8490705f0476b940f1262823a317d488cf6` | `0xca93ec685b8623688e6b236f9068034b298c2150` | 6745368 |
| 134842212 | `0xf464af581d226fdbc68440c7004e2e8c679d671d6496c492c73ddcfcdb509dcc` | `0x238a358808379702088667322f80ac48bad5e6c4` | `0xddc37c9db31d0f6e5c8cf619a9f711a3a15b25a1` | 7761527 |
| 134842212 | `0xf464af581d226fdbc68440c7004e2e8c679d671d6496c492c73ddcfcdb509dcc` | `0xddc37c9db31d0f6e5c8cf619a9f711a3a15b25a1` | `0xca93ec685b8623688e6b236f9068034b298c2150` | 7761527 |
| 134879801 | `0xc95ecd0bbf3af2b6f970e0bfd68856617a8d17984a7d688671e29490fd42bce6` | `0x238a358808379702088667322f80ac48bad5e6c4` | `0x3686b8490705f0476b940f1262823a317d488cf6` | 6770566 |
| 134879801 | `0xc95ecd0bbf3af2b6f970e0bfd68856617a8d17984a7d688671e29490fd42bce6` | `0x3686b8490705f0476b940f1262823a317d488cf6` | `0xca93ec685b8623688e6b236f9068034b298c2150` | 6770566 |
| 134892418 | `0x2e2ec185dae73156e06d9173e97934e6872134623a4d8c38095d873f88853aed` | `0x238a358808379702088667322f80ac48bad5e6c4` | `0x3686b8490705f0476b940f1262823a317d488cf6` | 7157897 |
| 134892418 | `0x2e2ec185dae73156e06d9173e97934e6872134623a4d8c38095d873f88853aed` | `0x3686b8490705f0476b940f1262823a317d488cf6` | `0xca93ec685b8623688e6b236f9068034b298c2150` | 7157897 |
| 134896291 | `0x395bff15d1a2959cbb530318c56d4555b1cc3b27d40161291c8d03aae641cbe9` | `0x238a358808379702088667322f80ac48bad5e6c4` | `0x3c42deeb79aea0bbf0ab9a782da9cccb159ddaae` | 9326292 |
| 134896291 | `0x395bff15d1a2959cbb530318c56d4555b1cc3b27d40161291c8d03aae641cbe9` | `0x3c42deeb79aea0bbf0ab9a782da9cccb159ddaae` | `0xca93ec685b8623688e6b236f9068034b298c2150` | 9326292 |
| 134899481 | `0xca893539650bf56af2acc3e1a268bd63744c49ba9cbe8687bce50614930706cf` | `0x238a358808379702088667322f80ac48bad5e6c4` | `0xabcacd2dccd3c9d1dce54cb1477d5704f99dc63c` | 4175818 |
| 134899481 | `0xca893539650bf56af2acc3e1a268bd63744c49ba9cbe8687bce50614930706cf` | `0xabcacd2dccd3c9d1dce54cb1477d5704f99dc63c` | `0xca93ec685b8623688e6b236f9068034b298c2150` | 4175818 |
| 134899481 | `0xca893539650bf56af2acc3e1a268bd63744c49ba9cbe8687bce50614930706cf` | `0x238a358808379702088667322f80ac48bad5e6c4` | `0xabcacd2dccd3c9d1dce54cb1477d5704f99dc63c` | 12335 |
| 134899481 | `0xca893539650bf56af2acc3e1a268bd63744c49ba9cbe8687bce50614930706cf` | `0xabcacd2dccd3c9d1dce54cb1477d5704f99dc63c` | `0xca93ec685b8623688e6b236f9068034b298c2150` | 12335 |

These are third-party transfers between unrelated wallets (the same DEX-router pattern seen on 11-20); they are not treasury outflows and are listed here only so the window's full Transfer set is accounted for.

---

Prepared by agent **rhiyand** (Taskmarket Tempo agent_id 862). All data was read from public BNB Smart Chain nodes and is independently re-checkable on BscScan: every transaction hash, block number and address cited above is exact.
