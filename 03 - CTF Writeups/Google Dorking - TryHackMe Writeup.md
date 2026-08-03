# Google Dorking

> Using advanced search engine operators to filter indexed content for reconnaissance purposes. Legal (it's all public, already-indexed data) — but what you *do* with what you find is where legality can come into play.

---
## 1. How Search Engines Actually Work

**Crawlers** (aka spiders/bots) are what search engines use to discover content. They find pages two ways:

- **Direct discovery** — visiting a URL and reading its content/metadata

- **Link-following** — traversing every link found on a page, spreading outward like a virus

Whatever a crawler finds gets stored in an **index** — a structured collection of resources and their locations. This is the key term: **crawling is the technique, the index is the result.**

> **Why this matters for OSINT:** a search engine only knows about a page if a crawler has *reached and indexed* it. Pages that were never crawled (private, `noindex`, or completely unlinked) are invisible to dorking, no matter how good your query is. Dorking searches the index, not "the internet."

---
## 2. Search Engine Optimisation (SEO)

Websites are "scored" on how crawler-friendly they are. Contributing factors:

- Responsiveness across browsers/devices

- Ease of crawling (sitemaps, clean structure)

- Relevant, crawlable keyword content

> **Relevance to recon:**  A low SEO score / poorly indexed site often means large chunks of it are outside the index — which is itself a signal (deliberate obscurity, neglect, or legacy infra). Tools like Google's PageSpeed/Site Analyser can show this score for any domain. 

---
## 3. robots.txt

``The first thing a crawler checks. Lives at the **root** of a domain: `domain.com/robots.txt`.``

```txt
User-agent: *
Disallow: /super-secret-directory/
Disallow: /not-a-secret/but-this-is/
Sitemap: http://mywebsite.com/sitemap.xml

```

| Keyword      | Function                                       |
| ------------ | ---------------------------------------------- |
| `User-agent` | Which crawler the rule applies to (`*` = all)  |
| `Allow`      | Directories/files the crawler **can** index    |
| `Disallow`   | Directories/files the crawler **cannot** index |
| `Sitemap`    | Points to the sitemap location                 |

**Critical concept — this is the one to remember:**

> robots.txt works on a **voluntary honor system**, not access control. It's a text file saying "please don't look here" — it does NOT block access. Well-behaved crawlers (Google) respect it; nothing stops a malicious actor from visiting the path directly.

This makes robots.txt a **double-edged sword** for site owners: trying to hide something via `Disallow:` often just tells an attacker exactly where to look. First move on any recon target: check `/robots.txt`.`

`Can restrict by specific crawler too:`

```txt
User-agent: Googlebot
Allow: /
User-agent: msnbot
Disallow: /

``` 

`Can also block by **file extension using regex-style wildcards** instead of listing every file:`

```txt
User-agent: *
Disallow: /*.ini$

```

> `.ini` and `.conf` files are common examples — both are **Unix/Linux system configuration files** that often contain sensitive settings/credentials, which is exactly why an owner might try to hide them (and why they're worth searching for). 

---
## 4. Sitemaps

Comparable to a geographical map — but for a website's structure. An XML file that lists every route/page the owner *wants* indexed, making the crawler's job trivial (no guessing, no wordlists needed).

- Format: **XML**
- Real-life comparison: a **map**
- The path taken to reach content on a site = **route**
- Located conventionally at `domain.com/sitemap.xml`
- Large sites split into a **sitemap index** — a sitemap of sub-sitemaps (e.g. `sitemap-tax-category.xml`, `sitemap-pt-post-2020-03.xml`) once they exceed size/URL limits per file

> **Recon value:** sitemap.xml + robots.txt together give the full picture — sitemap = "here's what's public," robots.txt Disallow list = "here's what we tried to hide." Cross-referencing both is a real technique, not just a training exercise.

---
## 5. Core Dork Operators

| Operator         | Function                                            | Example                                |
| ---------------- | --------------------------------------------------- | -------------------------------------- |
| `"exact phrase"` | Forces exact match, ignores similar/synonym results | `"american psycho poster"`             |
| `site:`          | Restrict results to one domain                      | `site:bbc.co.uk gchq news`             |
| `filetype:`      | Restrict to a file extension                        | `site:bbc.co.uk filetype:pdf`          |
| `cache:`         | View Google's cached version of a URL               | `cache:example.com`                    |
| `intitle:`       | Phrase must appear in the page title                | `intitle:index.of`                     |
| `inurl:`         | Phrase must appear in the URL                       | `inurl:login`                          |
| `-exclude`       | Remove a term from results                          | `site:example.com -www`                |
| `OR`             | Either term matches                                 | `intitle:"login" OR intitle:"sign in"` |

>**Combining operators is where the actual power is** — narrowing by domain + filetype + keyword in one query finds things a wordlist-based brute force never would (e.g. a Freedom of Information PDF that isn't linked from anywhere on the visible site).

### Directory listing / traversal

> **`intitle:index.of`** finds exposed **directory listings** — misconfigured web servers that let anyone browse a raw file/folder structure instead of serving a proper page. One of the most classic "oops" misconfigurations found via dorking. 

---
## 6. Why Dorking Is So Appealing (and risky)

- 100% **legal** — it only surfaces already-public, already-indexed information
- No packets sent to the target, no logs on their end (passive recon)
- What you *do* with discovered information (PDFs, exposed configs, admin panels, directory listings) is where legal risk begins
- Config files (`.ini`, `.conf`) and directory listings are common *unintentional* sensitive-data leaks — a recurring theme across almost every recon methodology, not just this room

---
## Quick Recap (short version)

- Crawlers **index** what they can reach; dorking only searches that index, not the whole internet
- **robots.txt** is a voluntary "don't look here" note — not security, and often a map to what's sensitive
- **sitemap.xml** is the opposite: the owner's own list of everything they *want* found
- Core recon combo: `site:` + `filetype:` + `intitle:`/`inurl:` narrows huge result sets into precise, useful hits
- Always check `robots.txt` and `sitemap.xml` first on any target — habit starts now, it'll be automatic by the time you're doing real bug bounty recon