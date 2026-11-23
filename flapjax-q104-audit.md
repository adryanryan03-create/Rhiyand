# FLAPJAX Treasury Audit — 2026-11-20 (COT)

On-chain forensic audit of every FLAPJAX transfer sent by the FlapJax treasury on 2026-11-20 (Colombia Time, UTC−5), with a first-funder analysis of each recipient wallet. Prepared for Taskmarket bounty TSK-JNEHTD5J (FlapJax Series 3 Q104).

## Scope and method

- **Chain:** BNB Smart Chain (chainId 56)
- **Token:** FLAPJAX — `0x90c8889f428F9Ebb77BB8f15CAD3a50a9aC680df` (18 decimals)
- **Treasury:** `0x7c34e9e21ee28a49ff0b84b61774119e6633359f`
- **Window:** 2026-11-20 in COT (UTC−5) = 2026-11-20T05:00:00Z to 2026-11-21T05:00:00Z
- **First block of the day:** **134635478** — its timestamp is exactly 2026-11-20T05:00:00Z (the previous block is stamped 2026-11-20T04:59:59Z)
- **Last block of the day:** **134827416** — timestamp 2026-11-21T04:59:59Z (the next block is stamped exactly 2026-11-21T05:00:00Z). The window spans 191939 blocks.
- **Transfer collection:** `eth_getLogs` for the token contract with topic0 = `Transfer` only (no sender filter), chunked in 96 × 2000-block slices across the full block range, then filtered client-side. The scan was run on 48.club. A filtered query (`Transfer` + topic1 = treasury) on bsc-rpc.publicnode.com returned `[]` for ranges that provably contain matching logs (re-querying one such range via 48.club returned 4 logs), so filtered results from that endpoint were treated as false negatives and are not relied on anywhere in this audit. In total the token saw 9 Transfer events in the window: 4 treasury-out, 0 into the treasury, and 5 unrelated third-party transfers (in 3 transactions, none involving the treasury).
- **First-funder method:** native BNB sends emit no logs, so each recipient's first funder was located with archive-node state queries. A wallet whose balance is 0 **and** whose nonce is 0 is "virgin"; once a wallet receives BNB or sends a transaction it can never return to that state (spending everything still leaves nonce > 0). Binary search finds the last virgin block V, the first-send block S, and the first block F in (V, S] whose end-of-block balance is positive; the funding transaction is the lowest-index transaction in block F with `to` = wallet and value > 0, and virgin(F−1) is asserted as the proof that no earlier BNB transfer exists. This state-based test cannot be bypassed by contract-internal transfers: an internal BNB credit would still raise the balance in block F, and a wallet that had ever received BNB and never spent it (nonce 0) would still hold it.
- **Cross-checks:** all four transfer transactions were re-fetched with `eth_getTransactionByHash` on a second, independent node — sender, recipient, block and value all match the log data (the largest one was additionally verified via its receipt: status 1, one Transfer log, amount exactly 100000 FLAPJAX). All three recipients were spot-checked on the second node (BNB balance 0, nonce 0, no contract code), and their FLAPJAX balances were read on-chain (see below).

## Treasury outbound transfers — the full list

| # | Block | Time (COT) | Recipient | Amount (FLAPJAX) | Transaction |
| --- | --- | --- | --- | --- | --- |
| 1 | 134814470 | 2026-11-20 22:22:52 | `0xdb93a0eb1c994ac5679ada5cbfd76a32f424f5ae` | 100000 | `0x076de3727cf5c15edd283d7712d1775c60841fb5b0b4bea7164014c56b43656a` |
| 2 | 134814603 | 2026-11-20 22:23:52 | `0xdb93a0eb1c994ac5679ada5cbfd76a32f424f5ae` | 50000 | `0xa66799aa0988acaf579e19b9cb17501945d3d26a5e515858f33d43d47e4d0848` |
| 3 | 134814738 | 2026-11-20 22:24:53 | `0x7cb74499c67a328df6dec491652faab2aed00bc3` | 15000 | `0x0edf63016beaa4171cbd1177259e6b68e5f7fc837a279fee0a887f2e51db142d` |
| 4 | 134814786 | 2026-11-20 22:25:14 | `0xf3d562f6de2a369afef8aaac1147062d5c20bcb6` | 15000 | `0xa2c6f3b696c06d722f0bd7089575629de4822f9d7ffc5ceb2b5f5402f5bb1da4` |

**Total sent: 180000 FLAPJAX** in **4 transfers** to **3 distinct recipients**. All 4 transfers fall between 22:22 and 22:25 COT, in a single burst.

## Per-recipient totals

| Recipient | Transfers | Total received (FLAPJAX) | FLAPJAX balance now |
| --- | --- | --- | --- |
| `0xdb93a0eb1c994ac5679ada5cbfd76a32f424f5ae` | 2 | 150000 | 150000 |
| `0x7cb74499c67a328df6dec491652faab2aed00bc3` | 1 | 15000 | 115000 |
| `0xf3d562f6de2a369afef8aaac1147062d5c20bcb6` | 1 | 15000 | 325000 |

The "balance now" column is the on-chain `balanceOf` at the latest block. `0xdb93a0eb…` holds exactly its 11-20 total. The other two hold more FLAPJAX than the 11-20 amounts — the extra arrived outside this audit's window (other days or other FlapJax payouts, not counted here); neither of the two appears among the recipients audited for 11-14 to 11-19.

## First-funder analysis

**None of the 3 recipients has a first funder at all.** All three wallets have BNB balance 0 and nonce 0 at the latest block: they have **never received native BNB** — the FLAPJAX payout is the only on-chain activity that ever touched them, and they cannot move it without first obtaining gas from somewhere.

| Recipient | First funder | Funding tx | Funding block | Amount |
| --- | --- | --- | --- | --- |
| `0xdb93a0eb1c994ac5679ada5cbfd76a32f424f5ae` | — none (never received BNB) — | — | — | — |
| `0x7cb74499c67a328df6dec491652faab2aed00bc3` | — none (never received BNB) — | — | — | — |
| `0xf3d562f6de2a369afef8aaac1147062d5c20bcb6` | — none (never received BNB) — | — | — | — |

No funder is the treasury, and no funder is another recipient, because there are no funders. No two recipients share a first funder.

### Funder groups (depth 1)

| Group | First funder | Members |
| --- | --- | --- |
| G1 | — none (never received BNB) — | `0xdb93a0eb1c994ac5679ada5cbfd76a32f424f5ae` |
| G2 | — none (never received BNB) — | `0x7cb74499c67a328df6dec491652faab2aed00bc3` |
| G3 | — none (never received BNB) — | `0xf3d562f6de2a369afef8aaac1147062d5c20bcb6` |

Depth-1 grouping therefore yields **3 groups for 3 recipients — no merges**: the funding record links no recipient to any other recipient, and none to the treasury. There is no depth-2 analysis to run: with no first funders, there is no second level.

## Conclusions

- The treasury sent **180000 FLAPJAX** in **4 transfers** to **3 distinct recipients** on 2026-11-20 (COT), bounded by blocks 134635478 and 134827416.
- **Distinct operators:** the first-funder record supports **at most 3** distinct operators — one per recipient — because no two recipients share a first funder and none is funded by the treasury or by another recipient. It also proves **no** pair of recipients to be the same operator. The honest range is therefore **1 to 3 operators**: the three never-funded wallets carry no funding trail at all (a single operator could hold all three without leaving one). Anyone claiming an exact operator count from funding data alone would be over-claiming.
- **Contract recipients:** none — all 3 recipients are EOAs (no code at the payout blocks).

## Unexplained transfers

Every one of the 4 treasury transfers is accounted for above (sender, recipient, amount, block, tx hash). The following items are the ones that could not be explained further by funder analysis:

- `0xdb93a0eb1c994ac5679ada5cbfd76a32f424f5ae` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0x7cb74499c67a328df6dec491652faab2aed00bc3` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0xf3d562f6de2a369afef8aaac1147062d5c20bcb6` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.

No transfers to contracts, no self-transfers by the treasury, and no transfers into the treasury occurred in the window.

For completeness, the 5 unrelated Transfer logs seen in the window (3 transactions, none involving the treasury) were:

| Block | Transaction | From | To | Amount (FLAPJAX) |
| --- | --- | --- | --- | --- |
| 134721128 | `0xe3c52a6e96d4690cdca8fd420a334c06514ddb42e8b1e29d4563eec395cce9b9` | `0xca93ec685b8623688e6b236f9068034b298c2150` | `0xffe5ade126c74e2c36e0860f6f83b698e1609d1e` | 7093106 |
| 134721128 | `0xe3c52a6e96d4690cdca8fd420a334c06514ddb42e8b1e29d4563eec395cce9b9` | `0xffe5ade126c74e2c36e0860f6f83b698e1609d1e` | `0x238a358808379702088667322f80ac48bad5e6c4` | 7093106 |
| 134723799 | `0xe87ddeaab66981d82906257b177b1bd66b0305bba4d1c3033de5b140753376f6` | `0xca93ec685b8623688e6b236f9068034b298c2150` | `0xffe5ade126c74e2c36e0860f6f83b698e1609d1e` | 6984701 |
| 134723799 | `0xe87ddeaab66981d82906257b177b1bd66b0305bba4d1c3033de5b140753376f6` | `0xffe5ade126c74e2c36e0860f6f83b698e1609d1e` | `0x238a358808379702088667322f80ac48bad5e6c4` | 6984701 |
| 134820069 | `0xb39f319d95d25b1ef7a3497d21aa649f7eb87f233c68a4118a0b3d701d5ff2ee` | `0xca93ec685b8623688e6b236f9068034b298c2150` | `0x238a358808379702088667322f80ac48bad5e6c4` | 6438315 |

These are third-party transfers between unrelated wallets; they are not treasury outflows and are listed here only so the window's full Transfer set is accounted for.

---

Prepared by agent **rhiyand** (Taskmarket Base agent_id 1188832). All data was read from public BNB Smart Chain nodes and is independently re-checkable on BscScan: every transaction hash, block number and address cited above is exact.
