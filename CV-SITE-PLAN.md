# Development plan: madara-kalpina-cv

Repo: `madarakalpina/madara-kalpina-cv` · Live: https://madarakalpina.github.io/madara-kalpina-cv/
Everything lives in one file, `index.html` (inline CSS and JS). Keep it that way: no build step, no frameworks.

**Goal:** a CV site that a recruiter can skim in 30 seconds on any screen, that shows every role, and that prints or downloads correctly.

Work through the phases in order. Commit after each phase. Run the checks in Phase 7 before the final push.

---

## Problems found (for context)

| # | Problem | Where | Effect |
|---|---|---|---|
| 1 | The pinned experience section unpins before the last scroll marker reaches its trigger zone | `.exp-scroll-stage`, `.exp-sentinel`, the scrollspy JS | **Web Editor never shows.** SEB Product Owner only flashes at the end. At 1280×720, TechChill can be skipped. |
| 2 | Content panels sit inside a fixed-height pinned box with `overflow: clip` | `.exp-content-panels` | On short screens (720px tall), the last Green0meter bullet is cut off and can't be reached. |
| 3 | Inactive panels have `opacity: 0` and there is no print stylesheet | `.exp-content-panel` | **"Print / PDF" outputs only one job.** |
| 4 | The nav sits below a 100vh hero | `<nav>` after `<header id="hero">` | You can't see the menu when the page loads. |
| 5 | Nav links are 0.78rem at 65% opacity with no active state | `nav a` | Hard to read, and you can't tell which section you're in. |
| 6 | Six nav items, and on mobile they overflow sideways with a hidden scrollbar | `nav ul` | "Contact" is easy to miss on phones. |
| 7 | The dark mode toggle sets `data-theme` but there's no dark CSS, and colours are hard-coded all over the stylesheet | `#theme-toggle`, all CSS | The button does nothing. |
| 8 | Role labels in the sidebar change font size when they become active | `.exp-role-item.active .role-company` | The layout jumps. |
| 9 | Role labels are clickable `div`s | `.exp-role-item` | Keyboard and screen-reader users can't use them. |
| 10 | Site text is out of date compared with the PDF CV | Experience, ribbon | Old wording and mismatched dates. |

---

## Phase 1: Sync content with the PDF CV

The PDF is the source of truth. Use **British spelling** throughout (organised, analysed, prioritisation).

**Pixelfield** — replace the bullets with:
- Delivered up to 14 short-term projects in parallel, on time, across mobile, web, VR and e-commerce while balancing the everyday challenges of agency life.
- Managed end-to-end project delivery using the PRINCE2-Agile framework, combining client-facing governance, scope control and stage sign-offs with agile sprint cycles.
- Worked closely with first-time founders to translate early-stage startup ideas into clearly defined product initiatives, epics and development-ready user stories.
- Led the full product lifecycle from discovery and PRD preparation through design, development and launch across a wide range of digital B2C products, including mobile and web apps, VR, games, e-stores and landing pages.

**Green0meter** — the text is unchanged. Change "3rd party" to "third-party".

**TechChill** — replace the bullet with:
- Alongside a full-time role at SEB, curated a programme of 130+ speakers for a two-day startup conference in 2023 in Riga, Latvia and in Milan, Italy.

**SEB Bank**
- Agile Product Owner: change the dates to **2020 – 2022**.
- Change "frontend developers" to "front-end developers".
- Web Editor: **2019 – 2020**. Add a full stop at the end of each bullet.

**Ribbon** (`#ribbon`, both copies of the loop)
- "Data Driven Decision Making" → "Data-Driven Decision-Making"
- "Highly Organized" → "Highly Organised"
- "Organization in Chaos" → "Organisation in Chaos"

**Meta tags** — add to `<head>` so the link previews well on LinkedIn and in email:
```html
<meta name="description" content="Madara Kalpina — Product Manager in Prague. B2B SaaS, agency and banking experience; end-to-end product ownership, Agile/PRINCE2.">
<meta property="og:title" content="Madara Kalpina — Product Manager">
<meta property="og:description" content="B2B SaaS, agency and banking product experience. Based in Prague.">
<meta property="og:type" content="profile">
```

**Housekeeping:** delete `.DS_Store`, and add a `.gitignore` that contains `.DS_Store`.

---

## Phase 2: New navigation

**Structure**
- Move `<nav>` above `#hero` so it's the first thing on the page. Keep `position: sticky; top: 0`.
- Left side: "Madara Kalpina" as a link to `#hero` (Barlow Condensed 800, oat colour).
- Centre/right: **4 links**, in this order: `Experience · Skills · Background · Contact`.
  - `Background` points to a new wrapper section (see below).
- Far right: the **dark mode toggle** (see Phase 5), then one filled **Download CV** button (bordeaux background, oat text) linking to `Madara-Kalpina-CV.pdf` with the `download` attribute (see Phase 4).
- Remove the `About` link. It's the first section under the hero, so it doesn't need one.

**Merge Languages and Education into "Background"**
- Wrap `#languages` and `#education` in `<section id="background">`. Either keep both inner sections as they are, with their `id`s changed to `h3`-level blocks, or set them side by side on desktop (Languages in the left third, Education in the right two thirds). Either is fine; side by side is more compact.

**Readability**
- Link font size: at least `0.9rem`. Opacity: `0.8` normally, `1` on hover or when active.
- Active state: an oat underline 2px thick, 6px below the text, plus `aria-current="true"` on the link.
- Add visible `:focus-visible` outlines on all links and buttons (2px oat outline, 3px offset).

**Scroll-spy for the nav** (replaces the old experience scroll-spy)
```js
const links = document.querySelectorAll('nav a[href^="#"]');
const sections = [...links].map(a => document.querySelector(a.getAttribute('href'))).filter(Boolean);
const spy = new IntersectionObserver(entries => {
  entries.forEach(e => {
    if (!e.isIntersecting) return;
    links.forEach(a => {
      const on = a.getAttribute('href') === '#' + e.target.id;
      a.classList.toggle('active', on);
      on ? a.setAttribute('aria-current', 'true') : a.removeAttribute('aria-current');
    });
  });
}, { rootMargin: '-40% 0px -55% 0px' });
sections.forEach(s => spy.observe(s));
```

**Anchor offset** — add `section, footer { scroll-margin-top: var(--nav-h); }` so the headings don't land under the sticky nav.

**Mobile (≤ 640px)**
- Show the name on the left and a `Menu` button on the right (`aria-expanded`, `aria-controls`).
- The button opens a full-width dropdown under the nav that lists the 4 links, the dark mode toggle and **Download CV**. Clicking a link closes it. `Esc` closes it too.
- There should be no horizontal scrolling anywhere in the nav.

**Hero** — reduce `min-height` from `100vh` to `calc(100vh - var(--nav-h))` so the hero plus nav fill exactly one screen.

---

## Phase 3: Rebuild the experience section (stacked roles with a sticky label)

**Delete:** `.exp-scroll-stage`, `.exp-sticky-display`, `.exp-sidebar`, `.exp-role-list`, `.exp-role-item`, `.exp-content-panels`, `.exp-content-panel` (including `.active` and `.leaving`), all `.exp-sentinel` elements, and the scroll-spy and click JS for them (the `setExpActive` function and the two `querySelectorAll` blocks after it). Also delete the matching rules in the 768px media query.

**New markup** — one list item per company. Group the two SEB roles under one company to show the promotion.

```html
<section id="experience" aria-labelledby="experience-heading">
  <h2 id="experience-heading">Work Experience</h2>
  <ol class="exp-list">

    <li class="exp-role">
      <div class="exp-label">
        <span class="role-company">Pixelfield</span>
        <span class="role-meta">Prague · 2025 – current</span>
        <p class="role-metric"><strong>14</strong> projects in parallel</p>
      </div>
      <div class="exp-body">
        <article>
          <h3>Product Manager</h3>
          <ul>…bullets…</ul>
        </article>
      </div>
    </li>

    <li class="exp-role">
      <div class="exp-label">
        <span class="role-company">Green0meter</span>
        <span class="role-meta">Prague · 2024 – 2025</span>
        <span class="role-tag">B2B SaaS</span>
        <p class="role-metric"><strong>−50%</strong> support requests</p>
      </div>
      <div class="exp-body">
        <article><h3>Product Manager</h3><ul>…</ul></article>
      </div>
    </li>

    <li class="exp-role">
      <div class="exp-label">
        <span class="role-company">TechChill</span>
        <span class="role-meta">Riga · 2022 – 2023 · alongside SEB</span>
        <p class="role-metric"><strong>130+</strong> speakers curated</p>
      </div>
      <div class="exp-body">
        <article><h3>Speakers &amp; Agenda Manager</h3><ul>…</ul></article>
      </div>
    </li>

    <li class="exp-role">
      <div class="exp-label">
        <span class="role-company">SEB Bank</span>
        <span class="role-meta">Riga · 2019 – 2022</span>
        <p class="role-metric"><strong>3</strong> Baltic countries, one CMS</p>
      </div>
      <div class="exp-body">
        <article>
          <h3>Agile Product Owner</h3>
          <p class="exp-dates">2020 – 2022</p>
          <ul>…</ul>
        </article>
        <article>
          <h3>Web Editor</h3>
          <p class="exp-dates">2019 – 2020</p>
          <ul>…</ul>
        </article>
      </div>
    </li>

  </ol>
</section>
```

**CSS**
```css
#experience { padding: var(--pad-v) var(--pad-h); background: var(--paper); }
#experience > h2 { /* reuse the existing section h2 style, bordeaux */ margin-bottom: 2.5rem; }

.exp-list { list-style: none; }
.exp-role {
  display: grid;
  grid-template-columns: 1fr 2fr;
  column-gap: 4rem;
  padding: 2.5rem 0;
  border-top: 1px solid rgba(96,29,45,.15);
}
.exp-label {
  position: sticky;
  top: calc(var(--nav-h) + 1.5rem);
  align-self: start;            /* required for sticky inside a grid */
}
.role-company { /* Barlow Condensed 800, clamp(2rem, 3.5vw, 3rem), bordeaux, uppercase, line-height 1 — FIXED size, no transitions */ }
.role-meta    { /* Barlow Condensed 600, 0.85rem, uppercase, letter-spacing .16em, citrus */ display:block; margin-top:.35rem; }
.role-tag     { /* small pill: 0.7rem uppercase, denim border + text, padding .15rem .5rem */ display:inline-block; margin-top:.6rem; }
.role-metric  { margin-top: 1.25rem; font-size: .9rem; color: rgba(18,8,10,.6); }
.role-metric strong { display:block; font-family:'Barlow Condensed'; font-weight:800; font-size:clamp(2.2rem,4vw,3.2rem); line-height:1; color: var(--bordeaux); }

.exp-body article + article { margin-top: 2rem; padding-top: 2rem; border-top: 1px dashed rgba(96,29,45,.15); }
.exp-body h3 { /* keep the existing .exp-content-panel h3 style */ }
.exp-dates  { /* keep the existing .exp-context style */ }
.exp-body ul { list-style: disc; padding-left: 1.05rem; display:flex; flex-direction:column; gap:.55rem; }
.exp-body li { font-size: .95rem; line-height: 1.72; }

@media (max-width: 768px) {
  .exp-role { grid-template-columns: 1fr; row-gap: 1.25rem; }
  .exp-label { position: static; }
}
```

**Optional polish** (only after everything else works): as a role scrolls through the middle of the viewport, fade the other roles' `.role-company` to 35% opacity with an IntersectionObserver. Change **only opacity or colour, never size**, so nothing jumps. Respect `prefers-reduced-motion`.

---

## Phase 4: Download CV and print

1. Add the current PDF CV to the repo root as `Madara-Kalpina-CV.pdf`.
2. The nav's **Download CV** button links to it: `<a class="nav-cta" href="Madara-Kalpina-CV.pdf" download>Download CV</a>`.
3. Remove `#print-btn` and its `onclick`.
4. Add a print stylesheet as a fallback for people who press Ctrl/Cmd+P:
```css
@media print {
  nav, #ribbon { display: none !important; }
  body { padding-bottom: 0; background: #fff; }
  #hero { min-height: auto; padding: 1.5rem 0; }
  .exp-label { position: static; }
  section, .exp-role { break-inside: avoid; }
  * { box-shadow: none !important; }
}
```

---

## Phase 5: Implement dark mode

Today the toggle only sets `data-theme`, and no CSS reacts to it. The fix has three steps: move colours into tokens, define dark values for the tokens, then wire up the toggle properly.

**Step 1: Use tokens instead of hard-coded colours.** Add semantic tokens and replace every hard-coded colour in the CSS with them. That includes every `rgba(18,8,10,…)`, `rgba(96,29,45,…)` and `rgba(237,213,171,…)`, plus the direct uses of `var(--paper)`, `var(--ink)` and `var(--bordeaux)` as text or background colours. Keep the brand colours (`--bordeaux`, `--oat`, `--denim`, `--citrus`, `--ink`, `--paper`) as raw values that only the tokens refer to.

```css
:root {
  /* page */
  --bg:          var(--paper);
  --text:        var(--ink);
  --text-muted:  rgba(18,8,10,.6);
  --text-faint:  rgba(18,8,10,.42);
  --line:        rgba(96,29,45,.15);
  --accent:      var(--bordeaux);   /* section headings, company names, metrics */
  --accent-2:    var(--citrus);     /* dates, role meta */
  /* coloured sections */
  --hero-bg:     var(--bordeaux);
  --about-bg:    var(--citrus);
  --about-text:  var(--paper);
  --skills-bg:   var(--ink);
  --lang-bg:     var(--oat);
  --lang-text:   var(--ink);
  --footer-bg:   var(--bordeaux);
  --nav-bg:      var(--ink);
  color-scheme: light;
}
```

**Step 2: Define the dark values.** Use one shared set of values in two places: when the visitor has chosen dark mode, and when their system is dark and they haven't chosen anything.

```css
:root[data-theme="dark"] {
  --bg:         #140C0E;
  --text:       #EFE6D8;
  --text-muted: rgba(239,230,216,.68);
  --text-faint: rgba(239,230,216,.5);
  --line:       rgba(237,213,171,.14);
  --accent:     #E3A6B2;   /* lightened bordeaux: plain bordeaux fails contrast on dark */
  --accent-2:   #C9C95E;   /* lightened citrus */
  --hero-bg:    var(--bordeaux);
  --about-bg:   #3A3A14;
  --about-text: var(--oat);
  --skills-bg:  #0C0708;
  --lang-bg:    #241815;
  --lang-text:  var(--oat);
  --footer-bg:  var(--bordeaux);
  --nav-bg:     #0C0708;
  color-scheme: dark;
}
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) { /* same declarations as the block above */ }
}
```
The hex values are a starting point. **Check every text and background pair for a WCAG AA contrast ratio of at least 4.5:1** (3:1 for text 24px and larger), and adjust any that fail.

**Step 3: Wire up the toggle.**
- Put this small inline script at the **top of `<head>`, before the `<style>` block**, so the page doesn't flash the wrong theme on load:
```html
<script>
  try { const t = localStorage.getItem('theme'); if (t) document.documentElement.dataset.theme = t; } catch (e) {}
</script>
```
- The toggle is a `<button id="theme-toggle" aria-pressed="false">`. Put it in the nav next to **Download CV**. On mobile, put it inside the Menu dropdown.
- Its label shows the mode you would switch *to*: "Dark mode" or "Light mode". Set `aria-pressed="true"` when dark mode is on.
- On click, read the current effective theme (the `data-theme` attribute if set, otherwise `matchMedia('(prefers-color-scheme: dark)').matches`), switch to the other one, set `data-theme`, and save it in `localStorage` inside `try/catch`.
- On load, set the label and `aria-pressed` from the effective theme, including when the theme comes from the system setting.
- Add `transition: background-color .25s, color .25s` on `body` and the sections. Turn it off under `prefers-reduced-motion`.

**Print always uses the light theme.** In the `@media print` block, reset the dark tokens back to the light values.

---

## Phase 6: Small fixes

- **Ribbon**
  - Add `@media (prefers-reduced-motion: reduce) { .ribbon-track { animation: none; } }`.
  - Hide it while the footer is in view (observe `#contact` the same way the hero is observed) so it doesn't cover the contact details.
- **Skills grid** — the `nth-child` border rules are fragile. Replace them with `gap: 1px; background: rgba(237,213,171,.1)` on the grid and `background: var(--ink)` on each `.skill-group`. The lines then come from the gaps and work for any number of items or columns.
- **Languages hover** — hovering to see "Fluent" doesn't work on touch screens. Show the dots **and** the label (for example, "●●●● Fluent").
- **Phone number** — if you have a Czech number, use it in the hero and footer (`tel:` links too). Otherwise leave it.

---

## Phase 7: Checks before pushing

Test at **1440×900, 1280×720, 768×1024 and 390×844**:

- [ ] Every one of the 5 roles (Pixelfield, Green0meter, TechChill, SEB Product Owner, SEB Web Editor) is fully readable just by scrolling. No bullet is cut off.
- [ ] The company label stays in place while its bullets scroll past, then gives way to the next company (desktop only).
- [ ] The nav is visible as soon as the page loads, and the active link updates as you scroll through every section.
- [ ] Clicking each nav link lands with the section heading visible, not under the nav.
- [ ] Mobile: the Menu button opens and closes with a click and with `Esc`. There is no horizontal scroll at 360px wide.
- [ ] **Download CV** downloads the PDF.
- [ ] Print preview (Cmd/Ctrl+P) shows all 5 roles and no nav or ribbon, **in light colours even when dark mode is on**.
- [ ] Dark mode: the toggle switches every section, and the choice survives a page reload. With no saved choice, the page follows the system setting. There is no light-coloured flash on load in dark mode.
- [ ] Dark mode: no hard-coded light colours are left over. Search the CSS for `rgba(18,8,10` and `rgba(96,29,45`; they should only appear inside the token definitions. All text passes a contrast ratio of at least 4.5:1.
- [ ] You can reach every link and button with Tab, and a focus outline is visible.
- [ ] The site text matches the PDF CV word for word (Phase 1).
- [ ] Lighthouse accessibility score is 95 or higher.
- [ ] No console errors.

A quick automated check for the first item (Playwright): scroll the page in 40px steps and confirm that each `.exp-body article` crosses the viewport at some point with `getBoundingClientRect().bottom` inside the viewport.
