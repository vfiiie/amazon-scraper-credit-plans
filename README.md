# amazon scraper api: how the credit system really works, and how to pick a plan for product, price, and review data

Most people searching for an amazon scraper api want the same three things: product data by ASIN, current prices, and reviews — in clean JSON, without maintaining proxies or fighting CAPTCHAs. The catch is that pricing pages make comparison nearly impossible, because every vendor counts usage differently. This article walks through ScraperAPI's Amazon-specific endpoints, what each plan actually costs for Amazon traffic (which is very different from what the sticker price suggests), the limits that only show up in the docs, and where a dedicated Amazon-only API beats a general-purpose one.

> **Quick pick:** for most teams scraping Amazon products and reviews, the **Business plan ($299/month, 3,000,000 credits)** is the first tier where per-request cost drops to a sensible level and you get full country-level geotargeting. Smaller projects can start free. 👉 [See current plan pricing and start the free trial](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## How ScraperAPI handles Amazon requests

ScraperAPI gives Amazon its own structured endpoints instead of asking you to parse raw HTML. The core one is the Amazon Product API: you send a GET request with your API key and an ASIN, and it returns product data transformed into usable JSON, including variant details like size and color, plus all publicly available reviews for that ASIN.

A minimal call hits the `/structured/amazon/product` endpoint with three parameters:

- `api_key` — your account key
- `asin` — the product identifier, e.g. `B07FTKQ97Q`
- `tld` — the Amazon marketplace, e.g. `com`
- `country_code` — geotargeting, e.g. `us`

The returned fields go well beyond title and price. Based on the documented response structure, you get brand, dimensions, weight, wattage and other product_information entries, feature bullets, full description, images, category ladder, best sellers rank (formatted like "#49 in Rice Cookers"), ships_from and sold_by, coupon presence, and the reviews section.

A few details worth knowing before you write code:

- **21 Amazon marketplaces are supported** through the `tld` parameter: com, co.uk, ca, de, es, fr, ie, it, co.jp, co.za, in, cn, com.sg, com.mx, ae, com.br, nl, com.au, com.tr, sa, se, and pl.
- **ASINs are market-specific.** The docs are explicit that ASINs are TLD-specific and can be pulled from the Amazon product URL. If you scrape the same product across marketplaces, you need each marketplace's ASIN.
- **Cross-country scraping needs both parameters.** If you want to scrape amazon.com from Canada (say, to get Canadian shipping information), you must pass both `tld` and `country_code`.
- **ZIP code targeting** lets you request a page as seen from a specific US ZIP code, which matters for delivery estimates and buybox details that vary by location.
- **Output format** supports JSON (default) and CSV.
- **Async version for batches.** The async structured endpoint accepts a single ASIN or an array of ASINs in one POST, then delivers results to a webhook callback. If you're monitoring hundreds of products daily, this is the version to build on.

All plans also include the general stack: JS rendering, premium proxies, rotating proxy pools, CAPTCHA and anti-bot detection, automatic retries, custom headers and sessions, unlimited bandwidth, and a 99.9% uptime guarantee.

## The full plan list — and what Amazon requests actually cost

Here is the current lineup from ScraperAPI's pricing page, with monthly and annual (10% off) pricing:

| Plan | Monthly | Annual (per month) | Credits | Concurrent requests | Buy |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Get Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [Get Startup](https://www.scraperapi.com/pricing/?fp_ref=vigpgogogo) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Get Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [Get Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Get Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Get Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Talk to sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

There's also a free plan with 1,000 credits and 5 concurrent connections, plus a 7-day trial with 5,000 credits that requires no credit card.

Now the part almost nobody explains on their pricing page: **credits are not requests.** A standard page costs 1 credit, but Amazon costs 5 credits per request, according to both the pricing page and the billing documentation. Google and Bing cost 25, LinkedIn 30, and bypassing bot protection systems like Cloudflare, Datadome, or PerimeterX adds 10 credits per scrape.

That single line changes every comparison. Divide the credit allowance by 5 and you get your real Amazon volume:

- **Hobby ($49):** 100,000 credits → about **20,000 Amazon requests**, or roughly $2.45 per 1,000 pages
- **Business ($299):** 3,000,000 credits → about **600,000 Amazon requests**, roughly $0.50 per 1,000 pages
- **Scaling ($475):** 5,000,000 credits → about **1,000,000 Amazon requests**, roughly $0.48 per 1,000 pages

So Hobby looks like the "cheap starter plan" but is actually the most expensive way to buy Amazon data on this list — about 5x the effective per-page cost of Business. If your Amazon volume exceeds ~20,000 pages a month, Hobby runs out fast, and the jump to Startup only gets you to ~200,000 Amazon requests at $149.

Before committing, verify your actual costs. Every response includes an `sa-credit-cost` header showing what that request consumed. For planning, the dashboard's Domain Multiplier looks up the credit cost of any URL, and there's a cost-check endpoint on the API host you can call directly — pass your key plus the target URL (and any parameters you plan to use), and it returns the exact cost before you run anything at scale. That's the single most useful pre-purchase step, and it costs nothing. 👉 [Check the pricing page and size your credit budget](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Limits and billing rules buried in the docs

These are the details that tend to surprise people mid-project:

**You're only billed for completed work.** ScraperAPI charges for requests returning 200 or 404, and for requests you cancel yourself before the 70-second window closes. Failed scrapes that time out or error out don't consume credits. One caveat from the billing docs: accounts with custom discounted pricing pay higher per-request credit costs on Ultra Premium domains to meet a minimum floor of $3 per 1,000 requests without rendering, $7 with rendering — only relevant if you negotiate a custom deal.

**Geotargeting is restricted on the lower two plans.** Hobby and Startup are limited to US and EU geotargeting. Country-level geotargeting anywhere starts at Business. If your use case is scraping amazon.co.jp or amazon.com.br from specific regions, Hobby and Startup can't do it properly, and that's not printed anywhere near the price.

**Concurrency limits reject rather than queue.** Exceeding your plan's concurrent thread limit returns a 429 error asking you to retry within your limit. No extra charge, but no queuing either — your scheduler needs to handle this.

**Running out of credits pauses scraping by default.** Three ways around it, depending on plan:

1. Auto-upgrade (Hobby through Professional): your account moves up to the next plan automatically when credits run out. Opt-in.
2. Pay-as-you-go (Scaling and above): keep scraping at a fixed per-credit rate up to a monthly spending cap you set, so you never get an unexpected bill.
3. Renew annual plans early for a fresh batch of credits.

If you want to test before committing any money, the trial route is generous by API standards: 👉 [Start with 5,000 free credits — no card required](https://www.scraperapi.com/signup?fp_ref=coupons)

## How this compares with Amazon-only scrapers

The main alternative category is a dedicated Amazon scraper API — services that only do Amazon and price per successful request rather than per credit. One vendor that publishes these numbers on its own site:

|  | ScraperAPI | Amazon Scraper API |
| --- | --- | --- |
| Coverage | 21 Amazon marketplaces, plus 150+ countries of general scraping infrastructure | 20 Amazon marketplaces only |
| Pricing model | Credits; Amazon costs 5 per request | Requests; only 2xx responses billed |
| Entry paid plan | Hobby, $49/mo, ~20k Amazon requests | Vibe, $19/mo, 27,000 requests |
| Bulk option | Business at $299/mo, ~600k Amazon requests | Custom from $100/mo, 200k requests, flat $0.50/1k |
| Pay-as-you-go | Available on Scaling and above | $0.90 per 1,000 successful requests, works even on the free plan, balance never expires |
| Free tier | 1,000 credits (5 concurrent) + 7-day/5,000-credit trial | 1,000 requests on signup, no card |
| Other sites | Yes — general web, SERP, e-commerce | No |

Both figures come from the vendors' own published pricing, so treat them as self-reported. But the pattern is real: for pure Amazon work at small-to-medium volume, a dedicated Amazon-only API is usually cheaper per request, because there's no credit multiplier and no cross-subsidizing of harder targets. Its cancellation terms are also clearly stated: monthly plans cancel at the end of the billing cycle without prorated refund, and unused prepaid PAYG balance is refundable within 30 days.

Where ScraperAPI wins is scope and depth. If your pipeline also touches Google Shopping, Walmart, eBay, or general product pages, you get all of it under one API key, one billing system, and one integration. Structured data endpoints, DataPipeline, and full crawler access are available on paid plans. And per the vendor's own documentation, its infrastructure spans 150+ countries with 50M+ proxies, collecting data as a local customer would see it — relevant if you need marketplace-specific pricing rather than averaged global data.

A fair summary: single-marketplace Amazon monitoring → the dedicated APIs cost less. Multi-domain scraping or enterprise-scale collection → a general API like ScraperAPI consolidates everything and the per-credit cost becomes competitive at Business tier and above.

On developer experience, the dedicated vendor ships official SDKs for Node, Python, and Go, plus a CLI and an MCP server aimed at AI agents; its async batch endpoint takes up to 1,000 ASINs or queries per call. ScraperAPI's async endpoint handles batches too, but the documented pattern there is arrays of ASINs per request without a stated cap — if batch size limits matter to your pipeline, that's worth confirming on a trial before committing. Its cancellation terms: monthly plans stop at the end of the billing cycle without prorated refund, and unused prepaid PAYG balance is refundable within 30 days.

A note on refunds: this review found no published refund policy for ScraperAPI itself — its billing docs cover credit consumption and plan changes but not money-back terms. If a refund guarantee matters to you, ask their support directly before paying. That's an honest gap worth knowing, and it's another reason the 5,000-credit trial exists: 👉 [Test the endpoints with 5,000 free trial credits](https://www.scraperapi.com/signup?fp_ref=coupons)

## Who each setup fits

Based on the verified numbers above:

- **Hobby ($49):** fits if you scrape under ~15,000 Amazon pages monthly and stick to US/EU geotargeting. Above that threshold, Startup's $149 for 10x the credits is the better buy.
- **Startup ($149):** the awkward middle tier. 200,000 Amazon requests/month is real volume, but you're still locked to US & EU geotargeting. If you need Japanese or Brazilian marketplace data, skip straight to Business.
- **Business ($299):** the sweet spot for most serious Amazon work — 600,000 requests, country-level geotargeting, unlimited analytics, and an effective cost that finally competes with Amazon-only vendors.
- **Scaling and above:** for teams past 1M Amazon requests monthly, where PAYG headroom and higher concurrency (200–500 threads) matter more than the per-unit price.
- **Enterprise:** custom concurrency and volume pricing; the site suggests contacting sales if you're above 22M requests/month.

## FAQ

**How many Amazon pages can I scrape with 100,000 credits?**
About 20,000, since Amazon costs 5 credits per request. Use the `max_cost` parameter on individual scrapes to keep any single request within a fixed budget.

**Does ScraperAPI return Amazon reviews?**
Yes. The Amazon Product API endpoint returns all publicly available reviews for an ASIN, alongside the product data and variant details, in the same JSON response.

**Which Amazon marketplaces are supported?**
21, including the US, UK, Germany, Japan, India, Australia, Brazil, UAE, Saudi Arabia, Singapore, Mexico, and others listed in the TLD parameter documentation.

**Does using geotargeting cost extra credits?**
No. The billing documentation lists `country_code`, `session_number`, `device_type`, and `output_format` among the parameters that add no cost. What does cost extra: premium proxies (+10), JS rendering (+10), and their combinations.

**Can I batch multiple ASINs in one call?**
Yes, through the async endpoint — pass an array of ASINs in a single POST and receive results via webhook callback when each job finishes.

## Before you pick a plan

Match the plan to your actual monthly Amazon page count, not to the credit number on the sticker. Under 15,000 pages: Hobby, or better, the free trial to see if the JSON fields cover what you need. 50,000–500,000 pages with multi-country needs: Business at $299 is the effective-price turning point. Past 1M monthly requests: Scaling or Professional, with PAYG enabled so a spike month doesn't pause your pipeline.

One last check before checkout: run your 10 most representative URLs through the cost-check endpoint first. If your real per-request cost lands where the docs say it should, the math above holds — and if you're ready, 👉 [pick your plan on ScraperAPI's pricing page](https://www.scraperapi.com/pricing/?fp_ref=coupons) and start with the trial credits before the first invoice.
