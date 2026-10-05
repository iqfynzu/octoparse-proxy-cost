# Octoparse Proxy: Set Up Your Own Residential Proxies, Work Around the Login Limitation, and Pay $1/GB Instead of $3/GB

Octoparse works fine on friendly sites. Then you point it at a marketplace, a SERP, or anything with Cloudflare in front of it, and the run comes back with empty fields, timeouts, or a "blocked" message. That's usually not a selector problem. It's your IP.

Sooner or later every Octoparse user ends up researching proxies, and the research usually ends in the same place: Octoparse sells its own proxies at roughly $3/GB, and third-party providers sell residential traffic for a lot less. The catch is buried in Octoparse's help documentation, and it's the reason a lot of people buy the wrong plan: Octoparse's custom proxy field doesn't accept usernames and passwords. If you paste `user:pass@host:port` in there, nothing happens.

This guide covers both routes, the exact settings, and the workaround that makes cheaper proxies usable in Octoparse.

## What you're actually choosing between

There are two ways to get proxies into Octoparse, and they behave differently in ways that matter more than the price.

|  | Octoparse built-in proxies | Your own proxies |
| --- | --- | --- |
| Works on local runs | Yes | Yes |
| Works on Cloud runs | Yes | No |
| Price | ~$3/GB | Set by your provider |
| Proxy type | Residential, auto-rotating | Depends on provider (HTTP only) |
| Authentication | Handled inside Octoparse | IP whitelisting only |
| Country/region selection | Built into the UI | Not available in the proxy field |
| Setup effort | Two clicks | Whitelist your IP, then paste host:port |

Two lines in that table decide most people's setup. Cloud runs only accept Octoparse's built-in proxies, and custom proxies only work when the task runs on your own machine. If your workflow depends on scheduled cloud execution, you're on the $3/GB path whether you like it or not.

> Octoparse's help center states plainly that built-in proxy usage is billed separately from your plan, credits can't be refunded, and there is no unlimited proxy subscription.

## The built-in route, in numbers

Worth understanding before you decide it's expensive, because it does buy convenience.

- **$3 per GB**, billed against your Octoparse account credits
- You need **at least $3 in credits** to enable IP proxies at all
- Traffic is measured by page loading, and Octoparse estimates **1 GB covers roughly 500 pages**
- Credits are non-refundable
- The proxies are residential IPs, which is why they survive better than datacenter IPs
- You pick either Default (random countries) or a specific country/region, plus a rotation interval

Enable it per task: open the task, go to Task Settings, then Anti-blocking, tick "Access website via proxies", choose "Use Octoparse proxies", pick a country, set the rotation interval, and save. It's genuinely two minutes of work, and it works on both local and cloud runs.

At Octoparse's own estimate, $3/GB works out to about **$0.006 per page loaded**. That's the number to compare against when you look at outside providers.

## The custom proxy route, and the limitation nobody reads first

If you'd rather supply your own traffic, the setup path is:

1. Open your task
2. **Task Settings → Anti-blocking**
3. Tick **Access website via proxies**, then select **Use my own proxies**
4. Click **Configure**
5. Enter each proxy as `host:port`

And that's where it stops. Octoparse accepts HTTP proxies in a plain `IP:port` format. Its documentation is explicit that proxies requiring a username and password aren't supported, and cloud runs ignore your custom proxies entirely.

There's a second, less obvious place this matters: if your target site blocks your own IP while you're still building the task, you can enable a proxy from the Proxy button in the upper right. That panel lets you use Octoparse's proxies or your own, and you can tick "Use the same proxies for task runs" if you want those settings applied at runtime too.

### Why the host:port format breaks most providers

Almost every residential proxy network authenticates with credentials: a login, a password, and a gateway host, usually written as `login:password@gw.provider.com:823`. Country and session targeting often ride inside that username string.

Feed that to Octoparse and you get nothing, because there's no field for it.

### The workaround: IP whitelisting

IP whitelisting flips authentication from "who are you" to "where are you connecting from". You register the public IP of the machine running Octoparse in your provider's dashboard, and the gateway then accepts connections from that address with no credentials at all. Suddenly the endpoint is just `gateway-host:port`, which is exactly what Octoparse's field expects.

The trade-offs are real, so factor them in:

- **Your public IP has to stay put.** Residential broadband with a dynamic IP will break the whitelist every time it changes. Office or VPS connections are better suited.
- **Session and geo strings lose their home.** Anything you'd normally express in the username has nowhere to go in Octoparse's proxy field. If per-request country targeting inside Octoparse is a hard requirement, the built-in proxies are the route that supports it. Check with your provider's support what a whitelisted endpoint falls back to on targeting before you build a workflow around it.
- **One whitelisted IP per machine.** Running Octoparse on a laptop and a server means registering both, subject to your account's limit.

## Setting up DataImpulse as your Octoparse proxy

DataImpulse is the cheapest clean option for this specific job, mainly because of one feature that most low-cost providers bury: it supports IP whitelisting alongside username/password auth. Its residential traffic is $1/GB, pay-as-you-go, with a $5 minimum purchase and no subscription. For a local Octoparse workflow, that's the configuration that clicks into place.

The steps:

**1. Create the account and add the entry package.** The residential entry package is $5 for 5 GB. Traffic doesn't expire, so an unused balance sits there until you need it.

**2. Whitelist your machine.** Find your public IP first (your router's WAN address, not your local 192.168.x.x), then add it in the DataImpulse dashboard under the authentication settings. DataImpulse documents both username/password and IP whitelisting as supported methods.

**3. Create a residential endpoint and note the gateway.** Residential runs through `gw.dataimpulse.com` on port `823`.

**4. Paste it into Octoparse.** Task Settings → Anti-blocking → Use my own proxies → Configure → `gw.dataimpulse.com:823`. No credentials, because you're authenticated by IP.

**5. Set the switch interval.** This controls how often Octoparse changes exit IP. Aggressive switching suits list scraping; multi-step flows that need to stay logged in or hold a session prefer longer intervals. Third-party integration guides commonly use 1 for rotating sessions and around 600 for sticky ones, but tune it against your actual target.

**6. Run locally to test, then scale.** Custom proxies don't apply to cloud runs, so if Octoparse's cloud is part of your workflow, you'll be running two configurations in parallel.

Getting the account and whitelist done takes longer than the Octoparse side: 👉 [👉 Grab the $5 / 5GB residential entry package and whitelist your IP](https://bit.ly/dataimPulse)

## All current packages, side by side

DataImpulse prices by traffic used rather than by monthly seat, so "plans" here means proxy type plus volume tier. All four types are pay-as-you-go, all traffic is non-expiring, and the $5 minimum purchase applies to each.

| Proxy type | Entry package | Volume tiers | Price per GB | Good fit in Octoparse | Buy |
| --- | --- | --- | --- | --- | --- |
| Residential | $5 / 5 GB | $800 / 1 TB; $0.70/GB at 5 TB | $1/GB (from $0.80/GB at 1 TB) | Protected targets: marketplaces, SERPs, listings behind anti-bot | [ Start with residential at $1/GB](https://bit.ly/dataimPulse) |
| Datacenter | $5 / 10 GB | $50 / 100 GB; $450 / 1 TB; custom from $2,250 at 5 TB+ | $0.50/GB (from $0.45/GB at 1 TB) | Unprotected sites, high-speed local runs, large page counts | [ Compare the $0.50/GB datacenter option](https://bit.ly/dataimPulse) |
| Mobile (4G/5G/LTE) | $5 / 2.5 GB | $50 / 25 GB; $1,600 / 1 TB; custom from $8,000 at 5 TB+ | $2/GB (from $1.60/GB at 1 TB) | Targets that only serve mobile versions; app-adjacent data | [ Check the $2/GB mobile package](https://bit.ly/dataimPulse) |
| Premium residential | $5 / 1 GB | $50 / 10 GB; custom from $20,000 at 5 TB+ | $5/GB | High-trust pools, dedicated account manager, all targeting included | [ See the premium residential pool](https://dataimpulse.com/premium-residential-proxies/?aff=86938) |

A few notes that don't fit in the cells. Country-level targeting is included in the base price across the residential tiers; city, ZIP, and ASN targeting are paid add-ons, and one third-party pricing breakdown reports them billed at a multiple of the standard rate on residential plans. The datacenter offering is the one to reach for when your target doesn't fight back, because $0.50/GB halves your bill for the same volume. Mobile traffic exists for the narrow set of cases where a cellular IP is the difference between data and no data.

DataImpulse's own materials describe a first-party pool of 90M+ IPs across 195 countries, HTTP/HTTPS and SOCKS5 support, rotating and sticky sessions, and a published 99.51% success rate, with a 4.8/5 rating on G2. Third-party reviews consistently land on the same headline: the $1/GB residential rate is at the bottom of the market, comfortably under the $3–8/GB range most providers charge.

There's no free trial anywhere in that table, by the way. Access starts at the $5 purchase. Independent reviews note intro packages come with a 7-day money-back option on card payments provided you've used less than 80% of the traffic, which is worth confirming at checkout rather than assuming.

## The cost math with Octoparse's own estimate

Octoparse's help center figures 1 GB gets you roughly 500 pages. Run the numbers on a 100 GB month:

|  | Octoparse built-in | DataImpulse residential |
| --- | --- | --- |
| Price per GB | $3 | $1 |
| 100 GB cost | $300 | $100 |
| Estimated pages | ~50,000 | ~50,000 |
| Cost per page | ~$0.006 | ~$0.002 |

Same traffic, $200 apart. For a hobby run it's noise. For anyone scraping tens of thousands of pages a month it's the entire budget conversation.

Two caveats before you treat those figures as gospel. Page weight varies wildly, so a heavy image gallery burns far more traffic per page than a text listing, and blocked requests still consume traffic. Octoparse's own note that "the proxies might not work for all web pages" and that users should top up credits to test first applies equally to any outside provider. Budget with a test run, not with arithmetic.

## Picking the right proxy type for the job

The mistake to avoid is defaulting to residential for everything because it sounds safest.

**Datacenter at $0.50/GB** handles static sites, government portals, company registries, and anything without meaningful bot protection. It's fast and it's cheap. There's no reason to route a Wikipedia-shaped target through residential traffic.

**Residential at $1/GB** is for sites that actually check who's asking: e-commerce listings, review platforms, search results, travel pricing. Real consumer IPs get through where datacenter ranges get filtered on reputation.

**Mobile at $2/GB** is the specialist tier. Use it when a target serves different content to cellular visitors, or when residential IPs are getting flagged and you need one more step up in trust. It costs double, so use it selectively rather than as a default.

**Premium residential at $5/GB** is aimed at teams that need consistent high-trust traffic, all targeting options included, and a named account contact. If a single bad run costs more than the traffic does, that's the tier the pricing is structured around.

## Errors you'll meet, and what they mean

Proxy failures in Octoparse usually come back as vague timeouts, but the provider's logs are more specific. Common ones and their fixes:

- **407 with a traffic message** — your balance is empty. Top up, then retry.
- **407 with a thread message** — too many concurrent connections open. DataImpulse's own tooling documentation cites a ceiling around 2,000 active connections per account, so dial down parallel tasks.
- **403 from the destination** — the site blocked that exit IP. Rotate to another country or start a new session. Don't hammer it with retries; that's how a soft block becomes a hard ban.
- **503 / no matching IP** — targeting too narrow. Drop the city filter and keep the country.
- **Every request fails instantly** — the most common cause is a public IP that changed after you whitelisted it. Check that first.

One more: credentials containing unusual characters can produce odd socket errors in some HTTP clients when they're encoded for proxy authentication. If you're hitting confusing failures, switching to IP whitelisting removes the credential path entirely, which is the same move Octoparse forces on you anyway.

## Cloud or local: pick before you buy

This is the decision that determines whether any of the above saves you money.

If your Octoparse workload runs on your own machine — a desktop, a VPS, a spare laptop that stays on — then buying your own residential traffic is straightforwardly cheaper, and DataImpulse's whitelist support is what makes it work at all. You get $1/GB instead of $3/GB, an extra proxy type or two to choose from, and a balance that doesn't evaporate at the end of the month.

If your workload depends on Octoparse's Cloud for scheduling or for running when your machine is off, your custom proxies won't be used, no matter how well you've configured them. The practical answer for most teams is a hybrid: cloud runs on built-in proxies for small or overnight jobs, local runs on your own traffic for the heavy extraction. Budget accordingly and don't buy 500 GB of residential traffic expecting it to serve your cloud tasks.

If you're starting from zero, the entry packages are cheap enough to test both paths in an afternoon: 👉 [👉 Set up your DataImpulse account with $5 of residential traffic](https://bit.ly/dataimPulse)

## Quick answers

**Does Octoparse support SOCKS5 proxies?** Not through its custom proxy field. It accepts HTTP proxies in `host:port` format. DataImpulse supports SOCKS5, but you'd use that with other tools rather than this integration.

**Can I use a proxy just to log in to Octoparse?** Yes. The login screen has its own proxy settings, useful when a corporate network blocks external requests. That proxy only applies during login and isn't used for task editing or runs.

**Can I use my own proxy while building a task?** Yes, via the Proxy button in the upper right. You can also tick "Use the same proxies for task runs" to carry those settings into the run.

**Does Octoparse offer unlimited proxies?** No. Built-in proxy usage is metered and billed separately, with no unlimited subscription.

**What's the cheapest way to test?** Octoparse's built-in option needs $3 in credits; DataImpulse's minimum is $5. Either is a small enough spend to see whether your target cooperates before you commit to a volume tier.

**Will cheap proxies get my account banned?** Providers don't control what you scrape, and neither does Octoparse. Rate, targeting, and respect for the site's terms are your responsibility, and they matter more to block rates than which provider you pay.
