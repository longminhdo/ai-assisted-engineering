# DESIGN.md — "Signal" Taste Skill
### AI-Assisted Frontend Engineering — Training Deck

This file is the design system this deck (and any future deck for this curriculum) must follow. Section 1-6 are the reusable **taste skill** — the rules. Section 7 is the **layout mapping** — the concrete slide-by-slide plan. Nothing gets coded until this is approved.

---

## 0. Design Read

> Reading this as: an internal engineering-training deck for a frontend team, speaker-led, with a modern AI-tool language — leaning toward a custom dark-slate system with cinematic GSAP-driven transitions instead of any off-the-shelf template.

**Format:** fixed 16:9 stage, 1920×1080, single self-contained HTML file (per `frontend-slides` architecture). Click/arrow-key driven, not scroll-driven — GSAP is used for slide-switch choreography and in-slide reveal sequencing, not `ScrollTrigger` pin/scrub (there is no scroll).

**Density mode:** speaker-led / low-density. One idea per slide, large type, generous negative space, presenter fills the gaps live.

---

## 1. The Three Dials (locked by user brief)

| Dial | Value | Meaning here |
|---|---|---|
| `DESIGN_VARIANCE` | **8** | Asymmetric grids, fractional columns, off-grid numerals, no centered-hero default. No layout family repeats twice. |
| `MOTION_INTENSITY` | **7** | Advanced choreography tier — every slide transition and every content reveal is GSAP-timed, staggered, eased. Not static, not chaotic. |
| `VISUAL_DENSITY` | **4** | Art-gallery spacing. `120px` slide margins, one focal idea per slide, never more than ~5 text elements on screen at once. |

These are locked. Do not drift toward centered symmetry, toward flat CSS-transition-only motion, or toward denser slides as the deck grows — variance/motion/density hold constant from slide 1 to slide 33.

---

## 2. Color System — Monochromatic Slate + One Signal

One accent, used identically everywhere it appears (Color Consistency Lock). No purple/blue "AI gradient" cliché — the accent is a warm signal color instead.

```css
:root {
  /* Slate scale — the monochrome base, all cool near-blacks/grays, no warm beige */
  --stage-bg:      #050608;  /* letterbox behind the 16:9 canvas */
  --bg-primary:    #0A0C10;  /* default slide background */
  --bg-elevated:   #14171D;  /* card / panel surface, one step up */
  --bg-elevated-2: #1C2028;  /* second elevation — used sparingly (active states, code blocks) */
  --border-hair:   rgba(255,255,255,0.08);
  --border-hair-2: rgba(255,255,255,0.14);

  --text-primary:   #F3F5F7;
  --text-secondary: #9AA1AC;
  --text-tertiary:  #5C6270;

  /* The one signal color. Warm orange — reads as "alert / active / alive"
     without touching the purple-blue AI cliche. Locked across all 33 slides. */
  --accent:        #FF5A1F;
  --accent-soft:   rgba(255, 90, 31, 0.14);   /* tinted backgrounds, never a flat fill */
  --accent-line:   rgba(255, 90, 31, 0.45);   /* accent borders/underlines */

  /* Used ONLY on the 2-3 highest-emphasis elements in the whole deck
     (loop diagram core, human-review gates, final quote mark) — not a global effect. */
  --accent-glow:   0 0 60px rgba(255, 90, 31, 0.25);
}
```

**Rules:**
- No pure `#000000` / `#FFFFFF` anywhere (off-black / off-white only).
- Accent never appears as a gradient. Solid fills or tinted 8-14% backgrounds only.
- Glow (`--accent-glow`) is reserved for exactly three moments across the whole deck (flagged in Section 7) — not a default treatment. Everywhere else, elevation is a tinted shadow (`0 20px 40px rgba(0,0,0,0.4)`), never a neon outer glow.
- Shape Consistency Lock: panels/cards/diagram boxes use `4px` radius (reads sharp/engineering, not soft-SaaS). Tags, pills, and the nav progress dots use full pill radius. No other radius values anywhere.

---

## 3. Typography

Display and body pair — no Inter, no Space Grotesk (both banned as this generation's default). Mono is reserved for code, diagram labels, and the numeric section markers — never for eyebrows on every slide.

```css
:root {
  --font-display: 'Cabinet Grotesk', sans-serif;  /* Fontshare — headlines, section numerals */
  --font-body:    'Switzer', sans-serif;          /* Fontshare — body copy, captions */
  --font-mono:    'JetBrains Mono', monospace;    /* Google Fonts — code, diagram labels, kickers */

  --size-hero:     168px;  /* title slide only */
  --size-display:  108px;  /* section-divider numerals */
  --size-h1:        72px;  /* slide titles */
  --size-h2:        44px;  /* sub-headers within a slide */
  --size-body-lg:   30px;
  --size-body:      24px;
  --size-small:     18px;
  --size-micro:     14px;  /* mono labels only */
}
```

**Emphasis rule:** to punch a word inside a headline, use `font-style: italic` or heavier weight of the *same* family — never drop in a serif or a different font family mid-headline.

**Eyebrow discipline:** mono micro-labels (`--size-micro`, uppercase, `letter-spacing: 0.18em`) appear on at most **1 in 3 slides** (per the anti-slop budget) — reserved for section dividers and diagram axis labels, never pasted above every slide title out of habit.

---

## 4. Motion System

Three transition **modes**, chosen by what the slide-change means — never randomized per-slide, always motivated:

| Mode | When | What it communicates |
|---|---|---|
| **Cut** (default) | Topic -> topic inside the same section | Continuity — we're still in the same conversation |
| **Wipe** | Entering a new section divider | A chapter boundary — deliberate, not incidental |
| **Dissolve** | Into/out of the two closing quote slides | Gravitas — slow down, let it land |

### 4.1 Slide-switch skeleton (GSAP core, no ScrollTrigger — this is click-driven)

```js
gsap.registerPlugin(); // no ScrollTrigger needed; deck advances on click/key, not scroll

const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

function goToSlide(nextIndex, mode = 'cut') {
  const slides = document.querySelectorAll('.slide');
  const from = slides[currentIndex];
  const to = slides[nextIndex];
  if (reduceMotion) { swapInstant(from, to); return; }

  const tl = gsap.timeline({
    defaults: { ease: 'expo.out' },
    onComplete: () => { from.classList.remove('active'); revealContent(to); }
  });

  if (mode === 'cut') {
    gsap.set(to, { autoAlpha: 0, scale: 1.04, y: 24 });
    to.classList.add('active');
    tl.to(from, { autoAlpha: 0, scale: 0.97, y: -16, duration: 0.5 }, 0)
      .to(to,   { autoAlpha: 1, scale: 1,    y: 0,   duration: 0.6 }, 0.08);

  } else if (mode === 'wipe') {
    // Accent panel sweeps left->right covering `from`, then reveals `to` as it exits right
    const panel = document.querySelector('.wipe-panel');
    to.classList.add('active');
    gsap.set(panel, { xPercent: -100 });
    gsap.set(to, { autoAlpha: 1 });
    tl.to(panel, { xPercent: 0,   duration: 0.45, ease: 'power4.in' })
      .set(from, { autoAlpha: 0 })
      .to(panel, { xPercent: 100, duration: 0.55, ease: 'power4.out' }, '+=0.05');

  } else if (mode === 'dissolve') {
    gsap.set(to, { autoAlpha: 0, filter: 'blur(16px)' });
    to.classList.add('active');
    tl.to(from, { autoAlpha: 0, filter: 'blur(20px)', duration: 0.9 }, 0)
      .to(to,   { autoAlpha: 1, filter: 'blur(0px)',  duration: 1.1 }, 0.2);
  }
  currentIndex = nextIndex;
}
```

### 4.2 Per-slide content reveal (runs on `onComplete` above, once a slide is active)

```js
function revealContent(slideEl) {
  if (reduceMotion) return; // content is already at rest state, no motion
  const tl = gsap.timeline({ defaults: { ease: 'expo.out' } });

  tl.from(slideEl.querySelectorAll('.r-kicker'),  { autoAlpha: 0, y: 12,  duration: 0.5 })
    .from(slideEl.querySelectorAll('.r-title'),   { autoAlpha: 0, y: 28,  duration: 0.7 }, '-=0.3')
    .from(slideEl.querySelectorAll('.r-item'),    { autoAlpha: 0, y: 20,  duration: 0.5, stagger: 0.08 }, '-=0.35')
    .from(slideEl.querySelectorAll('.r-diagram'), { autoAlpha: 0, scale: 0.94, duration: 0.6 }, '-=0.3');
}
```

### 4.3 Signature component animations (used where Section 7 flags them)

- **Loop diagram (slide 2):** nodes draw in via `stagger` around the circle, then the connecting arc animates with a stroke-dashoffset sweep, then a small pulse travels once around the ring on loop-completion (motivated: shows the cycle "runs").
- **Pipeline diagrams (slides 10, 27):** stages reveal left-to-right / top-to-bottom in sequence, each stage's connector line draws before the next stage fades in — reinforces "this is a sequence," not a list.
- **Rule ticker (slide 28):** genuine `requestAnimationFrame`-free CSS/GSAP `xPercent` loop (one continuous marquee, the deck's only marquee — per the one-marquee-per-deck rule).

### 4.4 Reduced motion (mandatory)

Every timeline above checks `reduceMotion` first and short-circuits to an instant `gsap.set` end-state. No infinite loops (the rule ticker included) run under reduced motion — they render as a static single frame instead.

---

## 5. Anti-Slop Checklist (adapted from the taste skill for a slide-deck context)

- [ ] Zero em-dashes anywhere on any slide (headlines, captions, code comments, quotes).
- [ ] One accent color (`--accent`) used identically on all 33 slides — no drifting to a second "status" color.
- [ ] One corner-radius system (4px panels / full-pill tags) — no mixed radii.
- [ ] No centered-hero-with-bullet-list slide anywhere (the exact pattern the brief bans). Every content slide is asymmetric, split, diagrammatic, or a full-bleed statement — never "headline centered, bullets centered below."
- [ ] No default drop shadows (`box-shadow: 0 4px 6px rgba(0,0,0,0.1)` Bootstrap-card style). Elevation = tinted dark shadow (`rgba(0,0,0,0.4-0.6)`, larger blur, low spread) or a hairline border, never a soft gray card shadow.
- [ ] No layout family repeats twice anywhere in the deck (enforced in Section 7's table — check the "Layout family" column for duplicates before build).
- [ ] No 3-equal-column generic feature-card row. Grids are asymmetric (bento-style, mixed cell sizes) wherever a grid is used.
- [ ] No section-number eyebrows like "01 / 06" pasted above every title. Section numerals appear ONLY on the 6 section-divider slides, as the dominant visual, not a small label.
- [ ] No decorative status dots, no scroll-cue arrows (deck is click-navigated, not scrolled), no fake version-label chrome ("v0.6", "BETA").
- [ ] No hand-rolled fake product screenshots. All diagrams are drawn with real CSS/SVG shapes representing real concepts (pyramids, pipelines, loops, matrices) — never a fake browser-window mockup.
- [ ] Every code snippet is a real, verbatim example already in `CONTENT-OUTLINE.md` — nothing invented for decoration.
- [ ] Icons (where used) come from one library only, one stroke-width, no hand-drawn SVGs — used sparingly (this deck leans on typography and diagrams over iconography, per the AI/engineering register).
- [ ] Contrast: every text/background pairing hits WCAG AA against the dark slate backgrounds (verified against `--text-secondary` on `--bg-primary`/`--bg-elevated`).

---

## 6. Technical Architecture

- Single self-contained HTML file: `slides/ai-fe-training-deck.html`. All CSS and JS inline.
- Full contents of `viewport-base.css` (fixed 1920×1080 stage, scaled uniformly, `.active`/`.visible` visibility model) included verbatim in the `<style>` block.
- GSAP loaded from a CDN `<script>` tag (only external dependency — acceptable per `frontend-slides` since the deck still runs standalone in any browser with network access; no build step).
- Slide switching and content reveal driven by GSAP timelines (Section 4), replacing the base template's CSS-only `.reveal` transitions with real choreography.
- Keyboard (arrows/space/Page Up-Down), mouse wheel, and touch/swipe navigation, plus a minimal progress indicator (dot rail, bottom center, outside the 16:9 stage per `deck-controls`).
- Inline text editing included by default (hover-hotzone + `E` key, per `frontend-slides` convention), unless you'd rather ship it locked — flag if so.
- `prefers-reduced-motion` supported at both the CSS level (`viewport-base.css`) and the JS/GSAP level (Section 4.4).

---

## 7. Layout Mapping — All 33 Slides

Every row is a distinct layout family. No family repeats (Variance 8 requirement). Transition = the mode used to *arrive* at that slide.

| # | Section | Slide | Layout family | Key visual treatment | Transition |
|---|---|---|---|---|---|
| 1 | — | Title | Editorial Manifesto Hero | Massive display type, off-center (not centered), mono kicker "AI-ASSISTED FRONTENDENGINEERING", subtle grid-pattern background | — (opening) |
| 2 | — | Training Objective | Radial Loop Diagram | 7-node circular loop, center-lit with `--accent-glow` (moment 1 of 3), quote beneath | Cut |
| 3 | — | Expected Capabilities | Asymmetric Bento (7 cells) | A-G tiles, mixed sizes (2 large + 5 small), not a uniform grid | Cut |
| 4 | Foundations | Section Divider "01" | Full-bleed Numeral | Giant `01` in display font bleeding off-canvas, section name small, bottom-left | Wipe |
| 5 | Foundations | Pilot vs. Copilot | Asymmetric Split-Screen (60/40) | Left: Engineer column (accent-underlined). Right: AI column (slate/muted) | Cut |
| 6 | Foundations | Everything Lives in the Window | Custom Metaphor Diagram | Vertical "filling glass" of token blocks, opacity-fading older blocks toward the bottom | Cut |
| 7 | Foundations | Start Fresh, or Keep Going? | Horizontal Decision Flow | 3 gate-diamonds in sequence, routing lines to two end-states | Cut |
| 8 | Foundations | Resource by Risk, Not Size | 2x2 Matrix | Quadrant plot, two examples placed as dots with labels | Cut |
| 9 | Thinking Tools | Section Divider "02" | Full-bleed Numeral | Same numeral system as slide 4, different section name | Wipe |
| 10 | Thinking Tools | Requirement -> Contract | Horizontal Pipeline (7 stage) | Stage-by-stage draw-in, code snippet inset bottom-right | Cut |
| 11 | Thinking Tools | Context Beats Clever Prompt | Vertical Pyramid | 5-tier pyramid, off-center-left, generous right white-space with the core quote | Cut |
| 12 | Thinking Tools | The Design Is the Spec | Before/After Split | Left: quoted vague text treated as "noise" (low contrast). Right: annotated diagram-style callouts on a wireframe | Cut |
| 13 | Thinking Tools | Context->Goal->Constraints->References->Output | Vertical Stepper Card | 5-row fillable template, mono labels + example text per row | Cut |
| 14 | Workflow | Section Divider "03" | Full-bleed Numeral | Same system, section name "Workflow" | Wipe |
| 15 | Workflow | Analyze -> Plan -> Review -> Implement | 4-Stage Relay + Cost Curve | Relay boxes on top, rising cost-of-mistake curve underneath, curve spikes right after "Review" if skipped | Cut |
| 16 | Workflow | Generate / Transform / Explain / Discover | Bento Quadrant (4 equal-ish, asymmetric padding) | 4 cards, each its own accent-tinted icon zone | Cut |
| 17 | Workflow | Hypotheses Before Fixes | Dual Loop Comparison | Left: tight converging spiral loop (accent). Right: flat dead-end loop (muted slate) | Cut |
| 18 | Workflow | Refactor Is Not "Clean This Up" | 3-Gate Sequence | 3 padlock-style gates left to right, only the current gate lit | Cut |
| 19 | Workflow | Ask It to Prove You Wrong | Speech-Bubble Contrast | Two chat-bubble panels stacked diagonally (not aligned), weak vs strong prompt | Cut |
| 20 | Workflow | Generated Code Is Not Verified Code | Verification Pyramid (6-tier) | Literal 6-tier pyramid, mono annotation per tier ("catches: ___") | Cut |
| 21 | Frontend Quality | Section Divider "04" | Full-bleed Numeral | Same system, section name "Frontend & Quality" | Wipe |
| 22 | Frontend Quality | Six Frontend-Only Blind Spots | Asymmetric 6-cell Bento | Mixed cell sizes (not a neat 3x2 grid), one code snippet visible in the largest cell | Cut |
| 23 | Frontend Quality | Coverage Is Not Correctness | Before/After Code Diff | Two stacked code blocks, red strike vs accent-highlighted line | Cut |
| 24 | Frontend Quality | AI as a Knowledge Tool | Risk/Reward 2x2 | Quadrant plot, "Documentation" plotted high-reward/low-risk vs. "Code-gen" plotted elsewhere | Cut |
| 25 | Frontend Quality | The Seven Ways This Goes Wrong | Masonry Wall (7 cards) | Uneven card heights, staggered vertical offsets, mono violation-quote per card | Cut |
| 26 | Team Standard | Section Divider "05" | Full-bleed Numeral | Same system, section name "Team Standard" | Wipe |
| 27 | Team Standard | The One Loop the Whole Team Runs | Vertical Pipeline Stepper (10 stage) | Tall vertical stepper, 2 "HUMAN REVIEW" gates lit with `--accent-glow` (moments 2 & 3 of 3), Verify branches into 3 parallel lanes that rejoin | Cut |
| 28 | Team Standard | Ten Rules, One Team | Kinetic Rule Marquee | Continuous horizontal ticker of the 10 rules, mono-numbered, one accent word per rule (deck's single marquee) | Cut |
| 29 | Playbooks + Advanced | Section Divider "06" | Full-bleed Numeral | Section name "Playbooks & Beyond" | Wipe |
| 30 | Playbooks | A Recipe for Every Situation | 3x3 Situation Grid | 9 cards, uniform grid intentionally (menu metaphor — the one deliberate exception, since "menu of situations" is the point) | Cut |
| 31 | Advanced Track | Beyond Your Own Session | Layered Stack Diagram | Two horizontal bands stacked (session habits below, shared tooling above), connecting arrows upward | Cut |
| 32 | Closing | The Five Things to Remember | Numbered Stack Reveal | 5 lines, large display type, left-aligned ragged edge (not centered), revealed in sequence | Dissolve |
| 33 | Closing | The Actual Goal | Full-bleed Centered Quote | Single largest-type moment in the deck, true center (the one deliberate exception to anti-center-bias — closing statement, not a hero) | Dissolve |

**Family-repeat audit:** "Full-bleed Numeral" appears 6x (dividers only — same family by design, that's the section-marker system, not a content-layout repeat) and "Cut" transition is the default connective tissue (28 of 33 uses) — both are intentional systems, not variance failures. Every *content* slide (5-33 excluding dividers) uses a distinct layout family. Two deliberate centered exceptions are called out explicitly (30, 33) with their justification inline.

---

## Open questions before build

1. **Accent color** — proposing signal orange (`#FF5A1F`). Fine, or would you rather see a different single accent (e.g. acid lime, electric red)?
2. **Fonts** — Cabinet Grotesk + Switzer (Fontshare) + JetBrains Mono. Fine, or a different pairing?
3. **GSAP as the one CDN dependency** — acceptable, since the deck still opens as a single HTML file, it just needs network access once to load the library (or I can vendor it inline instead if you want a fully offline file).
