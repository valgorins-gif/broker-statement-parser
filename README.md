# Broker statement parser

Turns broker PDF statements into clean CSV. Runs entirely in your browser — the file never leaves your computer.

**[Try it →](https://valgorins-gif.github.io/broker-statement-parser/)**

## Why this exists

Broker statement parsing is a maintenance treadmill. Brokers change their PDF layout without notice, the parser breaks, someone opens an issue, a volunteer fixes it. Portfolio Performance has 397 open issues; some import bugs have been open since 2020.

This project takes a different approach to the same problem.

## How it's different

**It anchors on the ISIN, not on column positions.** An ISIN is the only token in a statement line that verifies itself (Luhn checksum). Everything else is read outward from it by token type. When a broker drops a column — and they do, silently, whenever a fee happens to be zero — a positional parser silently shifts every field by one. This one doesn't count columns, so there's nothing to shift.

**It knows when it's right.** Fee and exchange rate are syntactically identical in a statement: a number followed by a currency. Rather than guessing with heuristics, the parser tries the combinations and keeps the one where

```
quantity × price ÷ fx + fee = total
```

The document contains its own proof. Rows that don't balance are flagged, not silently returned wrong.

**Two structural families, not one parser per broker.** Statements come in shapes, not just layouts:

- *tabular* — one transaction per line (Degiro)
- *receipt* — one transaction per document, facts scattered across labelled sections (Trade Republic, Scalable Capital)

Within a family, a new broker costs vocabulary entries, not code. Trade Republic and Scalable Capital run on the same engine and the same dictionary.

## Status

| | |
|---|---|
| Degiro | 21/21 transactions, all arithmetically verified |
| Trade Republic | buy/sell, dividends (incl. foreign currency + German tax) |
| Scalable Capital | dividends, return of capital |
| Revolut | not yet — different shape again (ticker→ISIN lookup, no ISIN in the transaction rows) |

Output imports into Portfolio Performance without a mapping wizard. Localised for English, Italian and German — PP validates the *values* of the Type column against its UI language, so `Buy` is rejected by an Italian install.

## What it doesn't do yet

- Splits, mergers, and most corporate actions
- Scanned PDFs (no text layer to read)
- The vocabulary covers German and English labels only

## Tested against

Real statements from the Portfolio Performance issue tracker — files reported as unparseable, including [#4418](https://github.com/portfolio-performance/portfolio/issues/4418) (Degiro transaction.pdf unreadable for a year).

---

MIT. No tracking, no server, no upload.
