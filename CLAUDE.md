# thjw

Co-hosting organisations logo grid for The Hague Justice Week 2026. Live on
`humanityhub.org/the-hague-justice-week/` and on the staging page
`humanityhub.org/test-categories/`.

Owner: Giovanni, Communications and Marketing Manager, The Hague Humanity Hub.
He works from two MacBooks. This repo is the only shared state between them.

Sibling repo `thehaguehumanityhub/justice-week-events` holds the programme
listing, a second widget on the same page. Same delivery method, same house
style, different data. It has its own `CLAUDE.md`.

## What is in here

| File | Role |
|---|---|
| `orgs-embed.html` | The widget. Self-contained HTML fragment, scoped CSS, no dependencies. The organisations are baked into it as a JavaScript array, `ORGS`. |
| `logos/` | Normalised PNG logos, one per organisation, named by slug. |
| `logos/MISSING.md` | The logos still wanted, as a table of exact filenames. Rows get deleted as logos arrive. Currently empty. |
| `overrides.json` | Hand-edited corrections to links, names, initials and navy tiles, merged over the baked-in data at load time. |
| `organisations.json` | The source list with verification notes and where each logo came from. Kept in step with `ORGS`. |
| `README.md` | Human-facing notes: the Shortcoder snippet (`jw-orgs`) and the update workflows. |

## How it reaches the website

GitHub, then jsDelivr, then the Shortcoder WordPress plugin. No iframes, no
`.html` uploads into WordPress. WordPress strips `<script>` from page content;
Shortcoder stores snippets outside page content, so scripts survive.

The Shortcoder snippet fetches `orgs-embed.html`, injects it with `innerHTML`,
then **re-creates every `<script>` tag** so they execute. `innerHTML` alone does
not run injected scripts. An early attempt loaded the fragment with
`<script src="...">` instead: that silently never runs, because jsDelivr serves
`.html` as `text/plain` with `nosniff`.

**The repo must stay public.** jsDelivr will not serve a private repo.

`orgs-embed.html` holds a `ROOT` constant with the jsDelivr base URL, used for
`logos/` and `overrides.json`. If the repo is renamed or moved, that constant
must change too, or the grid renders with every logo missing.

## Two mechanisms that exist so Giovanni never edits the fragment

**Logos load by convention.** Every card requests `logos/<slug>.png` regardless
of what the data says, and falls back to a styled initials tile when the file
404s. So adding or replacing a logo is just committing a correctly named file.

**Text corrections live in `overrides.json`.** Keyed by the organisation's `k`
(its original name) in `ORGS`, not its display name. Each entry may set `u`
(URL; an empty string makes the card non-clickable), `n` (display name), `i`
(initials), `d` (`1` backs a light logo with navy, `0` undoes it). Keys starting
with an underscore are ignored. If the file is missing or malformed the widget
silently falls back to the baked-in data.

Two things overrides cannot do: **remove** an organisation, or **re-sort** one.
A renamed card stays where its original name sorted (City of The Hague still
sits under G, where Gemeente Den Haag was).

## Changing the roster (add, remove, rename at source)

There is no generator script. Roster changes edit the `ORGS` array in
`orgs-embed.html` **and** `organisations.json`, the same way each time:

- Parse `ORGS` as JSON with a regex on `ORGS = (\[.*?\]);`, change the list,
  write it back with `json.dumps(arr, ensure_ascii=False, separators=(",", ":"))`.
  That round-trips byte-identically, so the diff shows only the real change.
  Never hand-edit the single-line array.
- Entry shape: `{"n": display, "k": key, "s": slug, "u": url, "d": 0|1, "i": initials}`.
- Order is alphabetical ignoring a leading "The" and case. Insert by finding
  the right neighbour, not by sorting.
- `organisations.json` is indented one space; keep that.
- When removing, also delete the logo file, the `overrides.json` entry and any
  `MISSING.md` row. When renaming at source, rename the slug and logo file too,
  and the `overrides.json` key.
- Afterwards, compare old and new `ORGS` (from `git show HEAD:orgs-embed.html`)
  by key: only the intended entries should be added, removed or changed, and
  the order of the rest unchanged.

## Logos: how they are made

Target: PNG, transparent, fitting 440 × 160 px, quantised to 128 colours. The
card shows 76 px of media; logos wider than about 3:1 come out short.

Tooling is not installed system-wide. Use a throwaway venv in the session
scratchpad with Pillow (the scratchpad gets cleared between sessions, so expect
to recreate it). The processing used throughout:

1. If the corner pixel is opaque, treat it as a flat background and knock it
   out with a soft edge. **Check the corner is really background**: IFLA's green
   block touches the corner and got erased. If the file is already transparent,
   just trim.
2. Trim to the alpha bounding box, resize to fit 440 × 160, quantise, save.
3. Mean luminance of opaque pixels above about 180, or white/pale artwork:
   set `"d": 1` in `overrides.json` for a navy tile.
4. Render a card-sized preview (white card, navy tile if `d`) and look at it
   before committing.

Special cases already met:

- Black artwork on a coloured square (EcoJustice): derive alpha from darkness
  and paint pure black, or the old background leaves a coloured fringe.
- SVG logos: no converter is installed; `qlmanage -t -s <px> -o . file.svg`
  (macOS Quick Look) renders them. It handles a simple `viewBox="0 0 w h"`
  reliably and mangles offset viewBoxes.
- Some sites draw the logo as inline SVG text (Omnijuris): extract the `<svg>`,
  render it on navy, crop, then treat as a light logo.

Where logos come from, in order: what Giovanni supplies; the organisation's own
site (header `<img>` with logo/brand in it, then `og:image`, `apple-touch-icon`);
Wikimedia Commons. Never Google's favicon service: low resolution, and
hotlinking it sends every visitor to Google, a consent question on a Cookiebot
site. Nothing is hotlinked at runtime.

Sites that block automated requests (Cloudflare or similar): ifla.org,
americanbar.org, inter-just.org. Their links work in a browser; they just cannot
be checked from a script. IFLA's logo came from Commons, which also holds the
*landscape architects'* IFLA logo. Do not swap it in.

## Publishing, and checking it worked

Commit, push, then purge every changed file:

```
https://purge.jsdelivr.net/gh/thehaguehumanityhub/thjw@main/<path>
```

jsDelivr holds a branch URL for twelve hours at the edge and seven days in the
visitor's browser. Without the purge an update does not appear.

Then verify on the CDN, not just locally: fetch the file from
`cdn.jsdelivr.net/gh/thehaguehumanityhub/thjw@main/...`, compare byte size with
the local file, and for the fragment parse `ORGS` and check the count. After a
purge, `@main` occasionally keeps serving the old version for several minutes
(12 October). `@<commit-sha>` shows whether the commit itself is right; re-purge
and re-check `@main` until it matches. A replaced logo can also stay in a
visitor's browser cache: a private window shows the current one.

Pushing publishes to a public repo and CDN. Giovanni has been asking for each
change explicitly, and that request is the go-ahead; do not push work he has
not asked for.

## Current state, as of 10 November 2026

- **62 organisations, all with a logo.** `logos/MISSING.md` is empty.
- Only the Ministry of Justice of Cameroon has no link (`minjustice.cm` did not
  respond on 12 October). Its `overrides.json` entry holds `"u": ""`.
- Navy tiles (`d: 1`) in `overrides.json`: InterJust, Legitmov, Omnijuris.
  Pathfinders is forced to `d: 0` (its new logo is navy on white).
  International Nuremberg Principles Academy is forced to `d: 0` too: its new logo
  (12 Nov, supplied by Giovanni) is blue on white, the old one was white. Other light
  logos carry `d` in `ORGS` from the original build.
- Display-name overrides: Gemeente Den Haag shows as **City of The Hague**.
  FIBGAR, Columbia Society, Pathfinders and City Law School are shortened in
  `ORGS` itself.
- Removed on request: UAnimals, Center for International Legal Cooperation
  (6 Oct), International Bar Organisation (IBO) (12 Oct). Stichting NowHere was
  a wrong match and was replaced by **Nowhere Bookstore Netherlands**
  (`nowherenl.myshopify.com`), 12 Oct.
- Added: Partos (7 Oct), Cordaid (8 Oct), Stichting ABA ROLI Netherlands
  (19 Oct, links to the ABA Rule of Law Initiative page; not checkable by script),
  Climate Counsel, International Bar Association, European Society of
  International Law and Netherlands Institute for Multiparty Democracy (6 Nov;
  ESIL and NIMD had been left out on 24 Sep, Giovanni asked for them now).
  Lex Populi (10 Nov, logo from "Outstanding Logos", only 200 px; lexpopuli.org
  answers automated requests with 406, so its link is unchecked).
  The IBA logo is the roundel only; its tagline was cropped off because it is
  unreadable at card size. None of the four sites could be reached from the
  cloud session, so their links are unchecked.
- New logos now arrive in the SharePoint folder "Outstanding Logos", next to
  Giovanni's spreadsheet under THJW 2026 > 02. Communications > Website
  Visibility > templates to update content on the website. The Microsoft 365
  connector can list and download it.
- The spreadsheet's red row "International Bar Association (IBA)" is the old
  IBO row (removed 12 Oct); the new, unfilled row at the bottom is the real IBA.
- Benjamin Duerr (renamed from the typo "Benjamin Dewar") is an individual. His
  logo is a handwritten "dü" mark Giovanni supplied; it may be his signature.
- Logos worth replacing if better files arrive: Omnijuris (rendered from site
  text, cropped), InterJust (only 200 px source), Pro Bono Connect (200 px),
  UNDP (stacked portrait version, renders narrow), Cameroon (it is the
  ministry's CELCOM unit lockup, about 3.7:1), Partos (about 6:1 with the Dutch
  slogan "samen werkt."; Giovanni was offered a crop to the PARTOS block only).
- The NIMD wordmark's clipped "y" is in the supplied file and looks deliberate.
  It is about 6:1, so it renders short.
- 34 source URLs in `organisations.json` are still marked "please check".
  Every card is a public outbound link under the Hub's name, so a pass before
  the week is worthwhile.

## Giovanni's spreadsheet

`~/Downloads/participating organisations template.xlsx` is his working list.
Rows filled red mean "remove from the grid". On 12 October it was compared row
by row against the live grid; the comparison found the IBO removal, the NowHere
mix-up and five missing links. Redo that comparison whenever he sends a new
copy: names against `k`, URLs against the effective `u` after overrides.

## Rules

**The repo is public twice over**, on GitHub and on the CDN. Nothing goes in
here that should not be on a public web page.

**Do not put a clone in iCloud Drive, Dropbox or Google Drive.** Sync clients
corrupt `.git`. Keep it in `~/Developer/`.

**Logos are the property of their organisations** and are used here to identify
co-hosts of the event. That is the basis for their use; do not repurpose them.
