# Web Conventions

## 0. Scope and General Principles

These conventions govern all first-party web pages in the codebase (sites, documentation pages, dashboards and reports): their HTML and CSS source, and the content they present. Scripts follow the [TypeScript Coding Conventions](typescript-coding-conventions.md); where a rule here has a counterpart there, the two read the same.

- **Third-party code is out of scope.** Vendored libraries, generated bundles and framework scaffolding must not be reformatted, so they can be regenerated or re-synced with upstream.
- **These rules win over local habit.** Where an older page's own style disagrees, follow this document (see [Section 8](#8-changing-code-that-predates-these-rules)).

## 1. Comments

A comment states what a block is for in domain terms, never language mechanics.

- **Domain semantics only.** `<!-- pricing table -->` or `/* footer */` labels a region; do not explain the cascade, a browser quirk you worked around, or why you chose a selector.
- **No multi-line prose.** A one-line label is fine; a paragraph inside a stylesheet or a template is not.
- **No file header comment.** A file begins directly with code (`<!doctype html>` or the first rule): no license banner, file description or author block.

## 2. HTML

Markup states what the content is; styling and behaviour live elsewhere.

- **Semantic elements first.** Use `header`, `nav`, `main`, `article`, `section`, `footer`, `button` and `a` for what they mean; a `div` with a click handler is not a button.
- **One `h1` per page,** and headings descend without skipping levels.
- **Every page declares `lang`, `charset` and `viewport`.**
- **Every meaningful image has `alt` text;** a decorative image has `alt=""`.
- **No inline `style` attributes and no inline event handlers** (`onclick` and the like). Styles go in the stylesheet, behaviour in the script.

## 3. CSS

Every value with meaning is a named token, defined once.

- **Tokens on `:root`.** Colours, font families, font sizes, spacing and breakpoints are custom properties on `:root`; rules reference the tokens, never raw values. This is the CSS form of "no magic numbers".
- **Class selectors only.** No ID selectors and no element selectors for component styling; element selectors are for the base reset.
- **Flat selectors.** At most two levels of nesting or descendant combinators; a deeper selector means the markup needs a class.
- **No `!important`.** Fix the specificity instead.
- **Relative units.** Font sizes and spacing in `rem`; `px` only for hairlines such as a `1px` border.
- **Layout in proportions.** The widths of columns, sidebars, panels and the text column are shares of the screen or of their container (`%`, `fr`, `vw`), so a layout keeps its proportions from a laptop to a wide monitor. A fixed length may only bound a proportion, such as `max(12rem, 16%)`, so content still fits on a narrow screen; a width is never a fixed `px` or `rem` length alone.
- **Mobile first.** Base rules target the narrowest screen; `min-width` media queries add to them. No page scrolls horizontally on a phone.

## 4. Multi-line Formatting

Any element with several attributes, rule with several declarations, or selector list is fully expanded, one item per line.

**Elements with more than two attributes:** one attribute per line.

```html
<a
    class="nav-link"
    href="/papers/"
    aria-current="page"
>Papers</a>
```

**Rules:** one declaration per line, one selector per line.

```css
.card-title,
.section-title {
    font-family: var(--font-title);
    font-size: var(--size-title);
    margin-block-end: var(--space-2);
}
```

## 5. Naming

A name tells the reader what the element is, not what it looks like.

| Item | Rule | Example |
|---|---|---|
| Classes | `kebab-case`, named by role | `nav-link`, `paper-card` |
| Custom properties | `--<category>-<role>` | `--color-text`, `--font-title`, `--space-2` |
| Files | `kebab-case` | `paper-list.html`, `site.css` |

- **Name by role, not appearance.** `paper-card`, not `white-box`; `section-title`, not `big-serif`.
- **Descriptive full words, not terse abbreviations.** `navigation-menu` or `nav-menu`, not `nm`.

## 6. Structure and Placement

Every style has one canonical home.

- **One home per component.** A component's rules live together in one place; do not scatter overrides for it across page stylesheets.
- **Reuse tokens and components; do not reinvent.** Before adding a rule, check whether an existing class or token already does the job.
- **No ad-hoc utility classes.** Do not invent one-off helpers such as `.mt-13` or `.red-text` for a single element; give the element a role class and style that.
- **No logic in markup.** Templates render data; they do not compute it.

## 7. Content and Presentation

A page reads like a printed page: serif type, black and white, no decoration that imitates a terminal or a dashboard.

- **An explicit user request wins.** The typography and colour rules below are the default style. When the user explicitly asks for other fonts or colours, follow the request for what it covers and keep the defaults for everything else.

### 7.1 Typography

- **Titles are set in Instrument Serif.** The site title, page titles and every heading (`h1` to `h6`) use `Instrument Serif`, site-wide, with no exceptions per page.
- **Body text is set in Crimson Text.** Paragraphs, lists, captions, navigation, buttons and form controls use `Crimson Text`.
- **No mechanical fonts.** No monospace, geometric sans or "technical" display faces (such as JetBrains Mono, Space Grotesk or Space Mono) for headings, labels, numbers or body text. Monospace is allowed only inside `code` and `pre` for literal source code.
- **Load the two fonts once** from Google Fonts and expose them as tokens, with a serif fallback.

```css
:root {
    --font-title: "Instrument Serif", Georgia, serif;
    --font-body: "Crimson Text", Georgia, serif;
}
```

### 7.2 No All-caps Side Marks

- **No small uppercase labels.** Do not put an all-caps, small, letter-spaced label above or beside content as a marker (an "eyebrow" such as `FEATURES`, `NEW`, `SECTION 02`).
- **No uppercase styling to fake one.** No `text-transform: uppercase`, `font-variant: small-caps` or wide `letter-spacing` on small text.
- **Say it in the heading or drop it.** If the label carries meaning, fold it into the heading in sentence case; otherwise delete it.

### 7.3 Colour

- **White background, black text.** The light scheme is black text on a white background.
- **Black background, white text in dark mode.** The dark scheme inverts it, driven by `prefers-color-scheme`.
- **Neutral greys are allowed.** Greys between black and white may mark secondary text, hairline rules and chart fills; each is a token, and dark mode redefines it.
- **Code sits on a grey background.** Every `code` element, inline or inside a `pre` block, has a light grey background with black text; in dark mode it has a dark grey background with off-white text. Both are tokens, so no code block falls back to the page background.
- **No other colours.** No hue anywhere: no accent colours, coloured backgrounds, gradients or tinted cards. Links are distinguished by an underline, not by colour.

```css
:root {
    --color-background: #ffffff;
    --color-text: #000000;
    --color-muted: #666666;
    --color-rule: #dddddd;
    --color-code-background: #f2f2f2;
    --color-code-text: #000000;
}

@media (prefers-color-scheme: dark) {
    :root {
        --color-background: #000000;
        --color-text: #ffffff;
        --color-muted: #999999;
        --color-rule: #333333;
        --color-code-background: #262626;
        --color-code-text: #e6e6e6;
    }
}

code,
pre {
    background: var(--color-code-background);
    color: var(--color-code-text);
}
```

### 7.4 Every Screen Size

A page is designed for every width it can be shown at, not for the one it was written on.

- **Text scales with the screen.** Font sizes, line height and letter spacing are tokens that change at breakpoints or through `clamp()`; body text never drops below a comfortable reading size on a phone.
- **Spacing between widgets scales too.** Gaps, padding and margins between cards, sections and controls come from spacing tokens redefined per breakpoint, so a layout neither crowds on a phone nor floats apart on a wide monitor.
- **Graphics keep their shape.** A chart or diagram drawn in script takes its width from its container and its height as a share of that width, with a floor so its labels still fit; it never fixes its overall size in pixels. Details tied to the label text, such as tick gaps and legend swatches, stay at text size.
- **Layouts reflow, they do not shrink.** Multi-column layouts collapse to fewer columns on narrow screens; nothing is scaled down as a whole, clipped, or pushed off the side.
- **Animations are tuned per size.** Distances, durations and parallax depth that suit a desktop are reduced on small screens; no animation causes horizontal overflow or a layout jump. Every animation is disabled or reduced under `prefers-reduced-motion`.
- **Touch targets are at least `44px`** on touch screens, with enough space between them that a thumb does not hit the neighbour.
- **Verify at four widths.** Check every page at a phone (`375px`), a tablet (`768px`), a laptop (`1280px`) and a wide monitor (`1920px`), in both colour schemes.

```css
:root {
    --size-body: clamp(1.0625rem, 0.95rem + 0.5vw, 1.25rem);
    --width-sidebar: max(12rem, 16%);
    --space-section: 2.5rem;
    --motion-distance: 0.5rem;
}

@media (min-width: 768px) {
    :root {
        --space-section: 4rem;
        --motion-distance: 1.5rem;
    }
}

@media (prefers-reduced-motion: reduce) {
    :root {
        --motion-distance: 0;
    }
}
```

### 7.5 Writing

Page text follows the same rules as the [LaTeX Writing Conventions](latex-conventions.md#7-headings) for headings, tone and framing.

- **Headings name the topic in plain words.** No cryptic or clever headings such as "Two caches, two shared parts"; write the subject, e.g. "Sharing the L2 and the last-level cache across cores".
- **Support claims with evidence.** No preaching ("clearly", "it is crucial to understand", "you must"); support every claim with a number, a citation, a demo or a screenshot, and let the reader draw the conclusion.
- **No "not X, but Y".** Constructions such as "it is not a tool, it is a platform" or "rather than A, it does B" are replaced by a direct statement of what the thing is.

### 7.6 Numbered, Linkable Headings

Every section heading carries an index number and can be reached with a `#` link.

- **Number every section heading.** `h2` headings are numbered `1`, `2`, `3`; `h3` headings under them `1.1`, `1.2`; and so on down the levels in use. The page title (`h1`) is not numbered.
- **Generate the numbers.** Numbers come from CSS counters or the build step, never typed into the heading text, so inserting a section renumbers the rest.
- **Every heading has an `id`.** The `id` is a stable `kebab-case` slug of the heading text (`#cache-sharing`), not of its number, so links survive renumbering.
- **Expose the link.** Each heading carries a `#` anchor that links to its own `id`, shown on hover and focus, so a reader can copy a deep link to any section.
- **Land below sticky bars.** Headings set `scroll-margin-top` to the height of any fixed header, so a `#` link does not hide the heading under it.

```html
<h2 id="cache-sharing">
    Cache sharing across cores
    <a
        class="heading-anchor"
        href="#cache-sharing"
        aria-label="Link to this section"
    >#</a>
</h2>
```

```css
main {
    counter-reset: section;
}

main h2 {
    counter-increment: section;
    counter-reset: subsection;
    scroll-margin-top: var(--space-header);
}

main h2::before {
    content: counter(section) ". ";
}

main h3 {
    counter-increment: subsection;
    scroll-margin-top: var(--space-header);
}

main h3::before {
    content: counter(section) "." counter(subsection) " ";
}
```

### 7.7 Figure Titles

Every figure has a title, and the title is a short phrase that sums it up.

- **Every figure has a title.** Charts, diagrams, screenshots and tables sit in a `figure` with a `figcaption`; no figure appears without one.
- **The title is brief.** It names what is shown in a few words, the way the [LaTeX caption rule](latex-conventions.md#6-captions) does.
- **No walkthrough in the title.** No explanation of panels, bars, lines, colours or markers, and no definitions of the quantities shown; those go in the surrounding text, axis labels or column headers.

```html
<!-- Bad -->
<figcaption>Cost per clip. The left panel compares layouts on a log scale; the dark bar is our system and starred rows are estimated.</figcaption>

<!-- Good -->
<figcaption>Cost per clip</figcaption>
```

### 7.8 No Middle-dot Separators

Items shown side by side are told apart by layout, not by a dot between them.

- **No `·` anywhere.** Do not join items with a middle dot, as in `Ada Lovelace · 8 October 2026` or `Docs · Blog · About`; this holds for bylines, metadata, navigation, footers and the browser tab title alike.
- **No stand-in separators either.** No `•`, `|`, `/` or `—` placed between items to do the same job.
- **Lay out the items instead.** Give each item its own element and separate them with spacing, a hairline rule or a labelled field; in a title, use the words of a sentence or drop the extra part.

```html
<!-- Bad -->
<p class="byline">Ada Lovelace · 8 October 2026 · 5 min read</p>

<!-- Good -->
<dl class="byline">
    <div class="byline-field">
        <dt>Author</dt>
        <dd>Ada Lovelace</dd>
    </div>
    <div class="byline-field">
        <dt>Updated</dt>
        <dd><time datetime="2026-10-08">8 October 2026</time></dd>
    </div>
</dl>
```

## 8. Changing Code That Predates These Rules

When you touch a page or a component, bring it into line.

- **The house rules win; local consistency does not excuse a violation.** A stylesheet with seventeen hard-coded colours has seventeen violations, not a palette of its own.
- **Fix neighbours that share the same violation.** Leaving one fixed component beside unfixed ones on the same page leaves the page with two styles instead of one.
- **A class rename crosses into scripts and templates.** Search for the old name in `querySelector` strings, `classList` calls and templates; none of these fail to build, they fail late or quietly stop matching.
- **Check the page, not just the source.** Load it in both colour schemes and at the four widths of [Section 7.4](#74-every-screen-size) before calling a change done.
