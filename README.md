# ScraperAPI JavaScript Rendering: How Do You Turn It On, What Do 10/25/75 Credits Actually Cost, and Which Plan Do You Need? (Full Plan Table + Where It Fails)

Add `render=true` to a request and ScraperAPI sends it through a Chromium instance, executes the page's JavaScript, and returns the rendered HTML instead of the empty shell. That request costs **10 API credits**, **25** if you pair rendering with premium proxies, and **75** with ultra premium proxies. Those three numbers decide most of your budget, and they are where people get surprised.

What follows covers the switch itself, how the credit math actually stacks against per-domain multipliers, the concurrency ceiling you'll hit at scale, the selector parameter that silently does nothing if you forget `render=true`, and the targets where rendering still returns nothing at all.

> **Quick pick:** most developers reading this run one scraper and need a page to render. Hobby at **$49/month** gives 100,000 credits and 20 concurrent threads, which is roughly 9,000 rendered requests on a plain HTML target once the base credit is counted. Confirm your target renders at all on the free tier first.
> 👉 [See what the Hobby plan includes](https://www.scraperapi.com/?fp_ref=coupons)

## All ScraperAPI plans at a glance

Annual billing takes 10% off automatically at checkout, no code needed. The prices below are monthly rates; the annual column shows the effective monthly cost when you pay for a year.

| Plan | Monthly | Annual (per month) | Credits/month | Threads | Sign up |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 (+5,000 for the first 7 days) | 5 | [Start free](https://www.scraperapi.com/?fp_ref=coupons) |
| Hobby | $49 | $44.10 | 100,000 | 20 | [Get Hobby](https://www.scraperapi.com/?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [Get Startup](https://www.scraperapi.com/?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [Get Business](https://www.scraperapi.com/?fp_ref=coupons) |
| Scaling | $475 | $427.50 | 5,000,000 | 200 | [Get Scaling](https://www.scraperapi.com/?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [Get Professional](https://www.scraperapi.com/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [Get Advanced](https://www.scraperapi.com/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Talk to sales](https://www.scraperapi.com/?fp_ref=coupons) |

Two parameters split the tiers beyond raw credit volume:

| Plan | Proxy geotargeting | Pay-as-you-go overflow |
| --- | --- | --- |
| Free | None | No |
| Hobby, Startup | US and EU only | No |
| Business | Global, 50+ countries | No |
| Scaling and above | Global | Yes |

If you need localized pricing from Southeast Asia, Latin America or Australia, Hobby and Startup cannot do it. On Hobby, Startup and Business, running out of credits mid-month means waiting for renewal; only Scaling upward lets you keep spending past the allowance. Unused credits expire at renewal and do not roll over.

## Turning rendering on takes one parameter

Your existing request keeps its URL and API key. You append `render=true` to the query string, so the parameters you already pass (`country_code`, `session_number`, `premium=true`) stay in place.

Rendering adds latency and can lower your success rate, which is why ScraperAPI recommends using it only where the target actually needs it. Static pages that already return full HTML should skip it. Running every request through Chromium doubles your cost and slows the pipeline for no gain.

## The real credit cost: 10, 25, 75, and why combining features costs more than adding them

ScraperAPI's documentation lists three prices for a rendered request:

- **10 credits** for JS rendering on its own
- **25 credits** for rendering with premium proxies
- **75 credits** for rendering with ultra premium proxies

The combination prices are the part worth re-reading. Premium proxies and rendering together cost 25, not 20. Ultra premium plus rendering costs 75, not 40. Features combine at a discount to neither; the pair is priced as its own tier.

A second layer sits underneath: per-domain base costs, applied automatically without you selecting anything. Standard sites bill at 1 credit, Amazon, eBay and Walmart at 5, Google and Bing at 25, LinkedIn at 30. A rendered Google query therefore bills at the search-engine rate plus rendering, not the flat 10. Third-party cost write-ups put that at 35 credits per request, which turns a 100,000-credit Hobby allowance into roughly 2,850 searches. Run that arithmetic before you size a plan around a target with a multiplier.

There is a small ambiguity in the published wording. The docs say a JS-rendered request "cost 10 API Credits," which reads as a total; third-party breakdowns read it as base + 10, so 11 on a standard page. The gap is one credit per request, and on a 100,000-credit plan that is about 9% of your volume. Check the credit counter in the dashboard after your first hundred rendered calls and you will know which reading applies to your traffic.

Three things that cost nothing extra: `wait_for_selector`, `country_code`, `session_number`, `device_type`, `output_format` and `keep_headers=true` all bill at the base rate. Failed requests do not burn credits either; only 200 and 404 responses are charged.

👉 [Check current plan credits before you commit](https://www.scraperapi.com/?fp_ref=coupons)

## `wait_for_selector` does nothing without `render=true`

The API does not render everything on a page by default, since loading images and tracking scripts would slow every request. That means the element you want can still be missing after a successful render, usually because it sits below a lazy-loaded list or behind a chart that needs another second or two. `wait_for_selector=x` tells the API to hold the response until that selector appears.

The catch is flagged in a danger callout in the docs: if you specify `wait_for_selector` without `render=true`, **the API ignores it**. No error, no warning, just the same incomplete HTML you were getting before. If your selector-based wait appears to have no effect, check that both parameters are present in the same request.

## Concurrency: 10 rendered requests per second, and what that means in flight

The rendering burst limit is **10 requests per second** by default. That figure caps how many rendered requests you can start each second, not how many can be open at once. ScraperAPI's own example: if each rendered request takes 25 seconds and you send 10 per second steadily, you can have about **250 rendered requests running concurrently**.

The thread counts in the plan table (20, 50, 100 and up) and this burst limit are separate constraints. A heavy rendering workload can hit the 10/sec ceiling long before it exhausts a 200-thread allowance. Raising the rendering burst limit is an Enterprise arrangement handled through support; there is no self-serve setting for it.

## The rendering instruction set, and its limits

Rendering alone gets you a page after its JavaScript runs. Many targets need a click, a scroll or a form submission before the data exists. The instruction set handles that inside the same request:

| Instruction | What it does |
| --- | --- |
| `click` | Clicks an element by CSS, XPath or text selector |
| `input` | Types a value into a field |
| `scroll` | Scrolls vertically or horizontally, by pixels or to top/bottom |
| `loop` | Repeats a block of instructions a set number of times, useful for infinite scroll |
| `wait` | Pauses for a number of seconds |
| `wait_for_event` | Waits for `networkidle`, `load`, `domcontentloaded` or `stabilize` |
| `wait_for_selector` | Waits for a matching DOM element to appear |

Practical limits from the documentation: keep chains to **3 to 4 actions per request**, because each action adds latency and raises the chance of a timeout. `loop` cannot be nested inside another `loop`, though it can contain the other instruction types. `wait_for_event` with `networkidle` is the right tool after a form submission, when you need the resulting network requests to settle before reading the page. Custom headers should carry an `x-sapi-` prefix so they don't collide with headers the target site already uses.

👉 [Try these against your own target on a free trial](https://www.scraperapi.com/?fp_ref=coupons)

## Where rendering won't save you

Independent benchmarks of ScraperAPI's success rates draw a sharp line, and it's worth checking your target list against it before paying.

Reliable: Zillow around 100%, Etsy around 99%, Amazon around 98% with structured-data endpoints that return parsed JSON, Walmart around 93%, LinkedIn around 95% (at 30 base credits per request, the cost adds up fast).

Effectively unsupported: Instagram, Twitter/X and Booking.com returned **0% success** in that testing. Realtor.com managed roughly 12%. No amount of `render=true` or instruction chaining fixes a target the proxy and anti-bot layer cannot get through.

Two structural limits matter just as much. ScraperAPI does not support scraping behind login walls; `session_number` keeps the same IP across requests, but it will not automate a form login or clear 2FA. And difficult targets carry a forced 10-minute cache, so anything you pull from them can be up to ten minutes stale. Live pricing and inventory tracking on those domains will show lag you cannot tune away.

## Which plan to pick

Free covers a proof of concept: 1,000 credits a month plus 5,000 for the first week is enough to confirm your target renders and to watch what a rendered request actually deducts. Hobby at $49 handles a personal project or a product prototype with steady volume on plain HTML targets; run the domain-multiplier math before pointing it at Google or LinkedIn. Startup at $149 with a million credits and 50 threads fits a small team scraping daily, as long as US and EU proxies are enough.

Business at $299 is the first tier with global geotargeting, which is the real reason to move up rather than the credit jump alone. Scaling at $475 adds pay-as-you-go overflow so a busy month doesn't cut you off. Professional (10.5M credits) and Advanced (21.5M credits) arrived in May 2026 for teams past the standard tiers; most individual readers will never touch either.

## Before you pay

Two checks save more money than any plan comparison. First, run your actual target on the free trial and read the credit counter in the dashboard afterward instead of estimating from the headline numbers. Second, confirm whether you need proxies outside the US and EU; if you do, Hobby and Startup are off the table regardless of how many credits you buy. ScraperAPI doesn't email credit usage alerts, so a target with an unexpected multiplier can burn an allowance quietly.

If your target renders and your geography is US or EU, Hobby or Startup is the whole decision. If either answer changes, so does the tier.

👉 [Start the free trial and check your real credit burn](https://www.scraperapi.com/?fp_ref=coupons)
