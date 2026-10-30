# Changelog — dataset

Dataset releases, separate from the software changelog. Each release is a full snapshot:
figures change between releases both because results are added and because corrections are
made, and a MAJOR benchmark version re-scores every stored run of its benchmark. An analysis
should cite the exact version it used.

## 2026-10-29.5

Schema 1.3.0. Data through 2026-10-29, built from source commit `ea9579e81`, software 0.8.0.

- Initial release: 3,805 runs, 166,380 requests, 3,902,043 metric observations, 124 metric definitions.
- 407,164 re-scored field observations in `rescored_fields`, produced by today's
  scorers over the stored responses and kept apart from the scores the runs
  recorded; see the README.
- Per-request cost is exported twice: as the request file records it, and as derived from recorded tokens and the price in force on the run's date. Both include reasoning billed outside output tokens.
- `timestamp_utc` is null throughout: every stored timestamp is naive and no source timezone is recorded.
- Licensed CC BY 4.0, separately from the software's GPL-3.0. The payload sidecars additionally contain third-party model output; see the README.
- Benchmark versions: `bibliographic_data` 2.0.0, `blacklist_cards` 2.0.0, `book_advert_xml` 2.0.0, `business_letters` 2.0.0, `company_lists` 2.0.0, `duty_rosters` 2.0.0, `fraktur_adverts` 2.0.0, `general_meeting_minutes` 2.0.0, `library_cards` 2.0.0, `magazine_pages` 1.0.1, `medieval_manuscripts` 2.0.1, `personnel_cards` 3.0.0.
