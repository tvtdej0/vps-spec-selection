# virtual cloud server: how to pick specs, routing, and a plan that fits what you're actually running

Searching "virtual cloud server" usually means you've outgrown shared hosting, need root access, want to deploy something specific, or have hit a wall with latency to a particular region. The term itself is a little fuzzy — providers use it interchangeably with VPS, cloud instance, cloud VM — so before comparing plans, it helps to know what you're actually buying and which specs matter for your workload.

This guide breaks down the decision into the parts that affect your bill and your uptime: CPU, RAM, storage, bandwidth, port speed, and — the part most generic comparisons skip — network routing. We'll use DMIT's cloud instance lineup as the concrete example, because it's one of the few providers that splits the same hardware across three clearly different routing tiers, which makes the trade-offs easy to see. If your use case has nothing to do with Asia-Pacific or China connectivity, the Tier 1 section is where most of the value lives.

## What a virtual cloud server actually is

A virtual cloud server is a virtualized machine running on a hypervisor (usually KVM for anything reputable) on shared physical hardware. You get your own OS install, root access, dedicated slice of CPU and RAM, and an IP address. The "cloud" part, in practice, means the instance is provisioned from a pool of physical nodes, often with distributed storage underneath, and you can snapshot, rebuild, or move it without waiting on a data center tech.

The thing that separates a real cloud instance from a cheap OpenVZ container is isolation. KVM gives you a real kernel, so you can load custom modules, run Docker without workarounds, configure iptables fully, and run VPN endpoints. OpenVZ and similar container-based virtualization share a kernel, which is why they're cheap and why they're limited. DMIT uses KVM across its entire fleet — worth noting if you've been burned by "VPS" offers that turned out to be containers.

## When you actually need one (and when you don't)

You need a virtual cloud server if any of these apply:

- You need root access to install, configure, or debug software at the OS level
- You're running a service that shared hosting blocks (VPN, proxy, game server, custom daemon, specific database)
- You need consistent latency to a specific region and shared hosting is on a congested node
- You want to run Docker, Kubernetes, or anything that needs a real kernel
- You need predictable CPU and RAM rather than "best effort on a busy box"

You probably don't need one if:

- Your site is a small WordPress install with modest traffic — managed hosting or even good shared hosting is fine
- You're not comfortable in a Linux shell and don't want to be — managed cloud (Cloudways, etc.) or PaaS (Render, Railway) costs more but saves you the sysadmin work
- You just need file storage — that's object storage, not a VM

The mistake people make is buying a cloud server because it sounds more "serious," then realizing they spend weekends patching it. If you don't want root, don't pay for root.

## How to choose specs without overbuying

**CPU (vCore).** For a single web app, a reverse proxy, or a small API, 1–2 vCores is enough. CPU matters most for compile work, video transcoding, CI runners, and anything that's actually CPU-bound. If your service is I/O-bound (most web apps), more vCores just sit idle.

**RAM.** This is the spec that bites people. 2GB is the realistic floor for a real workload — OS overhead, your app, a database, plus headroom. A LEMP stack with a small database and caching runs okay in 2GB. Below that, you're fighting swap. 4GB gives you comfortable room for a database-heavy app or multiple services.

**Storage.** SSD, not HDD — that ship has sailed. The question is size. 20GB is tight once you account for OS, logs, Docker images, and a database. 40–80GB is the practical range for most single-server setups. The other thing to check is whether it's local SSD or networked (Ceph) — local is faster for I/O, networked survives node failure better. DMIT uses Intel Datacenter SSDs in Ceph clusters, which is the resilient-but-not-blazing-fast end of that trade-off.

**Bandwidth and traffic.** Two different things. Bandwidth is the port speed (1Gbps, 4Gbps, 10Gbps); traffic is the monthly quota. Most small apps never come near their traffic quota. Where it matters: video, file hosting, CDN origins, anything proxying a lot of data. DMIT throttles rather than cuts you off when you hit the cap — bandwidth drops to roughly 100Mbps–1Gbps depending on tier. That's better than a hard shutoff, but worth knowing before you assume you have full speed forever.

**Port speed.** 1Gbps is fine for almost everything. 10Gbps matters for bursty traffic, large file transfers, or if you're running multiple services on one box and they compete. DMIT's Premium plans cap at 1Gbps on the smaller configs and go to 10Gbps on STARTER and above; Tier 1 is "based on performance," which is a polite way of saying it depends on the node.

**Location and routing.** This is the spec that's invisible on most comparison tables and the one that actually decides whether your users have a good experience. Physical location sets the baseline latency. Routing decides what happens during peak hours when the cheap paths congest. More on this next.

## The routing question most buyers skip

A server in Los Angeles sounds like a server in Los Angeles. It isn't. The path your traffic takes to reach a user in Beijing, Tokyo, or São Paulo depends on which transit providers your host buys from, and that's where cheap VPS providers and premium ones diverge hard.

DMIT is built around this single idea. The same AMD EPYC hardware, the same KVM virtualization, the same SSD storage — but three routing tiers, each priced differently because the underlying transit costs different amounts.

- **Premium Network** uses Tier 1 transit plus premium partners including DMIT's own backbone and China Telecom CN2 GIA. This is the highest-quality path to mainland China and the wider Asia-Pacific region. Expect lower latency, fewer hops, and significantly less packet loss during peak evening hours. This is what you pay for if your end users are in China and performance during 8pm–11pm Beijing time matters.
- **Eyeball Network** pairs Tier 1 transit with reasonable-effort China routing via CMIN2 (and similar). Not as tight as Premium, noticeably better than standard BGP, and meaningfully cheaper. The sweet spot for "I want China to work well but I'm not paying CN2 GIA prices."
- **Tier 1 Network** is standard international routing with no China-specific optimization. Good for general Asia-America and intra-Asia traffic. This is where you look if your users aren't in China at all — paying for CN2 routing you don't need is wasted money.

There's also **Premium Secure** (LAX only), which routes inbound traffic through Cloudflare Magic Transit for DDoS scrubbing before it hits CN2 GIA. That's a niche product for game servers, production APIs, or anything that actually gets attacked. Most readers don't need it, but it's the right answer if you do.

> The short version: if "virtual cloud server" to you means "cheap box in Asia," you want Tier 1. If it means "my Chinese users complain about evening lag," you want Premium or Eyeball. Buying Premium for users who aren't in China is paying for a highway you'll never drive on.

## DMIT cloud instance plans: full comparison

The table below covers the plans currently displayed on DMIT's official pricing page across the three locations (Los Angeles, Hong Kong, Tokyo) and three network tiers. Prices are monthly billing unless noted; annual billing unlocks additional discounts via promo codes at checkout.

| Plan | Network | Location | vCPU | RAM | Storage | Traffic | Port | Price (monthly) | Buy |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| LAX.Pro.STARTER | Premium | Los Angeles | 2 | 2GB DDR4 | 80GB SSD | 3000GB (BIDI) | 10Gbps | $29.90 | [View LAX Premium STARTER](https://bit.ly/DmiT) |
| LAX.Pro.MINI | Premium | Los Angeles | 4 | 4GB DDR4 | 80GB SSD | 5000GB (BIDI) | 10Gbps | $58.88 | [View LAX Premium MINI](https://bit.ly/DmiT) |
| LAX.Pro.MICRO | Premium | Los Angeles | 4 | 4GB DDR4 | 160GB SSD | 7000GB (BIDI) | 10Gbps | $74.99 | [View LAX Premium MICRO](https://bit.ly/DmiT) |
| LAX.EB.STARTER | Eyeball | Los Angeles | 2 | 2GB DDR4 | 80GB SSD | 5000GB (BIDI) | 10Gbps | $29.90 | [View LAX Eyeball STARTER](https://bit.ly/DmiT) |
| LAX.EB.MINI | Eyeball | Los Angeles | 4 | 4GB DDR4 | 80GB SSD | 10000GB (BIDI) | 10Gbps | $58.88 | [View LAX Eyeball MINI](https://bit.ly/DmiT) |
| LAX.EB.MICRO | Eyeball | Los Angeles | 4 | 4GB DDR4 | 160GB SSD | 14000GB (BIDI) | 10Gbps | $74.99 | [View LAX Eyeball MICRO](https://bit.ly/DmiT) |
| LAX.T1.STARTER | Tier 1 | Los Angeles | 1 | 2GB DDR4 | 40GB SSD | 4000GB (IN+OUT) | Performance-based | $12.90 | [View LAX Tier 1 STARTER](https://bit.ly/DmiT) |
| LAX.T1.MINI | Tier 1 | Los Angeles | 2 | 2GB DDR4 | 60GB SSD | 8000GB (IN+OUT) | Performance-based | $21.90 | [View LAX Tier 1 MINI](https://bit.ly/DmiT) |
| LAX.T1.MICRO | Tier 1 | Los Angeles | 4 | 4GB DDR4 | 80GB SSD | 16000GB (IN+OUT) | Performance-based | $32.90 | [View LAX Tier 1 MICRO](https://bit.ly/DmiT) |
| HKG.Pro.STARTER | Premium | Hong Kong | 1 | 2GB DDR4 | 40GB SSD | 800GB (BIDI) | 1Gbps | $79.90 | [View HKG Premium STARTER](https://bit.ly/DmiT) |
| HKG.Pro.MINI | Premium | Hong Kong | 2 | 2GB DDR4 | 60GB SSD | 1200GB (BIDI) | 1Gbps | $119.90 | [View HKG Premium MINI](https://bit.ly/DmiT) |
| HKG.Pro.MICRO | Premium | Hong Kong | 4 | 4GB DDR4 | 80GB SSD | 1600GB (BIDI) | 1Gbps | $159.90 | [View HKG Premium MICRO](https://bit.ly/DmiT) |
| HKG.EB.STARTERv2 | Eyeball | Hong Kong | 1 | 2GB DDR4 | 40GB SSD | 2000GB (BIDI) | 2Gbps (no guarantee) | $59.90 | [View HKG Eyeball STARTER](https://bit.ly/DmiT) |
| HKG.EB.MINIv2 | Eyeball | Hong Kong | 2 | 2GB DDR4 | 60GB SSD | 3000GB (BIDI) | 2Gbps (no guarantee) | $89.90 | [View HKG Eyeball MINI](https://bit.ly/DmiT) |
| HKG.EB.MICROv2 | Eyeball | Hong Kong | 4 | 4GB DDR4 | 80GB SSD | 4000GB (BIDI) | 4Gbps (no guarantee) | $129.90 | [View HKG Eyeball MICRO](https://bit.ly/DmiT) |
| HKG.T1.STARTER | Tier 1 | Hong Kong | 1 | 2GB DDR4 | 40GB SSD | 4000GB (IN+OUT) | Performance-based | $12.90 | [View HKG Tier 1 STARTER](https://bit.ly/DmiT) |
| HKG.T1.MINI | Tier 1 | Hong Kong | 2 | 2GB DDR4 | 60GB SSD | 8000GB (IN+OUT) | Performance-based | $21.90 | [View HKG Tier 1 MINI](https://bit.ly/DmiT) |
| HKG.T1.MICRO | Tier 1 | Hong Kong | 4 | 4GB DDR4 | 80GB SSD | 16000GB (IN+OUT) | Performance-based | $32.90 | [View HKG Tier 1 MICRO](https://bit.ly/DmiT) |
| TYO.Pro.STARTER | Premium | Tokyo | 1 | 2GB DDR4 | 40GB SSD | 500GB (BIDI) | 1Gbps | $39.90 | [View TYO Premium STARTER](https://bit.ly/DmiT) |
| TYO.Pro.MINI | Premium | Tokyo | 2 | 2GB DDR4 | 60GB SSD | 1000GB (BIDI) | 1Gbps | $79.90 | [View TYO Premium MINI](https://bit.ly/DmiT) |
| TYO.Pro.MICRO | Premium | Tokyo | 4 | 4GB DDR4 | 80GB SSD | 2000GB (BIDI) | 1Gbps | $159.90 | [View TYO Premium MICRO](https://bit.ly/DmiT) |
| TYO.EB.STARTER | Eyeball | Tokyo | 1 | 2GB DDR4 | 40GB SSD | 2000GB (BIDI) | 2Gbps (no guarantee) | $55.90 | [View TYO Eyeball STARTER](https://bit.ly/DmiT) |
| TYO.EB.MINI | Eyeball | Tokyo | 2 | 2GB DDR4 | 60GB SSD | 3000GB (BIDI) | 2Gbps (no guarantee) | $85.90 | [View TYO Eyeball MINI](https://bit.ly/DmiT) |
| TYO.EB.MICRO | Eyeball | Tokyo | 4 | 4GB DDR4 | 80GB SSD | 4000GB (BIDI) | 4Gbps (no guarantee) | $119.90 | [View TYO Eyeball MICRO](https://bit.ly/DmiT) |
| TYO.T1.STARTER | Tier 1 | Tokyo | 1 | 2GB DDR4 | 40GB SSD | 4000GB (IN+OUT) | Performance-based | $12.90 | [View TYO Tier 1 STARTER](https://bit.ly/DmiT) |
| TYO.T1.MINI | Tier 1 | Tokyo | 2 | 2GB DDR4 | 60GB SSD | 8000GB (IN+OUT) | Performance-based | $21.90 | [View TYO Tier 1 MINI](https://bit.ly/DmiT) |
| TYO.T1.MICRO | Tier 1 | Tokyo | 4 | 4GB DDR4 | 80GB SSD | 16000GB (IN+OUT) | Performance-based | $32.90 | [View TYO Tier 1 MICRO](https://bit.ly/DmiT) |

A few things worth noting from the pricing page itself:

- All plans include 1 IPv4 and 1 IPv6 (/64 on Premium, /64 or /128 on others), basic DDoS protection, and free instant setup.
- The LAX AS3 platform is still being built out — DMIT's own notice says you may experience reduced disk performance and a lower SLA than their mature platforms during this period. If you need guaranteed disk I/O, ask support which platform your specific plan ships on before ordering.
- Premium and Eyeball traffic is billed bidirectionally (BIDI). Tier 1 is "Max (IN, OUT)" — meaning inbound and outbound are counted but with a higher ceiling, which favors workloads that push a lot of content outbound.
- The Hong Kong Eyeball port speeds are listed as "no guarantee," which is honest but means you shouldn't design around hitting 4Gbps sustained.

If you want to see all current plans and confirm live availability before deciding, 👉 [browse the full DMIT cloud instance lineup](https://bit.ly/DmiT).

## Which plan fits which actual situation

**Serving mainland China users, peak hours matter.** LAX.Pro or HKG.Pro. Hong Kong gives lower baseline latency (20–50ms to domestic Chinese servers in testing), Los Angeles gives you more traffic for the dollar. If your users are mostly mobile and on China Mobile, HKG.EB with CMI routing is the cost-conscious alternative that still beats standard BGP.

**China optimization wanted, but Premium pricing is hard to justify.** LAX.EB STARTER at $29.90/mo gives you 2 vCores, 2GB RAM, 80GB SSD, and 5000GB of CMIN2-routed traffic on a 10Gbps port. For most China-facing workloads that aren't latency-critical, this is the value pick.

**Japan or HK IP, general Asia-Pacific users, no China focus.** TYO.T1 or HKG.T1 STARTER at $12.90/mo. Same 1 vCPU / 2GB / 40GB baseline, no China premium baked into the price. Annual billing promo codes on the Tier 1 line are where the genuinely cheap recurring rates live.

**Mostly US or non-China Asia traffic.** LAX.T1. Don't pay for CN2 GIA routing if your users aren't on Chinese networks. The $12.90 STARTER covers a lot of small-app use cases; MICRO at $32.90 gives you 4 vCores and 4GB if you're running something heavier.

**You get DDoSed and you need China connectivity.** LAX Premium Secure. It's the only tier that combines Cloudflare Magic Transit scrubbing with CN2 GIA return paths. Niche, but if you're running a game server or a production API that attracts volumetric attacks, this is the specific product built for that problem.

## What you get beyond the spec sheet

A few things that aren't in the pricing table but matter once you're running the server:

- **Auto-rebalance.** DMIT distributes instances across multiple nodes in each location and rebalances to avoid resource congestion. In practice this means a noisy neighbor on your physical host is less likely to ruin your night than on a single-node VPS.
- **Snapshots and online backup.** Snapshots are free and instant. Online backup is paid at $0.45/GB/month. Backups are your responsibility — DMIT's terms say so explicitly, same as every other VPS provider — so budget for it if your data matters.
- **ISO mount.** You can mount ISOs as CDROM for unusual OS installs. Most users won't need this, but if you want a specific hardened Linux distro or a BSD, it's there.
- **China IP replacement policy.** If your IP gets blocked in China, DMIT replaces it free once every 15 days, then $5 per change after that. Not unlimited, but a clear policy beats "email support and pray."
- **Money-back window.** 3 days, up to 30GB of usage. Enough to actually test latency from your real user locations before committing.
- **Payment methods.** PayPal, Alipay, WeChat Pay, and major credit cards. Useful if you're paying from China or want to avoid card fees.

## Ordering, promo codes, and how the discount structure works

The ordering flow is straightforward: pick location → pick network tier → pick plan → checkout → apply promo code → pay → instance is live in a few minutes.

The promo code situation is the part worth paying attention to. DMIT runs recurring discounts, not first-year teaser rates. That means the discounted price is what you renew at — the long-term cost is actually predictable, which is not how most of the VPS market works.

A few patterns from codes that have been active on the official promotions pages:

- **LAX Tier 1 annual** codes have historically offered 20–30% off recurring on annual billing (excluding the smallest WEE and TINY plans).
- **LAX Eyeball** launch-period codes offered 20% off recurring on quarterly or annual billing for TINY and above.
- **Hong Kong Tier 1 annual** codes have offered up to 45% off recurring on annual billing, in some cases bundled with spec upgrades.
- **Tokyo Tier 1** codes have offered 10% off monthly and 30% off recurring on quarterly or annual.

Codes are case-sensitive, one per order, and most are limited to one use per account. They don't stack. The bigger discounts require quarterly or annual commitments — monthly billing typically only qualifies for smaller one-time or 10% discounts. The math on annual is usually substantially better once you factor in both the lower base price and the recurring code.

Because promo codes change and expire, the reliable move is to check the current codes on the promotions page right before checkout, paste the one that matches your plan and billing cycle, hit "Validate Code," and confirm the discount applied before paying. If a code doesn't validate, don't assume — pick a different one or contact support.

> One thing worth repeating: these are recurring discounts, not first-year traps. The price you see at checkout with a code applied is the price you renew at. That's a real difference from providers that hook you with $5/mo year one and $25/mo year two.

When you're ready to configure an instance and see live pricing for your specific location and tier, 👉 [open the DMIT cloud instance order page](https://bit.ly/DmiT).

## Common questions

**Is a "virtual cloud server" the same as a VPS?** Mostly yes, in modern usage. The distinction that used to matter — VPS on a single physical box vs. cloud VMs distributed across a cluster with failover — has blurred because most providers calling themselves "cloud" now run distributed storage and auto-rebalance. The thing to actually check is virtualization type (KVM vs. OpenVZ/LXC) and whether storage is local or networked. KVM with distributed storage is what you want.

**Why is DMIT more expensive than Vultr or Hetzner for the same specs?** The hardware is comparable; the difference is routing. CN2 GIA and AS9929 transit to China costs more than standard BGP. If your users aren't in China, that premium is wasted and a general-purpose provider is the better call. If they are in China, you'll feel the difference during evening hours when the cheap paths congest.

**What happens when I hit my traffic limit?** DMIT throttles bandwidth to roughly 100Mbps–1Gbps depending on the plan tier, rather than cutting you off. Your service stays online. If you regularly hit the cap, the next plan up is usually cheaper than paying overage.

**Can I run a VPN on these?** Yes. KVM virtualization means full kernel access, so WireGuard, OpenVPN, Xray, and similar all work without workarounds. This is one of the most common real-world uses for DMIT instances, especially the Premium tier for users who want CN2 GIA return paths to China.

**Do I need Premium Secure?** Only if your service gets DDoSed. For normal web apps, APIs, and VPN endpoints, basic DDoS protection on the standard tiers is enough. Premium Secure is a specific product for a specific problem — game servers, production APIs that attract attacks, anything where volumetric DDoS is a known threat and you also need China routing.

**Is annual billing worth it?** Almost always, on DMIT specifically, because the promo codes are recurring. You lock in a lower price for the life of the subscription rather than paying month-to-month. The catch is the upfront commitment — if you're testing, do one month first, then switch to annual once you've confirmed the routing works for your users.

## The short version

A virtual cloud server is a KVM VM you fully control. The specs that matter for most workloads are RAM (don't go below 2GB), storage type (SSD, ideally with distributed storage underneath), and the routing your provider buys — which is invisible on most comparison pages and the thing that actually decides real-world latency. DMIT's lineup is a clean example of how the same hardware gets priced very differently based on routing: Tier 1 for general traffic, Eyeball for cost-conscious China optimization, Premium for CN2 GIA quality, Premium Secure for DDoS-protected China workloads.

Pick the tier that matches where your users actually are, pick the smallest plan that fits your workload, use a recurring promo code on annual billing if you've tested it works, and don't pay for CN2 GIA routing if nobody on the other end is on a Chinese network.

To configure an instance and check current pricing and promo codes for your specific location and tier, 👉 [start here with DMIT's cloud instance page](https://bit.ly/DmiT).
