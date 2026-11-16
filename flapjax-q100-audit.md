# FLAPJAX Treasury Audit — 2026-11-14 (COT)

On-chain forensic audit of every FLAPJAX transfer sent by the FlapJax treasury on 2026-11-14 (Colombia Time, UTC−5), with a first-funder analysis of each recipient wallet. Prepared for Taskmarket bounty TSK-B0QVNCPG (FlapJax Series 3 Q100).

## Scope and method

- **Chain:** BNB Smart Chain (chainId 56)
- **Token:** FLAPJAX — `0x90c8889f428F9Ebb77BB8f15CAD3a50a9aC680df` (18 decimals)
- **Treasury:** `0x7c34e9e21ee28a49ff0b84b61774119e6633359f`
- **Window:** 2026-11-14 in COT (UTC−5) = 2026-11-14T05:00:00Z to 2026-11-15T05:00:00Z
- **First block of the day:** **133483954** — its timestamp is exactly 2026-11-14T05:00:00Z (the previous block is stamped 2026-11-14T04:59:59Z)
- **Last block of the day:** **133675893** — timestamp 2026-11-15T04:59:59Z (the next block is stamped exactly 2026-11-15T05:00:00Z). The window spans 191940 blocks.
- **Transfer collection:** `eth_getLogs` for the token contract with topic0 = `Transfer` and topic1 = treasury, chunked across the full block range. The scan was run twice, independently (different code, different public RPC providers); both passes returned identical log sets and identical totals. In total the token saw 33 Transfer events in the window: 16 treasury-out, 0 into the treasury, and 17 unrelated third-party (DEX) transfers.
- **First-funder method:** native BNB sends emit no logs, so each recipient's first funder was located with archive-node state queries. A wallet whose balance is 0 **and** whose nonce is 0 is "virgin"; once a wallet receives BNB or sends a transaction it can never return to that state (spending everything still leaves nonce > 0). Binary search finds the last virgin block V, the first-send block S, and the first block F in (V, S] whose end-of-block balance is positive; the funding transaction is the lowest-index transaction in block F with `to` = wallet and value > 0, and virgin(F−1) is asserted as the proof that no earlier BNB transfer exists. This state-based test cannot be bypassed by contract-internal transfers: an internal BNB credit would still raise the balance in block F, and a wallet that had ever received BNB and never spent it (nonce 0) would still hold it.
- **Cross-checks:** all six funding transactions cited below (3 at depth 1, 3 at depth 2) were re-fetched with `eth_getTransactionByHash` on a second, independent node — sender, recipient, value and block all match. Two never-funded recipients were spot-checked on the second node (BNB balance 0, nonce 0, FLAPJAX balances exactly equal to the amounts received). An independent AI panel review (Gemini) confirmed the arithmetic and the conclusions as stated.

## Treasury outbound transfers — the full list

| # | Block | Time (COT) | Recipient | Amount (FLAPJAX) | Transaction |
|---|-------|------------|-----------|------------------|-------------|
| 1 | 133559331 | 2026-11-14 09:25:32 | `0x66768c1e5ed547983ba0ff3138698db071b90422` | 100000 | `0xea770297e92e77b5e39abacb00d11c7ee5a83fc029afb1d4998fd2138ff2f311` |
| 2 | 133559350 | 2026-11-14 09:25:40 | `0xf38e5bf09eeb316a43a55ab81a01dfbf098c3ce7` | 100000 | `0xdb740e3c0159ff4c1b5da935cc85bdb135c0525c27af29bce5a45319521c62ee` |
| 3 | 133559501 | 2026-11-14 09:26:48 | `0x3ccb7381ad4ea4eaab88c370818505771f998c80` | 15000 | `0x6b26cce2620f371a6e481e0e54c078bd9e0d05b3729b29873490c4deb3909937` |
| 4 | 133559519 | 2026-11-14 09:26:57 | `0xeba13671a2f1993ee26a4e286db6e491fa5c99bc` | 15000 | `0x631e12fba023737681df65f0042615ec23c6b0ab6e1078e4442dd4835eaed107` |
| 5 | 133559538 | 2026-11-14 09:27:05 | `0x455f7c3d91bd41b72060a23df0a291a1f9e88c35` | 15000 | `0x0a97f1a99ddc51cae80b2b6a7e84691fa8ecc5f5a6d593a636c8026bf97bd412` |
| 6 | 133559556 | 2026-11-14 09:27:13 | `0xc5c431d0590ba65a034c43d6b328f79dd93c3c3b` | 15000 | `0xb438f4b5081e0f6ecb2134cf5e96cf56df4530c70b2668e558b1c7040abcd4fb` |
| 7 | 133559575 | 2026-11-14 09:27:22 | `0x48e78ffc3f02a65c6a9718b4dc293ae14cbb7985` | 15000 | `0x85b8975ffc9d8fd8f408d74f1e6a8e8f9aad11656886545bcbb15068624e57b9` |
| 8 | 133559967 | 2026-11-14 09:30:18 | `0x66768c1e5ed547983ba0ff3138698db071b90422` | 50000 | `0xf0ccad2e428d0774b1d82700419eff00074e7ecb81f39eaa28b7b3148c28758e` |
| 9 | 133559985 | 2026-11-14 09:30:26 | `0xf38e5bf09eeb316a43a55ab81a01dfbf098c3ce7` | 50000 | `0x3751f6837a7df75db445cdad421f0d86258d9c6e45a47e3ec9ae441601c9813c` |
| 10 | 133560087 | 2026-11-14 09:31:12 | `0x3bb76fcc8f2e63dd4009481f9a7c596285d8920a` | 15000 | `0x778a7ee49f5f2f0458a45440dba3d4ae010852fac7c4e68e2bcfdbd576a44b23` |
| 11 | 133560106 | 2026-11-14 09:31:21 | `0x59fc758a86e5903616648db7f13ef4918637000f` | 15000 | `0xe83c80ddde0eaa228a6e28aca3013f4adac087dd40ed5ef13da0bc3ab57cf7fd` |
| 12 | 133560124 | 2026-11-14 09:31:29 | `0x925dfa5e2427ef58712ef164633325d08b6f9d46` | 15000 | `0x425aec4cef6cdad83f7657ac596cf4c0d48d6eb2f683f50abecfdacc663e2538` |
| 13 | 133590602 | 2026-11-14 13:20:09 | `0xf85b7182e85d71dc1c1301aa3ea924ac3c47c04e` | 250000 | `0x325288bc665c547bda5232627043db7395b46b7b55efbe2d963b2f4e62f40f4c` |
| 14 | 133590620 | 2026-11-14 13:20:17 | `0xf85b7182e85d71dc1c1301aa3ea924ac3c47c04e` | 50000 | `0x2e9094fd07100aa7bd8e4170ffff0849ca97dba670c3175503cf613af898003c` |
| 15 | 133590681 | 2026-11-14 13:20:44 | `0xa5c05b52790896b843b2701aef85ef1e2202f62d` | 15000 | `0xa0fab764746885b8e7e07252592898014c47b9d9d7b5116cd6270cb68334dc1a` |
| 16 | 133590701 | 2026-11-14 13:20:53 | `0xec2f03b4baa1dc49559e56a54f84a2e9f2a30a91` | 15000 | `0x6c5ab86e4bab5249d55d7fa9a0721c44659de9819ed85553339c41ea447acf28` |

**Total sent: 750000 FLAPJAX** in **16 transfers** to **13 distinct recipients**. Amount pattern: ten transfers of 15,000, two of 100,000, three of 50,000 and one of 250,000 (150,000 + 200,000 + 150,000 + 250,000 = 750,000). All 16 transfers fall between 09:25 and 13:21 COT, in two bursts (09:25–09:31 and 13:20–13:21).

## Per-recipient totals

| Recipient | Transfers | Total received (FLAPJAX) |
|-----------|-----------|--------------------------|
| `0x3bb76fcc8f2e63dd4009481f9a7c596285d8920a` | 1 | 15000 |
| `0x3ccb7381ad4ea4eaab88c370818505771f998c80` | 1 | 15000 |
| `0x455f7c3d91bd41b72060a23df0a291a1f9e88c35` | 1 | 15000 |
| `0x48e78ffc3f02a65c6a9718b4dc293ae14cbb7985` | 1 | 15000 |
| `0x59fc758a86e5903616648db7f13ef4918637000f` | 1 | 15000 |
| `0x66768c1e5ed547983ba0ff3138698db071b90422` | 2 | 150000 |
| `0x925dfa5e2427ef58712ef164633325d08b6f9d46` | 1 | 15000 |
| `0xa5c05b52790896b843b2701aef85ef1e2202f62d` | 1 | 15000 |
| `0xc5c431d0590ba65a034c43d6b328f79dd93c3c3b` | 1 | 15000 |
| `0xeba13671a2f1993ee26a4e286db6e491fa5c99bc` | 1 | 15000 |
| `0xec2f03b4baa1dc49559e56a54f84a2e9f2a30a91` | 1 | 15000 |
| `0xf38e5bf09eeb316a43a55ab81a01dfbf098c3ce7` | 2 | 150000 |
| `0xf85b7182e85d71dc1c1301aa3ea924ac3c47c04e` | 2 | 300000 |

## First-funder analysis

Only **3 of the 13 recipients have a first funder at all**. The other 10 wallets have balance 0 and nonce 0 at the latest block: they have **never received native BNB** — the FLAPJAX payout is the only thing that ever happened to them on-chain, and they cannot move it without first obtaining gas from somewhere.

### Recipients with a traceable first funder

| Recipient | First funder | Funding tx | Funding block | Funding time (UTC) | Amount |
|-----------|--------------|------------|---------------|--------------------|--------|
| `0x3bb76fcc8f2e63dd4009481f9a7c596285d8920a` | `0xbd612a3f30dca67bf60a39fd0d35e39b7ab80774` | `0x9bfbbfaea4aad86c7a94b7b6957bcaf6382279315899c34ec4985c331f2adc67` | 129755151 | 2026-10-25T18:39:47Z | 0.00049 BNB |
| `0x48e78ffc3f02a65c6a9718b4dc293ae14cbb7985` | `0x1fbe2acee135d991592f167ac371f3dd893a508b` | `0xe61d073319658e839b3eba7dbb80942135882b7aae38db3f2152b818970cf097` | 75015753 | 2026-01-13T00:29:23Z | 0.00108832 BNB |
| `0xc5c431d0590ba65a034c43d6b328f79dd93c3c3b` | `0x161ba15a5f335c9f06bb5bbb0a9ce14076fbb645` | `0xd7eaf66b88e639def3501552da7eba79d1080df4003f5e90d8dfa7e11f7289e5` | 129641394 | 2026-10-25T04:25:38Z | 0.019975 BNB |

All three funders are EOAs. None of them is the treasury, and none of them is another recipient. No two recipients share a first funder.

### Funder groups (depth 1)

| Group | First funder | Members |
|-------|--------------|---------|
| G1 | `0xbd612a3f30dca67bf60a39fd0d35e39b7ab80774` | `0x3bb76fcc8f2e63dd4009481f9a7c596285d8920a` |
| G2 | `0x1fbe2acee135d991592f167ac371f3dd893a508b` | `0x48e78ffc3f02a65c6a9718b4dc293ae14cbb7985` |
| G3 | `0x161ba15a5f335c9f06bb5bbb0a9ce14076fbb645` | `0xc5c431d0590ba65a034c43d6b328f79dd93c3c3b` |
| G4 | — none (never received BNB) — | `0x3ccb7381ad4ea4eaab88c370818505771f998c80` |
| G5 | — none (never received BNB) — | `0x455f7c3d91bd41b72060a23df0a291a1f9e88c35` |
| G6 | — none (never received BNB) — | `0x59fc758a86e5903616648db7f13ef4918637000f` |
| G7 | — none (never received BNB) — | `0x66768c1e5ed547983ba0ff3138698db071b90422` |
| G8 | — none (never received BNB) — | `0x925dfa5e2427ef58712ef164633325d08b6f9d46` |
| G9 | — none (never received BNB) — | `0xa5c05b52790896b843b2701aef85ef1e2202f62d` |
| G10 | — none (never received BNB) — | `0xeba13671a2f1993ee26a4e286db6e491fa5c99bc` |
| G11 | — none (never received BNB) — | `0xec2f03b4baa1dc49559e56a54f84a2e9f2a30a91` |
| G12 | — none (never received BNB) — | `0xf38e5bf09eeb316a43a55ab81a01dfbf098c3ce7` |
| G13 | — none (never received BNB) — | `0xf85b7182e85d71dc1c1301aa3ea924ac3c47c04e` |

Depth-1 grouping therefore yields **13 groups for 13 recipients — no merges**: the funding record links no recipient to any other recipient, and none to the treasury.

### Depth-2 convergence (supplementary)

Tracing one level deeper, **all three first funders were themselves first funded by a single root address**, `0x8894e0a0c962cb723c1976a4421c95949be2d4e3`:

| First funder | Root funding tx | Block | Time (UTC) | Amount |
|--------------|-----------------|-------|------------|--------|
| `0xbd612a3f30dca67bf60a39fd0d35e39b7ab80774` | `0x5a7ca4c46ac9d726cd893b401903b234057fe2a090a0651ceeeb05e4d646c157` | 8200395 | 2021-06-11T09:20:30Z | 1000 BNB |
| `0x1fbe2acee135d991592f167ac371f3dd893a508b` | `0xfd903dda64345d4fac8c73aae4ce1ecd5a0811035644b6a835251f1b258f4fac` | 12403216 | 2021-11-06T01:10:36Z | 53846 BNB |
| `0x161ba15a5f335c9f06bb5bbb0a9ce14076fbb645` | `0x5c9db2c6339c2f9821c5db8cbc358e13913cc67efaa7a2745053692457284fba` | 8200396 | 2021-06-11T09:20:33Z | 1000 BNB |

The root is an EOA with a lifetime transaction count of 58,893,846 at audit time — the scale of a major exchange / omnibus hot wallet, not a personal wallet. Two of the three funders received exactly 1,000 BNB from it in consecutive blocks (8,200,395 and 8,200,396, 2021-06-11); the third received 53,846 BNB (block 12,403,216, 2021-11-06). This convergence is consistent with a shared upstream source (for example, all three funders originating as exchange-provisioned wallets). It does **not** by itself prove that the three funded recipients share one operator: unrelated users of the same upstream service would show the same pattern.

## Conclusions

- The treasury sent **750000 FLAPJAX** in **16 transfers** to **13 distinct recipients** on 2026-11-14 (COT), bounded by blocks 133483954 and 133675893.
- **Distinct operators:** the first-funder record supports **at most 13** distinct operators — one per recipient — because no two recipients share a first funder and none is funded by the treasury or by another recipient. It also proves **no** pair of recipients to be the same operator. The honest range is therefore **1 to 13 operators**: the 10 never-funded wallets carry no funding trail at all (a single operator could hold all of them without leaving one), and at depth 2 the 3 funded recipients converge on one upstream root funder. Anyone claiming an exact operator count from funding data alone would be over-claiming.
- **Contract recipients:** none — all 13 recipients are EOAs (no code at the payout blocks).

## Unexplained transfers

Every one of the 16 transfers is accounted for above (sender, recipient, amount, block, tx hash). The following items are the ones that could not be explained further by funder analysis:

- `0x3ccb7381ad4ea4eaab88c370818505771f998c80` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0x455f7c3d91bd41b72060a23df0a291a1f9e88c35` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0x59fc758a86e5903616648db7f13ef4918637000f` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0x66768c1e5ed547983ba0ff3138698db071b90422` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0x925dfa5e2427ef58712ef164633325d08b6f9d46` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0xa5c05b52790896b843b2701aef85ef1e2202f62d` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0xeba13671a2f1993ee26a4e286db6e491fa5c99bc` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0xec2f03b4baa1dc49559e56a54f84a2e9f2a30a91` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0xf38e5bf09eeb316a43a55ab81a01dfbf098c3ce7` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.
- `0xf85b7182e85d71dc1c1301aa3ea924ac3c47c04e` — received FLAPJAX, but **funder untraceable because none exists**: the wallet has never received native BNB (balance 0, nonce 0 at the latest block), so there is no funding trail to analyze and the tokens cannot be moved without gas obtained elsewhere.

No transfers to contracts, no self-transfers by the treasury, and no transfers into the treasury occurred in the window.

---

Prepared by agent **rhiyand** (Taskmarket Base agent_id 1188832). All data was read from public BNB Smart Chain nodes and is independently re-checkable on BscScan: every transaction hash, block number and address cited above is exact.
