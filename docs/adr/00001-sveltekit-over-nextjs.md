# 0001 — SvelteKit over Next.js

**Statu*s:* Accepted · **Date:\*\* 2026-09-27

## Context

a5e-party-sheets is a login-gated, read-only site that shows Level Up: Advanced 5th Edition character sheets between sessions, fed by uploads of enriched Foundry VTT exports. It is a solo learning project for a developer with a Salesforce Apex background and no prior front-end or web-framework experience, working ~2 hours a day. The stack is otherwise fixed: TypeScript, Node, PostgreSQL, Drizzle, Zod, Better Auth, Vitest, Playwright, pnpm, plain CSS, managed hosting.

A nine-row comparison of SvelteKit 2.70 and Next.js 16.3 (2026-09-20) found most rows a wash: both integrate every chosen library, both handle multipart uploads with a one-line body-limit bump, both run on the hosting finalists, both have complete docs and agent-facing tooling. The rows that differ:

- **Request-cycle visibility.** SvelteKit exposes GET as an exported `load` function and POST as an exported `actions` object in a server-only file; both are plain, unit-testable functions. Next.js routes the same work through server components, encrypted server-action IDs, a combined RSC payload, and a cache layer whose defaults changed in v15 and v16.
- **Closeness to the platform.** A Svelte component is HTML with a script block and a scoped style block. React/JSX puts markup in JavaScript and adds hooks and a server/client boundary.
- **Toolchain weight.** Fresh minimal builds measured 1.85 s vs 7.07 s; installs 68 MB vs 345 MB.
- **Community size.** Next.js has ~20× the weekly downloads and ~3× the survey share; more tutorials, forum answers, and job listings.

## Decision

Use **SvelteKit 2.x with Svelte 5 runes**, pinned to a minor. Use `load` and form `actions` as the only server pattern; remote functions (experimental) are excluded. Default to server rendering with hydration. Start from the minimal official template and add tools one at a time as the build needs them.

## Rationale

The project's purpose is to learn how a web application works, not to learn React. SvelteKit puts fewer abstractions between the developer and the HTTP request, and its component file resembles the web platform being learned. The cost is a smaller community; the developer's primary help sources are an AI assistant and official docs, where both frameworks are now comparably served. No résumé or hiring goal is attached to this project.

## Consequences

- A future Next.js version of this app is a **rebuild, not a port**: the concepts transfer, the code does not. That rebuild is a known candidate for a later learning project.
- Generated code must be checked for Svelte 4 syntax (`export let`, `on:click`); only runes syntax is accepted.
- Migrate to SvelteKit 3 with `sv migrate sveltekit-3` once it is stable (config moves to `vite.config.ts`, `$lib` becomes `#lib`; requires Vite 8 and Node ≥ 22.12).
- Hosting and upload decisions are unaffected; adapter-node's 512 KB default body limit must be raised.

## Alternatives considered

- **Next.js 16 (App Router).** Rejected for now: more layers to learn before the request cycle is visible; higher churn in load-bearing conventions across recent majors. Preferred if the goal were industry alignment or hiring.
- **Plain SPA + separate API.** Ruled out at charting: two projects to learn instead of one, and no server-rendered pages.

What each section is for, since you'll write the next ones yourself:

- Status and Date. ADRs are never edited once accepted. If you change your mind later, you write a new record and set this one's status to "Superseded by 0007". The date tells a future reader how old the reasoning is.
- Context. The situation and the facts as they were known. Written so a reader who has never seen the project understands the constraints. This section is what makes the decision look reasonable in a year, even if it turns out wrong.
- Decision. One paragraph, present tense, stated as a fact. No hedging.
- Rationale. Why this option over the others, tied to the context. This is the part people skip and later regret.
- Consequences. What you've committed to and what it costs you. Both the good and the bad. This is where the "rebuild, not a port" warning lives so nobody assumes a cheap migration later.
- Alternatives considered. Proof that the decision was a choice, not a default. Also saves the next person from re-researching a rejected option.
