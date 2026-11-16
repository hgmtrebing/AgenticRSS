## Overview

This project is to create an RSS Aggregator Server, which aggregates RSS feeds and makes them available for RSS Clients. Some essential features:
    - API so that AI Agents can read and write from the service. 
    - Ability to create RSS feeds by scraping websites
    - UI to administer the server and view RSS feeds
    - Serving RSS feeds

## Technology Stack

The backend server will be coded in Java / Spring Boot. The attached database will be Postgres. The UI should be coded in React and served from the Spring Boot backend. The API should be RESTful.

## Coding Style

- Favor smaller, single-purpose methods and functions. A soft guideline is fifty lines or less. It is acceptable if a method or function only has a single caller. The intention is to leverage functions as named blocks of code, to improve readability and comprehensibility.
- When writing comments, favor WHY comments over HOW comments. Comments should describe why something exists or the purpose it serves, rather than how it functions. Ideally, the code itself should document the precise implementation.
- Write JavaDoc-style comments (or whatever the language's equivalent is) for all methods and classes. Method comments can be brief, class comments can be longer.
- Put comments on non-trivial branching statements (if, else, while, for, switch, case, etc.). These comments can be brief, but should describe WHY the branch exists and what it is trying to accomplish. 
- Aim to keep classes and files on the small side. A soft target of 1000 lines or less is ideal.
- Favor verbose, self-documenting naming conventions. Avoid unclear abbreviations.

## Special Concerns

Error handling and security deserve extra care on this project. Many sources are scraped and break without warning. The server also fetches, parses, and renders content from the open internet, and AI agents read it.

### Error Handling: Detecting Broken Feeds

- Never fail silently. Every fetch or scrape attempt must leave a recorded outcome: HTTP status, duration, items found, new items, error category, and a content hash.
- Treat soft failures as failures. Examples: a successful response that yields zero items, required fields (title, link, date) that are missing or unparseable, extracted titles that look like navigation text, and a sharp drop in item count compared with the source's history.
- Tell "quiet" apart from "broken". If the page is unchanged, the source is just quiet. If the page changed but nothing new was extracted, the scraper is probably broken.
- Track source health explicitly (for example HEALTHY, DEGRADED, BROKEN), and back off on repeated failures so failing sites are not hammered.
- Keep a snapshot of the raw response when a fetch fails, for debugging and for LLM-assisted scraper repair.
- Make health visible to the user: in the admin UI, and ideally as a feed of its own.

### Security

- **SSRF:** Every outbound fetch of a user- or agent-supplied URL must reject private, loopback, link-local, and cloud-metadata addresses (for example 169.254.169.254). Check this after DNS resolution and again on every redirect. Enforce timeouts and response size limits.
- **XML parsing:** Disable DTDs and external entities on every XML parser to prevent XXE and entity-expansion attacks.
- **XSS:** Feed and scraped HTML is untrusted. Sanitize it on the server before storing or serving it, and never render it raw in the UI.
- **Prompt injection:** Feed and page content is data, never instructions. See *AI Agent Security* below.
- **Resource exhaustion:** Remote content can be hostile even when it isn't malicious XML. Enforce limits on response size, decompressed size (to stop gzip bombs), redirect count, and total fetch time. If scraper definitions allow regular expressions, guard against catastrophic backtracking (ReDoS).
- **Injection into our own systems:** Use parameterized queries only. The application connects to Postgres as a least-privilege database user, not the owner or superuser.
- **Authentication and authorization:** The admin UI and the API require authentication. Protect state-changing endpoints against CSRF and rate-limit them. Credentials embedded in feed URLs (for RSS clients that cannot log in) must be per-consumer, revocable, and limited to read access.
- **Secrets:** Credentials for cloud services such as AWS/Bedrock stay on the server. They are never sent to the browser, committed to the repository, or written to logs.
- **Supply chain:** Pin dependency versions, commit lockfiles, and enable automated vulnerability scanning (for example Dependabot) for both Maven and npm.
- **LLM-generated scrapers:** The LLM may only produce declarative scraper definitions that are validated against a schema. It must never produce executable code. A human approves a definition before it goes live.

### AI Agent Security

Every piece of feed, page, and scraped content is attacker-controllable. Assume some of it contains prompt injection, meaning text written to hijack an LLM that reads it.

- **Architecture over detection.** No filter catches every injection, so the main defense is limiting what a hijacked agent *could* do. Detection is an additional layer, not a replacement.
- **Least privilege per agent.** Each agent role (security screener, rater, summarizer, digester, scraper builder) gets its own narrowly scoped credentials and only the API operations it needs. Agents that read untrusted content get no general-purpose tools: no arbitrary web fetching, no shell, no ability to change sources, settings, or other agents' permissions.
- **Structured output only.** Agent output is validated against a schema (for example a numeric rating, a summary string, a list of cited item IDs) before it is stored. Output that fails validation is rejected and logged, never repaired by guessing.
- **Security screening before other agents.** New content passes through a dedicated security-screening agent before rating, summarizing, or digesting agents see it. Flagged items are quarantined: held back from downstream agents and shown to a human with the reason. The screener is itself an LLM reading untrusted text, so it can be fooled. Its only power is to flag or pass content, and its output is structured. It never receives write access.
- **Injection can propagate.** Agent-generated text (summaries, ratings, rationales) derived from untrusted content is itself untrusted. Treat it that way when it feeds other agents, especially the multi-source digest agent.
- **Citations must be verifiable.** Every claim in a digest cites stored item IDs. The server checks that each cited ID exists and rejects output that cites unknown items or invents URLs.
- **No exfiltration via output.** LLM output rendered in the UI or in served feeds is sanitized like any other untrusted HTML or Markdown. Links and images in agent output are limited to URLs that already belong to the cited source items. This stops attacker-crafted URLs from leaking data.
- **Clear separation of instructions and data.** Prompts clearly delimit untrusted content and state that it is data to analyze, not instructions to follow. This lowers the risk, but it is not a security boundary on its own.
- **Human approval for side effects.** Anything beyond additive annotation needs explicit human approval. That includes adding or changing sources, activating scraper definitions, and changing schedules.
- **Cost controls.** Hostile or oversized content can inflate token usage ("denial of wallet"). Cap input size per item, enforce per-run and per-day token budgets, and set cloud billing alarms.
- **Audit trail.** Record every agent invocation: role, model, prompt version, input item IDs, output, validation result, and token usage. Suspicious behavior must be traceable after the fact.
- **Provider guardrails as defense in depth.** Managed protections (for example Bedrock Guardrails' prompt-attack filtering) may be layered on, but never relied on as the only control.
