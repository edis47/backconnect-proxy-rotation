# backconnect proxy: How a Single Entry Node Rotates Your IPs, What It Costs, and How to Set One Up Without Managing a List

If you've ever watched a scraper run perfectly for the first fifty requests and then start returning CAPTCHAs, you already understand the problem a backconnect proxy solves. The issue isn't your code. It's that the site noticed the same IP address coming back over and over, and one IP making hundreds of requests per minute looks exactly like what it is.

A backconnect proxy swaps that single IP for a rotating pool behind one stable entry point. You connect to one host and port, and the provider decides which of its IPs your request actually leaves from. Everything else — rotation timing, geolocation, session persistence — gets controlled through connection parameters rather than by you maintaining a spreadsheet of proxies.

That's the short version. The rest of this article covers how the mechanics actually work, where the model breaks down, what the two competing pricing structures mean for your budget, and how to set one up on 9Proxy's residential network without buying more than you need.

## What a backconnect proxy is, mechanically

Providers describe it slightly differently, but the architecture is the same everywhere. A backconnect proxy is a single gateway server that routes each outbound request through a different IP from a pool, and the pool rotates automatically [1]. Instead of holding a list of static IPs and swapping them yourself, you point your scraper or browser at one endpoint and let the network assign the exit IP [4].

Compare that to a regular proxy, which hands you one IP address that stays the same for the whole session. The trade-off is straightforward: static IPs are predictable and cheap, backconnect setups are far less likely to get rate-limited but cost more because residential and mobile IPs are expensive to source [2].

Three practical consequences follow from that design:

- You don't manage anything. The provider handles when an IP gets retired and what replaces it [4].
- You get access to the full pool. Residential providers work this way because home connections drop in and out constantly — the gateway absorbs that churn so your requests don't fail with it [2].
- Configuration moves into the connection string. Country, city, session length and session identity are usually passed as parameters in the proxy username rather than in your application code [5].

That last point is what makes backconnect proxies feel different to work with, and it's also where most of the confusion in provider documentation comes from.

## What people mix it up with

Four things get described as "the same thing" in forum threads. They aren't.

| Setup | Exit IP behaviour | Who controls the IPs | Main use |
| --- | --- | --- | --- |
| Regular (static) proxy | One IP, doesn't change | You | Simple single-session tasks |
| **Backconnect / rotating proxy** | New IP per request or per session | Provider's gateway | Scraping, monitoring, automation |
| Reverse proxy | Fixed IP presented to the client | You or your host | Protecting your own servers |
| VPN | Usually one IP per session, encrypted tunnel | VPN provider | Private browsing, not scale |

A reverse proxy sits in front of a server to protect it and hands the client a stable address; a backconnect proxy does the opposite — it hides many outbound identities behind one inbound door [3]. A VPN gives you one IP and a tunnel, which is useful for privacy but useless for running thousands of requests that each need to look like a different household.

## Why "backconnect" and "rotating proxy" are usually the same product

The naming persists because it comes from how residential networks were built. When residential proxies started being sold, the IPs came from real user devices that could disconnect at any moment, so providers built a gateway that managed the pool dynamically. That gateway became known as the backconnect proxy. Providers today list it under several names — rotating proxy, dynamic proxy, IP rotating proxy — all describing the same entry-point architecture [6].

So when you see a residential provider advertising "rotating IPs," you're almost always buying access to a backconnect-style gateway. 9Proxy follows this pattern: its GB-based product generates unlimited endpoints through one gateway host assigned in your dashboard, and each endpoint either rotates on every request or holds a sticky IP for a set number of minutes [5].

The exception worth knowing about: some providers sell rotating access as a separate add-on and charge extra for the gateway. Others bundle it into every residential plan. It's worth checking which one you're buying before comparing headline per-IP prices.

## How rotation is actually controlled

This is the part that decides whether a backconnect proxy is pleasant or infuriating, and it all happens in the username field.

Most providers, 9Proxy included, embed targeting and session settings in a structured username string [5]:

text
subaccount-country-us-st-ohio-city-newyork-isp-as22773_Cox_Communications_Inc.-sst-15-ssid-device1


The pieces that matter:

| Parameter | What it does |
| --- | --- |
| `country` | Two-letter country code for the exit IP |
| `st` | State or region filter (optional) |
| `city` | City-level targeting, underscores for spaces |
| `isp` | Filter by ISP name or ASN |
| `sst` | Sticky session duration in minutes |
| `ssid` | Session identifier, so you can hold several sticky IPs under one config |

Omit `sst` and `ssid` entirely and every request exits a different IP. Add `sst-15` and you hold one IP for fifteen minutes, which is what you want when a workflow depends on staying logged into an account or keeping a cart session alive. Add an `ssid` on top and you can run parallel sticky sessions from the same base configuration — each unique session ID gets its own IP [5].

The documented request format is a normal proxy call:

bash
curl -x your_proxy_host:your_port \
     -U "subuser-country-us-sst-15-ssid-bot01:yourpassword" \
     https://ipinfo.io


One thing the docs flag that people learn the hard way: over-filtering shrinks your pool. Asking for a specific state *and* city *and* ISP narrows the available IPs dramatically, and if you've over-constrained the target, requests start failing or falling back. Target by country alone for the fastest response, then add filters only when the job genuinely requires them [5].

> Rotating mode and sticky mode aren't competing features — they're two settings on the same gateway. Rotating suits scraping and price checks; sticky suits account logins and session-bound flows [5].

## The part that actually costs money: bandwidth vs IPs

Backconnect access is priced one of two ways, and picking the wrong one is the single most expensive mistake in this category.

**Per-gigabyte** pricing meters the traffic you push through the rotating pool. You generate unlimited endpoints; every request draws from the shared pool and deducts bandwidth. This is the natural fit for high-rotation work where each request moves very little data — SERP checks, ad verification, geo-testing, price polling [7].

**Per-IP** pricing charges for a fixed number of residential IPs with unlimited bandwidth. You pay for addresses, not traffic, and an IP is deducted only when you actually forward it. Unused IPs never expire, so a package bought for a project that ends early keeps its balance [8]. This model suits sustained sessions, heavy transfers, and anything where bandwidth is hard to predict [7].

9Proxy sells both, plus bundles that combine them. Worth flagging for anyone comparing prices against older blog posts: the company raised IP-based and bundle prices on June 1st, 2026, the first adjustment in its history, while explicitly leaving GB-based pricing untouched [9]. Several older comparison articles still quote the pre-adjustment IP rates, which is why you'll see inconsistent numbers floating around.

## Every current 9Proxy package

All of these are balance-based purchases rather than recurring subscriptions. You buy a package, the balance sits in your account, and nothing renews behind your back.

### Residential proxies by IP — unlimited bandwidth per IP

| Package | Total price | Effective per IP | Purchase |
| --- | --- | --- | --- |
| 100 IPs | $24 | $0.24 | [ Grab the 100 IP starter package](https://bit.ly/9-Proxy) |
| 500 IPs | $72 | $0.144 | [ Get 500 residential IPs](https://bit.ly/9-Proxy) |
| 1,000 IPs + 500 bonus (1,500 total) | $126 | $0.084 | [ Buy the 1,000 IP package](https://bit.ly/9-Proxy) |
| 2,500 IPs | $210 | $0.084 | [ Get 2,500 IPs](https://bit.ly/9-Proxy) |
| 5,000 IPs | $360 | $0.072 | [ Order 5,000 IPs](https://bit.ly/9-Proxy) |
| 15,000 IPs | $720 | $0.048 | [ Get the 15,000 IP package](https://bit.ly/9-Proxy) |
| 25,000 IPs | $863 | $0.035 | [ Buy 25,000 IPs](https://bit.ly/9-Proxy) |
| 50,000 IPs | $1,438 | $0.029 | [ Get the 50,000 IP package](https://bit.ly/9-Proxy) |

### Business IP packages

| Package | Total price | Effective per IP | Purchase |
| --- | --- | --- | --- |
| 100,000 IPs | $2,300 | $0.023 | [ Request the 100,000 IP business package](https://bit.ly/9-Proxy) |
| 200,000 IPs | $4,140 | $0.021 | [ Request the 200,000 IP business package](https://bit.ly/9-Proxy) |
| 500,000 IPs | $8,625 | $0.018 | [ Request the 500,000 IP business package](https://bit.ly/9-Proxy) |

### Residential proxies by GB — 180-day validity

| Package | Total price | Effective per GB | Purchase |
| --- | --- | --- | --- |
| 5 GB | $15 | $3.00 | [ Buy a 5 GB bandwidth package](https://bit.ly/9-Proxy) |
| 50 GB + 5 GB bonus | $105 | $2.10 | [ Buy the 50 GB package](https://bit.ly/9-Proxy) |
| 100 GB | $150 | $1.50 | [ Buy 100 GB of rotating traffic](https://bit.ly/9-Proxy) |
| 200 GB | $200 | $1.00 | [ Buy 200 GB of rotating traffic](https://bit.ly/9-Proxy) |
| 1,000 GB | $800 | $0.80 | [ Buy the 1,000 GB package](https://bit.ly/9-Proxy) |
| 2,000 GB | $1,500 | $0.75 | [ Buy the 2,000 GB package](https://bit.ly/9-Proxy) |

### Enterprise GB packages — no expiry

| Package | Total price | Effective per GB | Purchase |
| --- | --- | --- | --- |
| 3,000 GB | $2,160 | $0.72 | [ Request the 3,000 GB enterprise package](https://bit.ly/9-Proxy) |
| 6,000 GB | $4,200 | $0.70 | [ Request the 6,000 GB enterprise package](https://bit.ly/9-Proxy) |
| 10,000 GB | $6,800 | $0.68 | [ Request the 10,000 GB enterprise package](https://bit.ly/9-Proxy) |

### Bundle packages — IPs plus bandwidth

| Bundle | Contents | Total price | Purchase |
| --- | --- | --- | --- |
| Starter | 100 IPs + 5 GB | $30 | [ Get the Starter bundle](https://bit.ly/9-Proxy) |
| Popular | 1,500 IPs + 50 GB | $180 | [ Get the Popular bundle](https://bit.ly/9-Proxy) |
| Pro | 5,000 IPs + 500 GB | $720 | [ Get the Pro bundle](https://bit.ly/9-Proxy) |

## Which one you should actually buy

Ignore the per-unit headline for a second and think about what your job consumes.

If your workflow is "hit a thousand URLs, read a small JSON response from each, never log in," you want GB-based pricing. A 50 GB package at $105 goes a long way when each request moves a few kilobytes, and you never have to think about IP count [7].

If your workflow is "keep forty browser profiles alive all day, each on its own clean IP, each logged into a different account," per-IP is the only structure that makes sense. You're buying addresses with unlimited traffic, and the 180-day clock that applies to GB packages doesn't apply here — unused IPs sit in your balance indefinitely [8].

The bundles sit in between and are the right call when a project has both shapes of work in it. The Starter bundle at $30 is the cheapest way to test whether the rotation behaves the way your tooling expects before committing to a larger balance, and bundled traffic stays valid for 180 days.

Two pricing details that affect the maths more than they look like they should: the 1,000 IP package actually delivers 1,500 IPs, which drops the effective rate to $0.084 — the same as the 2,500 tier. And GB pricing falls steeply from $3.00/GB at 5 GB to $0.68/GB at the 10,000 GB enterprise tier, so small bandwidth packages are proportionally much more expensive per gigabyte.

New users who want to test before paying can ask support about trial availability — 9Proxy runs limited trials depending on availability and asks you to specify whether you want IP-based or GB-based access [10]. Payment options include credit cards, crypto, Alipay, Google Pay and Apple Pay; some listings also mention a bonus on crypto payments, so it's worth confirming current terms at checkout rather than assuming.

## Where backconnect proxies earn their money

The use cases are narrower than the marketing suggests, but the ones that work, work well.

**Web scraping and data collection.** This is the primary use case. Rotating IPs defeat the two blocking techniques that break scrapers: IP blocking, which blacklists datacenter ranges, and rate blocking, which limits requests per IP per minute [11]. Rotate before either threshold triggers and neither rule fires.

**Price and SERP monitoring.** Repeated checks from one IP get flagged fast, and a competitor watching your requests can serve you different prices or block you outright. Distributing checks across many residential IPs makes the traffic look like ordinary visitors [2].

**Ad verification.** Confirming that an ad renders correctly in a specific country generally requires a real IP in that country, and confirming it across many regions requires many IPs [3].

**Multi-account work.** Platforms limit how many accounts can operate from one IP. Spreading activity across residential addresses avoids tripping those limits, which is why anti-detect browser setups and backconnect proxies get used together [2].

**SEO and geo-specific checks.** Country, city and ISP-level targeting is what makes a backconnect gateway useful for looking at a search result the way a real user in Ohio or Hanoi would see it [5].

## The honest downsides

Shared bandwidth is the big one. Backconnect proxies route through a pool shared with other users, so throughput depends on how many people are hammering the same nodes at the same time [2]. Residential exits also introduce latency compared to datacenter proxies, and the further the exit is from your target, the more noticeable it gets [11].

Residential IPs expire by nature — a real home connection can drop. With 9Proxy's per-IP model, an IP typically stays online for a few hours up to around 24 hours, and the app's auto-refresh setting replaces dead ones with fresh addresses automatically [8]. Budget for that churn instead of assuming a fixed lifespan.

Application compatibility is another real constraint. Anything that speaks HTTP, HTTPS or SOCKS5 works. Software that insists on a fixed IP whitelist without username/password support won't. And the per-IP product requires 9Proxy's desktop app for local port forwarding, whereas the GB-based product works straight from the dashboard with credentials or an IP whitelist [8]. If you're running on a headless server and don't want to install a desktop app, that distinction matters before you pay, not after.

## Setting it up, start to finish

1. Create an account and pick a package size that matches your workload shape (bandwidth-heavy vs IP-heavy).
2. For GB-based access, open the proxy generator in the dashboard and create a sub-user — this is the username and password pair your requests will authenticate with.
3. Copy the gateway host and port shown in your dashboard. The host is fixed; you don't need to source it yourself [5].
4. Choose rotating or sticky mode. Rotating needs no extra parameters. Sticky needs `sst`, plus `ssid` if you want multiple simultaneous sticky IPs.
5. Build the username string with your targeting filters — country first, then state, city or ISP only if the job demands it [5].
6. Test with a single request against an IP-check endpoint before wiring it into production.
7. If you're on the per-IP product, install the app, filter the proxies you want, forward them to local ports and point your tools at `localhost:port` [8].

If you'd rather skip the reading and start clicking through account setup, [👉 create your 9Proxy account here](https://bit.ly/9-Proxy) and the dashboard will show the gateway details for whichever model you pick.

## Questions that come up a lot

**Is a backconnect proxy the same as a rotating proxy?** Practically yes. Both describe a single entry point that hands out a different exit IP per request or per session [6].

**Can I use rotating IPs and sticky sessions at the same time?** Yes, on the same gateway. Rotating is the default; adding a session duration switches that specific connection to sticky [5].

**Do unused IPs expire on the per-IP model?** No. IPs are deducted only when forwarded to a port, and unused balance stays in your account indefinitely [8].

**Does GB-based traffic expire?** Standard GB packages carry 180-day validity; enterprise tiers don't expire at all [8].

**What protocols does it support?** HTTP, HTTPS and SOCKS5, which covers most scraping frameworks, automation tools and anti-detect browsers.

**Will this get around paid streaming geo-blocks?** Don't count on it. Residential proxies are built for data collection and account work, and streaming platforms detect them aggressively [10].
