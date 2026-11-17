# NIGHTCLUB — REACT WEB APPLICATION (FRONTEND SPEC)

**Document:** `13_REACT_WEB_APPLICATION.md`
**Version:** 0.2 draft · 17 November 2026 (visual system updated to match the reference prototype)
**Parent:** `NIGHTCLUB_HOOKED_PRODUCT_BLUEPRINT.md` (sections 3, 6, 9, 11, 12 are binding on this file)
**Scope:** Desktop-first, responsive web app (React + TypeScript). Flutter iOS/Android is specified later in `12_FLUTTER_IOS_ANDROID_APPLICATION.md` and must reuse the tokens, copy and states defined here.
**Reference prototype:** `Nightclub · Web UI Prototype.html` is the **visual source of truth**. If this document and the prototype disagree on how something looks, the prototype wins; open it in a browser, switch Light/Dark and the Palette picker, and match it.
**Status:** Design direction and screen contracts. Items marked **[OPEN]** need a team decision. Items marked **[ASSUMPTION]** are proposals not yet confirmed against the existing Laravel backend.

---

## 0. What this document is for

This file tells a frontend developer and a designer **what to build, screen by screen**, so the web app expresses Nightclub's product loop:

> Understand me → Find my people → Build together → Prove my work → Unlock my next opportunity → Improve future matching.

Every screen below has: a job, one primary action, the Hooked loop it serves (Trigger → Action → Variable reward → Investment → Next trigger), its states, and its guardrails. If a screen cannot name its one primary action, it should not ship.

**Non-goals for the UI:** infinite scroll, streaks, public leaderboards, fake countdowns, fake scarcity, "X people are viewing", loss-aversion copy, red-dot badges that don't map to a real pending action. (Blueprint §1 guardrails.)

---

## 1. Design direction

### 1.1 References and what we take from each

| Reference | Take | Do **not** take |
|---|---|---|
| **Circle** (primary visual reference) | Persistent left sidebar with grouped "spaces"; persistent sidebar structure and generous whitespace; rounded cards with 1px borders; clear event/session cards; clean member profile; restrained accent colour; good empty states | Heavy admin/customisation surfaces for members |
| **Skool** (interaction reference) | Extreme simplicity: few top-level tabs, one obvious thing to do per page; a "classroom"-style progress view (map to **Sessions & Missions**); a member directory; clear "what to do next" onboarding | Points, levels, leaderboards (they reward activity volume, which contradicts our North Star) |
| **Hinge / YC Co-Founder Matching** | One person at a time with a human reason ("Why this match?"); structured profile prompts | Swipe mechanics; scores shown as percentages |
| **Timeleft / Bumble BFF Groups** | The "group of strangers becomes a table" moment | Scarcity framing |
| **GitHub / Polywork / Read.cv** | Proof-of-work profile; typographic calm | Raw contribution counts as status |

*Competitor notes come from public information and general product knowledge; please validate against the live products and the screenshots the team collects.*

### 1.2 Principles

1. **One next action.** Every page has exactly one visually primary button.
2. **Finite.** Lists end. Feeds end with "You're caught up."
3. **Honest signals.** Show real states (pending, accepted, artifact-verified). Never decorative counters.
4. **Explain everything ranked.** Anything recommended carries a plain-language reason.
5. **Consent is visible.** Opt-in, decline, mute, leave, and block are always one click away and never styled as destructive-shame.
6. **Clean canvas, vivid moments.** Backgrounds are pure white or pure black. Colour is spent where the member acts or progresses (see §3.3), so the one thing to do next is always the most colourful thing on screen.
7. **Feels good to touch.** Every pressable thing answers the press (§10.3). Small, fast, consistent motion is part of the product's identity, not a finishing layer.

### 1.3 Voice

Plain, warm, specific. Telugu-first audience at launch: build i18n from day one (see §10.6). Examples:

- Good: "Priya builds mobile apps in Flutter. She is looking for a backend partner this month."
- Bad: "🔥 96% match! Don't miss out — 2 spots left!"

### 1.4 Visual identity: "make people want to come back"

The look is closer to **Instagram, Swiggy Instamart, Blinkit and LinkedIn** than to a calm admin tool: crisp pure backgrounds, one confident gradient per palette, bold numbers, big friendly cards, and instant response to every tap. It must still obey §1.2 and the guardrails in §0: colour and motion draw attention to **real** actions and progress only, never to fake urgency.

- **Pure backgrounds.** Light mode `#ffffff`, dark mode `#000000`. Never tint the page background with the palette.
- **One gradient per palette** (§3.2) is the brand. It appears only in the places listed in §3.3.
- **Neutrals do the structural work.** Cards, borders, text and hover fills are neutral and identical in every palette. Only the accent ladder and the gradient change when the palette changes.
- **People are the imagery.** Avatars are flat tints, with a gradient ring on people the member has an open introduction with (the "story ring").

---

---

## 2. Technology (proposed)

| Concern | Choice | Notes |
|---|---|---|
| Language / UI | React 18+ with TypeScript (strict) | Per Blueprint §0 |
| Build | Vite | **[OPEN]** If public, SEO-critical pages (public Builder Passport, session landing pages) are required, evaluate Next.js or a separate prerendered public site for those routes only. The authenticated app can stay a SPA. |
| Routing | React Router (data routers) | Route-level code splitting |
| Server state | TanStack Query | Cache, retries, optimistic updates for accept/decline |
| Client state | Local component state + URL state; Zustand only if needed | Avoid global stores for server data |
| Forms | React Hook Form + Zod schemas | Schemas shared with API DTO types where possible |
| Styling | CSS variables for tokens + Tailwind (or CSS Modules) | Tokens in §3 are the contract, not the tool |
| Overlays | **Native `<dialog>` and Popover API** | No custom focus traps (see §10.1) |
| Realtime chat | WebSocket/SSE client behind an interface | **[ASSUMPTION]** Transport depends on backend design |
| API | Typed client generated from versioned Laravel API contract | Never trust a client-side "paid" flag; server enforces entitlements |
| Testing | Vitest + Testing Library, Playwright e2e, axe-core in CI | |
| i18n | `react-i18next` or equivalent; ICU plurals | English + Telugu from first release |

**Browser support policy (proposed):** last two versions of Chrome, Edge, Safari, Firefox. Use Baseline "widely available" features freely; for newer features, feature-detect and degrade gracefully. **[OPEN]** confirm.

---

## 3. Design tokens

All colour is defined as CSS custom properties on `:root` with `color-scheme: light dark` and `light-dark()`. Tokens are the contract; Tailwind/CSS Modules are only the delivery tool. Values below are taken from the reference prototype.

### 3.1 Neutrals (identical in every palette)

```css
:root {
  color-scheme: light dark;

  /* Pure white in light, pure black in dark. */
  --nc-bg:        light-dark(#ffffff, #000000);
  --nc-side:      light-dark(#ffffff, #000000);   /* sidebar, same as page; separated by a 1px border */
  --nc-surface:   light-dark(#ffffff, #121212);   /* cards */
  --nc-surface-2: light-dark(#f5f5f5, #1c1c1e);   /* chips, wells, inputs on hover */
  --nc-active:    light-dark(#efefef, #1f1f21);   /* selected nav row, selected list row */
  --nc-border:    light-dark(#ebebeb, #262626);
  --nc-control:   light-dark(#d4d4d4, #3d3d3f);   /* input and outlined-button borders */
  --nc-text:      light-dark(#0b0b0c, #f5f5f5);
  --nc-muted:     light-dark(#6b6b6f, #a8a8ad);   /* meets 4.5:1 on its surface */
  --nc-primary:   light-dark(#0b0b0c, #f5f5f5);   /* solid, for toasts and rare solid UI */
  --nc-primary-t: light-dark(#ffffff, #000000);

  /* Semantic (status only; never decoration) */
  --nc-success:   light-dark(#247b50, #4fc88a);
  --nc-success-bg:light-dark(#dcfce7, #12301f);
  --nc-success-ink:light-dark(#14532d, #8be0b2);
  --nc-warn:      light-dark(#92400e, #f2b45a);
  --nc-warn-bg:   light-dark(#fffbeb, #33270f);
  --nc-danger:    light-dark(#c4242a, #f0777c);
  --nc-danger-bg: light-dark(#ffedea, #3a1a1c);
  --nc-danger-ink:light-dark(#831c1e, #f5a3a6);

  --nc-shadow-lg: light-dark(0 12px 32px rgb(0 0 0 / .16), 0 12px 32px rgb(0 0 0 / .7)); /* only for floating layers */
  --ease: cubic-bezier(.23, 1, .32, 1);          /* the one UI easing curve */

  --nc-radius-sm: 6px;  --nc-radius-md: 10px;  --nc-radius-lg: 14px;
  --nc-gap: 24px;       /* one gutter, horizontal and vertical */
  --nc-font-sans: "DM Sans", system-ui, -apple-system, "Segoe UI", "Noto Sans Telugu", "Nirmala UI", sans-serif;
}
:root[data-theme="light"] { color-scheme: light; }
:root[data-theme="dark"]  { color-scheme: dark; }
```

Typeface: **DM Sans** (400, 500, 600, 700, 800) loaded from Google Fonts with `display=swap`, followed by Noto Sans Telugu / Nirmala UI for Telugu coverage. `font-variant-numeric: tabular-nums` on `body` so every figure aligns. Final brand font is still **[OPEN]** (§14), but build with DM Sans until decided.

### 3.2 Palettes (the accent ladder + gradient)

A palette is exactly these tokens and nothing else. The default is **Pulse**. The app ships a palette picker in the prototype so the team can choose; production may ship one palette or let members choose (**[OPEN]**, §14).

| Token | Role |
|---|---|
| `--nc-g1`, `--nc-g2` | Gradient start and end |
| `--nc-grad` | `linear-gradient(135deg, var(--nc-g1), var(--nc-g2))` |
| `--nc-on-grad` | Text/icon colour on the gradient |
| `--nc-accent` | Solid accent for small marks and links on neutral surfaces (must be 4.5:1 on `--nc-bg` in its mode) |
| `--nc-accent-t` | Text on a solid accent (equals `--nc-on-grad`) |
| `--nc-accent-bg` | Pale tint for areas: chips, selected options, banners |
| `--nc-accent-track` | Slightly stronger tint: progress tracks, tag fills |
| `--nc-accent-ink` | Dark text used on top of the tint |

| Palette | `--nc-g1` → `--nc-g2` | Accent (light / dark) | Tint (light / dark) | Mood |
|---|---|---|---|---|
| **Pulse** (default) | `#C81E5B` → `#E8561A` | `#C81E5B` / `#FF6B9C` | `#FFF0F5` / `#2A0F1A` | Pink into orange, social energy |
| **Zest** | `#0A7A1E` → `#2E9B3F` | `#0C831F` / `#4ED06B` | `#FFF4C2` / `#2E2606` | Fresh green, yellow tint |
| **Spark** | `#D9480F` → `#C2255C` | `#D9480F` / `#FF8A4C` | `#FFF0E6` / `#2E1608` | Orange into magenta |
| **Wave** | `#0A66C2` → `#0B8F8C` | `#0A66C2` / `#5AA9FF` | `#E8F1FB` / `#0B1A2E` | Blue into teal, professional |
| **Graphite** | `#111111` → `#3A3A3C` (dark: `#F5F5F5` → `#BDBDBD`, `--nc-on-grad` `#000`) | `#111111` / `#FFFFFF` | `#F2F2F2` / `#1C1C1E` | Neutral, no colour |

Full token sets (copy as-is):

```css
/* Pulse: default, declared on :root */
--nc-g1:#C81E5B; --nc-g2:#E8561A; --nc-on-grad:#ffffff;
--nc-grad:linear-gradient(135deg,var(--nc-g1),var(--nc-g2));
--nc-accent:light-dark(#C81E5B,#ff6b9c);  --nc-accent-t:#ffffff;
--nc-accent-bg:light-dark(#fff0f5,#2a0f1a); --nc-accent-track:light-dark(#ffd0e0,#6b1f3d); --nc-accent-ink:light-dark(#8a1240,#ffb8d0);

/* Zest */   :root[data-palette="zest"]  { --nc-g1:#0A7A1E; --nc-g2:#2E9B3F; --nc-accent:light-dark(#0C831F,#4ed06b); --nc-accent-bg:light-dark(#fff4c2,#2e2606); --nc-accent-track:light-dark(#f8cb46,#8a6f00); --nc-accent-ink:light-dark(#5c4700,#ffe07a); }
/* Spark */  :root[data-palette="spark"] { --nc-g1:#D9480F; --nc-g2:#C2255C; --nc-accent:light-dark(#D9480F,#ff8a4c); --nc-accent-bg:light-dark(#fff0e6,#2e1608); --nc-accent-track:light-dark(#ffd2b3,#6b2e0a); --nc-accent-ink:light-dark(#7a2300,#ffc9a3); }
/* Wave */   :root[data-palette="wave"]  { --nc-g1:#0A66C2; --nc-g2:#0B8F8C; --nc-accent:light-dark(#0A66C2,#5aa9ff); --nc-accent-bg:light-dark(#e8f1fb,#0b1a2e); --nc-accent-track:light-dark(#c4dcf5,#1b4a80); --nc-accent-ink:light-dark(#07407d,#b3d5ff); }
/* Graphite */ :root[data-palette="graphite"] { --nc-g1:light-dark(#111111,#f5f5f5); --nc-g2:light-dark(#3a3a3c,#bdbdbd); --nc-on-grad:light-dark(#ffffff,#000000); --nc-accent:light-dark(#111111,#ffffff); --nc-accent-t:var(--nc-on-grad); --nc-accent-bg:light-dark(#f2f2f2,#1c1c1e); --nc-accent-track:light-dark(#e0e0e0,#3a3a3c); --nc-accent-ink:light-dark(#111111,#ffffff); }
```

Every palette (all five) is fully specified for both light and dark. Palette is chosen by `data-palette` on `<html>`; absence means Pulse. Persist the member's choice (prototype uses `localStorage`; production should store it on the profile).

### 3.3 Where the gradient and accent appear (exhaustive)

| Use the gradient (`--nc-grad`) for | Use the solid accent / tints for | Never colour |
|---|---|---|
| Logo mark | Links and the active nav icon (`--nc-accent`) | Page background, sidebar, card backgrounds |
| Primary button (one per page) | Selected chips / options (`--nc-accent-bg` + `--nc-accent-ink`) | Body text |
| **The Next-Action card** on Home (full gradient, `--nc-on-grad` text, white-on-grad primary button) | Provenance "peer-acknowledged" badge | Table/list rows |
| Count badges that map to a real pending action | Pills and tags | Decorative shapes, blobs, borders |
| Progress bars (`--nc-grad` fill on `--nc-accent-bg` track) | Focus rings (`--nc-accent`) | |
| Toggles when on | | |
| Your own message bubbles (`--nc-on-grad` text) | | |
| Stat numbers (gradient text, 34px, weight 800) | | |
| Story ring around avatars with an open introduction | | |
| Project story thumbnails | | |

Everything else is neutral. When a second item on a page wants the gradient, it does not get it.

### 3.4 Rules

- Body and UI text meet WCAG 2.2 AA (4.5:1; 3:1 large text and UI boundaries) in **both** themes and **all five** palettes. White text on the right end of the Pulse and Zest gradients is marginal for body-size text; keep text on gradients bold or large, or darken `--nc-g2`. **Verify before launch.**
- Default theme follows the system. The sun/moon button in the top bar toggles Light/Dark and overrides the system (`data-theme` on `<html>`, remembered). Handle the `prefers-color-scheme` `change` event.
- Avatars: flat tint of the person's own hue (`hsl(h 60% 88%)` light, `hsl(h 35% 22%)` dark) with ink in the same hue. In **Graphite** avatars are neutral grey (saturation 0). Avatar tints are identity, not data.
- Telugu text needs line-height ≥ 1.6 and a font with full Telugu coverage. Test with Telugu strings, which are often longer.
- Support Forced Colors: never convey state by colour alone.
- Respect `prefers-reduced-motion` (see §10.3).

### 3.5 Type scale and weights

| Role | Size / weight |
|---|---|
| Page title (`h1`) | 28px / 700, letter-spacing −0.02em (24px on phone) |
| Card title (`h2`) | 16px / 600 |
| Next-action headline | 22px / 700 |
| Body | 14px / 400, line-height 1.5 |
| Small / meta | 12–13px / 400–500 |
| Stat numbers | 34px / 800, gradient text |

Sentence case everywhere. No uppercase tracked labels, no monospace outside code and IDs.

### 3.6 Component recipes (match the prototype)

| Component | Recipe |
|---|---|
| **Page** | White/black background, 8px top padding, 32px side padding, max width 1280px; main column + 320px context rail; one `--nc-gap` (24px) everywhere. |
| **Sidebar** | 240px, background `--nc-side`, 1px right border `--nc-border`. Logo (gradient mark + wordmark), search button (32px, outlined), nav rows 34px tall, 16px outline icons (1.5px stroke), active row `--nc-active` with the icon in `--nc-accent`. Count badge = gradient pill. Group label (sentence case, 12px, muted) before "Your circles". User row pinned to the bottom. No top bar chrome; the top strip only holds utilities (theme, palette, loop explainer, controls, notifications). |
| **Card** | `--nc-surface`, 1px `--nc-border`, radius 14px, padding 20px, no shadow. |
| **Next-action card** | Full gradient background, no border, padding 24px, headline 22/700 in `--nc-on-grad`, supporting text at 82% opacity, **primary button = solid `--nc-on-grad` background with `--nc-g1` text**, secondary buttons = 18% `--nc-on-grad` tint, links underlined at 55%. Exactly one per screen. |
| **Button** | 38px tall (32px small), radius 6px, 1px `--nc-control` border, 500 weight. Primary: gradient, no border, 600 weight, `filter: brightness(1.08)` on hover. Ghost: no border, muted text. `:active` → `transform: scale(.97)`. |
| **Chip / tag** | 22px tall pill, `--nc-surface-2` fill. Filter chip (pressable): 28px, outlined, selected = `--nc-accent-bg` + `--nc-accent-ink` + accent border. Status chips use semantic pairs (pale bg + dark ink). |
| **Progress bar** | 8px, fully rounded, track `--nc-accent-bg`, fill `--nc-grad`, width animated 500ms `var(--ease)`. Fill is always the true ratio. |
| **Goal mix bar** | 10px stacked, 2px gaps: segment 1 gradient, 2 = 55% of `--nc-g1`, 3 and 4 neutral greys. |
| **Stat** | Number 34/800 in gradient text over a 12px muted label. No icon, no sparkline. |
| **Message bubble** | Mine: gradient + `--nc-on-grad`. Theirs: `--nc-surface-2`. Radius 8px, max 75% width. |
| **Avatar** | Circle, flat tint (see §3.4). **Story ring**: 2px gap in the surface colour then a 2px gradient ring, shown only when there is a real open introduction/request. |
| **Toggle** | 32×18, off = `--nc-control`, on = gradient. |
| **Tabs** | Underline tabs on a 1px rule; selected = text colour + 2px accent underline. |
| **Inputs** | 38px, radius 6px, 1px `--nc-control`, white (surface) fill, label above (14/500). Errors via `:user-invalid`. Placeholder `#6b6b6b` minimum. |
| **Dialog / popover / toast** | The only elements with a shadow (`--nc-shadow-lg`). Toast = solid `--nc-primary` background. |
| **Notice banner** | `--nc-surface-2` fill, muted text, no border. |

### 3.7 Palette and theme switchers (prototype only)

The prototype has a sun/moon button and a **Palette** popover (native Popover API) so the team can compare. They are scaffolding: production keeps the theme control in Settings → Appearance and decides on palette per §14.

## 4. App shell and navigation

### 4.1 Desktop (≥ 1024px) — Circle-style

```
┌───────────────┬──────────────────────────────────────────────┬──────────────┐
│ Sidebar 260px │ Main column (max 720–880px)                  │ Context rail │
│               │                                              │ 320px        │
│ Logo          │ Page header (title + one primary action)     │ (optional,   │
│ Search  ⌘K    │ Content                                      │ per page)    │
│               │                                              │              │
│ Home          │                                              │ e.g. "Why    │
│ Find Your Four│                                              │ this match", │
│ Discover      │                                              │ upcoming     │
│ Circles       │                                              │ session,     │
│ Build Missions│                                              │ profile      │
│ Sessions      │                                              │ summary      │
│ Messages      │                                              │              │
│ Opportunities │                                              │              │
│ Passport      │                                              │              │
│ ───────────── │                                              │              │
│ Your circles  │                                              │              │
│  · Circle A   │                                              │              │
│  · Circle B   │                                              │              │
│ ───────────── │                                              │              │
│ Avatar/Settings│                                             │              │
└───────────────┴──────────────────────────────────────────────┴──────────────┘
```

- Primary nav order matches Blueprint §9: **Home · Find Your Four · Discover · Circles · Build Missions · Sessions · Messages · Opportunities · Builder Passport**.
- "Your circles" is a collapsible sidebar group of the member's active circles (Circle's "spaces" pattern). It lists real circles only.
- Badges on nav items appear **only** when there is a pending action the member can take (e.g. "Messages: 2 unread", "Find Your Four: 1 intro awaiting your reply"). No badges for passive activity. Badge count = real count of actionable items.
- Global search (⌘K / Ctrl+K) opens a native `<dialog>` command palette: members, circles, missions, sessions, settings.
- Secondary (avatar menu, Popover API): Settings, Membership, Notification preferences, Help, Admin (role-gated), Sign out.

### 4.2 Tablet (640–1023px)
Sidebar collapses to an icon rail; context rail moves below the main content as stacked cards.

### 4.3 Mobile web (< 640px)
Bottom tab bar mirroring the Flutter plan (Blueprint §9): **Home · Discover · Build · Inbox · Profile**.
- *Build* groups Circles, Missions, Sessions.
- *Opportunities* live inside Discover with a goal filter.
- Find Your Four is a card on Home and a full page reachable from Home.
Use container queries for components (cards adapt to their own width) and viewport/dynamic-viewport units for the shell; respect safe areas.

### 4.4 Routes

| Route | Screen | Auth | Entitlement |
|---|---|---|---|
| `/` | Home | member | active membership |
| `/onboarding/*` | Persona wizard | signed-in | any (before match eligibility) |
| `/four` | Find Your Four | member | active |
| `/discover` | Builder Feed, member search, opportunities tabs | member | active |
| `/members/:id` | Member profile (privacy-safe) | member | active |
| `/circles`, `/circles/:id/*` | Circle list, workspace | member | circle member |
| `/missions`, `/missions/:id` | Mission list, mission detail | member | active |
| `/sessions`, `/sessions/:id` | Session hub, session detail | member | per admin-configured entitlement |
| `/messages`, `/messages/:conversationId` | Inbox and thread | member | connection exists |
| `/opportunities`, `/opportunities/:id` | List, detail | member | active |
| `/passport`, `/passport/:projectId` | Own passport; project story | member | active |
| `/settings/*` | Profile, goals, privacy, notifications, appearance, data | member | any |
| `/membership` | Plan, billing, renewal, cancel | member | any |
| `/safety` | Report/block centre, report status | member | **always available, never paywalled** |
| `/admin/*` | Session admin, moderation, review queue | admin role | role |

Route guards redirect: not signed in → sign-in; no persona → onboarding; lapsed membership → `/membership` (read-only access to Safety, data export, and billing is always preserved).

---

## 5. Screen contracts

Each screen lists: **Job · Primary action · Hooked loop · Layout · States · Guardrails · Events.** Events refer to the taxonomy in Blueprint §11.

### 5.1 Onboarding — Builder Persona (F01)

- **Job:** Turn a new member into a useful, editable matching profile in minutes.
- **Primary action:** "Continue" through short steps; final "See my first introductions."
- **Loop:** Trigger = first authenticated entry. Action = fill essentials. Reward = readable persona summary + any real opportunities. Investment = confirm/correct tags, add evidence. Next trigger = completeness prompt *only* when it unlocks something real.
- **Layout:** Single centred column (max 560px), progress indicator ("Step 2 of 5"), Back always available, autosave each step.
- **Steps (minimum viable):**
  1. Role and what you're best at (primary + secondary skills).
  2. What you can offer / what you're looking for.
  3. Goals as **editable weights** (presets: "Mostly startup", "Mostly career", "Balanced", plus sliders that always sum to 100%). Shows live preview of the weights.
  4. Availability, language, timezone.
  5. Privacy and matching consent (explicit, plain language) + optional portfolio links.
- **Result screen:** "Your builder persona" with the summary, every inferred tag editable/removable, and a clear note on what is and isn't shared.
- **States:** empty, loading (skeleton), validation errors (after interaction, see §10.2), save-failed (retry + keep local draft), offline (cached draft).
- **Guardrails:** Never require phone, social handles, precise location. Matching consent is a separate, unticked-by-default control. AI-suggested tags are labelled "Suggested" and editable.
- **Events:** `profile_started`, `profile_completed`, `goal_weights_updated`.

### 5.2 Home (macro-loop surface)

Order is binding (Blueprint §9). Single main column + optional context rail.

1. **Next meaningful action** — one card, rendered as the full-gradient Next-Action card (§3.3, §3.6): reply, review task, join live session, or accept introduction. If none exists: "Nothing needs you right now" with a calm suggestion (e.g. update a goal). No fake tasks.
2. **Find Your Four** — compact strip of up to four people with role and opt-in state; "See all" → `/four`.
3. **Your active build** — circle/mission card with milestone and the member's responsible next step.
4. **Relevant opportunity** — one item consistent with the member's goal weights, with fit rationale.
5. **Upcoming session** — date/time in the member's timezone, RSVP, resources.
6. **Finite discovery** — a short ranked list (≈5–8 items) ending with "You're caught up" and a link to `/discover`.

States: first-run (persona just completed, inventory may be empty), no circle yet (replace "Your active build" with "Start or join a build"), loading skeletons per module (modules load independently), error per module (never blank the page).

### 5.3 Find Your Four (F03) — signature screen

- **Job:** Show four complementary, available introductions and let the member respond individually or propose a circle.
- **Primary action:** "Request introduction" on each person (individually); secondary "Propose a circle with these people".
- **Layout:** Four person cards in a 2×2 grid (stack on mobile). Each card:
  - name, role, 2–3 skills, availability, language
  - **"What they could contribute"** (their proposed role in your project)
  - **Why this match?** — expandable plain-language reasons *and limitations* (e.g. "Timezone overlap is limited")
  - evidence chips with provenance label: *self-reported / peer-acknowledged / artifact-verified*
  - states: Not requested · Requested · Accepted · Declined · Expired
  - secondary: "Save", "Not now" (with optional reason: not the skill I need / wrong goal / timing)
- **Team coverage strip** above the cards: which capabilities the four cover and what is still missing (e.g. "Covered: backend, design, growth · Missing: mobile"). Honest, never gamified.
- **Fewer than four:** show what exists plus "We'll tell you when someone relevant joins" (waitlist). Never fabricate or pad.
- **Declines:** replacement suggestion is offered after a real decline/timeout, as a new card with an explanation. Never silently swap.
- **States:** empty (insufficient inventory), loading, partial (1–3 cards), all requested, circle ready, error.
- **Guardrails:** match scores are never shown as percentages or "chance of success"; show a descriptive tier at most (e.g. "Strong fit on goals"). No automatic enrolment.
- **Events:** `match_impression`, `match_reason_viewed`, `introduction_requested`, `introduction_accepted`, `introduction_declined`, `circle_created`.

### 5.4 Discover (F10 feed, F11 search, F09 opportunities)

Tabs: **For you · Search · Opportunities** (Opportunities tab hidden under Discover on mobile, see §4.3).

- **For you (Builder Feed):** finite ranked list of build updates, projects, problems. Card: author, project, update summary, evidence/artifact preview, actions **Give feedback**, **Save**, **Ask to contribute**. Ends with "You're caught up." No autoplay, no infinite scroll, no reaction-count ranking. Use a "Load more" only if the list is paginated deliberately, with a clear total cap.
- **Search (F11):** filter by skills offered, goals, availability, language, evidence level. Results respect blocks and visibility preferences. "Save this search" + opt-in digest.
- **Opportunities (F09):** list with goal filter chips mapped to the goal vector (Startup / Career / Freelance / Learning). Each item shows **fit rationale** and a "verified" label where applicable. Detail page: express interest / apply / pitch / request introduction. Clear line: "Introductions are not hiring guarantees."
- **States:** empty feed (explain why; suggest following topics), search no-results (suggest broadening, offer saved search), opportunity expired.
- **Events:** `opportunity_applied`, `opportunity_response_received`.

### 5.5 Member profile (privacy-safe)

Header (avatar, name, role, language, availability), "What I offer / What I'm looking for", skills with provenance, Builder Passport highlights, shared circles/connections. Actions: **Request introduction** (or **Message** if connected), Save, Block/Report (in a Popover menu).
No email, phone, or social handles unless the member chooses to share after connecting. Show "Blocked you / you blocked" states without leaking details.

### 5.6 Messages (F04)

- Two-pane layout: conversation list + thread (mobile: list → thread navigation).
- First message: **context-aware starter** — pre-filled prompts about goals and availability, editable, never auto-sent.
- Thread: message grouping, unread divider, retry on failure, pagination upward, typing/read receipts only if privacy settings allow **[OPEN]**.
- Header actions: Mute, View profile, Block, Report.
- Composer: multi-line, send with Enter/Shift+Enter, attachments **[OPEN]**.
- **States:** no conversations (explain how connections start), connection pending (composer disabled with reason), blocked, network lost (queued with visible status), moderation-removed message placeholder.
- **Guardrails:** no contact data in profile or composer by default; rate limiting shown as a calm message, not an error; report/block always reachable.
- **Events:** `message_sent`, `first_qualified_conversation`, `report_submitted`, `block_created`.

### 5.6b Connection requests
Inbox section "Introductions" lists incoming requests with context (their message, why-match), **Accept / Not now / Block**. Accepting grants in-app messaging only.

### 5.7 Circles (F05)

- **Circles list:** cards for the member's circles + pending invitations.
- **Circle workspace** (Circle-style tabs): **Overview · Mission board · Chat · Sessions · Members · Settings**
  - *Overview:* objective, deliverables, owners, cadence, next milestone, norms.
  - *Chat:* circle conversation, same patterns as §5.6.
  - *Members:* roles, capacity, contribution summary (balanced contribution view, not a ranking).
- **Creation:** confirm objective, deliverables, ownership, cadence, communication norms (a short form). Team size 2–5 by default; larger only with a reason. No pressure to reach exactly five.
- **States:** forming (pending acceptances), active, paused, completed, archived. Leaving a circle is a plain, shame-free action with a confirmation dialog.
- **Events:** `circle_created`.

### 5.8 Build Missions (F06)

- **Mission list:** filter by status, goal, skills needed; mission cards with deliverable, owner, due date (only team-agreed dates).
- **Mission detail:** tasks, milestones, artifacts, reviews, activity log. Primary action: **Claim a deliverable**, or **Mark milestone complete (attach evidence)** if you own one.
- **Review flow:** request review → reviewer sees artifact + checklist → accept / request changes. Completion produces a Passport entry draft.
- **Missed milestone:** supportive rescope dialog ("Want to adjust the plan?"), no shame language, no red alarms.
- **Progress display:** derived only from verified milestones. No fake progress bars.
- **Events:** `mission_started`, `milestone_verified`, `artifact_published`.

### 5.9 Sessions (F08) — Skool-"Classroom" + Circle-"Events" inspiration

- **Session hub:** upcoming sessions (calendar list), "This month's included sessions: X of N used" (N admin-configured, never hardcoded), past sessions with resources/recordings.
- **Session detail:** title, time in the member's timezone, what you'll build, prerequisites checklist, RSVP, add-to-calendar, join link **revealed only to entitled, RSVP'd members within the join window** (server-checked).
- **During/after:** "Claim an experiment" and "Submit a demo" actions; notes/code saved to the member's workspace; **one** follow-up task suggested from what they actually did.
- **RSVP ≠ attendance:** attendance is recorded separately.
- **States:** no entitlement left (explain plainly, show renewal date), session cancelled/rescheduled (clear banner), join link not yet available.
- **Events:** `session_attended`.

### 5.10 Builder Passport (F07)

- **Passport home:** profile summary + project stories grid. Each project story: role, artifact(s), outcome, contributors, and a **verification badge with text**: *Self-reported · Peer-acknowledged · Artifact-verified* (icon + label, never colour alone).
- **Publish flow:** pick a completed mission/contribution → write the story → choose visibility → request team validation (consensual).
- **Visibility:** private / members-only / public link. **[OPEN]** public passports require the SEO/rendering decision in §2.
- **Guardrail:** warn when content may expose confidential client work; no fabricated endorsements.

### 5.11 Contribution feedback (F12)
Triggered at milestone completion or collaboration end. Short structured review (contribution delivered, responsiveness, would collaborate again) + optional private note. Shown as a dialog or inline card, always skippable. Never public star ratings. Provide an appeal/clarify path for anything affecting trust.

### 5.12 Notifications centre and preferences (F13)
- **Bell Popover:** list of actionable items only, each deep-linking to the exact conversation/task/session. Group and de-duplicate. "Mark all read."
- **Preferences:** per-category toggles (Messages, Introductions, Missions, Sessions, Opportunities, Digest), channels (in-app, email, push), quiet hours + timezone, daily non-essential push cap (default 2), digest frequency. Transactional/security items are labelled "Always on."
- Preferences are explained in plain language with examples.

### 5.13 Membership and renewal (F14)
Plan, included sessions, next billing date, payment method, renewal status, **cancel in the same place and in the same number of steps as sign-up**. No "are you sure you want to lose everything" copy. Receipts listed. Payment failure shows a calm, specific fix. Web-based payments are acceptable on web; native in-app subscription rules are a Flutter-phase concern.

### 5.14 Safety, privacy, data (F15)
Always-available centre: Block list, Report a member/message/circle, report status, visibility settings, **export my data**, **delete my account**, leave/mute controls. Never behind a paywall, never gamified, factual status updates only.

### 5.15 Admin (role-gated) — minimal v1
Session manager (configure session count/schedule/links/entitlements), notification reminder scheduler with counts (targeted / queued / sent / delivered / failed / skipped — never conflate sent with delivered), match review queue (for manual review of low-quality matches in Phase 1), moderation queue. Detailed spec lives in `14_ADMIN_MODERATION_ANALYTICS_AND_TESTING.md`.

---

## 6. Component library

Build a small, accessible set first, then extend.

| Component | Notes |
|---|---|
| `AppShell`, `Sidebar`, `BottomTabs`, `ContextRail` | Responsive shell per §4 |
| `Button` (primary/secondary/ghost/danger), `IconButton` | One primary per page; native `<button>` |
| `Card` | Container-query aware |
| `PersonCard` | Used in Find Your Four, search, introductions |
| `WhyMatch` | Expandable reasons + limitations |
| `ProvenanceBadge` | self-reported / peer-acknowledged / artifact-verified (icon + text) |
| `GoalWeights` | Sliders that always sum to 100%, presets, keyboard accessible |
| `SkillPicker` | Typeahead over skills taxonomy; chips with remove |
| `Dialog` | Wrapper over native `<dialog>` (modal) |
| `Menu` / `Popover` | Native Popover API |
| `Toast` | Non-blocking, with undo where relevant; polite live region |
| `Tabs`, `Stepper`, `Skeleton`, `EmptyState`, `ErrorState`, `Banner` | |
| `Avatar`, `Tag`, `Chip`, `Tooltip` | |
| `MessageBubble`, `Composer`, `ConversationList` | |
| `MilestoneList`, `ArtifactCard`, `ReviewChecklist` | |
| `SessionCard`, `RSVPButton` | |
| `NotificationItem` | Deep-linking |
| `CaughtUp` | The finite-list terminator |

Every component documents: props, states (default, hover, focus-visible, active, disabled, loading, error), keyboard behaviour, and a visual regression story.

**EmptyState rule:** an empty state says *why* it's empty and offers one honest next step. It never invents activity.

---

## 7. States checklist (applies to every screen)

For each screen, design and test: **initial/empty · loading (skeleton, not spinner-only) · partial · success · validation error · server error (retry) · permission denied/entitlement lapsed · offline/slow network · blocked/privacy-restricted · very long text / Telugu strings · reduced motion · dark mode · high zoom (200%).**

---

## 8. Hook-loop UI patterns (how each loop shows up on screen)

| Loop element | UI pattern | Anti-pattern (banned) |
|---|---|---|
| **Trigger** | In-app next-action card; deep-linked notification; member-selected reminder | "We miss you" pushes; red badges without actions |
| **Action** | One primary button, prefilled where possible, completes in seconds | Multi-page forms for simple actions |
| **Variable reward** | Real outcomes: a reply, an accepted intro, feedback, a verified milestone | Random prizes, slot-machine reveals, pull-to-refresh gambling |
| **Investment** | Edit tags, add evidence, give structured feedback; show "this improves your recommendations because…" | Streaks, guilt, sunk-cost nudging |
| **Next trigger** | Real events only; digest for the low-priority | Timer-based pressure, fake urgency |

**Honest feedback copy:** after an action, tell the member what actually happened and what's next ("Request sent to Priya. We'll tell you when she replies.").

**Optimistic UI:** allowed for accept/decline/save with rollback on failure. Never optimistic for payment, verification, or safety actions.

---

## 9. Data and API contract (placeholder)

**[ASSUMPTION]** The Laravel backend exposes versioned REST endpoints (`/api/v1/...`) with token or cookie auth; the web client will confirm the actual auth method in Phase 0 (Blueprint §10). Until then, the frontend depends on typed interfaces so the transport can change.

Frontend-facing entities (from Blueprint §8): `User`, `BuilderProfile`, `GoalWeights`, `Skill`, `MatchCandidate` (with `reasons[]`, `limitations[]`, `tier`), `Introduction`, `Connection`, `Conversation`, `Message`, `Circle`, `Mission`, `Milestone`, `Artifact`, `Contribution`, `Session`, `Rsvp`, `Opportunity`, `NotificationItem`, `NotificationPreferences`, `Membership`.

Conventions:
- All times in UTC over the wire; displayed in the member's chosen timezone.
- Every list endpoint is paginated; the UI handles "end of list" explicitly.
- Error shape includes a stable `code` and a user-safe `message`.
- The frontend never receives contact data it isn't entitled to see.
- Entitlements and block/privacy filters are enforced server-side; client hides UI as a convenience only.

---

## 10. Engineering standards (native-first web)

### 10.1 Overlays
- **Modal dialogs** use native `<dialog>` opened with `showModal()` or declaratively (`command="show-modal"` + `commandfor`). Do **not** add `role="dialog"`, `aria-modal`, manual focus traps, or `inert` toggling to native modals; the browser handles focus containment, Esc, and focus return.
- Name every dialog with `aria-labelledby`; put `autofocus` on the primary control (or on the safe action in destructive confirmations).
- Stop scroll chaining with `overscroll-behavior: contain` on `dialog:modal` and `dialog::backdrop`; do not lock scroll with `overflow: hidden` on `html`/`body`.
- Click-outside dismissal via `closedby="any"` is progressive enhancement (not in Safari at time of writing); Esc and back gesture still work. Unsaved-changes guard lives in a `cancel` handler.
- **Menus, dropdowns, tooltips, toasts** use the Popover API (`popover`), not `<dialog>`. Invoker Commands and Popover are Baseline newly available; load polyfills only behind feature detection if the browser policy requires.
- Read `dialog.returnValue` in the `close` handler, then reset it to `''`.

### 10.2 Forms
- Use native constraint validation attributes (`required`, `type="email"`, `pattern`, `minlength`) and `autocomplete` tokens.
- Show errors only after the user interacts, via `:user-invalid`; never on page load or mid-typing.
- Error text is visible, begins with an error icon with a text alternative (or is worded as an error), says how to fix it, and is linked to the field with `aria-describedby` **only while invalid**. Put hints above inputs so mobile keyboards and autofill popovers don't cover them.
- Persist drafts for long forms (onboarding, project story).

### 10.3 Motion
Motion makes the product feel alive and responsive, and always explains a real change.
- **Press feedback on everything pressable:** buttons, icon buttons, filter chips, nav rows scale to `0.97` (`0.95` for small icon buttons) on `:active`, 160ms, `var(--ease)`. Gate hover-only effects behind `@media (hover: hover) and (pointer: fine)`.
- **Use `transform` and `opacity` only**; transition specific properties, never `transition: all`. Never animate from `scale(0)`; start at `0.95` with opacity.
- **Timing:** UI transitions 150–250ms, modals/drawers ≤ 300ms, progress bars 500ms. Easing is `cubic-bezier(.23, 1, .32, 1)`; do not use `ease-in` for UI. Exits are faster than enters.
- **Never animate keyboard-initiated, high-frequency actions** (command palette, list navigation).
- **Entrances:** a card moving to "requested", a milestone completing, a toast arriving. Use CSS transitions (interruptible), not keyframes, for things that can be triggered rapidly. `@starting-style` for enter states where supported.
- Same-document View Transitions may be used for route/list changes **with** a `prefers-reduced-motion: reduce` override. Under reduced motion keep opacity/colour changes and drop movement.
- No autoplay, looping or attention-seeking animation; no confetti, slot-machine reveals or fake-urgency effects (§0, §8).

### 10.4 Layout
- CSS Grid/Flexbox; **container queries** for cards and panels; dynamic viewport units (`dvh`) on mobile; logical properties for future RTL support.
- Minimum touch target 44×44px on touch devices.

### 10.5 Performance (targets, to be validated)
- Core Web Vitals "good": LCP ≤ 2.5s, INP ≤ 200ms, CLS ≤ 0.1 (p75).
- Route-level code splitting; images with explicit dimensions and lazy loading below the fold; fonts with `font-display: swap` and subset (Latin + Telugu).
- Long lists (message history, search results) use `content-visibility: auto` with `contain-intrinsic-size`, or windowing for very long threads. Because feeds are finite, virtualization is rarely needed.
- No layout shift when skeletons resolve to content (reserve space).
- Analytics and chat transports must not block first render.

### 10.6 Accessibility and i18n
- Target **WCAG 2.2 AA**. Semantic landmarks (`header`, `nav`, `main`, `aside`), one `h1` per route, visible `:focus-visible` rings, skip-to-content link.
- Full keyboard operation of every flow, including GoalWeights sliders and the command palette.
- Status changes use polite live regions; toasts never contain the only copy of important information.
- Provenance and state are never colour-only.
- Telugu + English at launch; all strings externalised; ICU plurals; date/number formatting via `Intl`; layouts tolerate ~30–40% text expansion.
- Test with a screen reader (VoiceOver + NVDA) and axe in CI.

### 10.7 Security and privacy (frontend)
- No secrets in the client bundle. Tokens stored per the approved auth design (prefer HttpOnly cookies if the backend supports it **[OPEN]**).
- Sanitise any user-generated rich text; render plain text by default.
- Do not put private message bodies or contact fields into analytics or error logs.
- Content-Security-Policy and strict referrer policy set at the edge.

---

## 11. Analytics instrumentation

Use the event taxonomy in Blueprint §11. Rules: consent-aware, pseudonymous IDs, minimal properties, never log message bodies or contact data. Add `notification_*` events from deep-link landings so we measure **qualified action**, not opens. Instrumentation is a typed wrapper (`track(event, props)`) so event names cannot drift.

---

## 12. Build order (web)

| Step | Deliverable | Depends on |
|---|---|---|
| 0 | Repo scaffold, tokens, theme, i18n, lint/test/CI (axe) | — |
| 1 | App shell, routing, guards, component library v1 | 0 |
| 2 | Auth integration, onboarding wizard + persona result | Phase 0 API audit |
| 3 | Home + Find Your Four + introductions + member profile | matching API |
| 4 | Messages + safety/block/report + notification preferences | chat API |
| 5 | Sessions hub + membership/billing views | existing sessions/Razorpay |
| 6 | Circles + Build Missions + reviews | circles/missions API |
| 7 | Builder Passport + feedback | verification API |
| 8 | Discover (feed/search) + Opportunities | opportunities API |
| 9 | Admin minimum + hardening (a11y, perf, i18n audit) | all |

Phase mapping: steps 2–5 ≈ Blueprint Phase 1; 6–7 ≈ Phase 2; 8 ≈ Phase 3.

---

## 13. Acceptance checklist (per screen, from Blueprint §12)

- [ ] Names its job, one primary action, and loop (T/A/R/I/N)
- [ ] All states in §7 designed and implemented
- [ ] Privacy/consent/abuse/opt-out behaviour present
- [ ] Entitlements enforced server-side; UI reflects them
- [ ] Desktop, tablet, and mobile-web layouts verified
- [ ] Keyboard, screen-reader, contrast, reduced-motion, dark-mode checked
- [ ] Telugu and English strings verified
- [ ] Events instrumented; success + guardrail metrics defined
- [ ] No hidden phone/email sharing, no invented engagement claims

---

## 14. Open decisions

1. **Public SEO pages:** are public Passports/session landing pages required at launch? (Drives Next.js vs. SPA.)
2. **Brand:** which of the five palettes in §3.2 (or a custom one built to the same token contract) is the default, and whether members can choose; final typeface (DM Sans is the working choice), logo, and tone for Telugu copy. Verify gradient/text contrast per §3.4.
3. **Auth method** of the existing Laravel app (cookie session vs. token) and whether web and mobile share it.
4. **Chat transport** (WebSocket vs. SSE vs. polling) and read-receipt/typing indicator policy.
5. **Browser support policy** for newer platform features.
6. **Attachments** in chat and artifacts: allowed types, size limits, scanning.
7. **Skool-style sessions "classroom"**: do we show recordings/resources as modules, or keep sessions as events only?
8. **Reference screenshots:** the visual direction is now defined by `Nightclub · Web UI Prototype.html`; team to share any further Circle, Skool, Instagram, Blinkit or Instamart screens that should refine it.

---

## 15. Final statement

The web app should have Circle's clear structure and the colour, speed and polish of the best consumer apps (Instagram, Blinkit, Instamart), applied to Nightclub's purpose: **a place members want to return to because the right people find each other there, build something real, and turn that work into their next opportunity.** If a design choice increases screen time without increasing real collaboration, it is wrong.
