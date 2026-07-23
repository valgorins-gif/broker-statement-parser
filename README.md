# Broker statement parser

Turns broker PDF statements into CSV that imports into Portfolio Performance. Runs entirely in your browser — the file never leaves your computer.

**[Try it →](https://valgorins-gif.github.io/broker-statement-parser/)**

## What it does well

One thing, and it does it with a property most parsers don't have: **it knows when it's wrong.**

Fee and exchange rate are syntactically identical in a statement line — a number followed by a currency. Rather than guessing with heuristics, the parser tries the combinations and keeps the one where

```
quantity × price ÷ fx + fee = total
```

The document contains its own proof. Rows that don't balance are flagged, not silently returned wrong. This matters more than it sounds: a positional parser that shifts by one column returns plausible numbers that are simply false, and nobody notices until tax season.

It also anchors on the ISIN rather than on column positions. An ISIN is the only self-verifying token in the line (Luhn checksum), so when a broker drops a column — which they do silently, whenever a fee happens to be zero — nothing shifts, because nothing is counting columns.

## Honest scope

Tested against Portfolio Performance's own test corpus (2,642 real statement files across 130 brokers, public in their repo):

| | files | extracted something |
|---|---|---|
| Degiro | 66 | 21 (32%) |
| Trade Republic | 241 | 123 (51%) |
| Scalable Capital | 65 | 18 (28%) |

**Read that as a limitation, not a score.** It handles transaction *tables* — statements listing buys and sells. What it does not handle is the long tail that makes up most of the corpus: account statements (`Kontoauszug`, `EstrattoConto`, `EstadoDeCuenta`), crypto trades, corporate actions, delistings, interest, and a dozen other document classes per broker.

That long tail is the actual work. Portfolio Performance has 66 test files for Degiro alone because it takes 66 to cover it. Their per-broker parsers aren't over-engineered — the domain is.

## Coverage

| | |
|---|---|
| Degiro | transaction tables, incl. foreign currency and FX |
| Trade Republic | buy/sell and dividend receipts (incl. German tax breakdown) |
| Scalable Capital | dividends, return of capital |
| Revolut | trades, tickers resolved against the holdings table |

Three structural shapes, one engine each: *tabular* (one transaction per line), *receipt* (one transaction per document, facts across labelled sections), *report* (rows carry a ticker; the document holds its own lookup table). Within a shape, another broker costs vocabulary entries. A new shape costs an engine.

## Portfolio Performance export

Output imports without a mapping wizard. Some things that cost me a few evenings and aren't in any documentation:

- PP validates the **values** of the Type column against its UI language — `Buy` is rejected by an Italian install, which wants `Compra`
- Decimal separator follows the UI language too, so the field separator has to follow with it (`;` for IT/DE)
- The exchange rate is expected inverted relative to how Degiro states it
- Securities in a foreign currency must exist in the portfolio first, or PP assumes the account currency and rejects the row

The language selector in the export handles the first three.

## Not covered

- Account statements, corporate actions, crypto, interest, most non-trade documents
- Scanned PDFs (no text layer)
- Label vocabulary is German and English only

## Tested against

Real files from the Portfolio Performance issue tracker and test corpus, including [#4418](https://github.com/portfolio-performance/portfolio/issues/4418) — a Degiro statement that had been unparseable for a year. That file: 21/21 transactions, all balancing.

---

MIT. No tracking, no server, no upload.
