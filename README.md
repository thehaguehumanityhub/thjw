# The Hague Justice Week 2026 — co-hosting organisations widget

A self-contained, searchable logo grid for
`https://www.humanityhub.org/the-hague-justice-week/`.

## Contents

| File | What it is |
|---|---|
| `orgs-embed.html` | The embed fragment. No doctype/html/head/body; all CSS scoped to `#thjw-orgs`; data baked in. |
| `logos/` | 45 normalised PNG logos (160px tall, ~320 KB total). |
| `organisations.json` | The source list with URLs, notes and which logo each row uses. Regenerate the fragment from this. |

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

## Updating

- **Text only** (an organisation's name or link): edit `orgs-embed.html` in GitHub,
  change only what is inside quotes in the `var ORGS = [...]` line, commit, then open
  `https://purge.jsdelivr.net/gh/thehaguehumanityhub/thjw@main/orgs-embed.html` once.
- **Adding an organisation**: add its logo to `logos/` (PNG, ~160px tall, trimmed) and add
  `{"n":"Name","u":"https://…","l":"file-name.png","d":0,"i":"XX"}` to the `ORGS` array.
  `d:1` puts a light/white logo on a navy tile; `l:""` falls back to the initials in `i`.
- **Rebuilding from scratch**: change `organisations.json` and re-run the build script.

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
