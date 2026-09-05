# PRD: Personal Resume Website

> **Attendee: fill in the three fields below, then paste this entire document into Paperclip and attach your resume.**

| Field | Value |
|---|---|
| Your full name | `{{FULL_NAME}}` |
| Your GitHub username | `{{GITHUB_USERNAME}}` |
| Repository name | `{{REPO_NAME}}` |

---

## 1. Overview

Build and deploy a single-page personal resume website for **{{FULL_NAME}}**, generated from the resume file attached to this task. The site is pushed to `https://github.com/{{GITHUB_USERNAME}}/{{REPO_NAME}}` and deployed publicly on Vercel.

This is a one-day workshop project. Bias toward finishing and shipping over completeness. If a requirement below is slowing the build down, drop it and note the omission in the README.

## 2. Environment — already configured

The following environment variables are **already set** in this workspace. Use them. Do not stop and ask for them, and do not prompt for any credential that appears in this list.

| Variable | Purpose |
|---|---|
| `GH_TOKEN` | GitHub personal access token, for creating the repo and pushing |
| `SUPABASE_URL` | Supabase project URL |
| `SUPABASE_DB_URL` | Supabase Postgres connection string |
| `SUPABASE_PUBLISHABLE_KEY` | Supabase publishable (anon) key |

If a credential you need is genuinely not in that list, stop and say exactly which one is missing rather than guessing or generating a placeholder.

## 3. Input

A resume is attached to this task. It is the sole source of content. Do not invent employers, dates, degrees, certifications, or skills that do not appear in it. If a section of the resume is ambiguous, make the most conservative reading and move on.

## 4. Privacy requirements — mandatory

This site will be published to the public internet.

1. **Do not publish a residential or street address.** Check the attached resume. If it contains one, omit it entirely from the site. City and state are fine.
2. **Do not publish a phone number** unless the attendee has explicitly written `INCLUDE PHONE` on the line below.
3. An email address may be included only if it already appears in the resume.
4. When you finish, state in your summary which pieces of personal information you found and removed.

Attendee override (leave blank unless you want it):
```
INCLUDE PHONE:
```

## 5. Technical requirements

- **Framework:** Next.js (App Router) with TypeScript.
- **Styling:** Tailwind CSS.
- **Rendering:** static / server-rendered. No client-side data fetching is needed in phase 1.
- **Hosting:** Vercel, connected to the GitHub repo so that every push triggers a build.
- The build must pass with no type errors and no failing lint rules that block the build.
- Keep the dependency list minimal. No component library, no animation library, no analytics package.

## 6. Content and layout

A single page containing, in this order, using only what the resume actually provides:

1. **Header** — name, professional title, city/state, and links (email, LinkedIn, GitHub, portfolio) as available.
2. **Summary** — a short professional summary. If the resume has one, use it. If not, write two or three sentences drawn strictly from its content.
3. **Experience** — role, organization, dates, and bullet points, most recent first.
4. **Skills** — grouped sensibly if the resume groups them.
5. **Education and certifications.**
6. **Footer** — name and current year.

Layout requirements:

- Responsive: readable on a phone and on a laptop.
- A **light/dark theme toggle**, defaulting to the visitor's system preference.
- Semantic HTML with a sensible heading hierarchy, and a page `<title>` and meta description built from the name and title.

## 7. Out of scope

Do not build any of the following. If you think one is needed, say so and stop instead of building it.

- Authentication, login, or any admin area
- A CMS, blog, or database-backed content
- Analytics or third-party tracking
- A contact form *(this comes later, in the phase 2 section below — do not build it now)*
- Multiple pages or routes
- Downloadable PDF generation

## 8. Database safety

- Never run a destructive or schema-pushing command against the database. Specifically: no `drizzle-kit push`, no `DROP`, no `TRUNCATE`, no bulk `DELETE`, no reset commands.
- When database changes are needed, output the SQL for a human to review and run in the Supabase SQL editor. Do not apply it yourself.

## 9. Deliverables

1. A public GitHub repository at `https://github.com/{{GITHUB_USERNAME}}/{{REPO_NAME}}` with the full project committed.
2. A clear initial commit and push.
3. A live Vercel deployment.
4. A short `README.md` with the live URL, how to run locally, and anything you deliberately skipped.
5. A closing summary that reports: the live URL, the repo URL, and what personal information you removed under section 4.

## 10. Acceptance criteria

- [ ] Site is live on a public Vercel URL and loads without errors.
- [ ] All content traces back to the attached resume; nothing is invented.
- [ ] No residential address anywhere on the site or in the repo.
- [ ] No phone number unless explicitly opted in above.
- [ ] Light/dark toggle works and respects system preference on first load.
- [ ] Readable on a phone-width viewport.
- [ ] Repo is pushed and the Vercel build is green

      
# Iteration ideas

Once the site is live, submit small change requests as separate tasks and watch the commit → push → Vercel build → live cycle:

- Change the accent color to something you actually like.
- Make the header a hero section with your initials as a monogram.
- Add a print stylesheet so the page prints cleanly as a one-pager.
- Reorder the sections so skills sit above experience.
- Add subtle hover states to the links.
- Add a "currently looking for" line under the header.
