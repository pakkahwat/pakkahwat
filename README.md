# Pakkahwat Chuesaard

Software engineer, 5 years. I build and run the internal systems a business actually depends on — stock verification across a national branch network, an auction platform wired directly into Oracle ERP, and the sync that keeps them fed.

Based in Ayutthaya, Thailand. Open to remote and hybrid roles.

---

## Pundit — an AI vs human prediction league

**Live → [pundit.devda.fyi](https://pundit.devda.fyi) · Code → [pakkahwat/pundit](https://github.com/pakkahwat/pundit)**

A football prediction league where LLM agents compete against people under identical rules and deadlines. The hard part was not wiring up the model. It was making *"identical rules"* true in a way that a bug could not undo.

Every fairness guarantee is enforced in the database, not in the UI:

| Guarantee | How it is enforced |
| --- | --- |
| Predictions lock at kickoff | The deadline lives inside the same SQL statement as the write, checked against Postgres `now()` — no trust in client clocks, and no gap between checking and writing |
| Nobody sees another player's pick before lock | `FORCE ROW LEVEL SECURITY` on the predictions table, so a badly written query still cannot leak them |
| Re-running the scorer never drifts | Idempotent upsert keyed on `result_version`; a corrected match result re-scores automatically, and an unchanged one touches nothing |
| The AI gets no advantage | Agents write through the exact same function humans do, and their context is always filtered by the target match's kickoff time |
| Stays inside a free API quota | Cached with a stale-data fallback, so an upstream outage degrades the page instead of breaking it |

`Next.js 16` `TypeScript` `PostgreSQL` `Drizzle ORM` `Auth.js` `Tailwind CSS` `Vercel AI SDK`

**One thing I would do differently:** I designed the jobs to be idempotent but never wrote a real test suite. When the upstream API began serving stale standings, I found it in production and fixed it reactively over several commits. Next time the scoring logic gets tests before it gets features, and I do not trust an external API without a reconciliation step of my own.

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
| **Frontend** | Vue · Nuxt · Next.js · React |
| **Data** | PostgreSQL · MySQL · SQL Server · Oracle |
| **Ops** | GitLab CI/CD · Docker · Git · Postman |
| **Analysis** | Requirements · database design · system documentation |

---

## How I work with AI coding tools

I use Claude Code and Gemini daily, and I run Gemini as an automated reviewer on every merge request in our GitLab pipeline.

I treat generated code the way I treat any other contribution: I read it, I check what it touches, and I make sure I can explain it before it merges. The parts of Pundit that matter most — the deadline guard, the RLS policy, the idempotent scorer — carry comments explaining *why* they are written that way, because the reasoning is the part that has to survive after the code is forgotten.

---

📫 **pakkahwat.contact@gmail.com**
