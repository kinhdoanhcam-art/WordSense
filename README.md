WordSense does not grade grammar or style, and it does not decide which meaning a word "really" has. It checks one thing about each practice sentence: does the word carry the sense you said you were learning — and that alone sets when the card comes back.

# WordSense

GenLayer Intelligent Contract · StudioNet (chain 61999) · py-genlayer v0.2 (`# v0.2.16`)

**The contract holds no money and moves no money.** It records flashcards, how each practice sentence was read and
when each card is next due.

| | |
|---|---|
| Contract | `contracts/WordSense.py` |
| Address (StudioNet) | `0x8A38F1c8c3FF97fe67AE45dD4c029fAF9C92C6f4` ([explorer](https://explorer-studio.genlayer.com/address/0x8A38F1c8c3FF97fe67AE45dD4c029fAF9C92C6f4)) |
| Deploy tx | [`0x8ac1447e7d18cfb67ed54945f8538937ce1e37d949e353e34fa9d9331a569ba9`](https://explorer-studio.genlayer.com/tx/0x8ac1447e7d18cfb67ed54945f8538937ce1e37d949e353e34fa9d9331a569ba9) |
| Source SHA-256 | `a3c8003f4a6d9118d3c1580b6a31af649520b5e4a036259ef98b06f672608f76` |
| Tested on-chain | `TESTING.md` — 11 transactions in a 12-row table, one hash each; every must-verify check PASS |

## What it does

A learner adds a flashcard: one term and the one sense of it they are learning (`bank` = `the land along the side of a
river`). To practise a card that is due, the learner writes a sentence that uses the term. The contract first checks,
without the model, that the sentence contains the term as a whole word. Validators then read the sentence **once** and
decide one thing:

- `RIGHT_SENSE` — the term, as used in the sentence, carries the declared sense;
- `OTHER_SENSE` — it carries another sense, or no sense can be told.

The outcome sets the card's spaced-repetition schedule:

| | `RIGHT_SENSE` | `OTHER_SENSE` |
|---|---|---|
| Interval | `min(interval × 2, 32)` days | back to `1` day |
| Next due day | today + the new interval | tomorrow |

A new card has interval 1 and is due on the day it is added; a card that is not due cannot be practised. So a card answered
right every time comes back after 2, 4, 8, 16 and then every 32 days, and one sentence in another sense brings it back
tomorrow. Each sentence can be used once per card (letter case and spacing do not make a new sentence).

`After the flood, the bank was covered in broken branches.` and `After the crash, the bank was covered in angry
headlines.` share their frame. On a card declaring the river sense the first doubles the gap and the second brings the
card back tomorrow; on a card declaring `a business that keeps and lends money` the same two sentences swap outcomes. That
swap is the core test of the contract — and on StudioNet, on two money-sense cards, the flood sentence was read
`OTHER_SENSE` (back tomorrow) and the crash sentence `RIGHT_SENSE` (interval 2), while on two river-sense cards
`We sat on the bank…` and `We sat in the bank…` split the same way.

## Methods

| Write | Who | Checks, in order |
|---|---|---|
| `add_card(term, sense)` | anyone (adds to their own deck) | term 1–30 characters → sense 1–100 characters → no reserved token → fewer than 50 cards → not already in the deck |
| `practice(card_id, sentence)` | the card's owner | id → owner → card is due → sentence 1–140 characters → no reserved token → sentence uses the term as a whole word → sentence not used on this card before → **the only model call** |

The whole-word check ignores letter case and requires that the characters right before and right after the term are not
letters: `bank's` and `Bank` use `bank`; `banks`, `banked` and `riverbank` do not. Whitespace is collapsed first, so a
multi-word term such as `make up` matches across any spacing.

Views (JSON strings; an unknown id returns `"{}"`, never a revert): `get_card(card_id)` (every field plus `today`,
`due_now` and `days_until_due`), `get_deck(wallet)` (every card of that wallet, sorted by due day, plus `today`),
`get_rubric`, `get_limits`. There is no preview or dry-run view.

Card id: `keccak256("WORD_SENSE:CARD:V1|" + owner_lower + "|" + len(term) + "|" + term + "|" + len(sense) + "|" + sense)`,
where `term` and `sense` have all whitespace collapsed (Python `" ".join(text.split())`). The owner is part of the id, so
two learners with the same card get two cards.

Days are UTC day numbers (days since 1970-01-01) computed from the transaction's datetime with plain integer arithmetic.

## Design choices

- **Fail-safe is `OTHER_SENSE` — the card comes back tomorrow.** A wrong `RIGHT_SENSE` pushes the card up to 32 days away
  while the learner has not got the sense; a wrong `OTHER_SENSE` costs one extra review. When the model's output is
  unusable, the card comes back tomorrow.
- **The model sees the term, the declared sense and the sentence, nothing else.** No wallet, no interval, no due day, no
  review count, nothing about what the contract will do next. `RIGHT_SENSE`, `OTHER_SENSE` and the six tag markers are
  refused in any letter case in the term, the sense and the sentence before the model is called, and stripped to a fixed
  point inside the prompt.
- **The cheap check runs first.** Whether the sentence uses the term is decided by code, before the model, so a sentence
  that does not contain the term never costs a model call and never moves the schedule.
- **No re-grinding.** A sentence cannot be resent with different spacing or letter case for another reading on the same
  card, and the term and sense are part of the card id, so the same card cannot be added twice to one deck.
- **The rubric names the task, not the answers.** It never mentions the words or senses used in the test pairs, and it
  does not ask for grammar, spelling or style to be marked.

## How to try it in GenLayer Studio

Use **two wallets of your own** (A and B), each with its own deck.

1. A and B: `add_card("bank", "the land along the side of a river")`. Read `get_card(<id>)`: `due_day` equals `today`.
2. A: `practice(<A's id>, "We sat on the bank and watched the boats drift past.")` → `RIGHT_SENSE`, interval 2.
3. B: `practice(<B's id>, "We sat in the bank and watched the clerks count notes.")` → `OTHER_SENSE`, interval 1.
4. Read `get_deck(<A>)` and `get_deck(<B>)`; practising A's card again today reverts with *This card is not due yet*.

The card id is computed from the formula above (any keccak256 tool works). Text may not contain `RIGHT_SENSE` or
`OTHER_SENSE` in any letter case.

## Honest limitation

1. **The contract holds no money.** It keeps a schedule; it does not make anyone study.
2. **It does not stop copied sentences.** A dictionary example is accepted like a sentence the learner wrote; each sentence
   is accepted once per card.
3. **A wrong `RIGHT_SENSE` is the main risk** — the card is pushed far away while the sense is not learned. Nets: the
   fail-safe `OTHER_SENSE`, and the 32-day cap on the interval.
4. **The term check is an exact-form match.** `banks` and `banked` do not count for `bank`; the learner must use the form
   on the card.
5. **Days follow the transaction's UTC datetime.** "Today" can differ from the learner's local date near midnight.
6. **Sentences longer than about 159 characters are not proven.** The contract caps sentences at 140 characters, which
   measures 236 bytes of calldata, under the 255-byte RPC limit; sentences with many non-ASCII characters take more bytes
   per character.

License: MIT.
