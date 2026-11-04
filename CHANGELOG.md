# Changelog — dataset

Dataset releases, separate from the software changelog. Each release is a full snapshot:
figures change between releases both because results are added and because corrections are
made, and a MAJOR benchmark version re-scores every stored run of its benchmark. An analysis
should cite the exact version it used.

## 2026-10-29.6

Schema 1.3.0. Data through 2026-10-29, built from source commit `41141a5d4`, software 0.8.1.

- 3,805 runs (+0) and 166,380 requests (+0) since 2026-10-29.5.
- Benchmark versions since 2026-10-29.5:
  - `business_letters` 3.0.0 (major, 2026-11-03): ground truth corrected against the scans: `persons.json` accepts `F. Lützelschwab` (letter46), `Jean Frey` (letter55), `Rupprecht` (letter13) and `P. Christ` (letter02), drops two spellings of Lützelschwab found on no letter, the empty spelling of Ritter-Wehrle and a duplicate; the sender of the unsigned `letter46` is marked as inferred (`<Oettinger-Meili, Artur>`), as on the other unsigned letters; person names are matched ignoring case and whitespace, and empty names are dropped before matching, so they no longer match an empty spelling; all stored runs re-scored. Scores of all its stored runs were re-based.
  - `company_lists` 3.0.0 (major, 2026-11-03): ground truth corrected against the scan: in `156089_1321088_41` the locations of Imperial Airways (`Basle, Switzerland`) and the Schweiz. Schleppschiffahrtsgenossenschaft (`Basle`) were swapped; all stored runs re-scored. Scores of all its stored runs were re-based.
  - `library_cards` 3.0.0 (major, 2026-11-03): ground truth corrected against the scans: the format entered as `pages` on `00443771`, `00589241`, `00599509`, `00614130` and `00658520` moved to `format`; `00639255` pages as on the card (`1+III+(zus. 207) Bl.`, was `1+111`); `00443771` shelfmark added; `00414956`, a cross-reference, records the referenced author in full and no title, as the other cross-references do; all stored runs re-scored. Scores of all its stored runs were re-based.

## 2026-10-29.5 — released 2026-10-30

DOI [10.5281/zenodo.23821533](https://doi.org/10.5281/zenodo.23821533). Schema 1.3.0. Data through 2026-10-29, source commit `ea9579e81`, software 0.8.0.

- 3,805 runs, 166,380 requests.
- Benchmark versions: `bibliographic_data` 2.0.0, `blacklist_cards` 2.0.0, `book_advert_xml` 2.0.0, `business_letters` 2.0.0, `company_lists` 2.0.0, `duty_rosters` 2.0.0, `fraktur_adverts` 2.0.0, `general_meeting_minutes` 2.0.0, `library_cards` 2.0.0, `magazine_pages` 1.0.1, `medieval_manuscripts` 2.0.1, `personnel_cards` 3.0.0.
