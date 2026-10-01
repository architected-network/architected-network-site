# Architected Network — project website

Static website for the NSF DMREF / NSF–NSERC collaborative project
**"Codesign of highly entangled materials through the lens of network science."**

Plain HTML + CSS + one small script, no build step, no dependencies — ready to be served
by GitHub Pages or any static file server.

## Structure

```
index.html              News
team.html               Photo grid; click a portrait for name / position / institution
publications.html       Flat publication list, reverse chronological
resources.html          Flat list of links (tools, code, data)
css/style.css           All styles
assets/logo/            The mark (logo.svg; logo-on-dark.svg is unused now that the
                        sidebar is light), PDFs, institution wordmarks and nsf-logo.png
assets/people/*.jpg     Portraits shown in the team grid
source/                 Full-size originals (portrait PNGs, NSF.eps, event photos).
                        Git-ignored — never served, never committed.
CNAME                   Custom domain for GitHub Pages
```

## Layout

The header is a **light sidebar on the left** (`header.sidebar`) holding two blocks:
`div.sidebar-top` (the mark, the site name and tagline, a vertical nav) and, pinned to
the bottom with `margin-top: auto`, the `footer` (one institution logo per line, then the
copyright — there is no link list in the footer). Text and nav are right-aligned against
the page; the marks (site logo and institution logos) are centred.
Everything else is the page column to its right (`div.page`), which opens with the dark title
banner (`div.hero`) — on the home page the DMREF eyebrow, the project title and a
one-paragraph lede; on other pages the page title (and, where one is kept, a line of context) — then `main`.

The sidebar is `position: sticky` and full viewport height, so it and the footer stay in
view while the page scrolls. It needs `align-self: start` — without that a grid item
stretches to the row height and sticky does nothing.

**Below 820px** the sidebar gets `display: contents`, which dissolves its own box so its
two children become items of the body grid directly. `.sidebar-top` is then styled as a
horizontal top bar, and `order` puts the footer after the page — the footer is at the
bottom of the page on a phone and at the bottom of the sidebar on a desktop, from a
single copy of the markup. Alignment returns to the left on narrow screens; logo sizes
are the same in both layouts (institution wordmarks 30px, NSF seal 72px, site mark 80px
in the sidebar and 44px in the top bar).

Every page has the same shell; only the banner text and `main` differ. If you change the
sidebar, footer or `<head>`, change it in all four files.

## Preview locally

```sh
python3 -m http.server 4173
# then open http://localhost:4173
```

## Deploy with GitHub Pages

1. Push to the `main` branch of this repository.
2. On GitHub: **Settings → Pages → Source: Deploy from a branch**, branch `main`, folder `/ (root)`.
3. The site is served at `https://architected-network.github.io/architected-network-site/`
   until the custom domain below is live.

## Custom domain — architected.network

The domain is registered with Squarespace; the site is hosted by GitHub Pages. The
`CNAME` file in the repo root tells GitHub which domain to serve. All page links are
relative, so the site works at either address.

**1. DNS at Squarespace** — Domains → architected.network → DNS → DNS settings →
Custom records. As of 19 Sep 2026 the domain carries Squarespace's own site records —
four `A` records on `@` (`198.185.159.144`, `198.185.159.145`, `198.49.23.144`,
`198.49.23.145`) and `CNAME www → ext-sq.squarespace.com`. Delete those five, then add:

| Type | Host | Data |
| --- | --- | --- |
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| CNAME | `www` | `architected-network.github.io` |

The CNAME target is the *organization's* Pages host, with no repository path.

**2. GitHub** — repo → Settings → Pages: source `main` / root, then Custom domain
`architected.network` → Save. GitHub checks DNS (a few minutes after the records
propagate), then offers **Enforce HTTPS** — turn it on once the checkbox is enabled.
`www.architected.network` will redirect to the apex automatically.

**3. Verify** from a terminal:

```sh
dig +short architected.network          # the four 185.199.*.153 addresses
dig +short www.architected.network      # architected-network.github.io. then the same IPs
curl -sI https://architected.network | head -1   # HTTP/2 200 once the certificate exists
```

**If HTTPS never becomes available** — browsers warn that the connection is not private,
and `curl` reports `no alternative certificate subject name matches` — GitHub is still
serving its own `*.github.io` certificate. Check which one is served:

```sh
echo | openssl s_client -connect architected.network:443 -servername architected.network \
  2>/dev/null | openssl x509 -noout -subject     # CN=*.github.io means not issued yet
```

GitHub requests the domain's certificate when the custom domain is set. The `CNAME` file
here was pushed (19 Sep 2026) while DNS still pointed at Squarespace, so that request
failed and was not retried. To re-trigger it: Settings → Pages → Custom domain →
**Remove**, then enter `architected.network` again → **Save**. GitHub records the change
as commits to `CNAME` on `main`, so `git pull` before your next push.

Optionally verify the domain org-wide (GitHub org → Settings → Pages → Add a domain,
which gives a TXT record to add at Squarespace) so nobody else can claim it for their
Pages site.

## Theming

All colours and type sizes are custom properties at the top of `css/style.css` — edit
them there, not in individual rules.

**Type: three sizes, no others.** Hierarchy comes from weight, letter-spacing and
uppercasing, never from a new size.

| Token | Size | Used for |
| --- | --- | --- |
| `--fs-lg` | 1.35rem | page titles, `h2`, the lightbox name |
| `--fs-md` | 1rem | body copy, nav, item titles |
| `--fs-sm` | 0.7rem | eyebrow, dates, labels, footer |

**Colour: black and white, with violet used sparingly.** Greyscale tokens (`--bg`,
`--surface`, `--divider`, `--text`, `--muted`, `--heading`) carry the layout. The title banner and
the lightbox are the two dark surfaces, on `--dark` / `--dark-text` / `--dark-muted`. `--violet` `#665CA2`
appears only on links, the sidebar's edge rule and focus rings;
`--violet-light` is the same accent tuned to read on the dark surfaces (banner eyebrow, lightbox role and award link). Green appears only inside the logo
artwork; it is not a CSS token.

The three content lists (news, publications, resources) share one `.rows` class for
their row padding and dividers; each adds only what differs.

Portraits are forced monochrome with `filter: grayscale(1)`; news photos are in colour. There is **no dark mode** —
the site renders the same regardless of `prefers-color-scheme`; do not add one.

## Footer logo strip

The NSF seal plus the five institution wordmarks (`ul.partner-logos`), NSF first, on
every page. **Every logo keeps its own brand colours — none are recoloured or filtered.**
Several are dark (MIT maroon `#750014`, Harvard near-black `#1e1e1e`, the Toronto crest
has no `fill` so it renders black), which is why they only ever sit on light surfaces.

Each logo sits on its own line, centred. The wordmarks share one **width** (140px) so
they read as a set; the two exceptions are sized by height — MIT (`li.partner-logos-mit`,
30px), whose blocky mark would dominate at that width, and the NSF seal
(`li.partner-logos-funder`, 72px), which is round. It was supplied as a 6.3 MB EPS, which
browsers cannot render; `nsf-logo.png` is a trimmed 256px transparent PNG made with:

```sh
gs -dQUIET -dBATCH -dNOPAUSE -dEPSCrop -sDEVICE=pngalpha -r600 \
   -sOutputFile=/tmp/nsf_raw.png source/logo/NSF.eps
```

## Common edits

**Add a news item** — in `index.html`, copy an `<li>` inside `<ul class="news-list rows">`:
a `p.news-date`, then `h3`, then `p`, stacked in one column. Newest on top. An item may
carry a photo under its text: a `<figure><img></figure>` after the `p`, full width of the text
column, shown in colour. Put web-sized copies in `assets/news/`
(≈1600px wide, JPEG q82 — the kickoff photo went from 2.2 MB to 258 KB that way) and set
the `<img>` `width`/`height` attributes to the file's pixel size.

**Team photo focus** — portraits have no focus styling, and the lightbox leaves nothing
focused when it closes.

**Add a person** — in `team.html`, copy an `<li>` inside `<ul class="photo-grid">` and
set `data-name`, `data-role`, `data-inst`, an optional `data-bio="..."`, the `<img src>`
and the `alt`. The lightbox reads those attributes; nothing else to wire up. Empty or
missing fields are dropped rather than left as blank lines.

**Award links** — three optional attributes on the same button:

| Attribute | Example |
| --- | --- |
| `data-award-label` | `NSF award` |
| `data-award-id` | `2523080` |
| `data-award-url` | `https://www.nsf.gov/awardsearch/show-award?AWD_ID=2523080` |

With an id, the line reads `NSF award 2523080` and the **number** is the link. Without
one, the label itself is the link — that is how the NSERC entry works, since only a
funding-decision page was supplied, not an award number. Links are built with DOM nodes,
not `innerHTML`.

**Replace a portrait** — overwrite the file in `assets/people/`, keeping the name. The
grid cell is 2:3 with `object-fit: cover`, so a 2:3 source avoids cropping. Update the
`width`/`height` attributes on the `<img>` to match; they prevent layout shift.
Regenerate greyscale 1200px JPEGs from PNG originals with:

```sh
python3 -c "
from PIL import Image; import glob
for f in glob.glob('source/people/*.png'):
    im=Image.open(f)
    if im.mode in ('RGBA','LA','P'):
        im=im.convert('RGBA'); bg=Image.new('RGBA',im.size,(255,255,255,255))
        im=Image.alpha_composite(bg,im)
    im=im.convert('L'); w,h=im.size
    im=im.resize((round(w*1200/h),1200), Image.LANCZOS)
    im.save('assets/people/'+f.split('/')[-1][:-4]+'.jpg','JPEG',quality=82,optimize=True,progressive=True)"
```

**Add a publication** — in `publications.html`, copy an `<li>` inside `<ul class="rows">`.
One flat list in reverse chronological order; insert at the right year. Bold a team
member's name with `<strong>`.

**Add a resource** — in `resources.html`, copy an `<li>` inside `<ul class="link-list rows">`:
a bold link, an em dash, one line of description. External links use
`target="_blank" rel="noopener"`.

**To fill in later**

- Position and institution for Csaba Both, Yaqi Guo and Yue Wang (TODO in `team.html`).
- Optional `data-bio` text for anyone who wants one.
