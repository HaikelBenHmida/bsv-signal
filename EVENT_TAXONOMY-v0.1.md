# BSV Signal — EVENT TAXONOMY v0.1

**Status:** Frozen and hashed **before** the 184-report retrospective audit begins.
**Companion document:** `METHODOLOGY-v0.1.md`
**Document hash:** published in `LOCK-MANIFEST-v0.1.json`

---

## 1. Provenance of these classes

The classes below are defined from causal first principles — by the mechanism through which an event could plausibly change BSV's value relative to the crypto market — and **not** from anything observed in the retrospective corpus.

This ordering matters. If class membership were drawn from what looked coherent in a period whose outcomes are already known, the classes would themselves be fitted parameters and every subsequent event study would inherit that contamination.

The procedure is therefore:

```
EVENT_TAXONOMY v0.1  →  hash  →  publish  →  LOCK
                                              ↓
                            184-report retrospective audit
```

If the taxonomy proves unworkable, that is an acceptable finding. The response is: v0.1 classifications remain as issued, the problem is documented, `EVENT_TAXONOMY-v0.2` is written, and future events are classified under v0.2. **v0.1 classifications are never rewritten to make a historical study cleaner.**

---

## 2. Definition of a material event

An item qualifies as a material event if it satisfies **at least one** of:

1. It changes market access or liquidity for BSV.
2. It materially changes miner economics.
3. It changes the network's operating state.
4. It constitutes a new production Teranode deployment or operator change.
5. It is a material application launch, shutdown or outage.
6. It is a regulatory or legal change with BSV impact.
7. It is a credible commercial adoption event.
8. It is an unusual capital flow with a plausible market-relevant interpretation.

An item is **excluded** if it is: routine transaction statistics; ordinary block production; a repeated announcement; a minor software commit; promotional activity; ordinary whale reshuffling without interpretation; or an event already recorded previously.

Exclusion is recorded, not silently dropped. A candidate examined and rejected is logged with its rejection reason, so the ratio of candidates to qualifying events is itself measurable.

---

## 3. Classes

Each class states its mechanism, what it includes, what it excludes, and its single primary horizon and response variable. All other horizons and response variables are secondary and exploratory.

### 3.1 Exchange / market-access change — primary horizon 24h

**Mechanism:** direct change in who can buy or sell BSV, or at what cost.

Includes: listings and delistings; trading suspensions; withdrawal or deposit halts; pair additions or removals; market-maker withdrawal; custody or fiat-rail changes affecting BSV access.

Excludes: routine maintenance windows; exchange-wide incidents not specific to BSV; promotional listings on venues with negligible volume.

### 3.2 Capital-flow / whale event — primary horizon 6h

**Mechanism:** a large, identifiable movement of coin that plausibly precedes supply or demand reaching the market.

Includes: large transfers to or from exchange-attributed addresses; dormant-supply reactivation; treasury or foundation movements; concentrated accumulation or distribution with a defensible attribution.

Excludes: internal reshuffling without directional interpretation; routine consolidation; movements whose `classification_confidence` is below **70/100**, logged as rejected candidates with reason `LOW_ATTRIBUTION_CONFIDENCE`.

The 70 threshold is an operational judgement fixed before the retrospective audit. It was not selected from event outcomes.

The short horizon reflects that flow-driven effects, if they exist, should appear quickly; a 6h primary is also the least contaminated by unrelated news.

### 3.3 Mining / security event — primary horizon 24h

**Mechanism:** change in the cost, concentration or reliability of block production.

Includes: pool entry or exit; material shifts in hashrate concentration; miner policy changes; reorganizations; double-spend or security incidents; fee-policy changes affecting miner revenue.

Excludes: ordinary difficulty adjustment; normal variance in pool shares; routine hashrate fluctuation.

### 3.4 Network reliability / outage — primary horizon 24h

**Mechanism:** the network's ability to process and confirm transactions changes observably.

Includes: chain halts or stalls; sustained abnormal block intervals; propagation failures; node software defects affecting consensus; sustained mempool or throughput anomalies not attributable to known automated load.

Excludes: single slow blocks; automated-load spikes already identified as such; explorer or third-party API outages that do not reflect network state.

### 3.5 Teranode / protocol infrastructure — primary horizon 7d

**Mechanism:** the medium-term credibility of the scaling thesis changes.

Includes: Teranode releases; new production operators or operator departures; version transitions; acceptance-test results; protocol upgrades and activations; consensus-relevant specification changes.

Excludes: routine commits; testnet-only results presented without production relevance; restatements of existing capability claims.

The 7d primary reflects that infrastructure events are structural rather than immediate. Capacity demonstrated is not adoption demonstrated, and the two are classified separately.

### 3.6 Commercial adoption / application — primary horizon 7d

**Mechanism:** demonstrated economic use of the chain changes.

Includes: application launches, shutdowns or sustained outages; verifiable enterprise integrations; partnerships with observable on-chain or operational evidence; material changes in an application's usage where independently measurable.

Excludes: announcements of intent; memoranda of understanding without deliverables; adoption claims resting solely on interested-party statements; directory listings without deployment evidence.

### 3.7 Regulatory / legal — primary horizon 7d

**Mechanism:** the legal position, listing eligibility or reputational standing of BSV changes.

Includes: rulings and judgments; enforcement actions; classification decisions; legislation with direct BSV consequence; litigation outcomes involving BSV-associated entities where market access or reputation is affected.

Excludes: commentary and speculation about pending legislation; filings without decision; general crypto-regulatory news with no BSV-specific mechanism.

---

## 4. Event record

```
event_id
taxonomy_version
dataset_label            PROSPECTIVE_PREREGISTERED | RETROSPECTIVE_EXPLORATORY

observed_at
source
source_url
event_class
domain
novelty                  0–100
evidence_quality         0–100
source_authority         0–100
classification_confidence

pre_event_benchmark_state     (the 08:00 UTC state at or before observed_at)
benchmark_weights
beta_btc, beta_fork
sigma_bch, sigma_ltc, sigma_doge, sigma_etc
beta_estimation_start, beta_estimation_end, beta_estimation_method
bsv_price_at_observation
constituent_prices, constituent_snapshot_times

machine_interpretation   (null at v0.1)
human_interpretation
expected_residual_direction   POSITIVE | NEGATIVE | NEUTRAL | UNCERTAIN
expected_mechanism
invalidation_condition

primary_horizon
horizons_recorded        1h | 6h | 24h | 7d | 30d

content_hash
```

Outcomes are attached later, never overwriting the above:

```
outcome_bsv_price
outcome_constituent_prices
outcome_snapshot_times
realized_bsv_return
benchmark_implied_return
residual_return          (primary response)
residual_class           POSITIVE | NEUTRAL | NEGATIVE
volume_response          (secondary)
liquidity_response       (secondary)
network_response         (secondary)
outcome_status           PENDING | MEASURED | UNSCORABLE_DATA_QUALITY
measured_at
```

The benchmark state is taken from the most recent 08:00 UTC computation strictly at or before `observed_at`, per `METHODOLOGY-v0.1.md` §9.0, and is never re-estimated for that event. Storing the constituent sigmas and the outcome-side prices is what makes an event record recomputable from itself years later; without them the primary scientific track would not be reproducible. A missing required snapshot at either end yields `UNSCORABLE_DATA_QUALITY` — no fallback venue, no interpolation, no score.

`expected_residual_direction` takes exactly one of `POSITIVE`, `NEGATIVE`, `NEUTRAL`, `UNCERTAIN`.

**`UNCERTAIN` is a legitimate answer and is expected to be common.** An event may be plainly material without carrying a confident directional thesis — a resolved network outage, for instance. Forcing a sign onto every event would manufacture pseudo-predictions and inflate the apparent size of the directional sample.

---

## 5. Classification procedure

1. A candidate item is identified from a named source.
2. It is assessed against §2. Rejections are logged with a reason.
3. It is assigned exactly one class from §3. An item that appears to span two classes is assigned to the class describing its **primary causal mechanism**, and the alternative is recorded in a note.
4. Interpretation, expected direction, mechanism and invalidation condition are written.
5. The event representation is **frozen and hashed**.
6. Only then is subsequent price history retrieved and outcomes attached.

For the retrospective corpus, step 4 is performed on report text alone, with no access to subsequent price data. The blinding is procedural and imperfect — aggregate knowledge of the March–September period cannot be removed — which is why §9.1 of the methodology restricts what that corpus may be used for.

---

## 6. Reporting

Results are reported by class, descriptively, with counts:

```
Exchange / market-access      n = …
Capital-flow / whale          n = …
Mining / security             n = …
Network reliability           n = …
Teranode / infrastructure     n = …
Commercial adoption           n = …
Regulatory / legal            n = …
```

Reporting thresholds, as fixed in `METHODOLOGY-v0.1.md` §9.1:

- `n < 10` — reported `INSUFFICIENT N`; descriptive listing only.
- `n ≥ 10` — descriptive aggregate statistics permitted.

**v0.1 performs no inferential event-study testing at any n.** The class-level statistical test has not been specified, and specifying it hastily would be worse than leaving it open. Formal inference requires a separately preregistered protocol naming the test in advance; Holm correction across the seven class-level primary tests is fixed now as the correction policy for that future protocol.

A class below threshold is never merged with another class on the basis of results. Classes are pooled only under a rule stated in advance.

Prospective and retrospective statistics are reported separately and never combined.

---

## 7. Versioning

Any change to the material-event definition, the class list, class boundaries, primary horizons, the primary response variable, or the classification procedure requires a new taxonomy version. Prior classifications remain as issued and continue to be reported under the version that produced them.
