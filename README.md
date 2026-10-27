# Pakkahwat Chuesaard

Software engineer, 5 years. I build and run the internal systems a business actually depends on — stock verification across a national branch network, an auction platform wired directly into Oracle ERP, and the sync that keeps them fed.

Based in Ayutthaya, Thailand. Open to remote and hybrid roles.

Outside work I ship small production apps end to end, from schema to deploy, on free tiers. The three below are live and in use.

---

## Pundit — an AI vs human prediction league

**Live → [pundit.devda.fyi](https://pundit.devda.fyi) · Code → [pakkahwat/pundit](https://github.com/pakkahwat/pundit)**

A football prediction league where LLM agents compete against people under identical rules and deadlines. The hard part was not wiring up the model. It was making *"identical rules"* true in a way that a bug could not undo. Visitors can browse leagues, standings and every revealed pick without signing in.

Every fairness guarantee is enforced in the database, not in the UI:

| Guarantee | How it is enforced |
| --- | --- |
| Predictions lock at kickoff | The deadline lives inside the same SQL statement as the write, checked against Postgres `now()` — no trust in client clocks, and no gap between checking and writing |
| Nobody sees another player's pick before lock | `FORCE ROW LEVEL SECURITY` on the predictions table, so a badly written query still cannot leak them |
| Re-running the scorer never drifts | Idempotent upsert keyed on `result_version`; a corrected match result re-scores automatically, and an unchanged one touches nothing |
| The AI gets no advantage | Agents write through the exact same function humans do, and their context is always filtered by the target match's kickoff time |
| Live scores never wipe the fixture list | Live scores from a second provider are overlaid field by field onto our own fixtures, never swapped in wholesale, because the two APIs use different IDs |
| Stays inside a free API quota | Cached with a stale-data fallback, so an upstream outage degrades the page instead of breaking it |

`Next.js 16` `TypeScript` `PostgreSQL` `Drizzle ORM` `Auth.js` `Tailwind CSS` `Vercel AI SDK`

**What I changed after launch:** the first version had idempotent jobs but no real test suite, so when the upstream API began serving stale data I found it in production and fixed it reactively over several commits. Since then the pure logic — the live-score overlay, the gate that decides whether a sync is needed, the AI probability and Thai-text guards — is covered by a 16-file `node:test` suite, and an *overdue results* check flags any match still without a result four hours after kickoff and alerts me on Discord, so I no longer rely on the API to tell me it is wrong. The scorer itself still leans on idempotency rather than its own tests; that is the next gap to close.

---

## PDF Tools — PDF tools that never upload your file

**Live → [pdf.devda.fyi](https://pdf.devda.fyi) · Code → [pakkahwat/pdf-tools](https://github.com/pakkahwat/pdf-tools)**

Twelve tools — merge, split, organize, rotate, compress, image ↔ PDF, PDF → Word, watermark, page numbers, protect and unlock — running entirely in the browser with pdf-lib, pdf.js and WebAssembly. No file leaves the device, so there is no server to pay for and nothing to leak.

The hard part was PDF → Word for Thai documents. Plain pdf2docx scored 49.7 / 100 on a Thai job-application form; it now scores 99.0, and an 11-page table document holds at 98.2.

| Problem | What I did |
| --- | --- |
| "Looks close" is not a measurement | A scorer opens each .docx in real Microsoft Word, exports it back to PDF and compares the page images with the original. Every change to the converter was judged by that number, not by eye |
| Thai has no spaces between words, and Word on machines without Thai enabled breaks lines in the wrong places | Insert zero-width spaces at word boundaries found by `Intl.Segmenter`, so lines break where the PDF did |
| The output looked right only on machines that had the fonts | Embed the matching open-licensed (OFL) fonts in the .docx |
| The converter is a Python library | Run pdf2docx + PyMuPDF on Pyodide inside a Web Worker — about 40 MB on first use, then cached |

`Next.js 16` `TypeScript` `WebAssembly` `Pyodide` `mupdf.js` `pdf-lib` `pdf.js` `Vitest` `Playwright` `GitHub Actions`

Every push runs lint → typecheck → unit tests → build → E2E on desktop and mobile, and the E2E suite uploads real files and checks what comes back.

---

## Community equipment loans — built for volunteers on phones

**Code is private because the system holds real borrowers' data — happy to demo it**

A loan system for the tables, chairs and tents a local community lends out for events. The staff are volunteers, mostly non-technical and on phones, so every flow was cut to the fewest possible steps. Borrowers prove who they are with an ID-card photo and an on-screen signature, and the system prints the loan document.

- **The ID card is read on the phone itself.** Tesseract.js reads the card number and address zone by zone; a PaddleOCR PP-OCRv5 Thai model on onnxruntime-web reads the name, which Tesseract got wrong on real cards. No card image is sent to a cloud OCR service.
- **Personal data is deleted by default.** Once everything comes back in full, the card photo and signatures are deleted automatically. Until then they are served only through an authenticated, `no-store` route.
- **Stock that cannot drift.** Available = total − under repair − still out on open loans, computed from the loan lines on every read and never stored.
- **Several communities in one deployment**, each with its own staff, stock and documents; super admin / group admin / staff roles; every action in an audit log.
- **Self-service where it matters:** borrowers can file a request from a QR code without an account (rate-limited per phone and per IP, with a honeypot), and staff recover a forgotten password with a 6-digit PIN instead of waiting for an admin.
- **LINE alerts** to each staff group for new loans, returns and sign-up requests.

`Next.js 16` `TypeScript` `PostgreSQL (Neon)` `Drizzle ORM` `Tesseract.js` `onnxruntime-web` `LINE Messaging API` `Playwright`

**One thing I would do differently:** I started on Cloudflare Workers + D1 and ran into the free plan's 10 ms CPU limit within a day, then moved to Vercel + Neon. Now I check a platform's limits against the heaviest request before I write code for it.

---

## Work I cannot open-source, but can walk you through

**Check Stock** — replaced manual branch-by-branch stock counts across a national retail network with automated 24/7 random verification.
`PHP` `Vue` `Nuxt` `MySQL` `Cron`

**Project M** — auction and foreclosed-asset management, designed from the business process up. Replaced hand-keyed Oracle entry with direct ERP integration and cleared a standing backlog.
`PHP` `Vue` `Nuxt` `Oracle ERP` `SQL Server`

**ERP Connect** — bidirectional data sync between internal systems and Oracle ERP, delivered into a system already in daily use without downtime.
`PHP` `Oracle ERP` `REST API`

---

## Stack

| | |
| --- | --- |
| **Backend** | PHP · Node.js · TypeScript |
| **Frontend** | Vue · Nuxt · Next.js · React · Tailwind CSS |
| **Data** | PostgreSQL · MySQL · SQL Server · Oracle |
| **In the browser** | WebAssembly (Pyodide, mupdf.js, onnxruntime-web) · Web Workers |
| **Testing** | Vitest · Playwright · node:test |
| **Ops** | GitLab CI/CD · GitHub Actions · Docker · Vercel · Neon · Git · Postman |
| **Analysis** | Requirements · database design · system documentation |

---

## How I work with AI coding tools

I use Claude Code and Gemini daily, and I run Gemini as an automated reviewer on every merge request in our GitLab pipeline.

I treat generated code the way I treat any other contribution: I read it, I check what it touches, and I make sure I can explain it before it merges. The parts of Pundit that matter most — the deadline guard, the RLS policy, the idempotent scorer — carry comments explaining *why* they are written that way, because the reasoning is the part that has to survive after the code is forgotten. On PDF Tools the rule was numbers over impressions: a change to the Word converter counted only if the scorer said it helped.

---

📫 **pakkahwat.contact@gmail.com**
