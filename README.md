# best residential proxy provider: per-GB proxy pools vs. scraping APIs, with the real cost math most reviews skip

Searching for the best residential proxy provider usually turns up a dozen top-10 lists of companies selling raw residential IPs by the gigabyte. That framing misses something important: there are two different products competing for your money here. One sells you bandwidth from a residential IP pool and leaves rotation, CAPTCHAs, and browser rendering to your code. The other — ScraperAPI is the best-known example — is an API that routes every request through a managed residential pool and handles all of that for you, billing in credits per request instead of dollars per GB. Picking "the best" means first deciding which of those two models fits your workload. This guide walks through both, with verified pricing and the cost details that most comparison posts leave out.

> **Quick take:** If your job is scraping specific, heavily protected sites — Amazon product pages, Google SERPs, LinkedIn — and you'd rather not maintain proxy infrastructure, ScraperAPI's Hobby plan at **$49/month** (100,000 credits, 20 concurrent requests) is the practical starting point, and a 7-day trial with 5,000 credits costs nothing. If you're an experienced user who wants raw residential traffic at the lowest price per GB, IPRoyal's pool starts at **$1.75/GB** with traffic that never expires. Details on both below.
>
> 👉 [Start the free ScraperAPI trial — 5,000 credits, no card required](https://www.scraperapi.com/?fp_ref=coupons)

## What "best" actually means for residential proxies

A residential proxy routes your traffic through an IP assigned by an ISP to a real household connection. Those IPs carry more trust than datacenter IPs, so sites flag and block them far less often — which is why they're the standard choice for web scraping, price monitoring, and ad verification. One caveat worth knowing up front: of the four common proxy types (residential, ISP, mobile, datacenter), residential IPs are the slowest by most measures. Even IPRoyal's own documentation admits this. You're buying trust, not speed.

The market splits cleanly in two:

1. **Per-GB proxy pools** — IPRoyal, Bright Data, Oxylabs, Decodo and similar. You buy traffic, get proxy endpoints, and write the logic yourself: IP rotation, headers, JavaScript rendering via your own headless browser, CAPTCHA solving, retries.
2. **Scraping APIs with built-in residential pools** — you send a URL plus parameters, and the service picks a residential IP, renders the page, retries failures, and returns HTML or parsed JSON. ScraperAPI works this way. You pay in credits, one credit being the cost of a standard page request.

Most "best provider" lists only cover category 1. If your actual task is "get clean product data from this e-commerce site every morning," the second category often wins on total effort even when its sticker price looks higher — because the engineering you'd spend maintaining rotation and unblocking logic is the expensive part.

## ScraperAPI: full plan lineup and current pricing

ScraperAPI currently sells seven paid tiers, all with a 10% discount on annual billing. Every plan includes JavaScript rendering, premium proxies, rotating proxy pools, CAPTCHA and anti-bot detection, auto-parsing, custom headers and sessions, unlimited bandwidth, and a 99.9% uptime guarantee.

| Plan | Monthly | Annual (per month) | Credits | Concurrent requests | Plan page |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10 | 100,000 | 20 | [ Buy Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [ Buy Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [ Buy Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [ Buy Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [ Buy Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [ Buy Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [ Contact sales](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

There's also a free tier: sign up and get 1,000 free API credits with up to 5 concurrent connections, no payment involved. The Hobby plan is the smallest paid option, and the annual price of $44.10/month works out to roughly $0.44 per thousand standard pages — cheap for standard pages, less cheap for the heavier domains we'll get to next.

If you want to check the lineup against your own volume before committing, 👉 [compare all ScraperAPI plans on the official pricing page](https://www.scraperapi.com/pricing/?fp_ref=coupons).

## The credit math that decides whether it's actually cheap

Here's the part almost no comparison article explains properly: a "100,000 credits" plan does not mean 100,000 pages. ScraperAPI charges different credit amounts by domain and by request type, and the gaps are large.

**Per-domain costs (per request):**

| Domain type | Credits |
| --- | --- |
| Normal pages (most of the web) | 1 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing (all subdomains) | 25 |
| LinkedIn | 30 |

**Per-parameter add-ons:**

- `premium=true` (residential pool): +10 credits per request
- `render=true` (JavaScript rendering): +10
- `premium=true` + `render=true`: 25 total
- `ultra_premium=true`: +30 (paid plans only)
- Bypassing Cloudflare, Datadome, PerimeterX, or Akamai: +10 per scrape

Put those together and the arithmetic changes fast. On Hobby at annual pricing ($44.10 for 100,000 credits), a standard page costs about $0.44 per thousand. A Google SERP request — 25 credits — costs about $11 per thousand pages. An Amazon page with rendering (5 + 10 = 15 credits) lands around $6.60 per thousand. This is why the free Domain Multiplier in the dashboard matters: you can check the exact credit cost of any target URL before you commit to a plan, or call the `urlcost` API endpoint programmatically. Every response also includes an `sa-credit-cost` header showing what that request actually consumed, and you can set a `max_cost` cap per scrape so one request can't blow through your budget.

The honest takeaway: for plain pages, the per-credit model is very affordable. For SERP-heavy or heavily protected targets, credits evaporate quickly, and you should run your real workload numbers through the cost estimator before choosing a tier.

## Hidden details that affect real usage

These are the fine-print items that live in the docs rather than on the pricing page, and they're exactly what vendor comparison lists tend to skip:

- **Billing only happens on successful requests** — status codes 200 and 404. Failed requests after retries cost nothing. But note: if you cancel a request yourself before the 70-second completion window ends, that request still gets billed.
- **Geotargeting has plan-level limits.** `country_code` itself doesn't cost extra credits, but on Hobby and Startup, geographic targeting is restricted to US and EU locations. Country-level targeting anywhere opens up from Business ($299) upward.
- **Analytics history differs.** Hobby and Startup only show the last 30 days of analytics; Business and above get unlimited history.
- **Concurrency overages are refused, not queued.** Exceed your thread limit and you get a 429 response telling you to retry within your plan's limit — no extra charge, but the request simply doesn't run. Plan concurrency around your peak parallel jobs, not your average.
- **Running out of credits has three different behaviors.** By default, scraping pauses until credits renew. Hobby through Professional can opt into auto-upgrade, which moves you to the next plan automatically when you run dry. Scaling and above can instead use pay-as-you-go at a fixed per-credit rate, capped at a monthly spending limit you set. Annual subscribers can also renew early for a fresh batch of credits.
- **Discounted custom Enterprise deals have a floor on Ultra Premium usage:** a minimum of $3 per thousand requests without rendering, $7 with rendered pages.

## How ScraperAPI compares to the per-GB providers

For readers comparing the per-GB players that dominate this keyword, here's the verified landscape. None of these companies' numbers below include links — this is purely for decision-making context.

| Provider | Entry pricing | Billing model | Notable terms |
| --- | --- | --- | --- |
| ScraperAPI | $49/mo (100k credits) | Credits per request | Residential pool + rendering + bypasses included; per-domain multipliers apply |
| IPRoyal | From $1.75/GB (bulk); plans start at $7/GB | Per GB, pay-as-you-go | Traffic never expires; 64M+ IPs, 195+ countries; HTTP(S) and SOCKS5; sticky sessions up to 7 days |
| Bright Data | From $2.50/GB | Per GB / plans | G2 rating 4.6/5; 150M+ IPs, 195+ countries |
| Oxylabs | From $2.50/GB | Per GB / plans | G2 rating 4.5/5 (422 reviews); 175M+ pool cited in comparison posts |
| Decodo | From $2/GB | Per GB / plans | Trustpilot 4.2/5 (2,000+ reviews); 99.92% success rate and 0.53s response in Proxyway's benchmark |

A few things worth reading closely in that table. IPRoyal's headline "$1.75/GB" is a bulk price — their standard plans start at $7/GB, and you get down to $1.75 by purchasing large traffic volumes upfront. The offsetting perk is that the traffic never expires, so stockpiling is genuinely safe. Their ISP proxies (static residential) are billed per IP instead, with 1/30/60/90-day leases, and IPRoyal itself recommends static residential over rotating residential for anyone managing multiple social media accounts.

Pool-size claims across the industry deserve skepticism too. The figures floating around comparison posts range from 40M to 175M+, and one review site openly warns readers to be cautious with "very large advertised pool figures that cannot be independently verified." ScraperAPI's own pricing page describes 40M+ IPs across residential, datacenter, and mobile networks spanning 50+ countries; some third-party reviews cite a higher residential-only figure. Treat any of these as marketing-scale indicators rather than precise specifications.

## Which model fits which reader

The decision mostly comes down to three questions.

**How much infrastructure do you want to run?** If you're comfortable maintaining rotation logic, headless browsers, and CAPTCHA handling — or you already have that stack — a per-GB pool gives you maximum control and the lowest raw bandwidth cost. IPRoyal at $1.75/GB with non-expiring traffic is the strongest value among the verified options for irregular, bursty usage.

**What are you scraping?** If your targets are Amazon, Google, Bing, LinkedIn, or sites behind Cloudflare/Datadome/PerimeterX/Akamai, ScraperAPI's pricing deliberately reflects that: those domains cost more credits because the service invests custom bypass logic in them. You're paying for solved problems. If your targets are ordinary websites with light protection, the per-credit model on standard pages is very economical, and the free trial will show you exactly what your workload costs before you pay anything.

**How predictable is your volume?** Per-GB pricing scales linearly with bytes; credits scale with pages and complexity. Steady, high-volume scrapers of ordinary pages tend to do well on either. Spiky users benefit from IPRoyal's non-expiring traffic. Teams that hate surprise overages should note ScraperAPI's `max_cost` per-scrape cap and its refusal (rather than silent queuing) on concurrency overruns — costs stay bounded by design.

## Frequently asked questions

**Is there a free way to test before paying?**
Yes. ScraperAPI offers a 7-day trial with 5,000 API credits and no credit card required, plus a permanent free tier of 1,000 credits at up to 5 concurrent connections. Per-GB providers in this space generally don't offer meaningful free residential traffic; IPRoyal, for instance, doesn't advertise a free residential tier, and its own FAQ advises against trusting third-party free proxies for privacy reasons.

**Does using residential proxies for scraping mean extra ScraperAPI charges?**
Not by itself. Setting `country_code` for geotargeting costs zero additional credits — the residential routing is bundled into plan features. Extra credits come from specific add-ons: `premium=true` (+10), `render=true` (+10), `ultra_premium=true` (+30, paid plans only), and anti-bot bypasses (+10) when triggered.

**What happens if I exceed my plan's limits?**
Concurrency overages get a 429 response with no charge — you retry within your plan's thread count. When you exhaust credits, scraping pauses by default; auto-upgrade (Hobby through Professional) or pay-as-you-go (Scaling and above, with a self-set monthly cap) can keep jobs running.

**Are residential proxies legal to use?**
Residential proxies are legal for most purposes, such as market research and price monitoring, though specific uses can fall under particular laws and platform terms. Providers including IPRoyal explicitly prohibit activities like DDoS attacks and other illegal use in their terms of service. Check your target site's terms and your local regulations before large-scale scraping.

## Making the final call

Matching the situation to the product:

- **Building a production scraping pipeline for a handful of known, protected targets** (Amazon prices, SERP tracking, LinkedIn data): ScraperAPI, starting at Hobby and using the free Domain Multiplier to project credit burn first. The 7-day trial with 5,000 credits tells you your real per-page costs with zero commitment.
- **Scraping ordinary sites at high volume, or you already run your own unblocking stack:** a per-GB residential pool. IPRoyal's non-expiring traffic at $1.75/GB in bulk is the verified price leader; Bright Data and Oxylabs at $2.50/GB make more sense when you need their larger pools and enterprise tooling.
- **Multi-account social media management:** neither rotating product is ideal — static residential/ISP proxies billed per IP (IPRoyal leases run 1 to 90 days) exist for exactly this use case.
- **Just experimenting:** start with ScraperAPI's free 1,000-credit tier, then decide.

One last check before you subscribe to anything: estimate your monthly page count by domain type, multiply by the credit or GB cost, and compare that against the annual-discounted tier. Two minutes of arithmetic prevents most overspending in this market. When you're ready to test, 👉 [open ScraperAPI's pricing page and start the 5,000-credit trial](https://www.scraperapi.com/pricing/?fp_ref=coupons).
