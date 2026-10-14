# swiss-payment-files

A typed TypeScript toolkit for the files Swiss banks exchange with their business customers: bank statements (ISO 20022 `camt.053`) and payment orders (ISO 20022 `pain.001`), following the Swiss Payment Standards.

> **Status: planned, not implemented.** This repository holds the plan only. Nothing below exists yet. Every section describes what the first release must do and the bar it must meet.

## Why this exists

Swiss banks use ISO 20022 XML files for payment orders and account statements. The Swiss market is retiring the older 2009-based message versions in November 2026, leaving only the 2019-based ones (`pain.001.001.09` and `camt.053.001.08`) and making structured addresses mandatory. A GitHub search on 2026-10-13 found no maintained TypeScript library of note for either format: the largest statement parser in any language had 55 stars, and nothing in TypeScript for the payment-order format had more than 6.

> **To verify before writing any code:** the sources disagree on the exact cut-over day (14 or 21 November 2026), and they were read only as search summaries. Read the official Swiss Payment Standards documents from SIX before putting any date or rule in this README or in code:
> - [SIX: Swiss Payment Standards 2026 business rules](https://www.six-group.com/dam/download/banking-services/standardization/sps/business-rules-sps-2026-en.pdf)
> - [Zürcher Kantonalbank: ISO 20022 factsheet](https://zkb.ch/media/zkb/dokumente/sonstige/fs-iso-20022.pdf)
> - [UBS: structured addresses](https://www.ubs.com/ch/en/services/payments/connection-ubs/iso-20022/structured-addresses.html)

## Planned scope for version 1

| Piece | What it does |
|---|---|
| Statement parser | Reads a bank statement file (`camt.053.001.08`) into typed objects: accounts, balances, entries, transaction details, references |
| Payment-order generator | Builds a payment-order file (`pain.001.001.09`) from typed input, following the Swiss implementation guidelines |
| Schema validation | Validates every file read or written against the official XML schema files |
| Swiss rule checks | Checks the Swiss rules that the schema cannot express: structured addresses, reference types, character set, field lengths |
| Migration checker | Answers "would this older-version file pass under the new rules?" and lists what to change |
| Command-line tool | `parse`, `validate`, `generate` and `check` commands |
| Browser demo | Drop a file into a web page and see it parsed and validated, entirely in the browser, with nothing uploaded |
| Optional: QR-bill reading | Reads and validates the data in a Swiss QR-bill against the structured-address rules (reading only; see non-goals) |

### Non-goals

- **Generating QR-bills.** At least seven maintained libraries already do this, including [swissqrbill](https://github.com/schoero/swissqrbill) in TypeScript.
- **Talking to banks.** No EBICS, no open-banking connections, no credentials. The toolkit works on files only.
- **Older message versions as output.** It reads old versions only so the migration checker can explain them.

## Design guidelines

- **Money is never a floating-point number.** Amounts are integer minor units or exact decimal strings, always with an explicit currency.
- **Typed end to end.** TypeScript in strict mode. Public types mirror the ISO 20022 structures closely enough that someone who knows the standard recognises them.
- **Reference types with their check digits:**
  - the Swiss QR reference (`QRR`), 27 digits with a recursive modulo-10 check digit, used with a QR-IBAN;
  - the international creditor reference (`SCOR`, ISO 11649), `RF` plus two modulo-97 check digits;
  - IBAN and QR-IBAN validation.
- **Validation errors are specific.** Each error names the rule, the path in the file and the offending value, and gives a reference to the guideline section.
- **No network access** in the library. Schema files are bundled.
- **Small dependency footprint.** Only an XML parser and a schema validator, each chosen and justified in a short design note.

## Quality bar for the first release

- **Test corpus** built from the sample files that SIX and the banks publish. Where those files live is not yet confirmed.
- **Round-trip tests:** generate, then parse, then compare.
- **Property-based tests** for the check-digit algorithms and amount handling.
- **A coverage table of the rules:** which Swiss rules are implemented, which are not, and why.
- **Continuous integration** runs lint, type checking, tests and schema validation on every push.
- **Published to npm**, with semantic versioning and a changelog.
- **Licence:** MIT or Apache 2.0, chosen before the first code commit.
- **A design note** covering the parsing approach, error model and known limits.

## Planned milestones

1. Read the SIX documents; settle dates, versions and the list of rules; locate the official sample files.
2. Statement parser with schema validation and the test corpus.
3. Payment-order generator with the Swiss rule checks and round-trip tests.
4. Reference and IBAN check-digit module with property tests.
5. Command-line tool and npm release.
6. Migration checker.
7. Browser demo.
8. Optional: QR-bill reading and validation.

## Related work

- [moov-io/iso20022](https://github.com/moov-io/iso20022): Go library for the same message family (archived).
- [sebastienrousseau/bankstatementparser](https://github.com/sebastienrousseau/bankstatementparser): multi-format statement parser.
- [schoero/swissqrbill](https://github.com/schoero/swissqrbill): QR-bill generation in TypeScript.
