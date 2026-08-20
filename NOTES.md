# Hulloboats.com — Progress Log

## Vision
Peer-to-peer marketplace for buying/selling used boats. Compete with Boatrader-style sites but friendlier: fewer ads, more trust, scam-detection, closer to the Reverb.com model of bridging buyer/seller directly. Possible future: consignment lot with a mechanic in Gainesville, FL (not confirmed yet).

**Niche positioning (decided 2026-08-19, refined):** Broad powerboat marketplace — center console, bay boat, flats boat, high performance, deck boat, and other powerboat types are all fine ("deck boats and what not"). Explicitly and permanently EXCLUDED as a sellable/listable category: sailboats and yachts. Jon's stated reason: those categories effectively require a dealer's/broker's license to transact (plausible — FL specifically regulates yacht & ship brokers for larger vessels) and he doesn't want that complexity or the page clutter. Worth a quick confirm with someone versed in FL boat dealer/broker licensing before launch, since rules can hinge on vessel size/value, not just category — but the product decision (no sailboats, no yachts, ever, as listings) stands regardless.

**Demand-sensing idea (decided 2026-08-19):** Even though sailboats/yachts can't be listed, Jon wants to keep visibility into whether people are searching/looking for them — search is broader than the sell-side category list, and those queries get logged as a demand signal. Mechanism: **passive for now** — quietly log searches, no active "don't see your boat type? tell us" prompt at this stage. If real demand shows up in the data, that becomes a future decision point about whether to ever open that door — not an automatic green light. Separate from actually allowing those listings, which stays closed.

**Dealers vs. private sellers (decided 2026-08-19):** Jon raised this from a prior conversation, worried about dealers "flooding" the market like on other sites. Claude initially proposed structurally separating dealers into their own section from day one. Jon pushed back with two good points: (1) there's no real way to verify/enforce someone is an individual vs. a dealer — a dealer can just register as "Joe Blow"; (2) Reverb.com (Jon's model for the site) doesn't seem to wall dealers off and doesn't have a problem with it. Claude researched Reverb's actual approach and confirmed: Reverb requires real identity verification (government ID, matching bank info via a third-party KYC provider) for ALL sellers; "business" is just a self-declared account type at signup that unlocks a few extra features and requires extra docs (EIN, business registration) — it's not a hard gate, and nothing stops a dealer from registering as an individual. Reverb's buyer-facing trust signals are behavior-based (Quick Responder, Quick Shipper, Preferred Seller badges, star ratings), not individual-vs-dealer labels.
**Decision: adopt the Reverb model.** No hard gate/separation between private sellers and dealers. Instead: (a) require real identity verification for every seller regardless of type — serves the scam-prevention goal directly; (b) "dealer/business" is an optional self-declared account type at signup that can unlock dealer-friendly features for those who want it, not a wall; (c) build buyer trust through behavior/reputation signals (responsiveness, shipping/handoff speed, ratings, completed-deal history) rather than seller-type badges; (d) optionally flag unusually high-volume individual accounts for a human look, as a soft backstop rather than a hard rule. Confirmed with Jon as the direction to go — "this is fine."

## Team / roles
- Jon: product direction, business (LLC via Northwest Registered Agent, domain via Namecheap), backend work alongside Claude.
- Casey: web designer, building the front end. Has previously built a site using Claude. Recommended AWS for backend scalability — under discussion (see Backend platform section below).
- Claude: backend, infra, DNS/hosting setup, general build support.
- Plan to eventually connect via GitHub (and/or GitLab) so both Claude and Casey can collaborate on the codebase.

## Status as of 2026-08-19
- **Domain**: hulloboats.com registered at Namecheap (expires May 12, 2027).
- **Coming-soon site**: LIVE at hulloboats.com over HTTPS (DNS check passed, HTTPS enforced). Built as a single static index.html, navy/teal/gold nautical theme, headline "Hulloboats", tagline "The boat marketplace, built on trust.", short blurb on the peer-to-peer/honest/fewer-ads positioning. No Gainesville/consignment mention published (not confirmed by Jon yet).
- **Hosting**: GitHub Pages, repo `hulloboats/hulloboats.github.io` (public). This is the special GitHub "user site" repo name so it auto-publishes at hulloboats.github.io. Note: GitHub Pages is static-only — it cannot run the actual marketplace backend (listings, accounts, search logging, identity verification, etc.); a real backend platform still needs to be chosen/built for that.
- **DNS (Namecheap Advanced DNS for hulloboats.com)**:
  - 4 A records on `@` → 185.199.108.153 / .109.153 / .110.153 / .111.153 (GitHub Pages IPs)
  - CNAME on `www` → hulloboats.github.io
  - Existing TXT record (SPF for email forwarding) left untouched.
- **Not yet decided**: email capture/signup approach for the coming-soon page. LLC/Northwest Registered Agent — no specific task done there yet.

## Backend platform — open discussion
Claude recommended Supabase (managed Postgres + auth + file storage, generous free tier, standard Postgres under the hood so it's portable) to move fast pre-launch without a big DevOps lift. Casey recommended AWS directly, for scalability as traffic grows. Claude's counter: Supabase itself runs on AWS-grade infra and scales to serious production traffic; the real tradeoff is who builds/maintains the plumbing (raw AWS = full control but more ongoing ops work; managed platform = faster to build, still migratable to raw AWS later since the data is standard Postgres). **Not yet resolved** — next step is a conversation with Casey directly about her specific reasoning before deciding.

## Backend design — in progress, starting with Listings
Decided to start backend design with the listing data model (not accounts) since it defines the core product. Draft fields as of 2026-08-19 (not yet finalized, no code/infra built yet):
- **Category** (closed picklist for SELLING, broad powerboat coverage): Center Console, Bay Boat, Flats Boat, High Performance, Deck Boat, plus likely more common powerboat types (pontoon, bass boat, cuddy cabin/offshore, jon boat, etc. — TBD exact full list) and an "Other powerboat" catch-all. Structurally NO sailboat or yacht option ever exists in the SELL-side picklist — permanent exclusion, not just a moderation rule.
- **Search/demand side is broader than sell side**: buyers can search more openly (including terms like "sailboat"/"yacht"), and those searches get logged passively as a demand signal even though no matching listings will ever appear. See "Demand-sensing idea" above.
- **Seller type**: no hard private-vs-dealer gate — see "Dealers vs. private sellers" decision above. Every seller (individual or business) goes through the same identity-verification flow; business/dealer is an optional self-declared upgrade, not a separate track.
- **Basics**: make/model, year, length, price, location (city/state).
- **Condition & specs**: new/used, hull material, engine type, engine hours, horsepower.
- **Trust/verification**: Hull Identification Number (HIN — boat equivalent of a VIN, useful for confirming a boat isn't stolen/misrepresented), trailer included y/n, title/documentation status (clean title, liens, etc.). Plus seller-level identity verification (see above) and reputation/behavior signals (responsiveness, completed deals, ratings) once accounts exist.
- **Story**: multiple photos, free-text description, seller contact method.
- **Behind the scenes**: listing status (draft / pending review / live / sold) — leaves room for a moderation step before a listing goes public, fits the "safe and trustworthy" goal.
- Open question: exact full list of allowed sell-side categories beyond the 5 named so far.
- Open question: which identity-verification provider to use (Reverb uses Trulioo; alternatives include Stripe Identity, Persona, Onfido) — ties into backend platform choice.

## Next possible steps (not started)
- Talk with Casey about her AWS-vs-managed-platform reasoning and settle on a backend platform.
- Finish nailing down the full listing category list.
- Decide on an identity-verification provider/approach for sellers.
- Give Casey access to the GitHub repo for frontend work.
- Decide on email signup approach for the coming-soon page.
- Firm up whether the Gainesville, FL consignment lot idea is happening.
