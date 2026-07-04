# cella

**Resonant dequantizer — the spectrum as a chamber.**

A noise-driven resonator bank, companion and dual to **aliquoto**
(`../aliquoto/`): aliquoto writes a spectrum into silence (sum of sines);
cella writes the same spectrum onto noise (bank of resonances). Same `a(f)`,
opposite substrate. Partials keep aliquoto's full dequantized freedom — any
ratio of the fundamental, integer, irrational, subharmonic — but each line now
has a **width**, and width is the whole new instrument.

*cella* = the inner chamber of a temple, the room that houses the deity —
a resonant enclosure whose *modes* answer whatever sound enters. Pun on
**cello** intended. Like Horn of Plenty, the name names the vessel, not the
power: aliquoto is the string, cella is the room.

Sibling triad (the suite's three ways to own a spectrum):

- **aliquoto** — linear, exact: the spectrum *specified*. `y = Σ sines`.
- **horn of plenty** — statistical: the spectrum as *distribution* (stationary substrate).
- **cella** — resonant: the spectrum as *response*. Excitation × transfer function.

The chimera (working name reserved: **Fano** — the Fano resonance, a discrete
state interfering with a continuum, which is exactly sine partials + driven
partials sharing one spectrum) is not a future mashup. It is the source–filter
model these three were always inside: aliquoto = line-spectrum source,
horn of plenty = arbitrary broadband excitation, cella = transfer function.
Design the seams now, get the chimera free later.

---

## Philosophy

- **One recursion, whole chimera.** There is no "sine partial" and "reso
  partial" as separate types. One complex one-pole state update covers both
  (see Core unit). A pure sine is a resonator with zero damping and no drive.
  Per-partial sine-vs-reso is parameter values, not modes.
- **The equation is still the interface.** Cella reuses aliquoto's spectrum
  grammar, extended minimally (a Q column, a `q :` line). You type the
  chamber's mode structure as mathematics.
- **Release is physics, not a parameter.** The ADSR gates the *drive*, not the
  output. Release = stop bowing: each partial rings out at its own `1/γ`.
  Narrow lines ring long, wide lines die fast — per-partial release for free,
  physically coherent. This alone separates cella from every subtractive synth.
- **Dequantize the lineshape.** Most subtractive synths give you one filter
  topology. Cella gives every partial its own homogeneous width (Lorentzian)
  and inhomogeneous spread (Gaussian ensemble) — the two broadening mechanisms
  of real spectroscopy, as timbral controls.

---

## Core unit

One state update per partial per sample:

```
s[n+1] = p · s[n] + g · drive[n]      // s complex, p = exp((−γ + iω)·dt)
```

- `γ > 0`, `drive = noise` → damped driven resonator. Bandwidth = `γ/π` Hz.
- `γ = 0`, `drive = 0`, `s` initialized at amplitude → **exact sine oscillator**
  (coupled-form). That *is* an aliquoto partial in the σ→0 limit.

Cost per partial: one complex multiply-add — same class as aliquoto's sine
bank. Same per-voice AudioWorklet architecture as aliquoto's `WORKLET_SRC`,
same voice seam (`startNote(id,hz,vel)` / `bendNote` / `stopNote`; every input
surface just produces Hz).

### Lineshape physics (the load-bearing correction)

A driven damped oscillator has a **Lorentzian** line, not Gaussian. Gaussian
lines are *inhomogeneous* broadening — an ensemble of detuned oscillators.
Cella exposes both, because they sound different:

- **1 pole** → Lorentzian: long spectral tails, rings clean.
- **K micro-detuned poles sharing one drive** → Gaussian-ish line,
  chorus-thick (inhomogeneous ensemble).
- Both at once → **Voigt profile** (Lorentzian ⊛ Gaussian).

So the per-partial width controls are two, not one: `γ` (homogeneous width /
Q) and ensemble spread (inhomogeneous). Dirac delta = both zero = sine.

---

## Excitation

- **v1 sources:** white / pink noise, per voice.
- **Common vs independent** (one toggle): all partials share one noise stream
  (one breath, one bow — correlated responses, real interference/beating
  between overlapping lines) vs independent streams per partial (clean, no
  beating). Physical vs hygienic.
- **Input port designed now, filled later:** an arbitrary drive buffer is the
  chimera seam. Horn of Plenty's stationary substrate is the intended first
  guest (its open questions "grain-vs-consumer crossover" and "spread default"
  are answered by this consumer — coordinate when wiring).
- ADSR gates the drive (see Philosophy). `R` becomes an optional override;
  the natural release is `1/γ` per partial.

---

## Grammar (aliquoto grammar + width)

Reuse aliquoto's grammar wholesale; extend:

```
sum n=1..N : r(n) : a(n) : p(n) : q(n)     # Q column
q : f(r)                                    # global Q line, env-style
```

- **Canonical width storage: Q (dimensionless).** Constant-Q keyfollow is the
  default → self-similar timbre across pitch (roughly what strings do).
  Constant-Hz bandwidth is one expression away (`q : hz/50`) since `hz, f0, r`
  are in scope.
- **`Q = ∞`** (notation `q:*` or omitted) = sine partial. This is the chimera
  notation, already present in v1: some lines ring pure, some are driven.
- **Phase column** is meaningful only at `Q=∞`; a noise-driven partial has no
  phase. Keep the column, ignore it when driven.
- Second width (inhomogeneous ensemble spread) — column vs `spread :` line:
  decide during build.

---

## Sharing / dependencies

The spectrum grammar parser becomes a **shared artifact** the moment it is
copied here. House pattern applies (tabota-resolve precedent): vendored copy
with header stamp + hash-freshness chip, and an entry in `../DEPENDENCIES.md`
in the same sitting. Two consumers → copy, don't extract a module yet.

---

## Status

**v1 spine shipped (2026-07-03), single self-contained `index.html`.** Working:
resonator-bank AudioWorklet (complex one-pole per partial), noise drive
(white/pink · common/independent), ADSR-gates-drive with **natural ring-out on
release** (verified: signal keeps ringing after key-up, decays at 1/γ), grammar
port + Q column + `q :` law line + `Q=*` pure-sine chimera notation, emission-
plate spectrum (linewidth rendered as Lorentzian glow, sines as hairlines),
piano + QWERTY surfaces, n-EDO tuning, 6 presets incl. a Chimera demo (struck
sines + rung modes in one spectrum). Temple/organ/spectroscopy skin on the
`stone.webp` ground: bronze drawstops, carved `Y=` inscription tablet, plum-
black plate.

**Cluster A added (2026-07-03, v1.1):**
- **Inhomogeneous ensemble → Voigt (build step 4 done).** Each driven line is
  now `K` micro-detuned sub-poles (Gaussian-quantile spaced, `qnorm` host-side)
  sharing the line; `spread¢` + `voices K` controls. spread 0 / K 1 = pure
  Lorentzian; wide spread = Gaussian bloom; Q ⊛ spread = Voigt. Sub-pole states
  flattened `n·K` in the worklet; sine modes ignore K. Plate glow now shows the
  combined Voigt half-width (Lorentzian 1/Q ⊕ Gaussian spread in quadrature).
- **Q× fix.** The old "Q base" only filled *unset* partials, so presets with
  explicit Q showed no response — looked broken. Replaced with a global **Q×
  multiplier** (0.1–6) applied to every mode; unset modes fall to a constant
  `DEFAULTQ=40`. Verified audibly (✕1 vs ✕4 ring residual 0.014 → 0.095).
- **MIDI** — `requestMIDIAccess`, note on/off → velocity, CC123 all-notes-off,
  `MIDI ● n` status. (Untestable in headless preview → shows `✕` there.)
- **Audio-init hardening (fix).** First report: MIDI on → keys dead, no notes.
  Root cause was general, not MIDI: `realizeNote` throws if the AudioWorklet
  module isn't registered yet (Cella has no oscillator fallback), and MIDI
  events create a *suspended* context from a non-gesture. Fix: `initAudio` is
  now one idempotent awaited promise; `startNote` bails unless
  `AC&&workletReady` and is try-wrapped; note-on tracks a `held` set and fires
  once audio is ready (press-before-load no longer lost, quick taps don't
  stick). MIDI verified end-to-end by simulation.
- **Q× is now log-scaled** (slider = log10 exponent, ✕0.1 … ✕10). Note: past
  ~Q 100 the line is so narrow that timbre/plate converge — the useful spread
  lives roughly ✕0.1 … ✕1; kept anyway.
- **Enable-audio fix (round 2).** The promise refactor trapped the label update
  + `resume()` inside the create-block, so if MIDI (non-gesture) made the
  context first, clicking *enable audio* did nothing visible → "can't enable".
  Now `enableAudio()` creates the context only on a real gesture (so it starts
  *running*, not suspended), and resumes + refreshes the label on *every* call.
  MIDI-before-enable shows a `MIDI ▸ click page to enable` nudge instead of
  wedging. Verified: fresh enable, suspended→resume relabel, MIDI cycle.
- **Envelope** relabelled to plain **ADSR** (attack/decay/sustain/release);
  still gates the *drive* (the physics identity is kept). Live: slider changes
  post `{type:'adsr'}` to sounding voices.

**Still deferred:** spectrum graphic editing (aliquoto selection/override/bake
port — its own session), chthonic/stonepunk reskin (its own session), external
drive-buffer input (Horn of Plenty → Fano — step 6), Tabota note model /
`.tabota` I/O, hex/ribbon surfaces, r(t) dynamics, per-partial ensemble/ADSR.
`window.__cella` debug hook (voices/AC/rms) left in for verification.

Design record (aesthetic direction + uniqueness analysis) from the 2026-07-03
naming/architecture session is above and in the git-less session history.

## Build order (v1 spine)

1. Complex-phasor worklet: partial bank with `(ω, γ, g)` per partial; verify
   γ=0 undriven case reproduces aliquoto's sine (chimera degeneracy test —
   this is the load-bearing invariant).
2. Noise drive (white/pink; common/independent toggle), ADSR on drive,
   ring-out release.
3. Grammar port + Q column + `q :` line; vendor parser, stamp, DEPENDENCIES.md.
4. Inhomogeneous ensemble (K micro-detuned poles per partial).
5. Surfaces / tuning: lift from aliquoto (piano / hex / ribbon, n-EDO).
6. Later: external drive buffer input (Horn of Plenty substrate) → Fano.

## Open questions

- Skin / device identity — aliquoto is the cursed LCD calculator; what is the
  chamber? (Stone? Nave? Anechoic-inverted?) Undecided, don't rush it.
- Inhomogeneous spread: per-partial column vs global `spread :` line vs both.
- Ensemble size K: fixed small (3–7) vs user-set vs derived from spread.
- Tabota note model: adopt aliquoto's `noteToEvent` wholesale? (Probably yes —
  spectrum rides in `payload` the same way.)

---

*spine: the room answers with its own lines, and stops answering slowly.*
