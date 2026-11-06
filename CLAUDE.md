# NBA Lead Protection Analytics — Project Memory

Portfolio project analyzing how NBA teams protect (or lose) leads, built from a
16-table NBA database with play-by-play (PBP) data. The SQL work lives on the
user's local PC; this repo will hold the final SQL, outputs, and documentation.

**Portfolio story:** not "another comeback dashboard", but carefully defined
lead episodes, validated millions of PBP records, investigation of what happens
before leads disappear, and eventually a test of whether those patterns predict
vulnerable leads. Prefer a smaller, well-defined, reproducible analysis over a
larger one with uncertain labels.

## Status (latest review: through SQL 19)

| Stage | Status |
|---|---|
| 1. Dataset & schema audit | Largely complete |
| 2. SQL extraction & analytical tables | In progress (reusable views/materialized tables not yet confirmed) |
| 3. Descriptive & diagnostic analysis | In progress (lead changes + lead episodes done; duration, scoring runs, droughts, turnovers still to do) |
| 4. Predictive snapshots & evaluation | Not started |
| 5. Power BI dashboard | Not started |
| 6. GitHub docs & case study | Not started |

## Established facts (data audit)

- 65,698 game records; 13.59M PBP rows.
- 27,391 regular-season games with PBP coverage.
- PBP score orientation verified: **first score = away team, second = home team**.
- PBP final score matches official score in 27,210 / 27,391 games (99.34%).
- Duplicate PBP event keys examined were identical (true duplicates, not conflicting records).

Regular-season PBP game classification:

| Classification | Games |
|---|---|
| Matched score + terminal event | 26,319 |
| Post-terminal period regression | 755 |
| Score mismatch | 181 |
| Missing terminal event | 136 |

- 26,319 is a **provisional candidate set**, not the maximum usable set. Some of
  the 755 period-regression games have valid gameplay before the terminal event
  and anomalous records after it — likely recoverable by truncating at the
  verified terminal event.

## Lead analysis results so far (on 26,319 candidate games)

- 168,801 lead changes; avg 6.41 per game; max 40 in one game.
- 274,851 lead episodes (SQL 19).
- Surrendering a lead ≠ losing the game (episode counts, not unique games):

| Episode ending | Leader eventually lost | Leader eventually won |
|---|---|---|
| Opponent took the lead | 56,825 | 51,129 |
| Lead became tied | 68,401 | 72,177 |

- "Game ended while leading → won" is true by construction; do not present it as a finding.

## Key definitions / rules

- **Lead surrendered:** opponent ties or takes the lead after the snapshot.
- **Game lost:** the team leading at the snapshot eventually loses.
  Keep these separate in analysis and labels.
- NBA clock: 12 min per regulation quarter, 5 min per overtime period.
- Don't lock in the clean-game filter until post-terminal regressions are resolved.

## Modeling guardrails (for later)

- Features only from events available at snapshot time (no final scores, future turnovers, future runs).
- All snapshots of a `game_id` in the same split; chronological season-based train/val/test.
- Evaluate with ROC-AUC, PR-AUC, Brier score, calibration — not accuracy alone.
- Account for class imbalance and within-game correlation of snapshots.

## Current priority: validated `lead_episodes` dataset with duration

Extend SQL 19 to compute per episode:
start/end period, start/end game clock, elapsed game time at start/end,
lead duration (sec/min), starting lead size, ending reason, eventual win/loss.

Validation checks: durations nonnegative; ending event matches stated reason;
every game has exactly one final episode.

### This week's targets
1. Resolve treatment of post-terminal period regressions (truncate at verified terminal event, revalidate).
2. Add elapsed time and lead duration to the episode query.
3. Summarize surrendered leads ending in wins vs. losses (by duration, ending reason).
4. Document episode definition, game inclusion rules, known limitations.
5. Save final SQL + outputs in the repo so the analysis is reproducible.

Model and Power BI dashboard are explicitly **not** this week's goals.
