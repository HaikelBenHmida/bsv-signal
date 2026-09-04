# BSV Signal — METHODOLOGY v0.1

**Status:** Preregistration. Frozen before observation #1.
**First scheduled observation:** 2026-09-02, 08:00 UTC.
**Document hash:** published in `LOCK-MANIFEST-v0.1.json`

---

## 1. What BSV Signal is

BSV Signal is a public research record. It publishes timestamped, immutable probabilistic forecasts and event classifications about Bitcoin SV, then attaches measured outcomes to them without altering the original records.

It is not investment advice, not a trading signal service, and — for the reasons set out in §8.1 — it does not claim demonstrated predictive skill. Records are published so that a third party can check the process, including its failures.

Three separate experiments run under this methodology:

| Ledger | Question | Status |
|---|---|---|
| Event ledger | Do preregistered BSV-specific event classes produce abnormal BSV residual returns at their primary horizons? | Primary scientific track |
| Daily ledger | Can a discretionary analyst produce prospectively timestamped, calibrated residual probabilities without choosing when to speak? | Discipline track |
| Audience ledger | Do enough people recurrently read this to justify building a product? | Commercial decision variable |

No result from any ledger may be described as proven alpha at any point during v0.1.

---

## 2. Forecast target

The primary daily forecast is the probability that BSV's **beta-adjusted residual return** over the following 24 hours is positive.

For daily log returns:

```
r_BSV,t = α + β_BTC · r_BTC,t + β_FORK · F_t + ε_t
```

`F_t` is the fork factor: the equal-weighted mean of the standardized daily returns of BCH, LTC, DOGE and ETC, each divided by its own standard deviation over the estimation window.

The realized outcome for a forecast issued at cutoff `T` is:

```
residual = r_BSV[T, T+24h] − ( β_BTC · r_BTC[T, T+24h] + β_FORK · F[T, T+24h] )
```

**α is excluded from the expected return.** Including it would fold historical BSV drift into the benchmark and change what the residual means as that drift changes. The target is BSV's move relative to what the market implied, not relative to its own past trend.

`P(BSV/USD up)` is recorded as a secondary diagnostic. It is never the headline metric and is never optimized against.

### 2.1 Clock

**All benchmark daily bars run 08:00:00 UTC → 08:00:00 UTC**, aligned to the forecast cutoff. Returns are logarithmic:

```
r_t = ln( P_t / P_{t−24h} )
```

This applies identically to estimation, to the fork factor, and to outcome measurement. No other bar convention is used anywhere in v0.1.

### 2.2 Estimation and freezing

β is estimated by OLS on the 90 daily bars ending at the forecast cutoff, using only data with `observed_at ≤ cutoff`. Constituent standardization uses the standard deviation of each constituent's returns over the **same** 90-bar window.

**All estimated quantities are written into the forecast record and never re-estimated.** A future change to β, to the basket, or to the estimation method cannot alter what a past forecast meant. Any such change requires a new methodology version (§6); records issued under v0.1 remain scored with their frozen v0.1 benchmark forever.

Every forecast is recomputable from its own record alone. It stores:

```
forecast_id
methodology_version
generator_type
generator_version

forecast_cutoff_at
submission_window_opened_at
submitted_at
time_to_submit_seconds

bsv_price_source, bsv_price_at_cutoff
btc_price_source, bch_price_source, ltc_price_source,
doge_price_source, etc_price_source
constituent_prices
constituent_snapshot_times

benchmark_weights
beta_btc, beta_fork
sigma_bch, sigma_ltc, sigma_doge, sigma_etc
beta_estimation_start, beta_estimation_end, beta_estimation_method

residual_deadband                  D
p_residual_positive_24h
p_bsv_up_24h                       (secondary)
call                               (POSITIVE | NO_EDGE | NEGATIVE)
confidence                         (LOW | MEDIUM | HIGH)
note                               (≤200 characters)

content_hash
published_at
telegram_message_ref
daily_anchor_txid, merkle_root     (optional)
```

Outcome fields are appended later and never overwrite the above:

```
outcome_bsv_price
outcome_constituent_prices
outcome_snapshot_times
realized_residual
residual_class                     (POSITIVE | NEUTRAL | NEGATIVE)
outcome_status                     (PENDING | MEASURED | UNSCORABLE_DATA_QUALITY)
measured_at
```

Storing the outcome-side prices means a record from 2026 can still be reconstructed in 2028 without querying a venue for history it may no longer serve.

The four `sigma_*` values are mandatory: without them the stored prices and β coefficients are insufficient to reconstruct `F`, and the reproducibility guarantee above would be false.

### 2.3 Price sources

One venue and one pair per asset, named here and unchanged within v0.1. Aggregators that revise history are not used.

| Asset | Venue / pair | Price type |
|---|---|---|
| BSV | Gate — BSV/USDT | last trade |
| BTC | Gate — BTC/USDT | last trade |
| BCH | Gate — BCH/USDT | last trade |
| LTC | Gate — LTC/USDT | last trade |
| DOGE | Gate — DOGE/USDT | last trade |
| ETC | Gate — ETC/USDT | last trade |

**Snapshot rule (causal).** A snapshot at target timestamp `T` is the most recent trade print **at or before** `T`, with age ≤ 60 seconds. A trade occurring after `T` is never eligible, at any distance.

```
target = 08:00:00
07:59:43   eligible
07:59:01   eligible
07:58:59   NOT eligible  (age > 60s)
08:00:01   NOT eligible  (after target)
```

This applies identically to forecast snapshots, historical beta bars, event-time snapshots and outcome snapshots. A snapshot is **missing** if no eligible print exists, or if the venue reports a gap, halt or stale feed at that boundary. A missing snapshot on **any** of the six assets makes the observation `MISSED` (§5). **No substitution, no interpolation, no fallback venue.** Silent substitution would corrupt the benchmark in a way no reader could detect.

---

## 3. Dead band, scoring target and call thresholds

A residual near zero on a thin asset is dominated by β estimation error. The dead band exists to prevent a trivially small residual being reported as a directional success. **It is not part of the probability score.**

### 3.1 Scoring rule

> The Brier score is calculated against the binary outcome `Y = 1 if realized residual > 0, else 0`. The residual dead band is not used in Brier scoring or in calibration.
>
> Directional hit rate is scored separately:
> - a POSITIVE call is correct iff `residual > +D`
> - a NEGATIVE call is correct iff `residual < −D`
> - outcomes within `±D` are recorded `NEUTRAL` and excluded from **both** the numerator and the denominator of the conditional directional hit rate.
>
> The count and share of NEUTRAL outcomes are always published.

**Brier and calibration use every PUBLISHED forecast for which a valid outcome can be measured. PLANNED_PAUSE, MISSED and UNSCORABLE_DATA_QUALITY observations are reported but excluded from scoring.** Hit rate uses the subset of those where the move was large enough to interpret.

### 3.2 Two distinct neutrals

The vocabulary is deliberately separate and must not be conflated:

- **NO_EDGE** is a *forecast* state — the analyst's probability fell between the thresholds.
- **NEUTRAL** is an *outcome* state — the realized residual fell inside the dead band.

They are independent. A POSITIVE call can resolve NEUTRAL; a NO_EDGE forecast still receives a Brier score.

### 3.3 Dead-band value

```
D = 0.25 × σ(residual_24h)   over the calibration window
```

The 0.25 coefficient is a judgement, not a derivation. It is fixed here so that it cannot be chosen later in light of results. Under an approximately normal residual distribution it classifies roughly a fifth of outcomes as NEUTRAL, and that cost is accepted knowingly (see §8.1).

```
deadband_method             0.25 × σ(residual_24h)
deadband_value              D
calibration_start           2025-09-01T00:00:00Z
calibration_end             2026-02-28T23:59:59Z
calibration_dataset_hash
```

The calibration window ends before 2026-03-01 and therefore does not overlap the retrospective corpus of §9.2. `D` is computed once, before observation #1, and is immutable for v0.1.

### 3.4 Calibration recipe

Frozen so that two independent implementations produce the same `D`:

```
returns                ln(P_t / P_{t−24h}), 08:00 UTC bars
snapshot rule          §2.3 causal rule, identical to live operation
beta estimator         OLS with intercept
beta window            90 complete daily observations
constituent sigma      sample standard deviation, ddof = 1

per calibration date T:
    estimate beta and sigmas using only information available at T
    freeze them
    measure the T → T+24h residual under §2
    append to the calibration residual series

deadband sigma         sample standard deviation of the residual
                       series, ddof = 1
D                      0.25 × deadband sigma

missing input          exclude that calibration observation entirely;
                       never interpolate, never substitute a venue
```

**D is not derived from a single regression fitted across the whole calibration period.** Each calibration observation reproduces the prospective process using only information that existed at its own date; otherwise `D` would be estimated with information the live process will never have.

Boundary dates, exactly: 90 daily returns require 91 price snapshots, so price history is required from **2025-06-03T08:00:00Z**. The first calibration date is **2025-09-01T08:00:00Z**; the last is **2026-02-27T08:00:00Z**, so that its 24-hour outcome closes inside the window. That is **180 candidate observations** before data-quality exclusions.

The canonical input and output series are hashed as `calibration_dataset_sha256` and published in the lock manifest.

### 3.5 Call thresholds

| Submitted P(residual > 0) | Call |
|---|---|
| > 0.55 | POSITIVE |
| 0.45 – 0.55 | NO_EDGE |
| < 0.45 | NEGATIVE |

NO_EDGE is a forecast, not an abstention. Coverage — the share of scheduled observations carrying a directional call — is published. There is no expectation of producing a call every day.

---

## 4. The generator

**ANALYST-v0.1 is a discretionary human forecast informed by a preregistered set of quantitative and event inputs. It is not a trained statistical or machine-learning model.** Sentinel contributes nothing at v0.1 (`sentinel_influence: NONE`).

### 4.1 Information set

ANALYST-v0.1 **may use**, as of cutoff:

*Market*
- Gate pricing summary for BSV and the five benchmark constituents
- Poloniex market-state summary, **high-level liquidity regime only** (§10)
- the frozen residual benchmark state for the day

*Network*
- BSV Intel public Daily Intel editions
- UTXO Engineer public Teranode explorer observations (operator state, chain tip)
- WhatsOnChain chain and network statistics

*Ecosystem*
- BSV Radar directory state and changes
- BSV Association official announcements
- `bsv-blockchain/teranode` GitHub releases and release metadata

*Event feeds*
- the named X watchlist: @BSVBlockchain, UTXO Engineer (@HBGnostic), GorillaPool, HandCash, 1Sat Ordinals
- manually submitted URLs **from the named sources above**, recorded with provenance

A URL from any source not named above may be archived as an out-of-information-set candidate for later reference, but **may not influence an ANALYST-v0.1 forecast**. Using it requires a new generator version. Manual ingestion is a convenience for the named set, not a route around the freeze.

**May not use:** any outcome information after cutoff; unpublished forward-looking data; proprietary execution outputs; any indicator or source not named above.

Adding or removing a material source, feature or decision input produces `ANALYST-v0.2`. The list above is what makes v0.1 a single process rather than a drifting one.

### 4.2 Forecast-before-outcome

Enforced by the submission interface, not by intention. Until a forecast is committed, the interface hides all pending outcomes, all unresolved scoring, and the recent rolling hit rate. Performance is reviewed only in the scheduled weekly window. `time_to_submit_seconds` is logged for later analysis.

---

## 5. Ledger states

Each scheduled observation resolves to exactly one of:

- **PUBLISHED** — committed within the window (08:00–08:15 UTC).
- **PLANNED_PAUSE** — declared at least 48 hours before the forecast date, with a reason category (`TRAVEL`, `ILLNESS`, `TECHNICAL_MAINTENANCE`, `KNOWN_UNAVAILABILITY`, `OTHER`), counting against an allowance of 12 per rolling 180 scheduled days, not revocable once the window opens.
- **MISSED** — anything else, including any missing price snapshot under §2.3.

**Missed days are never backfilled.** No forecast is created retrospectively for any reason. Publication adherence is published, and whether misses cluster in high-volatility periods is testable precisely because the gaps are recorded.

### 5.1 Outcome status is separate from forecast status

A validly published forecast cannot retrospectively become `MISSED` because data failed a day later. Each PUBLISHED record therefore carries an independent `outcome_status`:

- **PENDING** — the horizon has not yet elapsed.
- **MEASURED** — all required end-of-horizon snapshots were available under §2.3.
- **UNSCORABLE_DATA_QUALITY** — a required end snapshot was missing. No fallback venue, no interpolation, no score.

`UNSCORABLE_DATA_QUALITY` records remain published and visible. Their count is reported alongside publication adherence.

---

## 6. Version policy

A new generator or methodology version is required for any material change to: the benchmark or its construction; the clock; the dead-band method; the feature or source set; the decision procedure; Sentinel involvement; forecast timing; threshold definitions; or analyst instructions.

**A new version never resets the lifetime record.** The performance page permanently reports every version separately and in aggregate:

| Generator | N | Brier | Calibration | Coverage | NEUTRAL share | Conditional hit rate |
|---|---|---|---|---|---|---|
| ANALYST-v0.1 | … | … | … | … | … | … |
| *(later versions)* | … | … | … | … | … | … |
| All forecasts | … | … | … | … | … | … |

---

## 7. Immutability, corrections and timestamping

Published records are never edited or deleted. An error produces an appended correction that leaves the original visible:

```
FORECAST #0081 — Correction #1
Reason: incorrect constituent price snapshot.
Original record remains published and remains scored.
```

Each record is canonicalized to JSON and hashed with SHA-256. The trust chain is: canonical JSON → SHA-256 → public archive + Telegram publication (+ optional daily Merkle root anchored in a BSV transaction). **Git history is version control, not proof of publication time** — commit timestamps are author-supplied fields. The independent timestamp is the Telegram publication; the on-chain anchor, where used, strengthens it.

---

## 8. Scoring, baselines and statistical limitations

Reported per generator version: Brier score; calibration by probability bucket; coverage; NEUTRAL share; conditional hit rate; mean subsequent residual; publication adherence. Baselines published alongside:

- **Probabilistic:** constant `p = 0.50` every day.
- **Directional descriptive:** always POSITIVE.
- The realized unconditional frequency of positive residuals, reported descriptively.

Momentum and moving-average baselines are deliberately excluded from v0.1. Naming them without specifying the lookback, the horizon and the mapping from a binary regime to a probability would make them free parameters wearing the label of a baseline. They may return under a separately defined benchmark version.

The performance page shows best calls, worst calls, recent misses and calibration — not a headline accuracy figure.

### 8.1 Expected statistical power

> BSV Signal expects roughly 180 scheduled observations in its first six months. If approximately 35% carry a directional call, about 63 will exist; after excluding outcomes inside the dead band, roughly 50 will be scored for hit rate.
>
> That is far too few to distinguish a modest edge from sampling variation. Detecting a true 55% hit rate against a 50% null at conventional power requires on the order of 600 scored directional calls one-sided, or about 780 two-sided. At 35% coverage with roughly a fifth of outcomes falling NEUTRAL, that is approximately **5.9 and 7.6 years** respectively. At a true 52.5% edge the requirement runs into the thousands.
>
> **The first six months are a conduct, calibration, infrastructure and audience experiment. They are not expected to establish statistically persuasive forecasting edge, and no such claim will be made regardless of the observed hit rate.**

This section is published verbatim on the public methodology page.

---

## 9. Event track

Event classes, inclusion criteria and the classification procedure are specified in `EVENT_TAXONOMY-v0.1.md`, frozen and hashed before the retrospective audit begins.

**Beta-adjusted residual return is the only primary response variable.** Volume, liquidity and network responses are secondary and descriptive. Each class has exactly one primary horizon:

| Event class | Primary horizon | Primary response |
|---|---|---|
| Exchange / market-access change | 24h | BSV residual return |
| Capital-flow / whale event | 6h | BSV residual return |
| Mining / security event | 24h | BSV residual return |
| Network reliability / outage | 24h | BSV residual return |
| Teranode / protocol infrastructure | 7d | BSV residual return |
| Commercial adoption / application | 7d | BSV residual return |
| Regulatory / legal | 7d | BSV residual return |

All other horizons and response variables are **secondary and exploratory; no standalone significance claims**.

### 9.0 Event benchmark state

Events occur at arbitrary times; the benchmark clock is defined only at 08:00 UTC (§2.1). Rather than build a second intraday estimation system:

> For an event observed at time `E`, the benchmark state is the most recent 08:00 UTC state computed **strictly at or before** `E`. Its `beta_btc`, `beta_fork`, `benchmark_weights`, `sigma_*` and estimation window are copied into the event record and are never re-estimated for that event.

An event observed at 13:47 UTC therefore uses that morning's 08:00 benchmark. Outcomes are measured from the event-time snapshot to `E + horizon` using those frozen coefficients. Event records store the same reproducibility fields as daily records, on both the observation and outcome sides.

### 9.1 No inferential testing in v0.1

The class-level statistical test has not been specified — whether mean or median residual, signed or unsigned, one- or two-sided, parametric or permutation. Specifying it hastily would be worse than not specifying it.

> **v0.1 performs no inferential event-study testing.** Event results are reported descriptively only. Formal inference requires a separately preregistered analysis protocol naming the test in advance.
>
> Reporting thresholds: `n < 10` → `INSUFFICIENT N`, descriptive listing only. `n ≥ 10` → descriptive aggregate statistics permitted.
>
> When a preregistered inferential protocol does exist, **Holm correction** is applied across the family of seven class-level primary tests. The correction method is fixed here and is not selected later.

A class below threshold is never merged with another class on the basis of results.

### 9.2 Retrospective corpus

The 184 BSV Intel reports (from 2026-03-01) are labelled `RETROSPECTIVE_SOURCE_SELECTED` / `RETROSPECTIVE_EXPLORATORY`. Prospective records are `PROSPECTIVE_PREREGISTERED`. **Their statistics are never pooled.**

Classification is performed on report text alone; subsequent price history is retrieved only after the event representation is frozen.

Permitted uses: measuring material-event frequency; testing the operability of the frozen taxonomy; identifying available source fields; validating ingestion and classification code; descriptive event studies.

**Prohibited uses.** The retrospective corpus MUST NOT be used to select or tune: Sentinel domain weights; Signal Score weights; forecast thresholds; probability mapping; the dead band; event importance thresholds; source-authority weights; or any combination of event classes chosen because it performed well. It may not originate the taxonomy.

The corpus reflects BSV Intel's editorial selection. Any event-rate estimate drawn from it is a floor on discoverable events, not a census.

---

## 10. Public / private firewall

Every measurement carries a classification: `PUBLIC_NOW`, `PUBLIC_DELAYED`, `INTERNAL_ONLY`, `EXECUTION_SECRET`.

> **Nothing derived from proprietary trading features may be published at a resolution equal to or shorter than the useful life of the trading edge it touches.**

Public output may include: market regime, network regime, event regime, high-level liquidity state, residual outlook. Public output may not include: specific support or resistance levels, liquidity wall locations, OBI thresholds, execution triggers, replenishment regions, entry zones, or live strategy parameters.

---

## 11. Audience ledger

> **Authoritative commercial KPI:** a distinct pseudonymous archive visitor or subscribed reader who opens at least three distinct BSV Signal publications during a rolling seven-day period.
>
> Measured first-party from the archive, and from subscribed-reader opens where privacy-compliant measurement is available. The archive assigns a random first-party UUID stored in a SameSite cookie; the server retains only its salted hash for recurring-reader counting, with no cross-site tracking and no third-party analytics platform. **Telegram subscriber counts and aggregate post views are reported separately and are never substituted for this metric.**

Telegram is distribution and third-party timestamping. A channel yields aggregate views, not identity-level engagement, and the decision at §12 depends on a metric that must actually be measurable.

Also tracked: subscribers by channel; four-week retention by joining cohort; open and click rates; unsolicited replies; requests for alerts; requests for data; expressions of willingness to pay; actual payments if tested. Raw follower and impression counts are not decision inputs.

---

## 12. Decision at observation #180

Evaluated on recurring weekly readers as defined in §11, sustained in at least four of the final six weeks:

| Recurring weekly readers | Decision |
|---|---|
| 0–9 | **Prospective ledger closes.** Archive stays online as a static record. |
| 10–14 | Daily ledger ends. Static archive only. No product build. |
| 15–29 | Weekly publication only, maximum 3 further months at ≤1 hr/week. |
| 30–59 | Continue; test willingness to pay. |
| 60–149 | Lightweight public product justified for further validation. |
| 150+ | Evaluate full commercial platform. |

**There is no indefinite continue-cheaply state.** At 0–9 the project ends, without reinterpretation and without another six months.

Audience evidence decides whether a public product exists. Event evidence decides what it should emphasize. Daily-ledger evidence establishes calibration and credibility. None of them may be presented as proof of alpha.

---

## 13. Attention budget

BSV Signal is subordinate to existing trading research (E4 / H0–H1), which does not slip.

Logged weekly: `ledger_minutes`, `event_review_minutes`, `publication_minutes`, `engineering_minutes`. Target: daily submission ≤8 minutes; weekly review ≤30 minutes; median total ≤2 hours per week before commercial validation.

If the two-hour median is breached for three consecutive weeks, scope reduces automatically: first, daily prose is removed; second, event commentary becomes weekly; third, only the ledger and weekly brief remain. Trading research time is never reallocated to BSV Signal.

---

## 14. Disclaimer

BSV Signal publishes probabilistic research about a thinly traded asset. It is not financial, investment or legal advice. Probabilities are estimates from an explicitly stated and limited process, published together with their failures and their statistical limitations.

---

## Appendix — values to compute before hashing

Everything else in this document is frozen. These two are computed once and pasted in:

1. `deadband_value` D — from `0.25 × σ(residual_24h)` over 2025-09-01 → 2026-02-28, together with `calibration_dataset_hash`.
2. Telegram channel identifier and archive base URL.
