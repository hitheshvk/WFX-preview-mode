# Handoff: Whatfix Preview Mode + Diagnostics

## Overview

A **Preview Mode** experience for a Whatfix-style in-app guidance product. A creator turns Preview Mode on from the Whatfix Studio panel; a dark floating banner appears at the top of the host web application and reports whether the guidance content on the current page is healthy. When something is wrong, the banner routes the creator into a **Diagnostics** panel that lists the content on the page and, for a played flow, which step failed.

Three concerns are being designed at once:

1. The **preview-mode banner** — the primary artifact. A compact, draggable status bar with several states.
2. The **Studio / Diagnostics side panel** — where detail and remediation live.
3. A **host application** — a generic corporate portal, present only so the banner and panel have something to sit on top of.

## About the Design Files

The files in this bundle are **design references created in HTML**. They are prototypes that demonstrate intended look, motion, and behavior. They are **not production code to copy**.

The task is to **recreate these designs in the target codebase's existing environment** — React, Vue, Angular, Svelte, or whatever the host product uses — following that codebase's established component patterns, styling approach, and state conventions. If no environment exists yet, choose an appropriate framework and implement there.

Two notes specific to this design:

- The banner and panel are an **overlay injected into a third-party page**. In production they must not inherit or leak host page styles. Expect a shadow DOM or heavily-scoped CSS layer, and expect the host page in the prototype to be replaced by whatever real application the extension runs against.
- The host portal in the prototype (nav, feed, task list, charts) is **scaffolding only**. Do not port it.

## Fidelity

**High-fidelity for the banner.** Its colors, sizes, radii, and type were sampled pixel-by-pixel from an approved 2× Figma export and should be reproduced exactly. The values in *Design Tokens → Banner* are authoritative.

**Medium-fidelity for the side panel and Self Help popover.** Structure, copy, hierarchy, and behavior are correct; exact spacing and iconography should be reconciled against the design system before shipping. Icons in the prototype are CSS/SVG stand-ins for the product's real icon set.

**Throwaway for the host page.** Layout scaffolding only.

---

## Screens / Views

### 1. Host application (context only)

A generic intranet portal at 1440px: white 64px top nav with logo, search, and six icon+label nav items; a three-column body (`240px / minmax(0,1fr) / 300px`, 20px gap, `max-width:1300px`, centered, with a **72px left inset** reserved so the floating Self Help button never overlaps content). Left column is a profile card; center column has a profile-completion card, a post composer, three stat cards, a task list, an activity list, and a bar chart; right column has company news, a promo placeholder, upcoming events, suggested people, and a footer. Page background `#f1efeb`; cards white with `1px solid #e6e3de` and `8px` radius.

Replace entirely with the real host page.

### 2. Preview-mode banner

**Purpose.** Tell the creator, in one glance, whether the guidance content on this page will work for end users — and give them one action when it won't.

**Position.** Absolutely positioned, horizontally centered, `top: 10px`, `z-index: 55` — above the side panel. Once dragged it becomes free-positioned (`left`/`top` from state, `margin: 0`).

**Anatomy** (left to right):

| Part | Spec |
|---|---|
| Drag grip | 2 cols × 3 rows of 3.5px dots, `#8C889F`, 4px gap, 6px horizontal padding. `opacity: 0` by default, `1` on banner hover or while dragging, 160ms ease. `cursor: grab` / `grabbing`. |
| Gap | 10px between grip and pill |
| Pill | `height: 48px`, `border-radius: 10px`, `background: #1F1F32`, `box-shadow: 0 8px 24px rgba(15,14,40,.32)`, `overflow: hidden`, `display: flex` |
| Identity block | `padding: 0 16px`, column, `gap: 3px`. Row 1: 16×13px Whatfix mark + `Preview mode` (14px/600, `#F2F2F8`, `line-height:1.15`), 8px gap. Row 2: `Draft • Ready • Production` (13px, `#8C899F`, `line-height:1.15`). |
| Divider | `border-left: 1px solid #434353` on the status segment |
| Status segment | `padding: 0 20px`, row, `align-items:center`, `gap: 10px`. Contents vary by state. |
| Action button | `height: 28px`, `padding: 0 18px`, `border-radius: 8px`, `background: #3D3C52`, label 14px/500 `#F2F2F8`, `margin-left: 10px`. Hover `#4C4A64`. |
| Close | `✕`, 16px, `#6B697B`, hover `#F2F2F8`, 160ms |

**States.** The identity block is constant; only the status segment changes.

| State | Status segment |
|---|---|
| `hidden` | Banner not mounted |
| `checking` | 15px spinner (`2px solid #434353`, `border-top-color:#F2F2F8`, 900ms linear) + `Checking this page…` + ✕ |
| `issues` | Warning triangle + `12 of 15 content has issues` + **Diagnose** + ✕ |
| `clean` | 18px `#3BB273` circle with white ✓ + `No issues on this page` + ✕ |
| `broken` | Warning triangle + `Step 3 isn't working · "<flow name>"` (truncates at `max-width:300px` with ellipsis, full text in `title`) + **Diagnose** + ✕ |
| `plain` | ✕ only, `padding: 0 18px 0 14px` — the resting state after Diagnose, and the state held while a flow plays |

Status text is 14px `#F2F2F8` in every state.

**Warning triangle** — inline SVG, 16×15, `viewBox="0 0 16 15"`:

```svg
<path d="M6.98 1.42a1.18 1.18 0 0 1 2.04 0l6.34 11.06c.45.79-.11 1.77-1.02 1.77H1.66c-.91 0-1.47-.98-1.02-1.77z" fill="#F6566B"/>
<rect x="7.2" y="5.1" width="1.6" height="4.3" rx=".8" fill="#fff"/>
<rect x="7.2" y="10.4" width="1.6" height="1.7" rx=".8" fill="#fff"/>
```

There is deliberately **no "playing" state**. While a flow plays the banner sits in `plain`; it speaks up only on failure.

### 3. Side panel

`width: 436px`, pinned right, full height, `box-shadow: -14px 0 40px rgba(20,18,40,.18)`. Its `top` insets to **58px** whenever the banner is visible and un-dragged, so the panel header clears the banner — animated `top 300ms cubic-bezier(.22,1,.36,1)`.

Composed of a 46px **icon rail** (`#1C1A3C`) and a white content column. Both the rail and the content are inside the inset wrapper so they move together.

**Icon rail.** Whatfix mark at top (24px, orange squares + white bar), then the tab set, then an exit glyph pinned to the bottom (`margin-top: auto`). Each tab is a 34×34 tile, `border-radius: 9px`, `display: grid; place-items: center`, `transition: background 180ms`:

- **Active**: `background: rgba(242,106,27,.18)`, icon `#F26A1B`, plus a 3×18px `#F26A1B` indicator at `left: -6px` with `border-radius: 0 2px 2px 0`.
- **Inactive**: transparent, icon `#8F8CB5`, hover `background: rgba(255,255,255,.08)`.

Two tabs: **Content** (plus glyph) → Studio view; **Diagnostics** (magnifier) → Diagnostics view. Diagnostics reads as active for both the page view and the step view.

**Studio view.** Peach header (`#FDECE0`) with ⋮ / ✕, the headline `Studio: Guide. Track. Get feedback. In context. All in one place.` (23px/700, `line-height:1.28`, `max-width:280px`, with `Studio:` in `#F26A1B`) and a decorative shape cluster at the right. Below: a row with the guide name `J&J_Salesforce_dem..` and the **Preview mode** toggle pill (`#F4F5F8`, 22px radius; 38×21 track, `#1A73E8` on / `#D3D6DD` off, 17px white knob). Then `Content` and `Widgets` sections — 3-column grids of 8px-radius `1px solid #E4E2EC` tiles, icon over label (13.5px): Flow, Link, Video, Text / Beacon, Smart-tip, Popup, Launchers.

**Diagnostics — page view.** Title `Diagnostics` (21px/600) with ⋮ / ✕. Guide name + Preview mode toggle. A search field (`Search content on this page`, 8px radius, `1px solid #DEDBE6`) and a 40px filter button. A segmented control on `#F2F2F6` with 22px radius: **On this page** (active, `#2B2850`, white, 600) / **Not on this page**. Then rows separated by `1px solid #EEECF2`, each icon + title (15px/600) + sub-line (13px) + `⌄`, hover `#FAF9FC`:

- `Flows` — `Play a flow to check every step` (`#6F6A85`). **Clicking this row plays the flow.**
- `Smart tips` — `All 6 showing on this page` (`#3F8B57`)
- `Beacons` — `All 4 showing on this page` (`#3F8B57`)
- `Launchers` — `2 won't show to users` (`#C0392B`)

Green for healthy, red only for the row that genuinely fails.

**Diagnostics — step view.** Title `Diagnostics`, then the flow name and step count (wraps, `text-wrap: pretty`, full text in `title`). A scrolling list of step cards; each card is `flex: none` so cards never compress:

- **Passed step** — `#F6F5F9`, 10px radius. Row: 30px `#E6F4EA` circle with the step number, label, `Working` (12.5px/600 `#1E8E3E`). Footer band `#EEF7F0`: ✓ + `Shown in the right place`.
- **Failed step** — `1.5px solid #E2483D`, 10px radius. Row: 30px `#FBE3E0` circle, label, `Not working` (`#C0392B`). Footer band `#FDEEEC`: ✕ + `We can't find the link this step points to`.
- **Pending steps** — `1px dashed #D8D5E2`, 10px radius, 30px `#F4F3F8` circle, label `#6F6A85`.

### 4. Self Help launcher and popover

A 44px `#F26A1B` circle at `left: 18px; top: 296px` (`box-shadow: 0 6px 16px rgba(242,106,27,.34)`), with a 15px white rounded square rotated 45°, `transform: scale(1.08)` on hover. A 44px-tall dot grip sits to its left, decorative only.

Clicking it opens a 296px popover, absolutely positioned at `left: 66px; top: -8px`: orange header (`Self Help` + ✕), a search field, then three rows — `Onboarding new users` / `Flow · 5 steps` with a **Play** affordance, `Submit an expense report` / `Article · 2 min read`, `Tour of the new dashboard` / `Video · 1:20`. Row hover `#F3F5F8`, 7px radius.

### 5. On-page flow callouts

While a flow plays, the targeted element gets a 2px `#F26A1B` ring at `inset: -6px` with `box-shadow: 0 0 0 4px rgba(242,106,27,.2)`, and a 270px white callout appears at `top: calc(100% + 14px)`: 10px radius, `1px solid #E7E3DC`, `box-shadow: 0 14px 36px rgba(20,18,40,.22)`, a rotated 12px square as the arrow at `left: 26px; top: -7px`. Inside: `STEP n OF 5` (11.5px/600 `#D9600A`, `letter-spacing: .04em`), title (15px/600), body (13px `#5F6672`, `line-height: 1.45`), then a footer row with 5 progress dots (active 16×4 `#F26A1B`, inactive 6×4 `#DCDFE4`) and `Skip` / **Next** (`#F26A1B`, 13px/600, 6px radius).

Orange is used for guidance so it never reads as host-app chrome.

### 6. Exit confirmation

Clicking ✕ on the banner does **not** exit directly. A modal opens over the app: scrim `rgba(20,19,39,.42)`, card 400px wide, white, 12px radius, `box-shadow: 0 24px 64px rgba(20,18,40,.32)`, `padding: 26px 26px 20px`.

- Title `Exit preview mode?` — 18px/600 `#1F1C33`
- Body `You'll stop seeing this page as an end user. Diagnostics results for this page will be cleared.` — 14px `#5F5A72`, `line-height: 1.5`
- Buttons right-aligned, 10px gap, both 38px tall / 8px radius: **Stay in preview** (white, `1px solid #DEDBE6`, hover `#F6F5F9`) and **Exit preview** (`#1F1F32`, `#F2F2F8` label, hover `#33314F`)

Cancel restores the previous state untouched; confirm collapses the banner and switches the Preview mode toggle off.

### 7. States strip (prototype-only)

A dark `#141327` strip above the app with monospace chips that jump to any state, a `LONG NAME` chip that swaps in a 98-character flow name to stress-test truncation, and `HIDE ✕`, which collapses it to a small `STATES ⌄` tab at the page's top-left. **Do not port this** — it is a review affordance.

---

## Interactions & Behavior

### Primary flow

1. Studio panel open, Preview mode off. Banner hidden.
2. Toggle **Preview mode** on → banner mounts in `checking`.
3. After **3000ms** → cross-fade to `issues`.
4. Click **Diagnose** → the panel switches to Diagnostics and slides in; **420ms later** the banner cross-fades to `plain`. The panel arrives first, deliberately — the two events must not fire together.
5. Click the **Flows** row (or Play in Self Help) → panel closes, banner goes to `plain`, step 1 callout appears on the page.
6. Click **Next** → step 2 callout. Steps **never** auto-advance.
7. Click **Next** on step 2 → step 3 is requested but its target element is not on the page; nothing renders. After **900ms** the banner cross-fades to `broken`.
8. Click **Diagnose** → step diagnostics, then the banner settles to `plain` after 420ms.
9. Click ✕ → exit confirmation.

### Motion

All banner state changes go through one fade-swap: set `fading`, wait **460ms**, apply the new state. The status segment animates `max-width 620ms cubic-bezier(.22,1,.36,1)` and `opacity 420ms cubic-bezier(.4,0,.2,1)`; collapsed is `max-width: 0`, open is `520px`.

| Animation | Spec |
|---|---|
| `barIn` | 560ms `cubic-bezier(.22,1,.36,1)` — `translateY(-14px) scale(.96)` + fade in |
| `barOut` | 540ms `cubic-bezier(.4,0,.2,1)` — reverse |
| `segIn` | 480ms `cubic-bezier(.22,1,.36,1)`, 120ms delay — `translateY(6px)` + `blur(1px)` → sharp |
| `panelIn` | 320ms `cubic-bezier(.2,.7,.3,1)` — `translateX(24px)` + fade |
| `attn` | 1500ms ease-in-out ×3 — red halo pulsing to `0 0 0 7px rgba(226,72,61,.22)` |
| `nudge` | 620ms `cubic-bezier(.3,.8,.3,1)` — ±2px vertical |
| `warnPop` | 420ms `cubic-bezier(.2,.8,.3,1)`, 120ms delay — `scale(.7 → 1.14 → 1)` |
| `spin` | 900ms linear infinite |

The attention treatment (`nudge` once, then `attn` three times) fires **only** on `issues` and `broken`. It is the one moment the banner asks for attention; do not extend it to other states.

Hiding the banner is a two-phase teardown: `fading` + `closing` → 560ms → unmount.

### Dragging

`mousedown` on the grip captures the pointer. `mousemove` sets position clamped to the app viewport with an 8px margin on all sides; `mouseup` releases. Position persists after release, and is re-clamped on window resize and on any re-render that isn't a drag, so the banner can never end up partly offscreen. A dragged banner also drops the panel's 58px top inset.

### Long content

Banner status text caps at `max-width: 300px` and truncates with an ellipsis, full string in `title`. Panel titles wrap with `text-wrap: pretty` rather than truncating. Self Help row titles truncate. Step-card labels wrap.

---

## State Management

```
bar:        'hidden' | 'checking' | 'issues' | 'clean' | 'broken' | 'plain'
panel:      'closed' | 'studio' | 'diagnostics' | 'steps'
previewOn:  boolean   — mirrors the Studio toggle
playing:    boolean   — a flow is walking the page (independent of `bar`)
step:       number    — 1-based current step
fading:     boolean   — mid cross-fade
closing:    boolean   — mid teardown
pos:        {x, y} | null   — null means default top-center
dragging:   boolean
barHover:   boolean   — reveals the drag grip
exitAsking: boolean   — confirmation modal open
selfHelpOpen: boolean
```

Transitions:

| Trigger | Effect |
|---|---|
| Toggle on | `bar: 'checking'`, `previewOn: true`; 3000ms → `issues` |
| Toggle off / confirm exit | teardown → `bar: 'hidden'`, `previewOn: false` |
| Diagnose (issues) | `panel: 'diagnostics'`; 420ms → `bar: 'plain'` |
| Diagnose (broken) | `panel: 'steps'`; 420ms → `bar: 'plain'` |
| Play flow | `panel: 'closed'`, `bar: 'plain'`, `playing: true`, `step: 1` |
| Next | `step += 1`; if `step >= 3`, 900ms → `bar: 'broken'`, `playing: false` |
| ✕ | `exitAsking: true` |

Every timer is cleared on any new transition and on unmount. Two handles are enough — one for scheduled state changes, one for the fade-swap — but they must both be cleared, or a stale timer will overwrite a newer state.

No data fetching in the prototype. In production, `checking` wraps the real page scan, its result decides `issues` vs `clean`, and the counts come from that response.

---

## Design Tokens

### Banner (authoritative — sampled from the 2× export)

| Token | Value |
|---|---|
| Pill background | `#1F1F32` |
| Divider | `#434353` |
| Button background / hover | `#3D3C52` / `#4C4A64` |
| Primary text | `#F2F2F8` |
| Secondary text | `#8C899F` |
| Grip dots | `#8C889F` |
| Close glyph | `#6B697B` |
| Warning red | `#F6566B` |
| Success green | `#3BB273` |
| Pill height / radius | 48px / 10px |
| Pill shadow | `0 8px 24px rgba(15,14,40,.32)` |
| Button height / radius | 28px / 8px |
| Logo | 16 × 13px |

### Panel and app

`#1C1A3C` rail · `#F26A1B` Whatfix orange (`#D9600A` text-safe, `#C0510D` on white) · `#FDECE0` Studio header · `#1F1C33` primary text · `#5F5A72` / `#6F6A85` secondary · `#E4E2EC` / `#DEDBE6` / `#EEECF2` borders · `#F6F5F9` / `#FAF9FC` surfaces · `#1E8E3E` pass · `#3F8B57` healthy · `#C0392B` fail · `#E2483D` fail border · `#1A73E8` host blue · `#1B4C8C` host navy · `#F1EFEB` page · `#E6E3DE` card border

### Type

Inter Tight 400/500/600/700 throughout; IBM Plex Mono for the prototype-only states strip and placeholders.

Scale: 11.5 · 12 · 12.5 · 13 · 13.5 · 14 · 14.5 · 15 · 15.5 · 16 · 17 · 18 · 21 · 23 · 28px. `line-height: 1.15` on banner text, `1.28` on the Studio headline, `1.45–1.5` on body copy.

### Spacing and radii

Spacing: 2 · 3 · 6 · 8 · 10 · 12 · 14 · 16 · 18 · 20 · 22 · 26px.
Radii: 5 · 6 · 7 · 8 · 9 · 10 · 11 · 12 · 20 · 22px, plus `50%`.

These are the design's real values. Do not snap them to a 4px or 8px grid.

---

## Assets

- `assets/whatfix-mark.png` — the Whatfix mark used in the banner, 136×104 (rendered at 16×13). Extracted from the approved banner export and background-keyed to transparency. **Replace with the vector mark from the brand library before shipping.**
- The `.fig` attached to this project referenced the mark but shipped no image bytes, which is why it was recovered from the raster export.
- Every other icon in the prototype (nav glyphs, rail glyphs, content-tile glyphs, spinner, tick, chevrons) is a CSS or inline-SVG stand-in. Swap all of them for the product's real icon set.
- Striped grey rectangles labelled `illustration / 960×420`, `promo image / 300×230` are deliberate placeholders in the host page. No real imagery exists for them.

---

## Files

- `Preview Mode Diagnostics.dc.html` — the complete design: host page, banner and all states, side panel views, Self Help, flow callouts, exit modal, and the prototype states strip. Markup is inline-styled; interaction logic sits in the component class at the bottom of the file.
- `assets/whatfix-mark.png` — the banner logo.
- `Preview Mode Diagnostics (standalone).html` — a single self-contained copy that opens offline in any browser. Use it to review behavior and timing; it is a build output, not a source file.
- `preview-mode-banner-reference.png` — the approved 2× export of the `issues` state. Every banner value above was measured from this. Treat it as the visual acceptance test.

## Implementation order

1. Banner shell and the six status states as a pure presentational component driven by props.
2. The fade-swap state machine and timer discipline.
3. Diagnose → panel-then-banner staging.
4. Panel views (Studio, Diagnostics page, Diagnostics step).
5. Flow playback: Self Help → callouts → manual advance → failure.
6. Dragging, clamping, and the exit confirmation.

Build 1 and 2 first and get them reviewed against the reference PNG. Everything else depends on the banner being right.
