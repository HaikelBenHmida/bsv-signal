# BSV Signal — METHODOLOGY v0.2

**Status:** Preregistration amendment. Frozen before v0.2 calibration.
**Base document:** `METHODOLOGY-v0.1.md`
**Event taxonomy:** `EVENT_TAXONOMY-v0.1.md` (unchanged)
**Document hash:** published in `LOCK-MANIFEST-v0.2.json`

---

## 1. Scope and precedence

This document is a narrow amendment to `METHODOLOGY-v0.1.md`. It changes only
the endpoint-price construction in §2.3 and the availability rule that follows
from that construction. If this amendment and v0.1 conflict on one of those
subjects, this amendment controls for records carrying
`methodology_version = v0.2`. Every other v0.1 provision remains in force
without reinterpretation.

`EVENT_TAXONOMY-v0.1.md` remains the applicable taxonomy. No event class,
definition, threshold, horizon, response variable or classification procedure
is changed.

This amendment exists because the frozen v0.1 causal last-trade rule produced
zero usable calibration observations. `BSVS-FEAS-001` measured the failure and
`BSVS-FEAS-002` applied the predeclared endpoint-selection rule. The selected
endpoint is the shortest tested window not exceeding 30 minutes that yielded
at least 95% usable calibration dates under the 90-valid-returns-within-120-days
regime and at least 90% of BSV boundary windows with two or more trades. The
selected window is **30 minutes**. The tested 60-minute window is diagnostic
only and is not an alternative v0.2 endpoint.

## 2. Causal trailing-VWAP endpoint

For every asset and target timestamp `T`, the v0.2 endpoint price is aggregate
quote notional divided by aggregate base amount for all Gate trades in the
half-open trailing window:

```
T − 30 minutes < trade_timestamp ≤ T
```

A print after `T` is never eligible. A print exactly at `T−30 minutes` is not
eligible; a print exactly at `T` is eligible. The venue and six pairs remain
those named in v0.1 §2.3. There is no substitution, interpolation, candle,
fallback venue or use of a later print.

Gate archive columns are interpreted in their published order as
`timestamp, trade_id, price, amount, side`. For each contributing trade `i`,
quote notional is `price_i × amount_i`, and the endpoint is:

```
VWAP_T = Σ(price_i × amount_i) / Σ(amount_i)
```

`price` and `amount` are parsed directly from their archive decimal strings;
they are not first converted to binary floating point. A contributing row must
have a parseable timestamp, integer trade ID, strictly positive decimal price,
strictly positive decimal amount and a valid Gate side field. A malformed row
is a data-quality failure, not a reason to silently change the contributing
set. Duplicate archive rows with the same pair and trade ID are a data-quality
failure and are not double-counted.

An endpoint is **available** iff its causal window contains at least one valid
contributing trade and its total amount is positive. Otherwise it is missing.
The two-trade BSV threshold used by `BSVS-FEAS-002` was a feasibility selection
criterion only; it is not an additional live endpoint-availability rule.

The endpoint price replaces the v0.1 last-trade snapshot everywhere v0.1 uses
an asset price: forecast cutoffs, historical benchmark boundaries, event-time
prices and outcome endpoints. The 08:00 UTC daily clock is unchanged.

## 3. Deterministic decimal calculation

All VWAP inputs, products and sums are base-10 decimal quantities. Products
and sums are exact. Division uses a decimal context of 50 significant digits,
round-to-nearest with ties to even (`ROUND_HALF_EVEN`). No earlier rounding is
permitted. The stored VWAP is the resulting ordinary fixed-point decimal with
trailing fractional zeroes removed; scientific notation is forbidden and zero
is serialized as `0`.

When a v0.1 downstream calculation requires a numeric endpoint, it consumes
that canonical stored VWAP decimal. Returns, constituent standardization, OLS,
the exclusion of alpha from expected return, residual construction, sample
standard-deviation conventions and `D = 0.25 × σ(residual_24h)` remain exactly
as specified in v0.1. This amendment introduces no new estimator, response,
threshold or performance rule.

## 4. Contributing-trade audit fields and hashes

Every stored endpoint must include:

```
window_start_exclusive
window_end_inclusive
window_minutes                 30
trade_count
total_base_amount
total_quote_notional
vwap
contributing_trades_sha256
```

The contributing-trade hash is calculated independently for each pair and
endpoint. Contributing rows are sorted ascending by exact timestamp and then
ascending by integer trade ID. Each row is serialized as the five original
archive field strings, in Gate column order, joined by ASCII comma and
terminated by one LF byte:

```
timestamp,trade_id,price,amount,side\n
```

The SHA-256 digest of the concatenated UTF-8 bytes is stored as 64 lowercase
hexadecimal characters. No header, filename, gzip metadata, endpoint metadata
or noncontributing row enters this digest. The original field strings are not
decimal-normalized for the trade-set hash. The archive filename and archive
SHA-256 remain part of calibration provenance.

The same audit fields and hashing rule apply to calibration, live forecast,
event and outcome endpoints.

## 5. Estimation window preserved from v0.1

The v0.2 benchmark preserves the original v0.1 primary calibration and
beta-estimation design: **exactly 90 consecutive complete daily returns ending
at cutoff `T`**, using only information available at `T`. All six assets must
have available v0.2 endpoints at every boundary needed to construct those 90
returns. OLS and constituent standardization use the same 90 consecutive
returns.

If any required endpoint is missing, the observation is unusable. Do not
interpolate, substitute, use nonconsecutive returns, shorten the estimation
sample or extend the estimation period. No fallback estimation regime is part
of the v0.2 computational specification.

## 6. Frozen calibration dates and exclusions

The calibration calendar is unchanged:

```
required endpoint history start   2025-06-03T08:00:00Z
first calibration date            2025-09-01T08:00:00Z
last calibration date             2026-02-27T08:00:00Z
last outcome endpoint              2026-02-28T08:00:00Z
candidate calibration dates        180
```

For each candidate date `T`, fit the frozen v0.1 benchmark on exactly 90
consecutive complete daily returns, freeze the estimated quantities, and
measure the unchanged `T` to `T+24h` residual. A date is excluded if it lacks
the complete consecutive estimation window, either outcome endpoint is missing
for any asset, or an input fails the data-quality rules. No excluded date is
replaced.

The final v0.2 calibration report must give usable and excluded date counts
under this preserved consecutive-return design. Calibration inputs, endpoint
audit fields, fitted quantities, residual series and final output are hashed
and published through the v0.2 lock manifest.

## 7. No retrospective changes

The v0.1 methodology, its failed calibration result and every v0.1 artifact
remain immutable. They are not recomputed under this amendment or relabeled as
v0.2. Records created under v0.2 identify `methodology_version = v0.2`; version
reporting remains separate as required by v0.1 §6.

## 8. Computed calibration values

These are mechanical outputs inserted after CAL-002 to satisfy the inherited
v0.1 pre-lock result-publication requirement. No methodological rule is changed.

residual_sigma:
0.023408918284644597

deadband_value:
0.0058522295711611492

calibration_observations_used:
180

calibration_observations_excluded:
0

calibration_dataset_sha256:
337642e364714b5cab11f1ab2511cc5ffad1e2b51247160e415d81d05e04380e

calibration_endpoint_dataset_sha256:
9f5930456d1b45a9704a25bce8d5839637c9a9f7a7036597710e226068f6e9b2
