# TESTING

```
COMPILE PASS ≠ RUNTIME PASS
SUBMITTED ≠ ACCEPTED ≠ FINALIZED ≠ EXECUTION SUCCESS ≠ POSTCONDITION PASS
```

## On-chain (GenLayer StudioNet, chain 61999)

Wallets: **A** = learner A `0x3065E31B1D993d7C0D59E6786844cBa56780B2d3` · **B** = learner B `0xdaE8968571C6E84f44F86d06F1071bbc8F807500`, each with their own deck. Contract [`0x8A38F1c8c3FF97fe67AE45dD4c029fAF9C92C6f4`](https://explorer-studio.genlayer.com/address/0x8A38F1c8c3FF97fe67AE45dD4c029fAF9C92C6f4) · deploy tx [`0x8ac1447e…1a569ba9`](https://explorer-studio.genlayer.com/tx/0x8ac1447e7d18cfb67ed54945f8538937ce1e37d949e353e34fa9d9331a569ba9) · source SHA-256 `a3c8003f4a6d9118d3c1580b6a31af649520b5e4a036259ef98b06f672608f76`. Run date 2026-10-06 (UTC day 20732), GenLayer Studio, Normal (Full Consensus).

Every case is judged on a card with the term `bank`. The gate cases assume the declared sense `the land along the side of a river` (cards A1, B1). Rows 9 and 11 reuse the R3/O3 frame on cards declaring `a business that keeps and lends money` (A2, B2), where the expected labels swap: the outcome depends on the declared sense, not on the sentence alone. `today` and `due_day` are UTC days since 1970-01-01, read from the transaction's datetime.

The table has 12 rows and 11 transactions, all FINALIZED (12 with the deploy).

Card ids (keccak of owner, term and sense): A1 `724ea068db37ca6731f26d69c0069c641507dfa68b35fbb97b1938b79acc79f1` · B1 `516f0ad407d9e14b8bb91e33dba33cfe36807f1169ddaebe40499d7ae8f59ac4` · A2 `f6451b5e220ab79ab018f992897ad9f45be14ca351354e37605a8866b70581d4` · B2 `69e8709471a2f9c86e6789f783b38eedea0c5c9fdd0a9a1aab5a327699a09ff1`.

| # | Wallet | Call | Expected | Tx hash | Result |
|---|---|---|---|---|---|
| 1 | A | `add_card("bank", "the land along the side of a river")` | card A1, interval 1; get_card(A1): due_day == today, due_now true — **Clock check** | [`0x5657864e…359410ce`](https://explorer-studio.genlayer.com/tx/0x5657864e156b1d61b66beffbb8d5f8892d6551453f7640973723558b359410ce) | SUCCESS; get_card(A1): interval 1, due_day **20732** = today **20732** (2026-10-06), due_now true |
| 2 | B | `add_card("bank", "the land along the side of a river")` | card B1 (a different id from A1), due_day == today | [`0xc6e4d1a7…5acd2740`](https://explorer-studio.genlayer.com/tx/0xc6e4d1a762642287f97ca5a8e3728061908a43d6167f7b0d3fdff7985acd2740) | SUCCESS; card B1 (id differs from A1), interval 1, due_day 20732 |
| 3 | A | `practice(A1, R1)` | RIGHT_SENSE; interval 2, due_day = today + 2 — **Check 1a** | [`0x15c36050…d2b5783f`](https://explorer-studio.genlayer.com/tx/0x15c360509f1b9fe4f986791d17e67d27438dabf3a5e90ac07ce9e9aad2b5783f) | **RIGHT_SENSE**; interval **2**, due_day **20734**, rights 1 |
| 4 | B | `practice(B1, O1)` | OTHER_SENSE; interval 1, due_day = today + 1 — **Check 1b** | [`0x8c4809be…38571658`](https://explorer-studio.genlayer.com/tx/0x8c4809be0d3b56e145720e5aabec57c928187c942f44274f39c36af638571658) | **OTHER_SENSE**; interval **1**, due_day **20733** |
| 5 | A | `practice(A1, R2)` | revert *This card is not due yet* | [`0x3bf26856…ab9be3d2`](https://explorer-studio.genlayer.com/tx/0x3bf268564245f464edbc27ac59f0b36241fe70d869df4ceceb83e6ddab9be3d2) | reverted, *This card is not due yet* |
| 6 | B | `practice(A1, R4)` | revert *Only the card's owner may practise it* | [`0xc41c4e8d…adf2dde0`](https://explorer-studio.genlayer.com/tx/0xc41c4e8d06102a3f788579a847f32fa535ce23d57a4180200e942d7eadf2dde0) | reverted, *Only the card's owner may practise it* |
| 7 | A | `add_card("bank", "a business that keeps and lends money")` | card A2 (the money sense), due_day == today | [`0x0db04583…c632ce9e`](https://explorer-studio.genlayer.com/tx/0x0db045838fff9816986243b4731ae934488b3a00d08c0fec78fa9f17c632ce9e) | SUCCESS; card A2, interval 1, due_day 20732 |
| 8 | A | `practice(A2, "We banked the fire before bed.")` | revert *The sentence does not use the term* | [`0x09a9f503…bfe16939`](https://explorer-studio.genlayer.com/tx/0x09a9f503c45c80439db741b0e54fcfca197f1e9023f5b3f40ee129c0bfe16939) | reverted, *The sentence does not use the term* |
| 9 | A | `practice(A2, R3)` | OTHER_SENSE; interval 1, due_day = today + 1 — **Check 2a** | [`0x13832209…46bbf3d5`](https://explorer-studio.genlayer.com/tx/0x138322098ed3f8ec53460fe06dba9f003f86c51c671a4e279eb734ba46bbf3d5) | **OTHER_SENSE**; interval **1**, due_day **20733** |
| 10 | B | `add_card("bank", "a business that keeps and lends money")` | card B2 (the money sense) | [`0x5c26efdf…de785010`](https://explorer-studio.genlayer.com/tx/0x5c26efdf61e1ade4d4c4b598a16402f1ba2473236c52cae84f3a98d5de785010) | SUCCESS; card B2, interval 1, due_day 20732 |
| 11 | B | `practice(B2, O3)` | RIGHT_SENSE; interval 2, due_day = today + 2 — **Check 2b** | [`0x17ec589b…205cf347`](https://explorer-studio.genlayer.com/tx/0x17ec589bd3826b2f7a643137368642ab18872dd0f4a1503a2cdedf06205cf347) | **RIGHT_SENSE**; interval **2**, due_day **20734**, rights 1 |
| 12 | — | `get_deck(A)` ; `get_deck(B)` | read: A — A2 (interval 1, today + 1) then A1 (interval 2, today + 2); B — B1 (interval 1, today + 1) then B2 (interval 2, today + 2) | — (read) | A: A2 (OTHER_SENSE, interval 1, due 20733) then A1 (RIGHT_SENSE, interval 2, due 20734); B: B1 (OTHER_SENSE, interval 1, due 20733) then B2 (RIGHT_SENSE, interval 2, due 20734); today 20732 |

Must-verify rows:

- **Clock check** — right after add_card, get_card(A1) shows due_day equal to today, a plausible UTC day number (row 1): **PASS**
- **Check 1** — R1 → RIGHT_SENSE (interval 2) and O1 → OTHER_SENSE (interval 1) on two cards with the same content (rows 3, 4): **PASS**
- **Check 2** — on a card declaring the money sense, R3 → OTHER_SENSE and O3 → RIGHT_SENSE (rows 9, 11): **PASS**

Test cases (each pair shares its surface and carries opposite labels):

| Case | Text | Expected | On-chain |
|---|---|---|---|
| R1 | We sat on the bank and watched the boats drift past. | RIGHT_SENSE | row 3 · RIGHT_SENSE ✓ |
| R2 | The heron stood very still on the muddy bank. | RIGHT_SENSE | row 5 (reverted before the model: card not due) |
| R3 | After the flood, the bank was covered in broken branches. | RIGHT_SENSE | row 9, on the money-sense card · OTHER_SENSE ✓ (labels swap there) |
| R4 | She tied the canoe to a willow on the far bank. | RIGHT_SENSE | row 6 (reverted before the model: not the owner) |
| R5 | Kids slid down the grassy bank into the shallow water. | RIGHT_SENSE | not sent |
| O1 | We sat in the bank and watched the clerks count notes. | OTHER_SENSE | row 4 · OTHER_SENSE ✓ |
| O2 | The heron logo stood out on the bank's new card. | OTHER_SENSE | not sent |
| O3 | After the crash, the bank was covered in angry headlines. | OTHER_SENSE | row 11, on the money-sense card · RIGHT_SENSE ✓ (labels swap there) |
| O4 | She tied her savings to a fund at the bank on Main Street. | OTHER_SENSE | not sent |
| O5 | Kids saved coins in a plastic bank shaped like a pig. | OTHER_SENSE | not sent |

R1/O1, R2/O2, R3/O3, R4/O4, R5/O5 are kill tests (the rubric does not hint at the mechanism).

## Offline (before deploy)

- **genvm-linter** `lint`, `schema`, `typecheck`: pass; 6 methods (2 write, 4 view), no type errors.
- **Direct Mode tests** (`genlayer-test`, the real py-genlayer v0.2.16 SDK, model mocked): 57 passed — the tooth (the same sentence on a river-sense card comes back tomorrow and on a money-sense card doubles its gap), the spaced-repetition schedule (1, 2, 4, 8, 16, 32 and the 32-day cap, a collapse to 1 after OTHER_SENSE, due exactly on the due day, late practice counted from the day it happens, new cards due on the day they are added), the deterministic whole-word term check run before the model (bank's, Bank, BANK accepted; banks, banked, riverbank, makeup refused; multi-word terms), one use per sentence per card across letter case and spacing, the day helper checked against Python date arithmetic (29 February, 1 January, 31 December and century years), the on-chain table replayed in order, the id recipe, a third wallet refused, two wallets with identical cards getting two ids, whitespace variants sharing one card id, wallet case, fail-safe on broken output, the validator function, the prompt fence (including cross-token rebuilds) and a prompt that never sees a wallet, an interval, a due day or a review count, views (the deck sorted by due day), limits, check order, and every revert string with a dedicated test.
- **Mutation check**: 30/30 deliberate faults caught, each by at least one test (RIGHT_SENSE does not double the interval, interval has no 32-day cap, OTHER_SENSE keeps the interval instead of collapsing to 1, next due day ignores the new interval, due check off by one (due day itself refused), new card not due on the day it is added, rights not counted, fail-safe flipped on unparseable output, unknown label maps to RIGHT_SENSE, validator accepts any valid label, anyone may practise a card, due checked before owner, same sentence may be reused, used key not lower-cased (case grinding), used key not whitespace-normalized (spacing grinding), term check skipped, term check: no boundary after the term (banks / banked accepted), term check: no boundary before the term (riverbank accepted), term check is case-sensitive, term check looks at the first occurrence only, reserved token not checked on sentences, reserved token not checked on the sense, deck cap off by one (51 cards), sentence length cap off by one, card id ignores whitespace normalization, owner missing from the card id, fence strip: no gap and a single pass (cross-token rebuild survives), wallet leaks into the prompt, day helper: leap shift on the wrong months, deck view not sorted by due day).
- **Rubric gate** on the contract's `RUBRIC`: no word or word pair separates the two test classes; the rubric shares no content word with any test case. Advisory near-matches read by eye: none.
- **Calldata** (offline, encoded exactly as genlayer-js 1.1.8 `writeContract`): every call in the table is between 69 and 153 bytes, under the 255-byte RPC limit. Caps: `add_card` with a 30-character term with a 100-character sense (both caps) measures 162 bytes, longest ASCII text that fits: 193 characters; `practice` with a sentence at the 140-character cap measures 236 bytes, longest ASCII text that fits: 159 characters.

## Consensus behaviour

The model is called once per practice sentence, in `practice`. Validators re-run the judgement and must agree on the exact label; a disagreement rotates the leader or reverts the transaction (fail-closed), and a reverted practice changes nothing — the card stays due and the sentence stays unused. Re-running does not defend against prompt injection — the fence and the reserved-token check do. `add_card` is deterministic; the whole-word term check in `practice` runs before the model.

## What this run does NOT prove

- Each case is sent once; label stability across repeated runs or validator sets is not measured. Three of the five gate pairs are not sent on-chain (R2/O2, R4/O4, R5/O5); R3 and O3 are sent on money-sense cards only.
- Only the first step of the schedule is checked on-chain (interval 1 to 2, or back to 1, on the day the card is added). Later doublings, the 32-day cap and the due-day boundary are covered by the offline tests with a moved clock.
- A 140-character ASCII sentence (the contract cap) measures 236 bytes of calldata, under the 255-byte limit; the longest ASCII sentence that fits is about 159 characters. A sentence with many non-ASCII characters takes more bytes per character and can exceed 255 bytes before it reaches 140 characters; no such sentence is sent.
- Prompt-injection resistance rests on the fence and the reserved-token check; no adversarial model run is done.
