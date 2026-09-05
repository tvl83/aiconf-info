# Workshop Runbook — Hands-On Paperclip Session

Extracted from the planning call with Verbus the night before.

---

## 1. Tonight — prep and dry run

- [ ] **Do the whole thing yourself, end to end**, using the OpenRouter key and an open-source model. Candidates: DeepSeek, Qwen, Kimi, GLM. Leaning DeepSeek (used it before, likes it).
- [ ] **Time the build.** Guess is ~20 min for a simple single-page site, but speed on the open models is unknown — the top-tier models are faster. This is the main unknown for the schedule.
- [ ] **Set up LiteLLM** for failover — if a model hits a limit or breaks, it rolls to the next on the list instead of failing outright.
- [ ] **Write the PRD template** and push it to a **public GitHub repo**.
  - Template declares the 4 env vars up front: `GH_TOKEN`, Supabase URL, DB URL, publishable key.
  - Template includes a line instructing the agent to check the resume for a residential address and not publish it (secondary safety net).
  - Template has a name field the attendee edits before pasting.
- [ ] **Build the Google Sheet**: one row per attendee.
  - Column: name
  - Column: their LLM API key
  - Column: their Vercel URL (filled in later by them)
- [ ] **Generate per-person API keys** — $1.50 spend cap each, short-lived (good for the day only).
- [ ] **Keep a private backup copy of the keys** in a separate sheet.
- [ ] **Delete the old test Supabase project** (paid, billed hourly — costs a dollar or two for the days it ran) and stand up a fresh one.
- [ ] **Update the event Details page** with the Vault 405 / TierPoint address — it was left off.
- [ ] **Send a follow-up email with the address** to the ~17 people who signed in. Cross-check the sign-in sheet against actual registrations first; some handwriting is rough.
- [ ] Pull ordered notes out of this transcript so the morning has a step order to follow.

### To verify tonight
- Does Vercel run the Supabase migrations during the build, or do they have to be run by hand in Supabase?
- ~~Is JB right that bots crawl public repos for exposed keys?~~ **Confirmed — JB was right.** See the talking point in section 3. Nothing in the plan changes; the sheet link isn't a scannable credential and the keys are capped and short-lived.
- The registration page was demoed but **never actually submitted**, so it's unconfirmed that it writes to the database.

---

## 2. Morning logistics

- Verbus picks up Ramira and Michaela around **8:15**, on site around **8:40**.
- Thomas arrives earlier to set up.
- ~17 signed in; actual turnout unknown.
- Post the Google Sheet URL into the GitHub repo **only once everyone is in the room** — not before.

---

## 3. Session order

**Set everything up before pasting the PRD.** This was the failure point in the demo: attendees paste a PRD, then get stuck mid-run being asked for credentials, and there's confusion over which system already has which key.

### Opening: explain the stack
1. Walk through the setup you went through last night, briefly.
2. What OpenRouter is.
3. What LiteLLM is and why failover matters.

### Credentials and accounts (all of it, up front)
4. **API keys** — everyone opens the shared sheet, finds their name, **copies** the key, pastes it into Paperclip, confirms it landed, *then* goes back and deletes it from the sheet.
   - Say plainly: this is not how you'd normally handle secrets. These are short-lived, spend-capped, one-day keys, so it's fine for today.

> **Talking point — how fast leaked keys actually get found.** JB raised this and he was right; the numbers are more extreme than they sound.
>
> - Attackers subscribe to the **GitHub Events API**, the real-time public firehose of every push, and ingest thousands of events per second. They regex the diffs for known credential shapes — AWS keys starting with `AKIA`, GCP service-account JSON, Stripe keys, hundreds of patterns.
> - Palo Alto Unit 42's EleKtra-Leak analysis: average gap between a valid AWS key hitting a public repo and a bot detecting it was **four minutes**. A Comparitech honeypot saw exploitation in **one minute**. Truffle Security and GitGuardian honeypots land in the same 1–5 minute range.
> - GitGuardian found **12.8 million secrets** in public repos in 2023, and over 90% were still valid five days later — most people never find out.
> - **Deleting the repo does not fix it.** Secrets stay reachable through dangling commits. The only fix is to revoke the key.
>
> **The distinction worth drawing for the room:** what's going in our repo is a *link* to a sheet, not a key. Those scanners match credential patterns in file contents — a Drive URL matches nothing, so the firehose never flags it. A committed key and a link to a key get found by completely different mechanisms, and only one of them is measured in minutes. Our exposure here is a human happening to browse the repo, which is a much smaller problem, and it's capped and expiring anyway.
>
> Then the practical takeaway: assume anything you push to a public repo is read by a machine within minutes. If you ever leak one, revoke first — don't delete the repo and hope.
5. **GitHub token** — each person creates a **fine-grained** personal access token.

   Have them fork the PRD repo *first* and use that fork as their project repo. Then the token only needs access to one repo that already exists, which keeps it minimal:

   | Setting | Value |
   |---|---|
   | Resource owner | Their **personal account** — not an organization |
   | Expiration | Custom date → **tomorrow** |
   | Repository access | Only select repositories → **their fork** |
   | Contents | **Read and write** |
   | Metadata | **Read-only** (auto-selected, can't be turned off) |

   Contents write covers clone, commit, branch, and push. That's everything Paperclip needs.

   **Don't grant:**
   - *Workflows* — only needed if the agent writes files under `.github/workflows/`. Vercel deploys through its own GitHub App, not Actions, so it shouldn't. **Know the failure mode:** if you see a push rejected with a message about refusing to create or update a workflow without the workflow permission, that's this. The fix is telling the agent not to add CI, not widening the token.
   - *Pull requests* — nobody's opening PRs in this flow.
   - *Administration* — see below.

   **If someone lets the agent create a new repo instead of using their fork**, the token needs **Administration: Read and write**, and because the repo doesn't exist yet it can't be selected — so the token has to be scoped to **All repositories**. That's a token that can create, rename, reconfigure, and delete anything in their account. Survivable for one day, much worse than a single-repo Contents token. Steer people to the fork. *(Confirm this in tonight's dry run — it's the one setting that decides whether the morning works.)*

   **Gotchas:**
   - If anyone picks an **organization** as resource owner and that org requires approval, the token is created in a pending state and silently fails to work. Personal accounts only.
   - Delete the token at end of day regardless of the expiry date. Both are cheap.
   - Tell the agent to use a credential helper or the `x-access-token` form rather than `https://<token>@github.com/...`, which writes the token into `.git/config` in plaintext and into shell history.

   **Callback to the morning talk:** fine-grained tokens all begin with `github_pat_`. That fixed prefix is exactly the kind of pattern the scanners regex against. Nice concrete example right after you've described the mechanism.
6. **Vercel project** — create it.
7. **Supabase project** — create it (free tier gives 2 databases), pull the secrets out.
8. **Set the 4 env vars in Paperclip**: GH token, Supabase URL, DB URL, publishable key.

### Resume privacy — cover this before anything is published
9. These sites go on the public web. Tell people to **strip their residential address** from the resume before using it. Mention **phone number** as well — their call, but they should make it deliberately.

### The build
10. Fork the public repo containing the PRD.
11. Edit the PRD in GitHub — change the name field. (Adds a couple of extra GitHub steps, which is the point.)
12. Copy the PRD into Paperclip and **attach the resume**.
13. Give it the GitHub URL for where to save the repo.
14. Talk to the CEO agent to hire the staff.
15. Let it run.

### Iterations
16. New task per person, their own choice of change: light/dark toggle, color changes, whatever they want. Suggest the light/dark toggle as a starting idea, then let them pick their own.
17. Point out the full loop as it happens — request → agent edits → git commit and push → GitHub → Vercel detects → builds → live.
18. Everyone pastes their **Vercel URL into the shared sheet** so people can browse each other's sites. This is the tangible payoff.

### Lunch
19. **Empire Pizza, Edmond.** Service is quick and they can handle a party of 8–10. Eat in Edmond, then head to the data center after.

### After lunch: Supabase
20. New prompt: add a contact form to the page, wired into Supabase.
21. The agent will likely hand back SQL to create the table and the RLS policy — run it in Supabase. One table.
22. Commit the contact form.
23. **Demo it**: everyone visits each other's sites from the shared sheet and leaves a friendly comment.

### Closing block on the contact form

This is the second teaching moment of the day, and it pairs with the morning one. Budget ten minutes.

**24. Public forms get found, but not the way keys do.**

Spambots crawl at scale looking for forms. They follow search engine indexes hunting for pages containing form elements, run their own crawlers that read page source for form markup, follow links in from any public page, and trade lists of URLs already known to host forms. The more sophisticated ones skip the page entirely and post straight to the form endpoint. Nobody has to link to you on purpose.

But draw the contrast with this morning explicitly, because the two cases feel similar and aren't:

- A key pushed to a public repo is found in **minutes**, because there's a real-time firehose — the GitHub Events API — and a regex.
- A form on a fresh `*.vercel.app` URL has **no equivalent firehose**. Every discovery route above needs the URL to be indexed, linked, or already on a list. And Vercel serves all deployment URLs under one wildcard certificate for `*.vercel.app`, so these subdomains never show up individually in Certificate Transparency logs — which is the usual fast way new hostnames get discovered. Realistic time to first spam here is weeks or months, not hours.

Point out that **we're the ones creating the discovery path**: the PRD has the agent write the live URL into the README of a public repo, and public repos get crawled. That link is the most likely reason these sites get found at all. Good thing to show on screen.

**25. CAPTCHA is the escalation, not the first line.**

If you only say "we skipped the CAPTCHA," people leave thinking CAPTCHA is the standard and everything else is exotic. It's the other way round. Standard practice is layered, invisible measures first:

1. **Honeypot field** — a hidden input real users never see and never fill. Bots read the HTML and fill it. Reject any submission where it's populated. Five minutes of work, zero friction.
2. **Minimum submission time** — reject anything submitted a second after page load. Humans are slower than that.
3. **Rate limiting** — cap submissions per IP at something a real person would never exceed.
4. **CAPTCHA** — add it if spam persists despite the above, and prefer a background one (reCAPTCHA v3, Cloudflare Turnstile) over a puzzle.

Worth adding: modern AI-driven spam bots increasingly solve CAPTCHAs and mimic human behavior well enough to get through anyway, so it's not the wall it was ten years ago. Layering is the actual answer.

**26. The consequence today is close to zero — say so.**

Nothing here is wired to email. Spam would land as rows in a Supabase table nobody reads. No flooded inbox, no damaged domain sending reputation, no meaningful cost. Most of what people write about form spam is worried about problems this setup doesn't have. Be honest about that rather than manufacturing urgency.

**27. The thing that would actually bite them is RLS, not spam.**

This is the important one and it's the direct callback to the morning:

- The Supabase publishable (anon) key ships inside the client-side bundle. Anyone can view source and read it. **That is by design.** It is not a leak.
- The only reason that's safe is **row-level security**. RLS is what makes a public key harmless.
- So if the agent writes a permissive read policy — or someone pastes SQL without reading it — anyone with that key can read every submission on that site. Names, emails, messages. That's a real data exposure, unlike the spam.
- The policy we want: **anonymous insert allowed, anonymous select denied.**

Have people actually read the SQL the agent hands them before running it. That's the habit worth transmitting — more than any specific policy.

**28. What to do with the form afterward.**

Three honest options; let people pick:

- **Add the honeypot.** Five minutes, and you learn a mitigation instead of a deletion. Best option if anyone wants to keep the site as a real portfolio piece.
- **Leave it.** Given the discovery picture and the zero-consequence table, this is defensible — provided the RLS policy is right.
- **Remove it.** Fine if you don't want to think about it again. It existed to demonstrate the database round trip, and it did that.

What *isn't* optional: verify the RLS policy before walking away, whichever of the three you pick.

---

## 4. Facilitation

- Build in deliberate stopping points.
- Verbus interjects and asks questions to make sure the room is caught up — this worked well on day one. Thomas tends to keep going.

---

## 5. After 3pm

Server work after the session wraps. Details tracked separately.

- Remind Verbus to grab a cart on arrival.

---

## 6. Open items

- ~~Bot-scraping-public-repos claim~~ — **resolved, JB confirmed.** Now a talking point in section 3.
- ~~Whether Vercel runs migrations at build time~~ — **resolved: it does not.** The Supabase–Vercel integration syncs env vars only. Migrations run only via a GitHub Action or an explicit build-command step. Plan on the manual SQL-editor step and present it as the normal path.
- ~~Open-model speed~~ — **partly resolved.** The variance is across providers, not models: OpenRouter defaults to price-weighted routing and DeepSeek throughput ranges 4–57 tok/s depending on who serves it. Append `:nitro` to the model slug (or set `provider.sort: "throughput"`) and set `require_parameters: true` in LiteLLM. Still time both ways tonight.
- Whether the registration form actually saves to the database.
- Open-model speed — affects whether the build finishes before lunch.
- OS choice for the servers.
- Actual headcount vs. the 17 sign-ins.
