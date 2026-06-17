# Crawlora Anti-Bot Adoption Index — top-1M open dataset

Per-domain results of an independent anti-bot / WAF scan of the **top 1,000,000 sites**
(seeded from the [Tranco](https://tranco-list.eu/) research ranking), run `top1m-20260614`.
Licensed **[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)**.

- **998,497** sites probed · **818,614** reachable ·
  **53.5%** of reachable run a managed anti-bot / WAF.
- Searchable explorer + analysis: <https://crawlora.net/anti-bot-index> ·
  methodology: <https://crawlora.net/anti-bot-index/methodology>

## Files
| file | what |
| --- | --- |
| `data/top1m.jsonl.gz` | one JSON record per line (gzipped) — one per probed domain |
| `data/sample.jsonl` | the first 200 records, uncompressed, for a quick look |
| `data/summary.json` | headline aggregates, vendor / kind share, and the rank gradient |

Read it: `gzip -dc data/top1m.jsonl.gz | head` or, in Python,
`for line in gzip.open("data/top1m.jsonl.gz","rt"): rec = json.loads(line)`.

## Record schema
| field | meaning |
| --- | --- |
| `domain` | the site |
| `rank` | its Tranco rank |
| `reachable` | did the homepage return a usable response from our datacenter vantage |
| `protected` | runs a managed WAF / bot-management / access-control vendor |
| `vendors` | detected vendor names (array) |
| `kinds` | protection kinds: `waf` / `captcha` / `bot_management` / `access_control` / `rate_limit` |
| `difficulty_band` | HTTP-tier access tier (`easy` / `medium` / `hard` / `blocked`) |
| `block_reason`, `enforcement` | why a request didn't pass; active vs passive |
| `custom_vm`, `captcha_types` | proprietary signed-payload VM flag; detected CAPTCHA types |
| `final_url`, `status`, `probed_at` | landing URL, HTTP status, probe timestamp |

## Top vendors (share of reachable)
| Vendor | % of reachable | Sites |
| --- | ---: | ---: |
| Cloudflare | 45.0% | 368,609 |
| Google reCAPTCHA | 7.5% | 61,238 |
| hCaptcha | 3.5% | 28,249 |
| Unidentified WAF / bot protection | 2.1% | 17,383 |
| Cloudflare Turnstile | 1.3% | 10,910 |
| Imperva (Incapsula) | 0.6% | 5,249 |
| Akamai (edge) | 0.6% | 5,185 |
| Akamai Bot Manager | 0.6% | 4,810 |
| DDoS-Guard | 0.6% | 4,503 |
| Sucuri | 0.3% | 2,224 |
| GeeTest | 0.2% | 2,034 |
| Citrix NetScaler | 0.2% | 1,964 |

## The rank gradient
Protection and Cloudflare **rise** as traffic rank falls; the busiest sites run the *least*
anti-bot, and Cloudflare's dominance is a long-tail story.

| Tranco rank band | Protected | Cloudflare |
| --- | ---: | ---: |
| top 1k | 44.2% | 23.4% |
| 1k-10k | 50.7% | 34.0% |
| 10k-100k | 52.3% | 40.2% |
| 100k-1M | 53.6% | 45.6% |

## Methodology (summary)
A passive homepage fingerprint from a datacenter IP: HTTP **status**, response **header
names**, **Set-Cookie names** (*names only — never values*, so no session tokens or PII), and
challenge-shaped body markers, matched against a database of public, documented vendor
fingerprints. One GET per homepage; we never submit forms, run CAPTCHAs, or log in. It is a
**lower bound** — homepages are more open than the deep pages people actually scrape, and a
datacenter IP sees more challenges than a residential one. Full method:
<https://crawlora.net/anti-bot-index/methodology>.

## License & citation
**CC BY 4.0** — <https://creativecommons.org/licenses/by/4.0/>. Please cite as
*"Crawlora Anti-Bot Adoption Index"* with a link to <https://crawlora.net/anti-bot-index>.

## Deep-page census (top-50k)

Homepages understate protection: the page you actually scrape — a product, listing,
profile or search page — is often walled differently. We re-probed the **same top-50,000
sites** at their deep pages (run `top50k-deep-20260613`, HTTP tier).

| file | what |
| --- | --- |
| `data/top50k-deep.jsonl.gz` | 86,792 records (homepages + deep pages), one JSON record per line, gzipped |
| `data/top50k-deep-sample.jsonl` | the first 200 records, uncompressed |
| `data/top50k-deep-summary.json` | homepage-vs-deep protection stats + page-type counts |

Each record uses the same schema as `top1m.jsonl.gz` plus a **`page_type`** field
(`homepage`, or the deep-page template: `page`, `category`, `products`, `blog`, …) and a
`category`. Filter `page_type == "homepage"` for the homepage baseline; anything else is a
deep page.

On the same 50k sites: **40.5%** of homepages carry a managed wall vs **48.4%** of deep
pages, and deep pages are ~**30×** more likely to sit behind a login wall. This is an
HTTP-tier (datacenter) probe — it detects a wall's *presence*, not full browser-tier
difficulty; treat it as a lower bound.

## Run the detector yourself

The passive anti-bot/WAF detector behind this dataset is open-source (MIT):

```sh
go install github.com/Crawlora-org/crawlora-antibot@latest
crawlora-antibot nike.com        # → Akamai Bot Manager, hard
```

Repo: https://github.com/Crawlora-org/crawlora-antibot — same signatures used to build this index.
