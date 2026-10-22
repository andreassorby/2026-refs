# Alt text guide

> Part of **Andreas-standardoppsett** (topic: `accessibility`). Installed into projects as `docs/standards/ALT-TEXT.md`.
> WCAG: 1.1.1 Non-text Content (A), 1.4.5 Images of Text (AA), 2.4.4 Link Purpose (A), 4.1.2 Name, Role, Value (A).
> More: W3C alt decision tree https://www.w3.org/WAI/tutorials/images/decision-tree/

## The one question

**"If the image disappeared, what would a sighted user lose?"** Write exactly that, in the context where the image sits. Nothing lost → `alt=""`.

## Quick decision table

| Image type | What to do | Example |
|---|---|---|
| **Informative** (adds content) | Short description of the *information*, not the pixels. | `alt="Band rehearsing on stage in Storsalen"` |
| **Functional** (the only content of a link/button) | Describe the **action or destination**, not the icon. | Trash icon button → `aria-label="Delete song"`; logo link → `alt="Chordinger – home"` |
| **Decorative** (mood, flourish, background) | `alt=""` (empty, *not* missing). Or CSS background. | Wavy divider, stock "music notes" illustration |
| **Redundant** (same info as adjacent text) | `alt=""` so it isn't read twice. | Album cover next to the visible title "Amazing Grace" |
| **Complex** (chart, diagram, chord chart, stage plot) | Short `alt` saying what it is + key takeaway; full detail nearby as text/table. | See "Complex images" |
| **Image of text** | Avoid: use real text. If unavoidable (logo, scanned sheet) `alt` = the text that matters. | Scanned PDF page → offer the ChordPro/text version |
| **Logo** | Company/product name (+ "home" if it links home). Not "logo". | `alt="Chordinger"` |
| **Avatar / profile photo** | If the person's name is shown next to it: `alt=""`. If alone: the person's name. | `alt="Kari Nordmann"` |
| **Icon with visible text label** | Hide icon: `aria-hidden="true"`. | `<MusicIcon aria-hidden="true" /> Setlists` |

## Rules of thumb

- **Length:** usually under ~125 characters, one sentence. Longer detail goes in visible text or a linked description.
- **Don't start with** "Image of", "Picture of", "Bilde av", "Icon". Screen readers already say "image"/"bilde". ("Photo of" / "Screenshot of" is fine only when the medium matters.)
- **No filenames or junk:** never `alt="IMG_2034.jpg"`, `alt="image"`, `alt="photo"`, `alt="undefined"`.
- **Context decides.** The same photo of a guitarist can be `alt="Ola Hansen playing a Gibson Les Paul"` on a gear article and `alt=""` on a page where it's mood only.
- **Language matches the page.** If `<html lang="nb">`, the alt is in Norwegian. If the UI language switches (i18next), alt texts come from the same translation files: `alt={t('home.heroAlt')}`. Never hard-code English alt on a Norwegian page.
- **Punctuation:** end with a period for a nicer screen reader pause; no ALL CAPS.
- **Don't repeat the caption or nearby text** word for word.
- **User-uploaded images:** let the user enter alt text (optional field "Beskrivelse for skjermleser"); fall back to `alt=""` when the image is clearly supplementary, never to the filename.

## Complex images (charts, diagrams, chord charts)

Give a short `alt` + a longer text alternative that's available to everyone:

```tsx
<figure>
  <img
    src="/charts/songs-per-month.png"
    alt="Bar chart: songs played per month in 2026, peaking in December."
    aria-describedby="songs-chart-desc"
  />
  <figcaption id="songs-chart-desc">
    Most songs were played in December (42) and fewest in July (6).
    <a href="#songs-table">See the data as a table</a>.
  </figcaption>
</figure>
<table id="songs-table">…</table>
```

- Chord diagram image (e.g. an Am fingering): `alt="Am chord: x 0 2 2 1 0"` (string notation), or render the chord as text.
- Stage plot / setlist screenshot: provide the setlist as an actual `<ol>`, image gets `alt=""` or a short summary.
- Inline SVG chart generated in React: `role="img"` + `aria-label` with the summary, plus a visible or toggleable data table.

## SVG

```tsx
// Informative SVG (standalone meaning)
<svg role="img" aria-labelledby="tuner-title">
  <title id="tuner-title">Tuner: 3 cents sharp</title>
  …
</svg>

// Simpler: role + aria-label
<svg role="img" aria-label="Recording">…</svg>

// Decorative SVG or icon next to visible text
<svg aria-hidden="true" focusable="false">…</svg>

// <img src="logo.svg"> works like any image: alt="Chordinger"
```

Icon libraries (lucide-react, heroicons): pass `aria-hidden="true"` on the icon, put the name on the button.

## Good vs bad – React / Next.js

```tsx
import Image from "next/image";

// ❌ Missing alt (build/lint error) or useless alt
<Image src={cover} alt="image" />
<Image src={cover} alt="cover.jpg" />

// ✅ Album cover next to visible title → redundant → empty alt
<li>
  <Image src={song.coverUrl} alt="" width={48} height={48} />
  <span>{song.title}</span>
</li>

// ✅ Album cover shown alone (e.g. a grid with no visible title)
<Image src={song.coverUrl} alt={t('song.coverAlt', { title: song.title, artist: song.artist })} width={160} height={160} />
// en: "Cover of {{title}} by {{artist}}"   nb: "Omslag til {{title}} av {{artist}}"

// ❌ Icon-only button with no name – screen reader says "button"
<button onClick={removeSong}><TrashIcon /></button>

// ✅ Functional: describe the action (include which item when there are many)
<button onClick={removeSong} aria-label={t('setlist.removeSong', { title: song.title })}>
  <TrashIcon aria-hidden="true" />
</button>
// nb: "Fjern «{{title}}» fra settlisten"

// ✅ Icon + visible text: hide the icon
<button><PlayIcon aria-hidden="true" /> {t('reader.play')}</button>

// ✅ Logo linking home
<Link href="/"><Image src="/logo.svg" alt="Chordinger – til forsiden" width={120} height={32} /></Link>

// ✅ Avatar next to name → empty; alone → name
<Image src={member.avatarUrl} alt="" width={32} height={32} /> <span>{member.name}</span>
<Image src={member.avatarUrl} alt={member.name} width={32} height={32} />

// ✅ Decorative hero background
<Image src="/hero-notes.jpg" alt="" fill priority />

// ✅ Informative photo, Norwegian page
<Image src="/storsalen.jpg" alt="Lovsangsteamet på scenen i Storsalen under søndagsmøtet." width={800} height={450} />
```

## Song / setlist app examples

| Situation | Good alt | Bad alt |
|---|---|---|
| Thumbnail of scanned sheet music in a list where the title is shown | `""` | `"Sheet music"` |
| Scanned sheet music opened in the reader (only content) | `"Amazing Grace, page 1 of 2 (scanned sheet music)"` + link to text/ChordPro version | `"PDF"` |
| QR code to join a setlist | `"QR code: open setlist Søndag 26. oktober on your phone"` + the link as text | `"QR"` |
| Key change badge as image | Use text instead: `<span>Key: G → A</span>` | `alt="key badge"` |
| Band photo on landing page (nb) | `"Band med gitar, bass og trommer øver i et lyst øvingslokale."` | `"Bilde av band"` |
| Instrument icon next to musician name "Kari – Bass" | `aria-hidden="true"` on icon | `alt="bass guitar icon"` |
| Drag handle icon in setlist reorder | Handle is a button: `aria-label="Move «Amazing Grace»"` + Move up/down buttons | no name |

## Checklist

- [ ] Every `<img>` / `<Image>` has an `alt` attribute (empty is a valid choice).
- [ ] Every icon-only button/link has an accessible name describing the action/destination.
- [ ] Decorative/redundant images and icons are hidden (`alt=""` / `aria-hidden="true"`).
- [ ] Complex images have a text or table alternative.
- [ ] Alt text is in the page language and comes from translations in multilingual apps.
- [ ] No "image of", filenames, or placeholder alt texts. `jsx-a11y/alt-text` and `jsx-a11y/img-redundant-alt` pass; axe `image-alt`, `button-name`, `link-name`, `svg-img-alt` pass.
