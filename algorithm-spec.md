# DAILY MAX v2 — Algorithm Specification
## Evidence-Based Readiness Estimation for RPE-Prescribed Powerlifting

---

## 1. e1RM Estimation (Baseline Strength Tracking)

### Formula: Blended Epley-Brzycki with RPE Adjustment

Every working set the lifter logs produces an estimated 1RM. We blend two formulas
for improved accuracy across rep ranges.

**RPE Adjustment (Tuchscherer principle):**
The lifter reports RPE after each set. RIR = 10 - RPE. Effective reps = performed reps + RIR.
This treats a set of 4 @ RPE 8 (2 RIR) identically to a set of 6 @ RPE 10 (0 RIR).

```
Source: Tuchscherer RPE table; Zourdos et al. (2016) — RIR-based RPE scale.
         %1RM(reps, RPE) = REPMAX(reps + (10 - RPE))
         (Exodus Strength forum, Tuchscherer's original formulation)
```

**Epley (most accurate at 1-5 reps):**
```
e1RM_epley = weight × (1 + 0.0333 × effective_reps)

Source: Epley (1985). DiStasio (2014) validated: Epley within 2.7kg from 3RM loads.
```

**Brzycki (most accurate at 3-6 reps, more conservative):**
```
e1RM_brzycki = weight / (1.0278 - 0.0278 × effective_reps)

Source: Brzycki (1993). DiStasio (2014): Brzycki within 3.1kg from 5RM loads.
```

**Blended e1RM:**
```
e1RM = (e1RM_epley + e1RM_brzycki) / 2

Source: Gemini report §4.1 — "Averaging multiple formulas is a mathematically sound
         strategy to smooth out the extremes of linear and exponential estimations."
         Also supported by arvo.guru comparison: "For accuracy, use the average of
         all three formulas."
```

**Baseline e1RM** = rolling average of last 3-4 session e1RMs for the same lift.
This creates a "chronic strength" baseline that naturally drifts upward with progression.

**Edge cases:**
- If effective_reps = 0 (i.e., single @ RPE 10), e1RM = weight.
- If effective_reps > 12, accuracy degrades significantly — flag to user but still calculate.
- Minimum 2 sessions required before baseline is meaningful; before that, use training max.

---

## 2. Velocity-Based Readiness (Primary Load Governor)

### Calibration Protocol

```
Source: Gemini report §5.1 — "The 1-point method utilizing a fixed percentage of the
         training max is the most efficient protocol for the solo lifter."
```

**Calibration set parameters:**
- Load: 70-75% of baseline e1RM (NOT training max from months ago — the living e1RM)
- Reps: 2-3, with MAXIMAL intentional concentric velocity
- Record: Peak concentric velocity of the FASTEST rep (not average of all reps)

```
Source: Gemini report §5.1 — "The application should track the fastest repetition (peak
         concentric velocity of the best rep) to eliminate artifacts from poor un-racking
         mechanics or a loss of brace on the first repetition."
```

**Load justification:**
```
Source: Gemini report §5.1 — "Loads below 60% are overly influenced by variations in
         intentional acceleration and technical groove. Loads above 80% incur a metabolic
         and neurological cost. The 70-75% range resides in the optimal bandwidth."
```

### Z-Score Calculation

```
velocity_zscore = (today_velocity - rolling_avg_velocity) / rolling_stdev_velocity

Rolling window: 30 days (same lift, same calibration load)

Source: Gemini report §6.1 — "By converting the morning subjective wellness score and
         the warm-up bar velocity into Z-scores, the variables are standardized on a single
         scale."
         Gemini report §3.3 — "A 7-day window is too short, as it is overly influenced by
         acute microcycle fatigue, while a 30-day window effectively captures chronic
         fitness levels."
```

**Minimum data requirement:** Need ≥5 sessions to compute a meaningful Z-score. Before
that, use percentage deviation from first recorded velocity as a proxy.

### Velocity Deviation Percentage

For the traffic light thresholds, we convert Z-score back to a percentage deviation:
```
velocity_pct_deviation = ((today_velocity - rolling_avg_velocity) / rolling_avg_velocity) × 100
```

This is more intuitive for the lifter than raw Z-scores.

---

## 3. Subjective Readiness (Volume Governor)

### 3-Item Modified Hooper Index

```
Source: Gemini report §3.5 — "Integrating a 3-item subset of the Hooper Index (Systemic
         Fatigue, Sleep Quality, Mental Stress) captures the autonomic and non-training
         stressors that physical velocity metrics cannot forecast."
         Saw et al. (2016) systematic review — subjective measures more sensitive than
         objective physiological markers.
         SimpliFaster (Fields et al.) — fatigue and soreness correlated at r=0.59 (redundant),
         stress and mood correlated at r=0.54 (redundant). Hence 3 items, not 5.
```

**Questions (scored 1-5):**
1. **Sleep Quality**: "How restorative was last night's sleep?" (1=Terrible, 5=Excellent)
2. **Systemic Fatigue**: "How heavy/sore/drained does the body feel?" (1=Wrecked, 5=Fresh)
3. **Mental Stress**: "Current level of life/psychological stress?" (1=Extreme, 5=None)

**Sum**: 3 (worst) to 15 (best)

**Z-Score:**
```
subjective_zscore = (today_sum - rolling_avg_sum) / rolling_stdev_sum

Rolling window: 14 days

Source: Gemini report §9.1 — "The system automatically compares today's score to the
         14-day rolling average to generate a subjective Z-score."
```

### Volume Recommendation (from subjective data)

```
Source: Gemini report §6.2 — "Objective VBT data should act as the primary governor of
         mechanical load (the weight on the bar), while subjective data serves as the context
         for altering set volume or accepting higher RPEs."
```

Decision rules:
- subjective_zscore > -0.5:  FULL VOLUME — execute all prescribed sets + accessories
- subjective_zscore -0.5 to -1.5: CAPPED VOLUME — do prescribed working sets, reduce accessories by 50%
- subjective_zscore < -1.5: REDUCED VOLUME — cap working sets at RPE 7, cut accessories entirely

---

## 4. Traffic Light Decision Matrix (Readiness Status)

```
Source: Gemini report §9.1, Table — exact thresholds from the report.
```

| Status    | Velocity Deviation    | Action (Load)                           |
|-----------|-----------------------|-----------------------------------------|
| GREEN     | > +3% above 30d avg  | May increase working weight 2.5-5%      |
| AMBER     | -5% to +3%           | Execute as prescribed                    |
| RED       | -5% to -10%          | Drop working weight 5-10%               |
| CRITICAL  | < -10%               | Deload: -10%+, cap RPE 7, cut volume    |

### Lift-Specific Sensitivity Adjustment

```
Source: Gemini report §2.3 — "The deadlift imposes the highest mechanical and systemic
         demands on the CNS. Heavy deadlift sessions cause prolonged suppression of both
         bar velocity and systemic power output for up to 48 hours. Because the concentric
         phase of the deadlift lacks an initial eccentric stretch-shortening cycle (SSC),
         starting velocity is heavily reliant on raw RFD and peak CNS arousal, making it
         highly susceptible to daily readiness drops compared to the squat."
```

**Deadlift adjustment:** RED threshold shifts from -5% to -4%, and the readiness modifier
is scaled 1.25× more aggressively:
```
if (lift === 'deadlift') {
  RED threshold = -4% (instead of -5%)
  modifier_magnitude *= 1.25
}
```

**Bench press:** Standard thresholds apply. Shorter local recovery but standard CNS impact.

**Squat:** Standard thresholds apply. Benefits from SSC stretch reflex.

---

## 5. Readiness Modifier (Continuous, Not Stepped)

The traffic light is a human-readable label. The actual weight adjustment is a continuous
function derived from velocity percentage deviation.

```
// Clamp velocity deviation to actionable range
clamped_dev = clamp(velocity_pct_deviation, -15%, +8%)

// Linear interpolation within range:
//   -15% → modifier 0.85
//   -10% → modifier 0.90
//    -5% → modifier 0.95
//     0% → modifier 1.00
//    +3% → modifier 1.025
//    +8% → modifier 1.05
readiness_modifier = 1.0 + (clamped_dev / 100)

// But we don't want 1:1 mapping — research suggests the modifier should be
// dampened because true 1RM only varies 1-2% (Gemini report §2.1).
// The velocity drop reflects PERCEPTION and RFD, not absolute strength loss.
// So we scale by 0.5 to avoid over-correction:

readiness_modifier = 1.0 + (clamped_dev / 100) × 0.5

// Result: a -10% velocity day → modifier of 0.95 (not 0.90)
//         a +6% velocity day  → modifier of 1.03 (not 1.06)

Source: Gemini report §2.1 — "True physiological maximal strength is remarkably stable,
         fluctuating by merely 1% to 2% within a standard microcycle. However, the
         perception of effort (RPE) and the velocity at which submaximal loads are moved
         can vary substantially."

// Apply deadlift sensitivity multiplier
if (lift === 'deadlift') {
  deviation_for_modifier = clamped_dev × 1.25
  readiness_modifier = 1.0 + (deviation_for_modifier / 100) × 0.5
}
```

**Why 0.5× dampening?** Because the goal is to adjust load such that the PRESCRIBED RPE
is achieved. A -10% velocity drop doesn't mean you're 10% weaker — it means the same
load will *feel* harder. The 0.5× factor maps velocity deviation to the approximate RPE shift,
which is what we're correcting for.

---

## 6. Working Weight Calculation (The Core Output)

```
working_weight = baseline_e1rm × rpe_table_pct(reps, target_rpe) × readiness_modifier

Round to nearest 2.5kg.
```

### Tuchscherer RPE-Percentage Table (Default)

```
Source: Tuchscherer, Reactive Training Systems.
         Gravitus, StrengthLog, rpecalculator.com all use this table.
         Gemini report §4.2 — the RTS model maps proximity-to-failure against relative
         intensity.
```

The table maps (reps, RPE) → percentage of 1RM:

| Reps | RPE 6  | RPE 6.5 | RPE 7  | RPE 7.5 | RPE 8  | RPE 8.5 | RPE 9  | RPE 9.5 | RPE 10 |
|------|--------|---------|--------|---------|--------|---------|--------|---------|--------|
| 1    | 0.880  | 0.895   | 0.910  | 0.920   | 0.930  | 0.945   | 0.960  | 0.978   | 1.000  |
| 2    | 0.855  | 0.870   | 0.885  | 0.895   | 0.905  | 0.920   | 0.935  | 0.955   | 0.978  |
| 3    | 0.830  | 0.845   | 0.860  | 0.870   | 0.880  | 0.895   | 0.910  | 0.930   | 0.955  |
| 4    | 0.808  | 0.820   | 0.835  | 0.845   | 0.855  | 0.870   | 0.885  | 0.905   | 0.930  |
| 5    | 0.785  | 0.798   | 0.810  | 0.820   | 0.830  | 0.845   | 0.860  | 0.880   | 0.905  |
| 6    | 0.763  | 0.775   | 0.788  | 0.798   | 0.808  | 0.820   | 0.835  | 0.855   | 0.880  |
| 7    | 0.740  | 0.753   | 0.765  | 0.775   | 0.785  | 0.798   | 0.810  | 0.830   | 0.855  |
| 8    | 0.720  | 0.733   | 0.745  | 0.755   | 0.765  | 0.775   | 0.788  | 0.808   | 0.830  |

**Individualisation over time:**
```
Source: Gemini report §4.3 — "Lifters must calibrate their own tables by tracking actual
         rep-max performances against historical RPE ratings over a mesocycle."
```

As the lifter logs sets (weight, reps, RPE), we calculate what percentage of their baseline
e1RM that set represented. Over time, we can adjust the table entries to match THEIR
actual reps-at-percentage relationship.

Algorithm:
1. For each logged set: actual_pct = weight / baseline_e1rm
2. Store (reps, rpe, actual_pct) tuples
3. After ≥20 data points for a given (reps, rpe) bucket (±0.5 RPE), update that cell
   with the median of observed percentages.
4. Blend: personalised_pct = 0.7 × observed_median + 0.3 × default_table_value
   (to prevent over-fitting to noisy data early on)

---

## 7. Minimum Velocity Thresholds (Per-Lift)

```
Source: Gemini report §5.2, citing Helms et al. (2017), González-Badillo & Sánchez-Medina:
  - Squat MVT:   0.28 – 0.34 m/s (sticking point requires momentum)
  - Bench MVT:   0.15 – 0.17 m/s (short ROM, can grind)
  - Deadlift MVT: 0.14 – 0.17 m/s (no SSC, extremely grindable)

Gemini report §5.2 — "Establishing an individual MVT by tracking the velocity of RPE 10
sets over time yields vastly superior 1RM estimations than relying on group averages."
```

**Defaults (midpoint of ranges):**
- Squat: 0.31 m/s
- Bench: 0.16 m/s
- Deadlift: 0.15 m/s

User can override with personal MVT once they have enough RPE 9.5-10 data points.

MVTs are used for:
1. Display context ("your calibration velocity of 0.52 m/s at 70% suggests ~X% of 1RM")
2. Future: if the user ever wants to use the velocity-based 1RM prediction (extrapolate
   to MVT from calibration velocity), the MVT is needed as the intercept.

---

## 8. Recovery Flagging

```
Source: Gemini report §2.3 — "Heavy deadlift sessions cause prolonged suppression of both
         bar velocity and systemic power output for up to 48 hours."
```

**Rules:**
- If deadlift session is <48hrs after previous deadlift: WARN "Insufficient recovery for deadlift — consider heightened readiness awareness"
- If squat session is <24hrs after heavy squat or deadlift: WARN
- Bench: no minimum flagging (shorter local recovery)

**Implementation:** Track timestamps per lift. Compare current session time to last
session time for same lift (and for deadlift, also check last squat session given shared
posterior chain demand).

---

## 9. Circadian Awareness

```
Source: Gemini report §2.2 — "Neuromuscular performance, joint compliance, and maximal
         isometric strength consistently map to core temperature peaks [late afternoon].
         Comparing a morning warm-up velocity to an evening rolling average will introduce
         a confounding circadian variable."
```

**Implementation:**
- On setup, user enters typical training time (e.g., "17:00-19:00")
- If current session time deviates >3 hours from typical: display warning
  "Training at an unusual time — velocity comparison may be less reliable"
- Future enhancement: segment rolling averages by time-of-day bucket
  (AM vs PM) if sufficient data exists

---

## 10. Acute:Chronic Workload Ratio

```
Source: Gemini report §6 references the Fitness-Fatigue Model (FFM).
         Shrier et al. PMC7575491 — readiness as a stochastic component of the FFM.
```

**Simplified implementation for a powerlifter:**

Track "estimated stimulus" per session:
```
session_load = Σ (weight × reps × sets) for all working sets in a session
```

```
acute_load  = sum of session_loads in last 7 days
chronic_load = average weekly session_load over last 28 days (i.e., sum/4)
ACWR = acute_load / chronic_load
```

**Thresholds:**
- ACWR 0.8 – 1.3: Sweet spot. No flag.
- ACWR > 1.5: WARN "Acute spike — high injury/overreach risk"
- ACWR < 0.6: WARN "Significant detraining — consider increasing volume"

---

## 11. Camera Setup Checklist

```
Source: Gemini report §5.3 — filming best practices for computer vision accuracy.
```

Display before first calibration set each session:
1. Phone on tripod or stable surface (NOT hand-held)
2. Camera perpendicular to barbell sleeve (0-10° max)
3. Lens at waist height for squat/bench, knee height for deadlift
4. Good lighting, no direct glare on plates
5. Record with MAXIMAL concentric intent

---

## 12. VBT App Recommendation

```
Source: Gemini report §7 —
  - MyLift: "84% failure rate on bench press" for rep detection. NOT RECOMMENDED.
  - Metric VBT v4.5+: "demonstrated low systematic error and strong correlations with
    commercial LPTs for bench press, provided recording parameters are strictly controlled."
  - Qwik VBT: "exceptional accuracy in recent PLOS ONE validation studies."
```

Show on setup: "Recommended VBT apps: **Metric VBT** (v4.5+) or **Qwik VBT**.
Avoid MyLift for bench press (84% rep detection failure rate in validation studies)."

---

## 13. Post-Set Feedback Loop

After each working set, the lifter logs:
- Weight used (pre-filled from suggestion)
- Reps completed
- Actual RPE

This produces a new e1RM data point:
```
effective_reps = reps + (10 - actual_rpe)
e1rm_epley = weight × (1 + 0.0333 × effective_reps)
e1rm_brzycki = weight / (1.0278 - 0.0278 × effective_reps)
e1rm_new = (e1rm_epley + e1rm_brzycki) / 2
```

The baseline e1RM (rolling average of last 3-4 sessions) updates to include this session.

Additionally, the (reps, actual_rpe, weight/baseline_e1rm) tuple feeds into the
personal RPE table calibration (§6).

---

## Summary of Evidence Mapping

| Algorithm Component           | Primary Source                                    |
|-------------------------------|---------------------------------------------------|
| Blended Epley/Brzycki         | DiStasio (2014), Gemini §4.1                      |
| RPE→RIR→effective reps        | Zourdos et al. (2016), Tuchscherer RTS            |
| 1-point calibration at 70-75% | Gemini §5.1                                       |
| Fastest rep velocity          | Gemini §5.1                                       |
| 30-day velocity rolling avg   | Gemini §3.3                                       |
| 14-day subjective rolling avg | Gemini §9.1                                       |
| Z-score normalisation         | Gemini §6.1                                       |
| 3-item Hooper subset          | Gemini §3.5, Saw et al. (2016), Fields et al.     |
| Traffic light thresholds      | Gemini §9.1 table                                 |
| Decision tree not weighted avg| Gemini §6.2                                       |
| VBT→load, subjective→volume   | Gemini §6.2                                       |
| 0.5× dampening factor         | Derived from Gemini §2.1 (1-2% true variance)     |
| Deadlift sensitivity ×1.25   | Gemini §2.3 (no SSC, highest CNS demand)           |
| MVTs per lift                 | Gemini §5.2, Helms et al. (2017)                  |
| 48hr deadlift recovery        | Gemini §2.3                                       |
| Circadian warning             | Gemini §2.2                                       |
| ACWR                          | FFM model, Shrier framework (PMC7575491)          |
| RPE table individualisation   | Gemini §4.3, Tuchscherer                          |
| Camera checklist              | Gemini §5.3                                       |
| App recommendation            | Gemini §7 (MyLift failures, Metric/Qwik validity) |
| Personal RPE table evolution  | Gemini §4.3                                       |
