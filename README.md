# The Hague Justice Week 2026 — co-hosting organisations widget

A self-contained, searchable logo grid for
`https://www.humanityhub.org/the-hague-justice-week/`.

## Contents

| File | What it is |
|---|---|
| `orgs-embed.html` | The embed fragment. No doctype/html/head/body; all CSS scoped to `#thjw-orgs`; data baked in. |
| `logos/` | 45 normalised PNG logos (160px tall, ~320 KB total). |
| `organisations.json` | The source list with URLs, notes and which logo each row uses. Regenerate the fragment from this. |
| `overrides.json` | Hand-edited corrections to links, names and initials. Merged over the fragment's data at load time. |
| `logos/MISSING.md` | The exact filenames still wanted in `logos/`. |

## Publishing (per the Hub's embeddable-dashboards playbook)

1. The repo `thehaguehumanityhub/thjw` holds
   `orgs-embed.html` plus the `logos/` folder at its root.
2. Check it serves:
   `https://cdn.jsdelivr.net/gh/thehaguehumanityhub/thjw@main/orgs-embed.html`
3. Shortcoder → Add New Shortcode, name it `jw-orgs`, paste the loader below.
4. Put `[sc name="jw-orgs"]` on the Justice Week page.

```html
<div id="jw-orgs-host">Loading…</div>
<script>
(function(){
  var URL = "https://cdn.jsdelivr.net/gh/thehaguehumanityhub/thjw@main/orgs-embed.html";
  fetch(URL).then(function(r){return r.text();}).then(function(html){
    var host = document.getElementById("jw-orgs-host");
    host.innerHTML = html;
    host.querySelectorAll("script").forEach(function(old){
      var s = document.createElement("script");
      if (old.src) s.src = old.src; else s.textContent = old.textContent;
      old.parentNode.replaceChild(s, old);
    });
  }).catch(function(){
    document.getElementById("jw-orgs-host").textContent = "Could not load the list.";
  });
})();
</script>
```

## Making corrections without touching the generated file

Two escape hatches, so day-to-day fixes never involve editing `orgs-embed.html`.

### Adding a missing logo

The widget asks for `logos/<slug>.png` for every organisation and quietly falls back to an
initials tile when the file is not there. So adding a logo is just committing a file with the
right name. `logos/MISSING.md` lists the exact filenames still wanted, one per row.

PNG, transparent background where possible, around 160px tall, no wider than about 3:1.
If the logo is white or very light and vanishes against the white card, add `"d": 1` for that
organisation in `overrides.json` to back it with the Hub navy.

### Correcting a link, a name or the initials

Edit `overrides.json`. It is keyed by the organisation's name exactly as it appears in
`orgs-embed.html`, and each entry may set any of:

| Key | Meaning |
|---|---|
| `u` | website URL (an empty string makes the card non-clickable) |
| `n` | display name shown on the card |
| `i` | initials used when there is no logo |
| `d` | `1` puts the logo on a navy tile, `0` undoes it |

```json
{
  "EcoJustice": { "u": "https://ecojustice.example/" },
  "InterJust":  { "n": "InterJust Foundation", "i": "IJF" }
}
```

Keys starting with an underscore are ignored, which is where the file's own notes live.
If `overrides.json` is missing or malformed the widget simply uses the built-in data, so a
typo degrades quietly rather than breaking the page.

### After any edit

Commit, then open the matching purge link once, then hard-refresh the page:

```
https://purge.jsdelivr.net/gh/thehaguehumanityhub/thjw@main/overrides.json
https://purge.jsdelivr.net/gh/thehaguehumanityhub/thjw@main/orgs-embed.html
https://purge.jsdelivr.net/gh/thehaguehumanityhub/thjw@main/logos/<file>.png
```

Nothing in WordPress changes. The shortcode keeps pointing at the same URL.

### Rebuilding from the source list

`organisations.json` is the full source list with verification notes. Change that and ask
Claude to regenerate `orgs-embed.html` when the list itself changes, rather than for
one-off corrections.

## How the logos were sourced

For each organisation the site's own `apple-touch-icon`, `og:image`, header `<img …logo…>`
and favicon were fetched, the largest usable one kept, trimmed and resized to 160px tall.
Nothing is hotlinked at runtime, so no visitor request goes to a third party and the grid
does not break when a partner redesigns their site.

Still needing a proper logo file from the organiser: UNDP, IFLA, Pro Bono Connect,
UCLA Promise Institute. Eight rows have no website in the source list and render as
initials tiles: EcoJustice, InterJust, Legitmov, International Bar Organisation,
Omnijuris, Ministry of Justice of Cameroon, Palestinian Mission to the Netherlands,
and Benjamin Dewar (an individual, not an organisation).
