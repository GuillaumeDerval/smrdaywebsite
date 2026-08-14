# Belgian SMR Day 2027 — conference website

Static site for the **Belgian SMR Day 2027** (18 March 2027, Brussels — both
provisional), built with [Jekyll](https://jekyllrb.com/) and the
[jekyll-theme-conference](https://github.com/DigitaleGesellschaft/jekyll-theme-conference)
template (MIT). All content is Markdown + YAML; no database, no build step
beyond Jekyll.

## Running it locally

```bash
bundle install
bundle exec jekyll serve --livereload
```

Then open <http://127.0.0.1:4000>. Edits to Markdown are picked up
automatically; edits to `_config.yml` need a restart.

To produce the files to upload:

```bash
JEKYLL_ENV=production bundle exec jekyll build
```

The finished site is in `_site/`.

> Ruby is installed via Homebrew here. If `bundle` is not found, run
> `export PATH="/opt/homebrew/opt/ruby/bin:$PATH"` first.

## What is where

| File | What it holds |
|---|---|
| `_config.yml` | Site title, date/venue banner, navigation, mailing-list form link |
| `index.md` | Landing page: date, location, what the conference is, CFA teaser |
| `call-for-abstracts.md` | Call for abstracts: topics, presentations vs posters, timeline |
| `updates.md` | Mailing-list sign-up page |
| `location/index.md` | Venue page and the OpenStreetMap map |
| `_includes/subscribe_form.html` | The reusable mailing-list box (links to / embeds the Google Form) |
| `_data/organizers.yml` | The organising institutions and their logo file names |
| `_includes/organizers.html` | The logo grid rendered from that file |
| `assets/css/main.scss` | Custom styles layered on top of the theme |
| `_data/program.yml` | The schedule: days, rooms and time slots |
| `_talks/`, `_speakers/`, `_rooms/` | One Markdown file per talk / speaker / room |
| `program/index.md` | The programme page and its intro text |
| `talks/`, `speakers/` | Overview pages, not yet linked in the menu |

## Things to do before going live

### 1. Check the mailing-list form

Sign-ups go through a Google Form (fields: email, organisation), set in
`_config.yml`:

```yaml
conference:
  mailing_list:
    form_url: "https://docs.google.com/forms/d/e/1FAIpQLSc38R57WDeBz-YeL7VUn_uuDaO1ADJ7s470qQiTuW0V39kHYg/viewform"
```

Every "Get notified" box links to it, and the `/updates/` page embeds it inline.
Responses land in the form's *Responses* tab — link it to a Google Sheet there
if you want a spreadsheet to email from.

Since this collects personal data from EU residents, the wording on the site
("used for this conference only, never shared") should match what you actually
do, and it is worth adding a one-line privacy note in the form's description
too. Make sure the form is set to accept responses without a Google sign-in, or
most external visitors will bounce.

### 2. Keep build warnings off in production

`url` is already set to `https://smrday.be` and `baseurl` is overridden by the
deploy workflow, so neither needs touching.

`conference.show_errors` controls the theme's own validation messages, and is
set to `false`. Those messages are useful while editing — they catch a talk in
`_data/program.yml` with no matching file in `_talks/`, a track that is not
declared, and similar mismatches — so turn it on while you work:

```yaml
conference:
  show_errors: true
```

Turn it back off before pushing: with it on, the warnings render as red banners
on every page of the live site. Right now the only warning is that `_speakers/`
is empty, which is expected until abstracts are accepted.

### 3. Replace the organisers' logos with official files

`_data/organizers.yml` lists the six organising institutions and the logo file
each tile uses. All six files are already in `assets/images/logos/`, taken
directly from each institution's own website:

| Institution | File | Fetched from |
|---|---|---|
| Vrije Universiteit Brussel | `vub.svg` | `vub.be/themes/custom/ocelot_vub/assets/vectors/vub-logo-long-color.svg` |
| Université libre de Bruxelles | `ulb.svg` | `ulb.be/uas/ulbout/LOGO/Logo-ULB.svg` |
| UCLouvain | `uclouvain.svg` | `uclouvain.be/themes/custom/uclouvain_theme/logo.svg` |
| KU Leuven | `kuleuven.svg` | `stijl.kuleuven.be/releases/latest/img/svg/logo.svg` |
| Université de Liège | `uliege.svg` | `uliege.be/plugins/ULiegePlugin/images/Header/logo.svg` |
| SCK CEN | `sck-cen.svg` | `sckcen.be/themes/custom/itr_theme/logo.svg` |

These are the site-header versions, not files obtained from a press kit. Before
the site goes public, ask each partner's communications office for their
official logo package and swap the files in — same file names, nothing else to
change. All six institutions have visual-identity guidelines governing
third-party use, and several (VUB and ULiège among them) distribute their
logos as a ZIP on request rather than as a public download.

Two things worth checking when you do:

- **KU Leuven** and **SCK CEN** ship as pure wordmarks with no clear-space
  padding baked in, so they read slightly larger than the others in the row.
- **SCK CEN**'s file has no fill colour set, so it inherits black. That is
  correct on the light tiles used here, but it will disappear if you ever move
  the grid onto a dark background.

Each logo sits on a light tile so it stays legible in dark mode without
recolouring anyone's brand. If a replacement file needs a dark variant, add
`logo_dark:` next to `logo:` in `_data/organizers.yml` and the grid swaps them
automatically. Removing a file (or pointing at one that does not exist) makes
that tile fall back to the institution's short name, so the section always
renders.

### 4. Add a preview image and confirm the details

Drop a 1200×630 image in `assets/images/` and reference it under
`conference.meta.link_preview.img` so shared links render a card. Then confirm
the date, the venue and the organiser name in `_config.yml`.

## Working on the programme

The provisional schedule lives in two places that have to agree:

- `_data/program.yml` holds the timetable — one day, one room (`Main Hall`), and
  fourteen slots with start and end times.
- `_talks/` holds one file per slot, breaks included. The `name:` in each file
  must match the `name:` used in `_data/program.yml` exactly, or the slot
  renders empty.

Colour coding comes from the `track:` in each talk file, matched against
`conference.talks.tracks` in `_config.yml`. The four tracks are
`Contributed talk`, `Plenary`, `Poster & networking` and `Break`.

### Filling a talk slot

`_talks/talk-1.md` … `talk-6.md` are the placeholders for contributions coming
out of the call for abstracts. For each one:

1. Rename the file and change `name:` to the real talk title, then update the
   matching entry in `_data/program.yml` to the same title.
2. Remove `hide: true` so the slot links to its own page.
3. Add `speakers:` listing the speaker names, and create a file per speaker in
   `_speakers/` whose `name:` matches. Speaker files need `first_name` and
   `last_name` as well.
4. Write the abstract in the body of the talk file.

Then uncomment the Talks and Speakers entries under
`conference.navigation.links` in `_config.yml` so the overview pages become
reachable.

### Changing the times

Edit `time_start` / `time_end` in `_data/program.yml`. The programme page is set
to `time_steps: start`, so it draws one row per slot rather than a fixed grid —
you can use any times you like without them having to land on quarter hours. To
add a parallel track, add a second room to `_rooms/` and to the day's `rooms:`
list.

The theme's [README](https://github.com/DigitaleGesellschaft/jekyll-theme-conference#content)
documents every front-matter field, and it can import a schedule directly from
a pretalx export if you end up using one.

## Deploying

The site is hosted on **GitHub Pages** at <https://smrday.be>, built by
`.github/workflows/pages.yml`. Every push to `main` rebuilds and redeploys; the
run also builds on pull requests to `main` only if you add that trigger.

The workflow runs `bundle exec jekyll build` with `JEKYLL_ENV=production` and
passes the Pages
`base_path` as `--baseurl`, which means the site resolves both at the custom
domain and at the `guillaumederval.github.io/smrdaywebsite/` fallback.

`Gemfile.lock` includes the `x86_64-linux` platform so bundler resolves on the
CI runner. If you add or update a gem, run `bundle lock --add-platform
x86_64-linux` again before pushing.

### One-time repository settings

Two things have to be set in the repository's **Settings → Pages**:

1. **Source**: *GitHub Actions*. The workflow tries to enable this itself
   (`enablement: true` on `actions/configure-pages`), but set it by hand if the
   first run fails on permissions.
2. **Custom domain**: `smrday.be`, then tick **Enforce HTTPS** once the
   certificate has been issued (this can take up to an hour after DNS
   propagates).

> A `CNAME` file in the repository does **not** work here. GitHub ignores it for
> workflow-based deployments — the custom domain lives only in the repository
> settings.

### DNS records for smrday.be

At the registrar / DNS host for `smrday.be`:

| Type | Name | Value | TTL |
|---|---|---|---|
| A | `@` | `185.199.108.153` | 3600 |
| A | `@` | `185.199.109.153` | 3600 |
| A | `@` | `185.199.110.153` | 3600 |
| A | `@` | `185.199.111.153` | 3600 |
| AAAA | `@` | `2606:50c0:8000::153` | 3600 |
| AAAA | `@` | `2606:50c0:8001::153` | 3600 |
| AAAA | `@` | `2606:50c0:8002::153` | 3600 |
| AAAA | `@` | `2606:50c0:8003::153` | 3600 |
| CNAME | `www` | `guillaumederval.github.io.` | 3600 |

All four A records are needed (they are alternatives, not a sequence); the AAAA
records add IPv6 and are optional but worth having. The `www` CNAME lets
`www.smrday.be` redirect to the apex — GitHub handles that redirect once the
custom domain is set.

Check propagation with:

```bash
dig +short smrday.be A
dig +short www.smrday.be CNAME
```

### Hosting somewhere else

`_site/` is plain static files, so any web host works —
`JEKYLL_ENV=production bundle exec jekyll build` and upload the folder. Note
that `_talks/`, `_speakers/` and `_rooms/` must exist for the build to succeed,
which is why the empty ones contain `.gitkeep` files.
