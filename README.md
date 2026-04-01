# Lavoisier Cornerstone — IT Portfolio Site

**Live URL:** [lavoisier.dev](https://lavoisier.dev)  
**Stack:** React + Vite · Cloudflare DNS · Vercel  
**Author:** Lavoisier Cornerstone · [github.com/cornerstonian](https://github.com/cornerstonian) · [linkedin.com/in/voiscornerstone](https://linkedin.com/in/voiscornerstone)

---

## Overview

Professional IT portfolio for Lavoisier Cornerstone — an IT professional and systems administrator with hands-on experience in network operations, cloud infrastructure, and systems administration. The site showcases deployed projects, certifications, and technical skills targeting roles in Help Desk, Network Administration, and Cloud/Network Security across Texas.

**Sections:**
- About — professional summary, location availability, cert badges
- Certifications — CompTIA Security+, AZ-900, Google IT Support Professional
- Projects — deployed lab work across networking, cloud, cybersecurity, and automation
- Skills — technical stack and tooling
- Contact — direct outreach and resume download

**Upcoming additions:**
- CCNA Command Center integration (browser-based IOS simulator, subnetting arena, flashcard drills)
- Blog/README showcase
- Links landing page subdomain

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend Framework | React 18 + Vite |
| Styling | CSS / Tailwind |
| Build Tool | Vite with SWC |
| Hosting | Vercel (Production) |
| DNS Provider | Cloudflare |
| Domain Registrar | Cloudflare Registrar |
| SSL/TLS | Vercel (Let's Encrypt, auto-provisioned) |
| Version Control | Git + GitHub |

---

## Deployment Architecture

```
User Request (lavoisier.dev)
        │
        ▼
Cloudflare DNS  ──  DNS only (gray cloud, no proxy)
        │
        ▼
Vercel Edge Network
        │
        ▼
React + Vite Production Build
        │
        ▼
lavoisier.dev (HTTPS, SSL via Let's Encrypt)
```

Cloudflare operates in **DNS only** mode — it resolves the domain and routes traffic directly to Vercel without proxying. Vercel handles SSL certificate provisioning independently via Let's Encrypt. This architecture avoids SSL conflicts that occur when Cloudflare proxy (orange cloud) intercepts traffic destined for a Vercel-managed certificate.

---

## DNS Configuration

| Type | Name | Content | Proxy Status |
|---|---|---|---|
| A | lavoisier.dev | 76.76.21.21 | DNS only |
| CNAME | www | cname.vercel-dns.com | DNS only |

**Why DNS only and not proxied:**  
Vercel provisions and manages its own SSL certificate for every custom domain. When Cloudflare proxy is enabled, it terminates the TLS connection at Cloudflare's edge and re-encrypts to Vercel — creating two competing certificate authorities for the same domain. This causes Vercel's domain validation to fail. Setting records to DNS only allows Vercel to own the full TLS chain end-to-end.

**Why `cname.vercel-dns.com` for `www`:**  
Vercel uses a shared CNAME target for all custom `www` subdomains. Pointing to a project-specific hash URL (as Vercel sometimes auto-generates) can break if the deployment is relinked or the project is renamed. The canonical `cname.vercel-dns.com` target is stable across all Vercel projects.

---

## Troubleshooting Case Study — DNS Resolution Failure

### Symptom
After configuring DNS records and adding the custom domain in Vercel, both `lavoisier.dev` and `www.lavoisier.dev` showed **Invalid Configuration** in the Vercel dashboard after nearly 24 hours. The site was not resolving.

### Diagnosis — Layer by Layer

**Layer 1 — Vercel (hosting)**  
Vercel dashboard confirmed both custom domains were showing Invalid Configuration while the default `.vercel.app` domain was valid. This indicated Vercel could not verify DNS ownership — meaning the issue was upstream, not in the application itself.

**Layer 2 — Cloudflare DNS zone**  
DNS records existed in Cloudflare (A record + CNAME) but both were set to **Proxied** (orange cloud). Proxied mode conflicts with Vercel's SSL provisioning. Additionally, the `www` CNAME was pointing to a Vercel-generated hash URL rather than the canonical `cname.vercel-dns.com`.

**Layer 3 — Cloudflare zone status**  
The Cloudflare zone for `lavoisier.dev` showed **Pending** — meaning Cloudflare had not verified nameserver ownership. Cloudflare's DNS records cannot resolve until the zone is Active.

**Layer 4 — Cloudflare Registrar**  
Navigating to Cloudflare Registrar revealed no registered domains. The zone had been created in Cloudflare (allowing DNS record configuration) without the domain ever being purchased. The domain `lavoisier.dev` was unregistered — no authoritative nameserver delegation existed at the registry level.

### Root Cause
A Cloudflare zone was created for `lavoisier.dev` without completing domain registration. Cloudflare allows zone creation (DNS management setup) independently of domain registration. Without registration, the `.dev` registry has no NS records pointing to Cloudflare's nameservers (`kianchau.ns.cloudflare.com`, `millie.ns.cloudflare.com`), so the zone can never activate and DNS records can never resolve.

### Resolution — In Order

1. Purchased `lavoisier.dev` through Cloudflare Registrar ($12.20/year at-cost, no markup)
2. Cloudflare automatically linked the registration to the existing zone, activating it
3. Edited the A record — toggled proxy from **Proxied → DNS only**
4. Edited the CNAME — corrected target from hash URL → `cname.vercel-dns.com`, toggled **DNS only**
5. Returned to Vercel → Settings → Domains → Refresh on both entries
6. Both domains flipped to **Valid Configuration** within minutes
7. Vercel auto-provisioned SSL — site live at `https://lavoisier.dev`

### Total Resolution Time
~45 minutes from diagnosis to live site.

### Key Takeaway
DNS troubleshooting requires understanding the full delegation chain: **Registry → Registrar → DNS Zone → Host**. A misconfiguration at any layer blocks all layers below it. In this case, the failure was at the registrar layer — everything else (DNS records, Vercel config, React build) was correctly configured and simply waiting on a domain that didn't legally exist yet.

---

## Local Development

```bash
git clone https://github.com/cornerstonian/big-net-cloud-sec-portfolio.git
cd big-net-cloud-sec-portfolio
npm install
npm run dev
```

Site runs at `http://localhost:5174`

---

## Deployment

Production deployments are handled automatically via Vercel's GitHub integration. Every push to `main` triggers a new production build and deployment.

```bash
git add .
git commit -m "your message"
git push origin main
# Vercel auto-deploys within ~30 seconds
```


