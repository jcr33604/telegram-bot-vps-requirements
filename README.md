# Best VPS for Telegram Bot: How Much RAM Do You Need, Are Free Hosts Enough, and Polling vs Webhook? (Plan Specs and Real Costs Included)

A Telegram bot spends most of its life doing nothing. It sits on 50–150 MB of RAM, wakes up when a message arrives, replies in milliseconds, and goes back to sleep. A 1 vCPU / 2 GB server holds that comfortably. In a plan database checked on **November 8, 2026**, machines that meet that spec start at **$3/month**, and nearly every one of those cheap entries is an annual prepayment rather than a monthly bill.

The rest of the decision comes down to three things: how big the machine really has to be, whether you can use a webhook or need to stick with long polling, and what the advertised monthly price actually costs you at checkout. SixtyNet's cheapest Los Angeles plan is $200/year, which is a very different purchase from a $3/month VPS, so it's worth knowing exactly where that money goes before you decide.

## The short version

Most people running one or two bots with no database need the smallest machine available. If you want to see what SixtyNet currently sells for Telegram-style workloads, its entry LAX plan is **2 cores / 2 GB / 40 GB NVMe at $200/year** (listed at $240).

👉 [Check SixtyNet's current VPS plans and annual pricing](https://sixtynet.net/aff.php?aff=39)

This guide works out three things in order: the machine spec you actually need, why free hosting platforms drop bots, and where the price differences between providers come from.

## How much server does a Telegram bot actually need?

A bot is almost always idle. It waits for an event, replies, and waits again. What it needs from the server isn't throughput but stability — enough memory that the process doesn't get killed, and a machine that doesn't reboot underneath it. That's why bots generally live on the smallest plan a provider offers, and why spending more usually goes toward backups and monitoring instead of extra cores.

On memory, the numbers from different sources line up reasonably well:

- A typical bot uses **50–150 MB RAM** when idle, rising during peak traffic, according to OliveVPS's own product page.
- NovaVPS puts the common runtimes (aiogram, python-telegram-bot, Telegraf, grammY) at **under 200 MB** idle.
- servercalc considers **1 GB "plenty"** for a bot on webhooks with simple logic and no database. The cases where 1 GB stops being enough are image processing, large job queues, or launching a headless browser to scrape.

Where the sources disagree is the recommended floor. servercalc treats **1 vCPU and 2 GB** as the working configuration. Xenax's planning table starts at **2 vCPU / 4 GB** for a simple command-and-reply bot, though the article states plainly that those are planning examples, not official Telegram requirements or measured benchmarks. OliveVPS claims 1 GB handles 3–4 bots, which is a vendor claim from a sales page and should be read that way.

A workable rule set, built from the more conservative sources:

| What the bot does | Starting spec | Why |
| --- | --- | --- |
| Command-and-reply, no database | 1 vCPU / 1 GB | The process idles at a few hundred MB at most |
| Database, job queue, scheduled tasks | 1 vCPU / 2 GB | The database and workers compete with the bot for memory |
| Several bots, media processing, external integrations | 2 vCPU / 4 GB | CPU peaks and temporary files during uploads |

Treat that as planning guidance. A bot that only answers `/start` and looks up a config file will run on less than the table says; one that transcodes video will run on more.

## What SixtyNet is actually selling

SixtyNet runs KVM virtualisation on AMD EPYC hardware in a single Los Angeles datacenter. Its catalogue splits into two families: **LAX** plans with standard datacenter IPs and DDoS protection, and **Residential** plans that come with a residential IP instead.

Full plan list from the current pricing page:

| Plan | CPU / RAM | Storage | Port / traffic | Annual price | Order |
| --- | --- | --- | --- | --- | --- |
| LAX-STARTER | 2 cores / 2 GB | 40 GB NVMe | 2 Gbps / 5 TB | $200/yr (list $240) | [View plan](https://sixtynet.net/aff.php?aff=39) |
| LAX-MICRO | 4 cores / 4 GB | 80 GB NVMe | 4 Gbps / 10 TB | $400/yr (list $480) | [View plan](https://sixtynet.net/aff.php?aff=39) |
| LAX-MEDIUM | 8 cores / 8 GB | 160 GB NVMe | 10 Gbps / 20 TB | $800/yr (list $960) | [View plan](https://sixtynet.net/aff.php?aff=39) |
| LAX-LARGE | 8 cores / 16 GB | 320 GB NVMe | 10 Gbps / 40 TB | $1,600/yr (list $1,920) | [View plan](https://sixtynet.net/aff.php?aff=39) |
| Residential-STARTER | 2 cores / 2 GB | 40 GB NVMe | 2 Gbps / 5 TB, residential IP | $260/yr (list $300) | [View plan](https://sixtynet.net/aff.php?aff=39) |
| Residential 4 / 8 / 16-core tiers | 4–16 cores / 4–16 GB | 80–320 GB NVMe | 4–10 Gbps / 10–40 TB, residential IP | Not shown on the pricing page | [View plans](https://sixtynet.net/aff.php?aff=39) |
| Residential AS9929 (2 and 4 core) | 2–4 cores / 2–4 GB | 40–80 GB NVMe | 5–10 TB, China-optimised routing | Not shown on the pricing page | [View plans](https://sixtynet.net/aff.php?aff=39) |
| China Unicom Premium | Bandwidth product | — | 100M port, 3× burst, 95th-percentile billing | $2,500/yr (list $3,000) | [View plan](https://sixtynet.net/aff.php?aff=39) |

The single most important line in that table for a bot operator is the residential notice. SixtyNet states that **residential IP plans do not have ports 80 and 443 open**, for IP risk-control reasons. That rules out webhooks on those plans entirely. If you want Telegram to push updates to an HTTPS endpoint, you need a LAX plan.

The AS9929 residential variants carry their own warning: they reject external probes and scans by default, and access is restricted unless you configure whitelisting or port forwarding. Long polling works because the bot dials out, but anything inbound needs setup first. Refunds on those specific plans are issued as site balance rather than back to the original payment method.

Two limits worth knowing before you buy: there is one location (Los Angeles), and a third-party review notes that the money-back window is **3 days** rather than the 30 days most hosts offer. That same review flags card payments as having been suspended at one point, so confirm the available payment methods at checkout.

## Why free hosting keeps dropping your bot

The most common story in bot-hosting threads looks like this: someone runs two bots with a pile of API keys, tries to keep them online for free, and watches them die.

A Reddit post from December 2025 lays out the specifics. The author's bot on Render went offline after a day, and adding a five-minute ping didn't help. On Koyeb, the bot dropped randomly once requests picked up and couldn't handle more than about ten concurrent users. Oracle's free tier came out as the most dependable actual VM of the three, but the setup is fiddly and Oracle rejected the author's prepaid card.

The replies converged on the same point: free application hosts shut down idle processes, so they can't hold a long-running bot. A $3–5/month VPS from a budget provider beats chasing free tiers, and one commenter mentioned running small projects on $7/year machines that only went down a handful of times over a year. Those are individual experiences from late 2025, not benchmark results, but the failure mode is consistent with how idle-shutdown works on PaaS platforms.

If you've already decided to move off free hosting and want a provider with an actual annual plan behind it:

👉 [Compare SixtyNet's annual VPS options](https://sixtynet.net/aff.php?aff=39)

## What a 2 GB VPS actually costs

Prices below were verified on **November 8, 2026** and reflect entry pricing, which often means the first billing term rather than a permanent rate.

| Provider | Spec | Price | Billing note |
| --- | --- | --- | --- |
| RackNerd KVM 2 GB | 2 cores / 2 GB / 35 GB SSD | $3/mo | Billed yearly, $35.99 upfront |
| RackNerd KVM 4 GB | 3 cores / 4 GB / 60 GB SSD | $5/mo | Billed yearly |
| Ethernet Servers VPS 2 GB | 1 core / 2 GB / 80 GB SSD | $6/mo | Monthly |
| Truehost Cloud Starter | 1 core / 2 GB / 50 GB SSD | $6/mo | Billed every 36 months |
| Hostman Cloud 3 | 2 cores / 2 GB / 60 GB NVMe | $7/mo | Monthly |
| Fornex Cloud NVMe 2 | 1 core / 2 GB / 20 GB NVMe | $8.64/mo | Monthly |
| Hostman Cloud 4 | 2 cores / 4 GB / 80 GB NVMe | $10/mo | Monthly |
| Vultr Regular Performance 2 GB | 1 core / 2 GB / 55 GB SSD | $10/mo | Monthly |
| is*hosting Start | 2 cores / 2 GB / 30 GB SSD | $10.19/mo | Monthly |
| Ethernet Servers VPS 4 GB | 2 cores / 4 GB / 160 GB SSD | $12/mo | Monthly |

The $3 headline deserves a second look. RackNerd's $3/month is a yearly plan at **$35.99 paid upfront**, not a monthly bill you can cancel after four weeks. Truehost's $6 is billed every 36 months. Advertised monthly prices in the budget segment frequently assume a long prepayment, and the same specification billed month-to-month costs more.

That puts SixtyNet's LAX-STARTER at $200/year, roughly **$16.67/month equivalent**, into context. It is not the cheap end of the market. What you're paying for is a 2 Gbps port, 5 TB of monthly traffic, DDoS protection, and a Los Angeles location. If you only need a place to run a command-response bot and nothing else, a $36/year machine does the same job. If you specifically want US west coast routing, a large traffic allowance, or SixtyNet's residential IP products for other projects, the comparison changes.

👉 [See what SixtyNet's entry LAX plan includes](https://sixtynet.net/aff.php?aff=39)

## Polling or webhook? The answer depends on your ports

This isn't a performance preference. It's a configuration question, and the first thing to check is whether you have a domain, a public IPv4 address, and open ports 80/443.

**Long polling** has your application open a connection to Telegram and pull updates. Nothing has to be reachable from the outside, so no domain and no open inbound port are required. Xenax suggests small bots start here because it's the simpler path.

**Webhooks** reverse the direction: Telegram posts updates to an HTTPS endpoint you control. That requires a public IP, a domain pointed at it, a valid TLS certificate, and ports 80/443 open for issuance and traffic. Xenax's wording is worth repeating — follow Telegram's current requirements for webhook URLs, certificates and supported ports rather than assuming any public port will work.

This is where SixtyNet's residential plans fall out of consideration for webhook bots. Ports 80 and 443 are closed on those, by the provider's own statement, so a webhook cannot be reached. Polling still works on them because the connection goes outbound.

## Keeping it running: systemd and the security basics

Uptime comes from process management, not machine size. A bot on a 2 GB server with no restart policy will stay down after a crash just as reliably as one on a 16 GB server.

Set up a systemd unit with automatic start on boot, restart-on-failure, and logging. That handles crashes and reboots. It does not fix a bad token, broken code, or an outage at Telegram's end, and it's worth being clear with yourself about which of those you're looking at when the bot goes quiet.

The rest is a short checklist that applies to any bot host:

- Keep the bot token in a protected environment file, never in a repository or in log output. If it leaks, revoke it in @BotFather and issue a new one.
- Run the bot under a dedicated user account rather than root.
- Open only the ports you need — SSH, plus HTTPS if you're using webhooks.
- Treat message text, uploaded files, and callback data as untrusted input.
- For webhooks, validate Telegram's secret token header instead of accepting any request that arrives.
- Snapshot the server before upgrading dependencies.
- Store user settings and task state in a database or files, not in memory. A small bot can keep SQLite on the same box; move to Postgres or Redis when it starts to strain.

## Don't fight Telegram's rate limits

Telegram throttles per bot, not per IP. OliveVPS states this directly on its Telegram hosting page: respect the **30 messages per second per chat** and **20 messages per minute per group** limits and the bot runs fine. That's a vendor page, but the limits themselves match Telegram's documented behaviour.

The account-level rules matter more than the throughput numbers. Spam bots, mass-DM tools, and unsolicited broadcast operations get bots and accounts stopped, and that applies whether you're on a $3 VPS or a $200/year one. MTProto userbots are permitted as long as they follow Telegram's terms; bulk account registration, spam DMs, and scraping that violates those terms are not.

## FAQ

**Is 1 GB of RAM enough for one Telegram bot?**
For a bot on webhooks with simple logic and no database, yes. servercalc calls 1 GB plenty for that case. Add a database, a queue, or image handling and 1 GB starts to bind.

**Do I need a dedicated IPv4 address?**
Not for polling. For webhooks you need a public IP that Telegram can reach over HTTPS, and a domain pointed at it. NovaVPS markets dedicated IPv4 as protection against shared-neighbour throttling, but that's a sales argument rather than a documented Telegram behaviour.

**Can I run a Telegram bot and a Discord bot on the same VPS?**
Yes, if the combined workload fits. Keep them as separate systemd services with separate credentials and separate logs, so a crash or update in one doesn't take the other down.

**Will the bot come back after a server reboot?**
Only if you've configured it to. A systemd unit with automatic start on boot handles this. Without it, the bot stays down until you log in and start it manually.

**Can a free tier run a bot long-term?**
Free application platforms shut down idle processes, which is exactly what a bot looks like between messages. Oracle's free VM tier is the closest thing to a real free option, but users report a fiddly setup and payment verification problems. A cheap paid VPS is the more predictable route.

## How to choose

- **One bot, command-and-reply, no database** → 1 vCPU / 1 GB is enough. Buy the cheapest annual plan you can find and spend the savings on backups.
- **A bot with a database, scheduled jobs, or a few hundred daily users** → 1 vCPU / 2 GB. This is the configuration most bots actually land on.
- **Multiple bots, media processing, or browser automation** → 2 vCPU / 4 GB, and watch memory during uploads.
- **You want a US west coast location, a large traffic allowance, or SixtyNet's residential IP products for other work** → SixtyNet's LAX-STARTER at $200/year covers the bot case, and the residential plans are there if you need a residential IP for something else entirely — but remember ports 80/443 are closed on those, so webhooks are out.

Before you pay, confirm four things: the billing cycle (SixtyNet's plans are priced annually), the refund window (a third-party review reports 3 days), whether your payment method is currently accepted, and whether the plan you're choosing has ports 80/443 open if you intend to use webhooks.

👉 [Open SixtyNet's pricing page and confirm the current billing terms](https://sixtynet.net/aff.php?aff=39)
