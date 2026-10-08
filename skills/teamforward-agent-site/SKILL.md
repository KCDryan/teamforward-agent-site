---
name: teamforward-agent-site
description: Build and ship a client real estate website for a Team Forward agent, cloned from the kirbychantoronto.com codebase. Use when the owner says "build a site for [agent]", "new client website", "Team Forward agent site", "make the Toronto site for a client", or gives a client's name and service area and wants the full package - neighbourhood guides, service guides, a personal "Meet [Name]" page, a blog that only OneCut Content can write for (Pro accounts), built into the site, free home evaluation page, IDX home search without the map, full SEO, and Team Forward branding. Runs a short intake, researches the area from primary sources, builds, deploys to Cloudflare Pages, checks the live site and hands over an owner checklist.
---

# Team Forward agent website

Builds one client website per run. The code comes from kirbychantoronto.com. Every word, number,
photo and neighbourhood is new for the client.

| | Path |
|---|---|
| Template (read-only) | `$HOME/kirbychan-toronto` (repo `KCDryan/kirbychan-toronto`) |
| Client sites | `$HOME/Client Sites/<client-slug>/` (repo `KCDryan/<client-slug>`, private) |
| Team Forward facts | skill `anthropic-skills:team-forward-knowledge-base` |

`<client-slug>` is the domain without the dot ending, for example `janedoerealty`.

---

## 0. Rules that never bend

1. **The template and the Markham repo are read-only.** Never edit, commit, push or deploy in
   `kirbychan-toronto` or `kirbychan-markham`, or touch their Cloudflare projects and databases. Before
   every push run `git remote -v` and confirm it names the client repo.
2. **Copy code, never words.** No sentence, FAQ, description or number from the Toronto or Markham sites
   may appear on a client site. Google treats shared text as duplicate content, and two clients must
   never share text either. Check with `grep` for "Kirby", "Toronto" (unless it is the client's area),
   "Markham", "kirbychan" and `1E3765` before the first deploy. Kirby Chan may be named only in the
   Team Forward section.
3. **Never invent.** No made-up reviews, sales figures, awards, years of experience, client stories or
   market numbers. A fact comes from the client's intake or from a primary source opened in this run
   (the real estate board, the municipality, ontario.ca, canada.ca, CRA, the transit agency, Statistics
   Canada). Anything unverified goes on the questions list and stays off the site.
4. **House style** (enforced by `npm run verify`): Canadian English, no em or en dashes, no comma before
   "and" or "or", no italics, exclamation marks or emoji, no predictions or superlatives.
5. **Compliance (RECO, TRESA, Ontario Human Rights Code):**
   - Every page shows the brokerage's registered name and the agent's registration title exactly as
     registered (Salesperson, Broker). No "top agent" or "number one" claims.
   - Describe homes, transit and services, never who a place "suits" by age, family status or any
     protected ground. No school catchment promises.
   - Sold data stays behind sign-in. Legal, tax and care topics are general information with "confirm
     with a lawyer, accountant or the agency".
   - If the client is licensed outside Ontario, research that regulator's advertising rules first and
     treat them as binding.
6. **Secrets.** Never ask anyone to paste a password, token or key into chat. Give exact dashboard
   clicks. The owner or client enters them.
7. **Gates.** `npm run verify` passes before every push. After every deploy, check the live pages, then
   `node scripts/indexnow.mjs <paths>` (pass paths one by one; zsh does not split a string variable).
8. **Only OneCut Content users can make blog posts.** Every client site ships with
   `"requireOneCutPro": true` and `"oneCutOnly": true` in `src/data/site.json`. Never turn either off,
   never add a paste box, TinaCMS or any other way to publish, and never hand over a site where the
   check in section 5 fails. If a client is not on OneCut Content Pro, their blog shows the starter
   posts only.
9. **Outward steps need a yes.** Creating the GitHub repo, the Cloudflare project and pointing a domain
   are confirmed with the owner once, in the intake.

---

## 1. Intake (one batch, then work alone)

Read what the owner gave with the request. Look the client up first (their current site, brokerage
profile, RECO public register, Google Business Profile, socials) and pre-fill what you can verify. Then
ask for everything still missing in **one message**, grouped like this. Use AskUserQuestion only for
the few choices with fixed options (plan, colour, IDX).

**Identity (required)**
- Full name as registered, and the first name for the "Meet ____" page
- Registration title (Salesperson, Broker, Broker of Record) and brokerage legal name
- Team name, if any, and how the brand line should read
- Phone, public email, and the email that receives leads
- Office address as registered (never invent a local one)
- Domain (owned already, or to be bought by the owner)

**Pictures (required)**
- One portrait for the hero and one for the "Meet ____" page (the largest files they have)
- Logo, light and dark versions if they exist. No logo: set the name in the site's heading font
- Optional: team photos, their own neighbourhood photos, YouTube video links

Ask the owner to drop the files in `$HOME/Client Sites/<client-slug>/intake/`. Neighbourhood
photos the client does not supply come from Wikimedia Commons with a recorded licence and credit
(`src/data/photo-credits.json`), never from a stock site or another agent's site.

**Market (required)**
- Every city and neighbourhood they work in (aim for 8 to 18; more than 18 only if each gets a
  researched, distinct guide)
- The services to promote, picked from: downsizing, first-time buyers, upsizing, estate sales,
  separation and divorce, new construction, luxury, relocation, investors. Other services are fine if
  the client really offers them
- Languages they serve clients in (drives `knowsLanguage` and any translated pages)

**Story (required for the "Meet ____" page)**
- Years licensed, why they work this area, how they work, designations
- Real reviews with the reviewer's permission and the source (Google, RankMyAgent). None: the reviews
  section is left out, not filled

**Plan and accounts**
- **OneCut Content Pro:** yes or no. The site checks the plan itself (section 5); this only decides
  whether the upload setup steps are done now or later
- **IDX:** is the client's brokerage a member of a board on PropTx (TRREB), and whose IDX token will
  this site use? No IDX access: build without home search and say so in the handover
- **Sold prices (VOW):** only if the client has a VOW agreement. Default off
- Brand colour (hex) or "pick one from the logo"
- Lead destination: email only, or a CRM webhook (Lofty, Follow Up Boss)
- Socials and Google Business Profile URL

Save the answers to `intake/CLIENT.md` in the client repo. That file is the source for every page.

---

## 2. Research (before any page is written)

Research can be split across subagents (one per group of neighbourhoods). Start each with
`model: "sonnet"` (Sonnet 5.5), as section 4 says.

Write `RESEARCH.md` in the client repo: each fact with its URL and the date opened.

1. **The area.** For each neighbourhood: the board's community name (the exact `CityRegion` value in
   PropTx, checked with a live `$filter` count), housing stock (Census profile), transit, parks,
   libraries, commute, and the board's price figures by property type. For cities outside Toronto use
   `City eq '<name>'` and that city's communities.
2. **The rules.** Land transfer tax (the municipal tax applies only inside the City of Toronto),
   property tax rates, local relief programs, vacant home tax if any.
3. **Demand.** Search autocomplete for "[area] real estate agent", "[neighbourhood] homes for sale",
   "[service] [city]". Record the phrases worth a page. Write them to `KEYWORDS.md`.
4. **Overlap.** If another Team Forward client site already covers a neighbourhood, this guide still
   has to be written fresh, with a different angle and structure.

Confirm with the owner in one message: neighbourhood list with slugs, service list, target phrases and
anything about the client that could not be verified. Then build without further questions.

---

## 3. Create the client repo

```bash
[ -d "$HOME/kirbychan-toronto" ] || git clone https://github.com/KCDryan/kirbychan-toronto "$HOME/kirbychan-toronto"
mkdir -p "$HOME/Client Sites/<client-slug>"
rsync -a --exclude .git --exclude node_modules --exclude dist --exclude .astro --exclude .claude \
  "$HOME/kirbychan-toronto/" "$HOME/Client Sites/<client-slug>/"
cd "$HOME/Client Sites/<client-slug>" && git init -b main
```

Pull the template first (`git -C <template> pull -q`) so the client gets the latest fixes.

**Remove (Toronto-only or not sold to clients):**
- The map search: `functions/api/map.ts`, `src/pages/homes-for-sale/map.astro`, the map styles in
  `home-search.css`, `mapQuery`, `GTA` and `MAP_PAGE` in `src/lib/proptx.ts` with their self-checks,
  the "Place new listings on the map" step in `.github/workflows/refresh-listing-counts.yml`,
  `leaflet`, `leaflet.markercluster` and their `@types` in `package.json`, every link to
  `/homes-for-sale/map/`, and the Leaflet and tile hosts in `public/_headers`. No Geocodio key is used
- `MARKHAM-SYNC.md`, `site.markhamSite` and its footer link, `toronto-vs-[city].astro` and
  `src/lib/comparisons.ts` (rebuild comparisons only if the research shows demand)
- `map-of-toronto.astro`, `scripts/build-toronto-map.mjs`, `src/data/toronto-map.json`
- TinaCMS (`tina/`, `TINA-CMS.md`, `check:tina-lock`, the `tinacms` packages, `/admin/`): clients
  publish through `/upload/`. Keep `src/lib/html-import.ts`, which the upload page uses
- All Toronto content: `src/content/**`, `src/assets/photos/**`, `src/data/*.json` values,
  `BLOG-LOG.md`, `BLOG-TOPICS.md`, `KEYWORDS.md`, `UPDATE-LOG.md`, the IndexNow key file in `public/`
- Translations: keep only languages the client serves clients in. English only is the default:
  remove `src/pages/[lang]/`, the other `src/i18n/*.json` files, the language switcher and the
  sitemap `i18n` block together, so no hreflang points at a missing page

**Rename and rewire (every place, found with `grep`, not from memory):**
- `src/data/site.json`: name, brand line, url, phone, email, office, brokerage (legal name, url,
  registrant, registration category), socials, lead email. Add `teamForward.url` and set both
  `requireOneCutPro` and `oneCutOnly` to `true`
- `astro.config.mjs` `SITE`, `package.json` name, `public/robots.txt` sitemap line, `public/_headers`,
  `public/.well-known/security.txt`, `scripts/indexnow.mjs` host and a new key file, `llms.txt.ts`
- `src/styles/tokens.css`: the brand colour replaces `#1E3765`, with text contrast of 4.5 to 1 or
  better checked. Regenerate `favicon`, `apple-touch-icon.png` and `og-default.png` in that colour
- `src/lib/proptx.ts`: `HOME_CITY` and `AREAS` (label plus the board's exact community names). An
  unknown `city=` falls back to the home city. Run its self-checks:
  `node --experimental-strip-types src/lib/proptx.ts`
- `src/lib/post-links.ts` `HOODS` and `SERVICES`, `src/lib/neighbourhood-hubs.ts`, `src/lib/guides.ts`,
  `src/lib/landing.ts`, `src/data/nav.json`, `public/_redirects`
- `MeetKirby.astro` becomes `MeetAgent.astro`, reading the name and portrait from `site.json`
- Page files with "toronto" in the name (`land-transfer-tax-calculator-toronto`,
  `mortgage-calculator-toronto`, `toronto-house-prices`, `best-toronto-neighbourhoods-for-[who]`,
  `<slug>-toronto` guide paths in `src/lib/format.ts`) take the client's city. Outside the City of
  Toronto the land transfer tax calculator computes the provincial tax only (`src/lib/ltt.ts`)
- `schema.ts`: `geo` from the real office address, opening hours only as the client states them,
  `knowsLanguage` from the intake

---

## 4. Pages every client gets

Write each in the client's voice from `CLIENT.md` and `RESEARCH.md`.

| Page | Notes |
|---|---|
| Home | Hero with the portrait, service grid, neighbourhood grid, live listing counts, reviews (only real ones), FAQ |
| **Meet [First name]** at `/about/` | Title and H1 "Meet [First name]". Their story, how they work, registration, languages, a Team Forward block. Person schema with `sameAs` |
| Neighbourhood guides | One per area, `/<slug>-<city>/`: board prices by type, housing stock, transit, parks, FAQ, sources, "By [Name], [Title], [Brokerage]. Facts checked [date]" byline and Article schema |
| Service guides | One pillar guide per chosen service, 2,000 words or more, opening with a direct answer, real FAQ, dated sources |
| **Free home evaluation** at `/home-valuation/` | Title "Free Home Evaluation in [City]". Lead form with Turnstile, what the client checks and how fast they reply (only what they commit to) |
| Home search | `/homes-for-sale/` one-click search, `/homes-for-sale/<type>/` and `/homes-for-sale/<area>/` landing pages with 24 listings in the HTML, the listing page (noindex). **No map** |
| Sold prices | `/sold/` only with a VOW agreement. Otherwise remove the page, `functions/api/sold.ts`, `functions/api/vow/` and every link to them |
| Blog | Index, categories, RSS, and 6 starter posts researched for the client's area and services, following `BLOG-PLAYBOOK.md` and logged in `BLOG-LOG.md` |
| Tools | Mortgage calculator, land transfer tax calculator, house prices page from the board's figures |
| Buyers, sellers, services, contact, thank-you, privacy, terms, accessibility, 404 | Rewritten, never copied |

**Subagents run on Sonnet 5.5.** Every subagent used for research or writing (area research,
neighbourhood guides, service guides, blog posts, translations) is started with `model: "sonnet"` on
the Agent tool, never Fable or Opus, to keep token use down. The main session keeps the code work,
the final fact check against `RESEARCH.md` and `npm run verify`.

Neighbourhood guides and posts can be written by parallel subagents. Brief each one fully: its own
file only, the research rows it may use, the style rules, no git, no build, run
`node scripts/check-style.mjs` and `node scripts/check-blog.mjs` and fix only its own errors. No
worktrees inside the repo (`.claude/worktrees/` stays in `.gitignore`).

**Team Forward on the site.** Load `anthropic-skills:team-forward-knowledge-base` before writing these,
and follow its voice and spelling rules ("eXp Realty", "Kirby Chan").
- Footer, every page: "Proud member of Team Forward, powered by eXp Realty" linking to
  `https://teamforwardexp.com/`
- "Meet [First name]" page: a short section on what Team Forward gives the client's customers (an
  agent with their own brand, backed by a network's training, tools and mentorship)
- One line for other agents: "Are you an agent? See how Team Forward works" linking to the same URL
- Never state fees, splits, revenue share figures or a founding date. The knowledge base lists those
  as unanswered

---

## 5. The blog system (every client site, OneCut Content only)

Every client site ships with the template's blog system unchanged. Do not rebuild it; keep these
files and check they still work after the rename pass.

**The rule: only OneCut Content can make a post.** A client cannot paste or upload an article. They
type a topic on their own site and onecutcontent.com researches and writes it. Both settings below
are `true` in `src/data/site.json` on every client site:

| Setting | What it does |
|---|---|
| `"requireOneCutPro": true` | The upload page works only while the client's OneCut account is on Pro or above. The site asks OneCut (`GET /api/v1/account`, answer kept ten minutes) |
| `"oneCutOnly": true` | No paste box and no "Upload a post I already have". A new post is saved only with the one-use ticket that comes with an article OneCut finished writing. Saved posts can still be edited, unpublished and removed |

**What the client sees** (`/upload/`, also reached at `/createblog`):
1. Sign in with the upload password.
2. First time only: "Connect your OneCut account". They create a website key under Account on
   onecutcontent.com ("Connect your website", shown on Pro and above) and paste it. It is checked
   with OneCut and kept encrypted in D1. If the upload password changes they paste it again.
3. "My blog posts" with one button, **Write a new post**, and Edit and Remove on each post.
4. They type a topic. A live panel shows what OneCut is doing: each step with a tick and a timer,
   the searches it ran, the pages it read and the keyword the article is aimed at. About a minute,
   2 tokens from their OneCut account.
5. The article opens in "Check the details" with an exact preview. They publish or save a draft.
   Blog cards lead with a picture: the photo of the neighbourhood the post is about (from its links,
   or its title and summary), or a colour band naming the topic (`BlogCardMedia.astro`). A new post
   gets its picture at once from the hidden set on the blog page.
6. A Done screen. The post is live at its address at once and at the top of `/blog/` at once.
   Category pages, the sitemap and the feed catch up at the next build (every morning).

Not Pro, no key, a revoked key or a plan that drops: after sign-in the connect screen shows with a
link to OneCut's pricing, and every action answers 403. Posts already published stay up. If
onecutcontent.com cannot be reached, signed-in clients keep access; writing simply fails with a
message.

**Files that make it work** (all must be in the client repo):
- `src/pages/upload/index.astro`: the screens (sign in, connect, posts, write, check, done)
- `functions/api/upload/[action].ts`: sign-in, connect, write (passes OneCut's steps through and
  issues the ticket), preview, publish, list, unpublish, export
- `functions/blog/[[path]].ts`: serves a new post at once and adds new posts to the top of `/blog/`
- `src/lib/uploads.ts`, `src/lib/html-import.ts`, `src/lib/post-links.ts`
- `src/pages/blog/upload-shell/[category].astro`: the built layout a new post is poured into
- `scripts/pull-uploads.mjs` and `scripts/prepare-quick-posts.mjs`: each build writes uploaded posts
  into the blog collection. A post that fails a check is held back from the list, never the deploy
- `public/_redirects`: `/createblog` and `/blogs`

**Set up once per client:**
1. Create the D1 database and bind it as `VOW_DB`.
2. The owner sets `UPLOAD_PASSWORD` (12 characters or more) in Cloudflare and gives it to the client.
3. Redeploy.

**Test before handover** (all must pass):
- Signed in with no key connected: `/api/upload/me` answers
  `{"ready":true,"signedIn":true,"needsPro":true}` and the connect screen shows.
- With the owner's own Pro key connected for the test: `me` answers `canWrite` and `oneCutOnly`,
  "Upload a post I already have" is absent, and one written post publishes and appears at the top of
  `/blog/` without a rebuild. Remove the test post and the owner's key afterwards
  (`DELETE FROM upload_settings` on the client's D1) so the client connects their own.
- The next Cloudflare build passes with the post pulled in.

`/upload/` and `/blog/upload-shell/` stay noindex and disallowed in robots.txt. kirbychantoronto.com
has both settings on too.

---

## 6. SEO checklist (all must hold before launch)

- One H1, a unique title under 60 characters and a unique description of 140 to 160 characters on
  every indexable page, with the target phrase in the title and H1
- Self-referencing canonicals, a sitemap with real `lastmod` dates, noindex pages left out of it
- `robots.txt`: sitemap line; disallow `/api/`, `/upload/`, `/blog/upload-shell/`, `/cdn-cgi/`,
  `/homes-for-sale/?` and `/sold/?`; named AI crawler groups allowed
- JSON-LD: RealEstateAgent (address, geo, hours as stated, `sameAs`, `parentOrganization` with url),
  Person, WebSite on the home page, BreadcrumbList, Article on guides and posts, Place on
  neighbourhood guides, FAQPage only where the questions are visible
- `llms.txt` and `llms-full.txt` generated for the client
- Images: AVIF and WebP with `srcset`, fallbacks capped at 1200 px, alt text, hero image
  `fetchpriority="high"`
- Internal links: every page reachable; guides link to their landing page, hubs, tools and posts
- No page under 300 words indexable (`check:thin`)
- Run the `seo` skill's `crawl_audit.py`, `schema_required_props.py` and `hreflang_checker.py` (if
  translated) on the live site with Python 3.12, and fix what is real. Check script flags against the
  HTML before acting: several are false positives

---

## 7. Deploy

1. `npm install`, then `npm run verify` until it passes.
2. With the owner's yes from the intake: `gh repo create KCDryan/<client-slug> --private --source . --push`.
3. Cloudflare Pages project connected to the repo (build `npm run build`, output `dist`). Give the
   owner exact clicks for anything the CLI cannot do.
4. Secrets the owner enters in Cloudflare (Settings, Variables and Secrets), then one redeploy:
   `PROPTX_IDX_TOKEN`, `TURNSTILE_SITE_KEY`, `TURNSTILE_SECRET_KEY`, `LEAD_WEBHOOK_URL` or the lead
   email setting, `UPLOAD_PASSWORD`, VOW keys (VOW only).
5. Poll `gh api repos/KCDryan/<client-slug>/commits/<sha>/check-runs` until "Cloudflare Pages" and CI
   succeed. Use `until` loops, not long sleeps.
6. Landing pages fetch the site's own `/api/listings` at build time, so the first build shows the
   fallback. Run the counts workflow once after the token is set
   (`gh workflow run refresh-listing-counts.yml`) and confirm every landing page then shows 24 cards
   from its own communities.
7. Live checks: every sitemap URL returns 200; listing counts equal PropTx's own `$count`; the lead
   form delivers a test lead to the owner's test address; `/upload/` behaves as the plan says; 1366 px
   and 375 px with no sideways scrolling; www redirects to the root with a single 301.
8. IndexNow for every URL.

**Known pitfalls** (learned on the Toronto and Markham builds):
- PropTx: `@odata.nextLink` returns 500, so page with `$top`, `$skip` and `$orderby=ListingKey`;
  `PropertySubType` has `'Semi-Detached '` with a trailing space; always exclude parking spaces and
  lockers; card photos use the `Medium` size
- Cloudflare: use 503, not 502; secrets reach the site only after a new deploy; D1 writes go through
  `db.batch()` in groups of 100
- Astro: scoped styles miss script-created elements; `getStaticPaths` cannot see frontmatter constants;
  `display: grid` beats `hidden`
- YAML values containing `: ` must be quoted
- Gate every push on verify in the same command (`npm run verify && git push`), never as separate
  steps: a push after a failed verify was sent once and Cloudflare rejected the build
- After a client publishes their first post, confirm the next Cloudflare build passes. Uploaded posts
  are pulled into the build, and a post that fails a check is held back from the list, not the deploy
- A new area's landing page refuses listings from outside its communities until the API knows the area

---

## 8. Handover

Write `HANDOVER.md` in the client repo and give the owner the same summary in plain words:

- **Live:** the URL, page counts by type, listing counts, what `/upload/` does for this client
- **Could not verify:** every fact left off the site and what is needed to add it
- **Owner checklist, with exact clicks:**
  1. Search Console: verify the domain with a TXT record, submit `sitemap-index.xml`, request indexing
     for the home page, hubs and guides (about 10 a day)
  2. Bing Webmaster Tools import
  3. Google Business Profile: add the site URL; send back the profile link for `sameAs`
  4. Cloudflare: www redirect rule, Email Address Obfuscation off
  5. Links from the client's eXp and realtor.ca profiles
- **Client guide:** one page on how to upload a post (Pro) and who to call for changes

Add a row to `$HOME/Client Sites/CLIENTS.md`: client, domain, repo, date, plan (Pro or not),
IDX and VOW status, neighbourhoods. Check it at the start of every run so two clients never get the
same text for a shared neighbourhood.

---

## 9. Keeping client sites current

When the template gains a fix (see its git log since the client's "template commit" recorded in
`HANDOVER.md`), port code changes only, client by client, with the same gates. Map, Markham and Tina
commits are skipped. Content is never ported.

When a run teaches something new (a pitfall, a better intake question, a step that had to be
repeated), add it to this file before finishing.
