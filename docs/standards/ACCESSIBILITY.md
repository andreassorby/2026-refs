# Accessibility standard (WCAG 2.2 AA)

> Part of **Andreas-standardoppsett** (topic: `accessibility`). Installed into projects as `docs/standards/ACCESSIBILITY.md`.
> Short rule for agents: `.cursor/rules/accessibility.mdc`. Alt text guide: `docs/standards/ALT-TEXT.md`.
> Legal facts checked: 22 Oct 2026. This is not legal advice.

## 1. Target

All UI we build targets **WCAG 2.2 level AA**. That is a superset of what Norwegian law asks for today (WCAG 2.0 / 2.1, see section 2), and it matches where EU law is going. Aiming higher costs almost nothing when it's done from the start.

## 2. Legal context (short and accurate)

| Rule | What it says | Who it applies to |
|---|---|---|
| **WCAG 2.2** (W3C Recommendation, 5 Oct 2023) | The technical standard. Adds 9 criteria to 2.1 (6 of them at A/AA, listed below) and removes 4.1.1 Parsing. | Not law by itself. Laws point to it (or to 2.0/2.1). |
| **Norway: likestillings- og diskrimineringsloven § 18** + **forskrift om universell utforming av IKT-løsninger** (FOR-2013-06-21-732) | ICT solutions aimed at the public must be universally designed. | Private **and** public organisations, incl. associations ("lag og organisasjoner"). |
| — private sector (forskriften § 4) | Minimum **WCAG 2.0 A + AA, except 1.2.3, 1.2.4, 1.2.5** = 35 success criteria. | Private businesses. |
| — public sector (since 1 Feb 2023, from EU Web Accessibility Directive 2016/2102) | **EN 301 549 v3.2.1 → WCAG 2.1**: 48 of 78 criteria, plus a published **tilgjengelighetserklæring** (accessibility statement) for every website and app. | Public bodies (state, municipalities, bodies covered by the public procurement rules), and suppliers delivering citizen-facing solutions for them. |
| Supervision (loven § 36, forskriften §§ 5–7) | **Tilsynet for universell utforming av IKT (Uu-tilsynet)**, part of **Digitaliseringsdirektoratet (Digdir)**. Can order fixes (pålegg om retting) and impose daily coercive fines (tvangsmulkt / dagbot). | Everyone covered above. |
| **EU: European Accessibility Act (EAA)**, Directive (EU) 2019/882 | Accessibility requirements for listed products and services (e.g. e-commerce, banking, e-books, electronic communications, passenger transport ticketing/info). **Applies in the EU from 28 June 2025.** Microenterprises (< 10 staff and ≤ €2 M turnover/balance) providing services are exempt. EN 301 549 (WCAG 2.1 AA for web) is the practical reference. | Businesses placing those products/services on the EU market. |
| — EAA in Norway | **Not yet incorporated into the EEA Agreement or Norwegian law** (consultation 2021; no date set). Norwegian businesses that sell covered products/services into EU countries must still follow the EAA there. | |

What this means for a small Norwegian private web app (e.g. chordinger): the legal floor is the 35 WCAG 2.0 criteria; if it sells to consumers in the EU and falls under an EAA service category, the EAA applies in those countries. We build to WCAG 2.2 AA anyway. A tilgjengelighetserklæring is only *required* for public bodies, but publishing a short one is good practice.

Sources:
- WCAG 2.2: https://www.w3.org/TR/WCAG22/ and https://www.w3.org/WAI/standards-guidelines/wcag/new-in-22/
- Forskriften: https://lovdata.no/dokument/SF/forskrift/2013-06-21-732
- Uu-tilsynet, what the regulation says: https://www.uutilsynet.no/regelverk/kva-seier-forskrifta/153
- Uu-tilsynet, Web Accessibility Directive / public sector: https://www.uutilsynet.no/webdirektivet-wad/eus-webdirektiv-wad/265
- Uu-tilsynet, EAA status: https://www.uutilsynet.no/tilgjengelighetsdirektivet-eaa/eus-tilgjengelegheitsdirektiv-eaa/268
- EAA text: https://eur-lex.europa.eu/eli/dir/2019/882/oj

## 3. New in WCAG 2.2 (A/AA) – easy to forget

| SC | Level | In practice |
|---|---|---|
| 2.4.11 Focus Not Obscured (Minimum) | AA | Sticky headers, bottom bars, chat widgets and cookie banners must not fully cover the focused element. Use `scroll-padding-top` / `scroll-margin`. |
| 2.5.7 Dragging Movements | AA | Every drag (reorder setlist, sliders, drag-to-upload) has a click/tap alternative (Move up/down buttons, input field, file picker). |
| 2.5.8 Target Size (Minimum) | AA | Pointer targets ≥ 24×24 CSS px, or spaced so a 24 px circle around each doesn't overlap another. (Our mobile rule is stricter: 44 px.) |
| 3.2.6 Consistent Help | A | Help/contact/FAQ links appear in the same relative place on every page. |
| 3.3.7 Redundant Entry | A | Don't ask for the same info twice in one process; prefill or offer "same as above". |
| 3.3.8 Accessible Authentication (Minimum) | AA | No memory/puzzle tests to log in. Allow paste + password managers; magic link, passkeys, OAuth are fine. |

## 4. Full checklist

### Structure and semantics
- [ ] Native elements first: `<button>`, `<a href>`, `<input>`, `<select>`, `<dialog>`, `<details>`, `<table>`.
- [ ] No clickable `<div>`/`<span>`. If unavoidable: `role`, `tabIndex={0}`, Enter/Space handlers (but fix it instead).
- [ ] Landmarks: `<header>`, `<nav>` (labelled if several), `<main>` (one), `<footer>`.
- [ ] One `<h1>` per view; headings don't skip levels; headings aren't used just for big text.
- [ ] Lists are `<ul>/<ol>`; data grids are `<table>` with `<th scope>` and `<caption>`.
- [ ] Unique, descriptive page `<title>` (Next.js: `metadata.title` / `generateMetadata`).
- [ ] "Skip to main content" link as the first focusable element.
- [ ] `<html lang>` matches the UI language and changes with it (i18next: update `document.documentElement.lang` on `languageChanged`). Use `lang="nb"` / `lang="nn"` / `lang="en"`. Song lyrics in another language: wrap in `lang="…"`.

### ARIA
- [ ] First rule of ARIA: don't use ARIA if HTML does it.
- [ ] Custom widgets (tabs, menu, combobox, listbox) follow the WAI-ARIA Authoring Practices pattern incl. keyboard: https://www.w3.org/WAI/ARIA/apg/
- [ ] `aria-expanded`, `aria-pressed`, `aria-selected`, `aria-current="page"` reflect state.
- [ ] Never `aria-hidden="true"` on something focusable or containing focusable children.
- [ ] `aria-label` only on interactive elements or landmarks, not on plain `<div>`/`<span>`.

### Keyboard and focus
- [ ] Everything reachable and operable with keyboard alone; order follows visual order.
- [ ] No positive `tabIndex`.
- [ ] Visible focus on every interactive element. Tailwind: replace `outline-none` with `focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[color]` (≥ 3:1 against background).
- [ ] Focus not hidden under sticky UI (`scroll-padding-top` equal to header height).
- [ ] Single-key shortcuts (e.g. in a reader/presenter) can be turned off or only work when the component has focus (2.1.4).
- [ ] After route change in an SPA, focus moves to the new `<h1>` or `<main>` (or the change is announced).

### Pointer, touch, hover, motion
- [ ] No hover-only reveal of controls; hover reveals must also happen on `:focus-within` and be reachable by touch.
- [ ] Tooltips/popovers: dismissable with Esc, pointer can move onto them, stay until dismissed (1.4.13).
- [ ] Targets ≥ 24×24 px (project rule may require 44 px).
- [ ] Drag-and-drop has button/keyboard alternative. (`@dnd-kit` has `KeyboardSensor` + screen-reader announcements; enable them *and* add Move up/down buttons.)
- [ ] Multi-point/path gestures (pinch, swipe) have single-tap alternatives (2.5.1).
- [ ] Actions trigger on pointer up / click, not pointer down (2.5.2).
- [ ] `prefers-reduced-motion` respected: Tailwind `motion-safe:animate-…`, CSS `@media (prefers-reduced-motion: reduce)`, framer-motion `useReducedMotion()` / `<MotionConfig reducedMotion="user">`.
- [ ] No flashing > 3/s; auto-scrolling/auto-advancing content (> 5 s) has pause/stop.

### Colour and visual
- [ ] Text contrast ≥ 4.5:1; large text (≥ 24 px, or ≥ 18.66 px bold) ≥ 3:1.
- [ ] UI component boundaries, focus indicators, icons, chart lines ≥ 3:1 against adjacent colours.
- [ ] Check light **and** dark theme; check disabled-looking-but-enabled states.
- [ ] Colour is never the only way to convey info (add text/icon/underline for links in body text).
- [ ] Reflow at 320 CSS px wide, 200 % zoom, and text-spacing override (1.4.12) without loss.
- [ ] Viewport meta never disables zoom.

### Images and media
- [ ] Follow `ALT-TEXT.md` (informative / decorative / functional / complex).
- [ ] Video with speech has captions; audio-only has a transcript; no autoplay with sound.

### Forms
- [ ] Every field has a visible `<label htmlFor>` (or wraps the input). Placeholder is not a label.
- [ ] Groups of radios/checkboxes in `<fieldset>` + `<legend>`.
- [ ] Required fields marked in text ("required" / "påkrevd"), plus `required` / `aria-required`.
- [ ] `autoComplete` on name, email, tel, address, `current-password`, `new-password`, `one-time-code`.
- [ ] Errors: text message next to field, `aria-invalid="true"`, `aria-describedby` → message id; on submit move focus to first invalid field or to an error summary with links.
- [ ] Error messages say how to fix ("Use format 12.05.2026"), not just "Invalid".
- [ ] Status messages announced without moving focus: `role="status"` (polite) for "Saved", "3 songs found"; `role="alert"` for urgent errors. The live region must exist in the DOM *before* the text is inserted.
- [ ] Destructive or legal/financial actions can be undone, confirmed, or reviewed (3.3.4).
- [ ] No redundant entry; login allows paste and password managers.

### Dialogs, menus, overlays
- [ ] Native `<dialog>` + `showModal()` preferred (gives focus trap, Esc, inert background).
- [ ] Otherwise: `role="dialog"`, `aria-modal="true"`, `aria-labelledby` → visible title; focus first sensible element on open; trap Tab; Esc closes; restore focus to the trigger on close; background `inert`.
- [ ] Full-screen views (reader/presenter) still need a keyboard-reachable, labelled exit button.
- [ ] Toasts don't steal focus, stay long enough (or until dismissed), and are announced.

### Consistency
- [ ] Navigation and help links in the same place on every page (3.2.3, 3.2.6).
- [ ] Same function = same name/icon everywhere (3.2.4).

## 5. Testing recipe

### 5.1 Automated (catches ~30–40 % of issues)

**Lint (JSX).** `eslint-config-next` registers `eslint-plugin-jsx-a11y` but enables only a few of its rules. Turn on the recommended set:

```js
// eslint.config.mjs (Next.js flat config)
import jsxA11y from "eslint-plugin-jsx-a11y";
// ...existing imports (nextVitals, nextTs)
const eslintConfig = defineConfig([
  ...nextVitals,
  ...nextTs,
  // plugin is already registered by eslint-config-next; only add the rules
  { files: ["**/*.{jsx,tsx}"], rules: jsxA11y.flatConfigs.recommended.rules },
  // ...
]);
```
```bash
npm i -D eslint-plugin-jsx-a11y   # in the app workspace, e.g. -w apps/web
```
Non-Next projects: add `jsxA11y.flatConfigs.recommended` to the config array directly.

**Unit/component tests (Vitest + jsdom).**
```ts
import { render } from "@testing-library/react";
import axe from "axe-core";

it("has no axe violations", async () => {
  const { container } = render(<SongForm />);
  const results = await axe.run(container, { rules: { "color-contrast": { enabled: false } } }); // jsdom can't compute colours
  expect(results.violations).toEqual([]);
});
```

**End-to-end (Playwright + axe) – the main gate.**
```bash
npm i -D @playwright/test @axe-core/playwright && npx playwright install
```
```ts
// e2e/a11y.spec.ts
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

const pages = ["/", "/library", "/setlists"]; // add your key routes

for (const path of pages) {
  test(`a11y: ${path}`, async ({ page }) => {
    await page.goto(path);
    const results = await new AxeBuilder({ page })
      .withTags(["wcag2a", "wcag2aa", "wcag21a", "wcag21aa", "wcag22aa"])
      .analyze();
    expect(results.violations).toEqual([]);
  });
}
```
Also run with `page.emulateMedia({ colorScheme: "dark" })` if the app has a dark theme, and open important modals before analysing.

**Quick manual tools:** browser DevTools Lighthouse (Accessibility), axe DevTools extension, Chrome "Rendering → Emulate prefers-reduced-motion", WebAIM contrast checker (https://webaim.org/resources/contrastchecker/).

### 5.2 Manual (the other 60–70 %)

1. **Keyboard only** (unplug the mouse): Tab through the page; operate every control; open/close every dialog; check focus is visible and never lost or hidden.
2. **Zoom** to 200 % and set window to 320 px wide: no lost content, no sideways scroll.
3. **Screen reader**, 5 minutes per changed screen:
   - macOS **VoiceOver**: Cmd+F5 to toggle; VO = Ctrl+Option; VO+U opens the rotor (headings, links, form controls).
   - iOS VoiceOver (Settings → Accessibility), Android **TalkBack**, Windows **NVDA** (free).
   - Check: page title read, headings make sense, every button/field has a sensible name, errors and "saved" messages are announced, dialogs announce their title.
4. **Reduced motion** on (macOS: System Settings → Accessibility → Display → Reduce motion): animations calm down.
5. **Touch** on a phone: everything reachable without hover, targets easy to hit.

## 6. Definition of done for UI work

Same list as in `.cursor/rules/accessibility.mdc`. An agent must not say a UI task is finished until it has run (or explicitly listed as not run) each item, and reported which checks need a human (screen reader, real devices).

## 7. Accessibility statement (tilgjengelighetserklæring)

Required for **public bodies** in Norway, published via Uu-tilsynet's solution: https://www.uutilsynet.no/ . For private apps it's optional; a simple `/tilgjengelighet` (or `/accessibility`) page saying the target standard (WCAG 2.2 AA), known issues, and a contact for feedback is good practice.
