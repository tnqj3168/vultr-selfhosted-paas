# self hosted vercel alternative: How to Run Coolify or Dokploy on a Vultr VPS for Less Than One Vercel Pro Seat

Everybody hunting for a self hosted Vercel alternative is chasing the same outcome: keep the git push → live URL workflow, drop the bill that scales with headcount. Vercel Pro runs $20 per team member per month before bandwidth overages and concurrency limits, and once you're past a hobby project the invoice tends to grow with the team. Self-hosting flips the math. The deployment software is free and open source; your only recurring cost is the server, a flat number that doesn't care how many apps or teammates you add.

The catch, which every honest comparison mentions: you become the ops team. Updates, backups, and the Saturday-night outage are yours now.

This guide covers both halves of that decision — which self-hosted platform to install, and which server to run it on, including a Vultr plan breakdown, because "how much server does this actually need" is the question most articles answer with a shrug.

> **Quick picks**
>
> - Solo dev, one or two small apps: the **1GB plan at $5/mo** with Dokploy, which idles around 600MB
> - Most teams: the **2GB plan at $10/mo** with Coolify — per-branch preview environments plus 280+ one-click service templates
> - Dozens of apps and bundled databases: the **4GB plan at $20/mo**, still less than a single Vercel Pro seat
> - 👉 [Check current Vultr Cloud Compute pricing](https://www.vultr.com/pricing/?ref=7268283)

## What a self hosted Vercel alternative actually replaces

A self-hosted setup replaces Vercel's deployment pipeline with open-source PaaS software running on a VPS you rent. You connect a Git repo, push, and the platform builds the app — Nixpacks auto-detects Next.js, Remix, SvelteKit, Nuxt, Django, Rails and 30+ other stacks — provisions a Let's Encrypt certificate automatically, and routes traffic to the new container. Coolify goes further and creates isolated preview environments per branch, the Vercel feature departing teams say they miss most.

What doesn't carry over: the global edge network and the "never think about infrastructure" part. Worth knowing that some of what feels like Next.js magic is actually platform glue — Edge Middleware, ISR behavior, Vercel's analytics — and those pieces tie your app to their platform. On your own server a standard Next.js build runs as a normal Node container. You trade platform-specific convenience for ownership, which is exactly the trade most people searching this keyword are trying to make.

## The five platforms worth shortlisting

The tools split into two families: container platforms that give you a dashboard for running anything in Docker, and deployment tooling that pushes code to a server you already manage. Here's how they compare:

| Tool | What it is | Standout | Trade-off |
| --- | --- | --- | --- |
| Coolify | Apache-2.0 PaaS, ~60,000 GitHub stars, PHP + Docker | Broadest coverage: apps, databases, 280+ one-click services (Postgres, Redis, MinIO, Ghost), per-branch previews, Docker Swarm for multi-node | Most feature surface to learn; idles at 300–500MB |
| Dokploy | Apache-2.0 PaaS, TypeScript | Lighter (~600MB idle), cleaner UI, Docker Compose support, built-in Traefik with automatic TLS | No per-branch previews; smaller template library |
| Dokku | MIT, minimal git-push PaaS, running since 2013 | Tiny footprint, decade of stability, plugin for everything | CLI only, no dashboard by default |
| Kamal | MIT deploy tool from 37signals (Basecamp, HEY), Ruby | Zero-downtime rolling deploys via Traefik health-checked swaps; no control-plane server at all | No UI, no previews; needs terminal comfort and a Docker registry |
| CapRover | Docker + nginx PaaS with web UI, around since 2017 | Mature and stable, one-click app catalogue, clustering | Its licence adds restrictions on the paid features; smaller community than Coolify or Dokploy |

One newer entry deserves a footnote: Temps, an Apache-2.0 project shipped as a single Rust binary, bundles git-push deploys with privacy-first analytics, session replay, Sentry-compatible error tracking and uptime monitoring — the observability stack that Coolify, Dokploy and Dokku all leave you to assemble from separate SaaS accounts. It's young and its newer features are marked beta, so the deployment-only tools above remain the safer default for now.

If your team wants a dashboard and previews, the choice is effectively Coolify versus Dokploy. If your team lives in SSH anyway, Dokku or Kamal get out of the way.

## How much server does a self-hosted PaaS actually need?

RAM is the spec that matters, and the software's own footprint sets the floor. Coolify idles at 300–500MB but builds run in Docker on the same box, so its working recommendation for a small team is 2GB, with 4GB running dozens of apps comfortably. Dokploy idles around 600MB. That translates directly to Vultr's shared-vCPU Cloud Compute line:

| Plan (RAM) | vCPU | Storage | Transfer | Monthly price | Deploy |
| --- | --- | --- | --- | --- | --- |
| 512MB | 1 | 10GB SSD | 0.5TB | $2.50–$3.50* | [ Deploy](https://www.vultr.com/pricing/?ref=7268283) |
| 1GB | 1 | 25GB SSD | 1TB | $5 | [ Deploy](https://www.vultr.com/pricing/?ref=7268283) |
| 2GB | 1 | 55GB SSD | 2TB | $10 | [ Deploy](https://www.vultr.com/pricing/?ref=7268283) |
| 4GB | 2 | 80GB SSD | 3TB | $20 | [ Deploy](https://www.vultr.com/pricing/?ref=7268283) |
| 8GB | 2 | 160GB SSD | 4TB | $40 | [ Deploy](https://www.vultr.com/pricing/?ref=7268283) |
| 16GB | 4 | 320GB SSD | 5TB | $80 | [ Deploy](https://www.vultr.com/pricing/?ref=7268283) |

*\*The $2.50 variant is IPv6-only; IPv4 costs $1 more.*

A few billing details that affect the real cost:

- Billing is hourly, capped at the monthly rate — spin up a test instance for an afternoon and you pay for the hours it existed.
- Automatic backups add 20% of the instance price. Transfer beyond the included amount is $0.01/GB.
- Vultr also sells a High Performance line that swaps in NVMe storage starting at $6/mo for 1GB, if you want faster disk for a dollar or two per tier.

Mapping plans to situations:

- **512MB — skip it for this job.** Fine for a static site; both Coolify and Dokploy will leave it no room to breathe during builds.
- **1GB ($5) — the solo floor.** Dokploy's ~600MB idle footprint fits. Coolify runs too, but a heavy Next.js build can push a 1GB box into swap.
- **2GB ($10) — the default answer.** This is the plan that handles Coolify for a small team without babysitting.
- **4GB ($20) — dozens of apps comfortably,** especially if you're stacking Postgres, Redis and MinIO from Coolify's one-click templates on the same machine.
- **8GB and up ($40+)** — when the box also hosts staging environments or a handful of client projects.

For calibration against the other usual suspects: Hetzner's CX23 runs $3.97/month for 2 vCPU and 4GB, and DigitalOcean's basic droplet is $4 for 1 vCPU and 512MB. On raw RAM per dollar, Hetzner wins that row. Vultr's case rests elsewhere — hourly billing with no minimum term, regions across North America, South America, Europe, Asia, Australia and Africa, and the tooling (firewall groups, snapshots, a full API) that starts to matter once the server is your production platform.

## Setup: bare Vultr instance to git push deploys in under an hour

Under an hour, start to finish, if you do DNS first. The Coolify installer is a single command; one developer who migrated three projects to Coolify timed the whole move at about 40 minutes and blamed most of it on DNS records done in the wrong order. Learn from that sequence:

1. **Deploy a fresh Ubuntu LTS instance** on Vultr in the region closest to your users, and note its IPv4 address.
2. **Point DNS before anything else.** Add an A record for your app's domain to the instance IP. Let's Encrypt can't issue a certificate until the domain resolves, so doing this first avoids a loop of failed issuance attempts.
3. **SSH in and run the installer:**

bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash


It pulls in Docker and stands up the control plane. Dokploy has its own one-liner, same idea.

4. **Open the right ports** — 22, 80 and 443, plus 8000 for Coolify's dashboard on first login — via Vultr's firewall or ufw.
5. **Create your admin account, connect your Git provider, create a project** and register your domain against the server.
6. **Push.** Pick the repo and branch; Nixpacks detects the framework and builds it. Certificate issuance is automatic once DNS is in place.
7. **Before you call it done:** enable Vultr automatic backups (that 20% of instance price) or take a snapshot of the freshly configured server. Future you will not regret this.

👉 [Deploy a Vultr instance and run the installer tonight](https://www.vultr.com/pricing/?ref=7268283)

## The real cost math against staying on Vercel

At one seat, self-hosting on Vultr roughly halves the bill. At five seats it's a 90%+ cut. The comparison:

- **Vercel Pro:** $20/seat/month, unlimited apps, but bandwidth overages and add-ons stack on top. A five-person team running three apps can cross $300–500/month once analytics and error tracking land on the invoice.
- **Vultr 2GB + Coolify:** $10/month flat, $120/year. Unlimited apps, unlimited collaborators, no per-seat anything. The same five-person team pays the same $120.
- **Bandwidth:** Vercel meters it; Vultr's 2GB plan includes 2TB of transfer at $0.01/GB after — more headroom than most side-project portfolios will ever touch.

The honest counterweight: the currency you pay with instead is time. Updates, backups, and the occasional weekend incident are on your calendar now, and if the server dies on a Saturday night, nobody fixes it but you. If nobody on your team wants that job, the managed route still exists — Railway's Pro tier is $20/seat, Fly.io runs a small production Next.js app on one shared-2x VM for roughly $5–8/month with anycast routing across 30+ locations, and Netlify sits at $19/seat (managed only). Those are legitimate choices; they just aren't self-hosting.

👉 [Compare Vultr's full plan list before you decide](https://www.vultr.com/pricing/?ref=7268283)

## Questions people ask before committing

**Do I still get preview deployments?**
On Coolify, yes — per-branch preview environments, the same concept Vercel made standard. Dokploy, Dokku and Kamal don't ship them, which is the most common reason comparisons in this space still put Coolify first for teams.

**Will my Next.js app actually run?**
Yes. Nixpacks detects Next.js and builds a standard Node container, which covers the bulk of what a typical app needs. What you leave behind is the edge network and platform-tied extras like Edge Middleware and Vercel's analytics.

**Is the $5 plan enough for Coolify?**
It boots and idles within budget at 300–500MB, but Docker builds run on the same machine and a heavy build can exhaust 1GB. For anything with a second set of hands on it, 2GB is the realistic floor.

**Who's on call when it breaks at 3 AM?**
You are. Backups and snapshots shrink the blast radius; they don't remove the pager. If that's a dealbreaker, the honest move is a managed host, not a VPS you'll grow to resent.

**Am I creating a new lock-in problem?**
No — your apps are Docker containers and your data lives in volumes on a server you control. Moving to a different VPS or a different PaaS later is a migration, not a rebuild.

## Pick by your situation

- Want the closest thing to the Vercel workflow, previews included, with room for apps and databases on one box → **Coolify on the 2GB plan, $10/mo**.
- Flying solo and allergic to dashboard clutter → **Dokploy on the 1GB plan, $5/mo**.
- Terminal-native, want no platform to maintain at all → Kamal or Dokku on whatever plan fits the app itself.
- Need global edge and a pager-free life → self-hosting is the wrong tool for you; the managed hosts are built for exactly that case.

Three checks before you click deploy: region near your users, 2GB rather than 1GB if you're building Next.js, and DNS sorted before the first deploy — in that order.

👉 [Start with Vultr's 2GB plan at $10/month](https://www.vultr.com/pricing/?ref=7268283) and you can be on git push deploys before dinner.
