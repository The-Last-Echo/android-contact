# DESIGN.md

`design_system_version: 1.2.0` (see Changelog at the end)

Shared design system for this Android-only Kotlin portfolio: **Soft Motion**. Rounded shapes, one
accent color that drives the palette, expressive typography, optional spring-based motion, and
optional navigation patterns. Implement using Android/Kotlin and Jetpack Compose. Do not copy code
or unlicensed assets from other projects. Each application selects only the styles and features it
actually needs; this document is a design vocabulary, not a mandate to implement every feature.

**Implementation rule:** consult this file before UI work. Section 11 (per-app config) states which
features this app actually offers and overrides optional defaults. Do not build optional capabilities
just because they are described here. If a needed reusable token/component is missing, first check
the existing project. Add it to the shared design layer only if reuse is justified, document it here,
and test it. For a small app, a package may be sufficient; a separate module is not mandatory.

---

## 1. Principles

1. **One accent, coherent palette.** When accent customisation is offered, derive tints,
   containers and highlights from the chosen colour. If the app does not need custom colours, use a
   carefully chosen fixed palette instead.
2. **Shapes carry state.** Pills and squircles; a shape morphs when its state changes.
3. **Motion has physics.** Springs, small overshoot, squash on press. Nothing linear.
4. **Calm by default, delight on success.** Everyday screens are quiet; achievements get
   a short celebration.
5. **Optional style packs, shared tokens.** `classic`, `minimal` and `expressive` are available
   design directions, but each app offers only the pack(s) appropriate to its purpose. Components
   read tokens so supported packs remain coherent.
6. **Customization is optional.** Brightness, accent, style and background pickers are not mandatory.
   Add and persist only settings that materially benefit the app. When live theme switching exists,
   avoid flashes and apply it accessibly; a short cross-fade is a suggestion, not a requirement.
7. **Professional restraint.** Clear hierarchy, generous spacing, no decoration without
   purpose, never guilt-trip copy.

## 2. Theme model and color

A theme can be described by **style pack** (`classic` | `minimal` | `expressive`) + **brightness**
(`light` | `dark` | `system`) + **accent** + **background**. These are design dimensions, not a
requirement to expose every option to users. Check the per-app configuration before implementing
pickers, persistence, previews, or optional background rendering.

**Accent engine:** take the seed; if dark mode and luminance < 0.06 use near-white
(`#F2F2F2`); if light mode and luminance > 0.82 use near-black (`#1C1C1E`); generate a
tonal scheme (Material 3 from-seed or HCT). On-accent text is black above luminance 0.5,
otherwise white. Luminance is **WCAG relative luminance** (0 to 1, sRGB). Helpers are
the accent at a given opacity, painted over the surface the element sits on:
`tonal` 16% (soft fills, icon wells), `selected` 18% (selected container),
`tintRest` 6% (resting tinted surfaces, e.g. rail items) and `tintHover` 16% (pointer
hover or keyboard focus).

**Surface ladder (reference values, tune per app):**

| Level | Dark | Light |
|---|---|---|
| base (page) | `#121212` | `#F2F2F2` |
| 0 lowest | `#171717` | `#FFFFFF` |
| 1 card | `#242424` | `#FFFFFF` |
| 2 raised | `#2C2C2C` | `#ECECEC` |
| 3 highest / disabled | `#363636` | `#E0E0E0` |
| line | `#383838` | `#DCDCDC` |

Hairline = line colour at 25%, 1 dp (or the thinnest accessible platform-appropriate stroke).

**Elevation and layering (bottom to top):**

| z | Layer | Shadow |
|---|---|---|
| 0 | Background | none |
| 1 | Content, cards, tiles | none (Classic light: soft 1-2 dp shadow) |
| 2 | Sticky headers, scrims | none |
| 3 | Dock / glass bar, rail, FAB | large soft |
| 4 | Menus | medium |
| 5 | Bottom sheets | medium |
| 6 | Dialogs (with scrim) | medium |
| 7 | Snackbars (above the dock, 12 dp gap) | medium |
| 8 | Celebrations (non-interactive overlay) | none |

Only layers 3-7 may cast shadows. Everything else is flat, separated by surface levels
and hairlines.

**Semantic tokens:** success `#22C55E`, warning `#F59E0B`, danger `#EF4444`, info
`#3B82F6`, muted text `#6B7280` (light) / `#9CA3AF` (dark). Read them from a `tokens`
theme extension, never as constants.

**Backgrounds:** solid = base surface; gradient = soft vertical; dots = grid, 26 dp gap,
~1.1 dp radius, 3-9% opacity; black = pure black/white (OLED); photo = user image with a
scrim (dark 45-65% black, light 55-72% white), falling back to solid if missing.

**Entity colours** (categories, labels): when category colours are useful, use a curated palette or
picker appropriate to the app. Any user-selectable colour must pass a legibility guard and on-colour
rule; a colour picker is not mandatory for apps without colour-coded entities.

## 3. Typography

Optional font families in the original design concept are **Google Sans Flex** (variable), **Figtree**
and **Playfair Display** (optional serif accent). Verify the actual font file's licence, version and
redistribution terms before bundling it; record it in `THIRD_PARTY.md`. Declare suitable system/Noto
fallbacks for scripts not covered by the chosen font. Prefer the platform font if a custom family does
not add enough value.

Expressive recipes (weight / width / optical size / roundness): `display` 950/85/30/100,
`headline` 700-800/115/32/60, `body` 450-500/100/16/0.

| Token | Size / line | Use |
|---|---|---|
| pageTitle | 38 / 1.05, tracking -0.6 | Screen heading |
| headlineS | 24 / 32 | Sheet and dialog titles |
| titleM | 16 / 24, w800 | Rows, cards |
| bodyM | 14 / 20 | Default text |
| bodyS | 13 / 1.35 | Subtitles |
| labelL | 15.5, w800 | Buttons |
| labelM | 12-13.5, w700-800 | Chips, captions |

Sizes in the table are `sp`; line heights are `sp`; tracking is in `sp` where relevant. These are
starting tokens, not immutable values—validate against Compose typography, font scale and readability.
Rules: tabular figures for every changing number; tight tracking on large sizes only;
primary text `onSurface`, secondary `muted`, values and links in accent; support OS text scale to 200%. Reflow content when the available width is constrained; at large text
sizes, stack values under titles and remove decorative (not informative) icons before text or controls
become clipped. Validate actual layouts rather than hard-coding the text-scale multiplier into width
breakpoints.

## 4. Shape, spacing, layout

**Unit rule:** use `dp` for layout, shape radii, offsets and touch targets; `sp` for text. Time values
are milliseconds and opacity values are percentages. The numerical values below are reference tokens,
not raw pixel values; implement them through the app's theme/token layer when those components are used.

- Spacing base 4 dp. Page padding 18-20 dp. Card padding 20 dp. Section gap 22-28 dp.
- Radii: `pill` = height/2, `card` 24, `cardLarge` 32, `control` 14-20, `joint` 8.
- **Grouped lists:** gap 4; first row large corners (24) on top, last row large on
  bottom, inner corners 8; single row large all round.
- **Squircle:** superellipse, exponent ~4.2. **Lobed shapes:**
  `r(a) = base(a) * (1 - amp + amp * cos(lobes * a))`; keep a family of ~7 (cookie 9
  lobes/0.085, flower 6/0.14, burst 12/0.055, clover 4/0.155, gem, pebble, sunny).
  Morph by interpolating parameters.
- **Responsive:** window width < 640 dp (phone patterns); rail >= 640 dp on larger windows; split (master-detail)
  >= 900 dp with list pane around 480 dp and detail pane max around 900 dp; compact icon-only rail
  below 1060 dp. Single-column pages may be centered with max width 680-760 dp. Treat these as
  starting breakpoints and validate using Android window-size guidance and the actual layout.
- Edge-to-edge with transparent system bars; respect all insets.

## 5. Motion

| Token | ms | Use |
|---|---|---|
| tap | 130-170 | Press-in |
| quick | 250-300 | Toggles, fades |
| base | 400-450 | Selection, size changes |
| slow | 650-700 | Entrances |
| morph | 520-560 | Container transforms |

Springs: `bouncy` (damping 0.55, stiffness ~380), `springy` (0.7, ~450), `loose`
(0.45, ~220). `emphasized` = cubic-bezier(0.2, 0, 0, 1). Classic/Minimal use ease-out
(Minimal without overshoot). Nothing interactive exceeds 700 ms.

**Patterns**
- **Press squish:** cards 0.965, buttons 0.94, FAB 0.90, check 0.88; release springs
  back with `springy`. Buttons go pill to radius 16 while pressed; icon buttons morph squircle to cookie.
- **Entrance:** fade + rise 16-18 dp + scale 0.97 to 1, stagger 40-45 ms per item
  (index capped at 8). Replay on Home after 5+ minutes away.
- **Tab switch:** fade-through 300 ms; keep visited tabs alive, build lazily.
- **Pushed screens:** slide-through (~10% offset + fade); optional container transform
  (520 ms forward, 400 ms back). Mirror for RTL.
- **Selection pill:** slides with `springy`, stretches up to 13% x distance mid-flight.
- **Completion pop (720 ms):** ring scales to 1.55 and fades; control overshoots +/-16%;
  icon cross-fades with scale.
- **Switch:** 56x34 track; thumb grows 20 to 26 and becomes a cookie with a check.
- **Progress:** wave bar (amplitude ~3.6 dp, wavelength ~26 dp, loop ~2.4 s).
- **Menus:** morph menu grows width then height (460 ms open / 340 ms close).
- **Celebration:** confetti 1.7 s or fireworks 2.4 s, only for the events listed in `celebrations` in the app config (none by default).
- **Haptics** (optional, user setting, never on scroll or per drag frame):

| Event | Feedback |
|---|---|
| Tab switch, toggle, chip select | light tick |
| Completion, long-press pickup | medium tick |
| Celebration | success pattern |
| Error or blocked action | error pattern |

**Reduced motion:** skip entrances, springs, loops and celebrations; keep state changes
visible (instant or 100-150 ms fade). **Performance:** animate transform/opacity,
isolate heavy painters, dispose controllers, stop ticking offstage tabs, target 60 fps.

## 6. Navigation

3-5 destinations, outline icons (one family, 18-21 dp) plus localized labels; first item
is Home. Selecting the current item does nothing.

- **Expressive dock (phone):** floating pill, 12 dp above bottom inset, padding 7,
  radius 34, fill = surface blended with accent (22% dark / 16% light), soft large
  shadow. Items 48 dp high: selected = accent pill with icon + label (w800, 14), icon
  scale 1.06; unselected = icon only. Spring + animated size.
- **Classic glass bar (phone):** capsule 64 dp high (collapses to 50 when scrolling
  down, restores on scroll up/top), backdrop blur ~22, surface 70-78%, 1 dp light
  border, double shadow. A **lens** highlight follows a spring (stiffness ~420, damping
  ~30), stretches with velocity (up to 28%); long-press (~220 ms) lets the user drag the
  lens across items and release to select. Soft bottom blur scrim behind.
- **Minimal (phone):** no bottom bar; single calm Home, other areas via in-page entries
  and sheets.
- **Rails (wide):** width 224 / 208 (80 / 76 compact), brand on top, optional footer.
  Expressive: selected pill radius 24 with a cookie icon well, others radius 14.
  Minimal: ink-inverted selected tile, radius 14. Hover tint, pointer cursor, tooltips
  when compact. Escape closes the detail pane.
- **Back:** close overlays and pushed screens first; then go to Home; then exit.
- **Page chrome:** transparent 60 dp app bar, 38 dp squircle back button, `pageTitle`
  with muted subtitle revealed on entry.

## 7. Components

Read tokens only. Expressive defaults:

- **Button:** filled / tonal (accent 16%) / ghost; height 52 (compact 40); pill; disabled
  uses surface 3 and muted text. **Icon button:** 38 dp, squircle, surface 3 at 65%.
- **FAB:** 62 dp cookie, accent glow, quarter-turn spin on tap with upright icon.
- **Check:** 48 dp, outline accent 45% when open, filled when done; hit area +12 dp.
- **Chips:** height 40; active = accent fill, radius 20; inactive = surface 2, hairline,
  radius 14, scale 0.94. **Segmented tabs:** 46 dp, stretching accent pill.
- **Card:** surface 1, radius 24, padding 20, hairline. **Tile:** 40 dp tinted icon well,
  title 16/w800, muted subtitle, accent value, group-driven corners.
- **Dialog:** surface 2, radius 32, scale-in with `springy`; **sheet:** top radius 32,
  max width 560, drag handle. Destructive actions use `danger` and confirm.
- **Empty state:** soft shape, short title, one sentence, one primary action.
- **Charts:** accent and tints, hairline grids, rounded bars, tabular figures.

- **Loading:** skeleton shapes in surface 2 that match the final layout, opacity pulsing
  100% to 60% over 1.2 s (static under reduced motion). Show only after 200 ms of waiting.
- **Errors:** field errors inline (danger text + icon, never color alone); screen-level
  failures use an empty-state layout with a Retry button; transient failures use a
  snackbar with a Retry action.
- **Offline:** a slim info banner under the app bar ("You're offline. Changes are saved on
  this device."), dismissible. In offline-first apps this is information, not an error.
- **Snackbar:** layer 7, radius 20, surface 3, `bodyM`, at most one action in accent,
  slides up 300 ms with `emphasized`, auto-dismiss 4 s (6 s with an action), swipe to
  dismiss, one at a time, queued. Danger variant adds an icon.
- **About / Licenses page:** app name and version, license of the app, list of bundled
  fonts, icons, sounds and libraries with their licenses, and a link to the source.
- **Icon sizes:** inside 38 dp icon buttons 17 dp; navigation 18-21 dp; tiles 19 dp;
  FAB 26 dp; chips 15 dp.
- **Component done:** every shared component implements the states rest, pressed,
  hovered, focused, disabled and (when relevant) selected and loading; works in all style packs enabled in this app, light and dark modes that the app supports; uses tokens only; exposes semantics; has a golden test; is
  documented in this file.

Per pack: Classic = radius 24, soft light shadow, glass; Minimal = ink buttons (radius
18), flat list container radius 22 with 1 dp dividers, press scale ~0.97, no bounce;
Expressive = pills, hairlines, springs.

## 8. Localization

No user-visible string hard-coded in UI code. Use Android string resources (`strings.xml`) with
stable descriptive keys, named placeholders, proper plurals, and locale-aware dates and numbers.
English may be the source/fallback language unless the product decides otherwise. The app name is
not translated unless there is a deliberate, documented brand/localisation decision. A translation platform such as Weblate may be used if it suits the contribution workflow; only list
languages that are actually shipped and maintainable. Budget +40% text width, support RTL
(start/end, mirrored directional icons and transitions), test a pseudo-locale and a
non-Latin script. CI fails on missing keys or broken placeholders.

## 9. Accessibility and quality

- Contrast: target at least 4.5:1 for normal text and 3:1 for large text and meaningful non-text
  controls, in the visual configurations the app actually offers. Verify combinations when accents
  are user-selectable; never assume any arbitrary accent automatically has sufficient contrast.
- Interactive touch targets should generally be at least 48 dp. Every custom control exposes an
  appropriate role, label and state.
- **Focus ring token:** 2 dp ring in accent (at least 3:1 against what is next to it),
  2 dp offset, following the component's shape; shown for keyboard and D-pad focus only.
  Logical order, no traps. Never rely on color alone.
- **Insets:** read system insets once into an `insets` token (top, bottom, start, end).
  Scrollable content pads by it; floating bars sit at `insets.bottom + 12`; page bottom
  padding = bar height + `insets.bottom` + 24.
- No tracking by default; permissions requested in context; no dark patterns.
- Tests: goldens for the actual configured styles/brightness modes at normal and large text where that testing
  method is used; interaction, semantics,
  reduced-motion, RTL, and breakpoint tests.

**Definition of done for UI work (apply relevant checks to features actually offered):**
- [ ] Shared tokens/components are used where available; avoid scattered unexplained visual constants.
- [ ] Every style pack, brightness mode, accent option and background type that this app actually offers
      has been checked; do not build/test unimplemented optional packs as a requirement.
- [ ] Motion is purposeful, respects reduced-motion needs, and does not block interaction.
- [ ] Large text and long translations have been checked for clipping where localisation/font scaling is supported.
- [ ] Screen-reader semantics and states are verified for interactive controls; phone and wider layouts
      are checked when within product scope.

## 10. Implementation

Implement the design layer in proportion to the app. If reuse is useful, organise tokens, theme,
typography, shapes, motion and components in a dedicated module; for a small app, a well-separated
package is enough. Implement only the features declared in section 11.

- **Jetpack Compose:** Material 3 colour-scheme APIs supported by the project's pinned Compose version (derive a scheme
  from a seed only when a supported API or verified implementation is available), a project
  typed token holder or `CompositionLocal` for custom tokens,
  Compose `spring(...)` animations when used, custom `Shape` only when needed, and variable font
  settings only when the bundled font and device support justify them. Avoid unconditional blur or
  expensive custom drawing; provide a stable fallback and test supported Android versions.
- Persist style, brightness, accent, background, language or other preferences only if those options
  are enabled for this app. If persisted settings evolve, use a versioned/migratable representation
  where needed, and define a safe fallback for unknown values.
- Assets: verify redistribution licences and record every bundled font/icon/image/sound in
  `THIRD_PARTY.md`. Add an About/Licences page when appropriate for the app; do not implement a large
  settings surface in a simple tool solely because this design system describes it.

## 11. Per-app configuration

Values below are the proposed configuration for the contacts app. The app name and application ID are provisional/undecided. Enable only the styles, settings and visual effects that genuinely improve contact management.

```yaml
design_system_version: 1.2.0
app_name: "Contacts (working title)"
application_id: "TO_BE_DECIDED"
purpose: "Android contact management with optional direct multi-account CardDAV sync"
platforms: [android]
default_style: minimal
styles_offered: [minimal, expressive]
default_brightness: system
amoled_variant: false
accent_customization: false
accent_presets: []
dynamic_color: false
background_options: [solid]
fonts: { primary: system, serif_accent: none, extra_scripts: [] }
navigation: []
fab: { enabled: true, action: create_contact, icon: plus }
celebrations: []
languages: [en, fr]
rtl_required: true
settings_schema_version: 1
exceptions:
  - "Use a calm, compact information hierarchy; contact data must stay more prominent than decoration."
  - "Do not implement all style packs/settings merely because the shared system describes them."
  - "Avoid blur-heavy navigation or custom animations that harm list scrolling, accessibility or battery."
```

### Product-specific design priorities

- The list and search are the primary surface; optimize for scanability and one-handed use.
- The contact detail screen prioritizes actionable data, with clear grouping for phone numbers, e-mail addresses, postal addresses, notes and other fields available from the source.
- Use initials or a neutral placeholder when no photo is available. Load photo thumbnails without blocking scrolling.
- Put source/account information where needed to prevent confusion between contacts from different accounts.
- Destructive operations need clear target and source context. Avoid decorative confirmation animations.
- Only implement a floating action button if it remains discoverable and does not obscure the list or navigation.
- Respect phone window insets, system bars, large text, TalkBack, keyboard/focus and RTL layout direction.
- Contact names, organization names and notes can contain any script; do not assume Latin-only names.
- No user-visible strings hardcoded in Kotlin. Use Android string resources, plurals and locale-aware formatting.

## Changelog

- **1.2.0**: Android/Compose-only implementation guidance; explicit optional adoption per app;
  Android dimension units; stronger accessibility defaults; removed cross-platform implementation details.
- **1.1.0**: precise accent opacity helpers and WCAG luminance; elevation and layering
  table; loading, error, offline and snackbar patterns; focus ring and insets tokens;
  haptics table; icon sizes; About page and `THIRD_PARTY.md`; settings schema version;
  component definition of done; versioned per-app config template and optional feature selection.
- **1.0.0**: first condensed version.
