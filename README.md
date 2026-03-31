# commandsP2P

Custom Claude Code slash commands built for the **Pitch to Prototype** class at UT Austin. These commands help non-technical builders ship better startups using vibe coding tools.

## Setup

1. Clone this repo
2. Open Claude Code from the project directory
3. Type `/` to see all available commands

---

## Startup Commands

Tools for founders to understand and strengthen their business.

### `/startup:unit-economics`
Reads your actual codebase to build a real unit economics model. Scans for pricing logic, paid API usage (OpenAI, Twilio, Stripe, etc.), infrastructure costs, and user acquisition channels. Outputs ARPU, cost per user, gross margin, LTV:CAC ratio, and flags cost landmines — all grounded in your code, not a generic template.

### `/startup:regulatory-scan`
Scans your codebase for compliance risks. Inventories every PII field in your database, audits every data collection point with file and line references, checks against GDPR, CCPA, COPPA, HIPAA, and SOC2, flags security red flags, and outputs a prioritized remediation plan with a compliance score.

---

## Vibe Tools Commands

Tools that give non-coders the instincts of a senior engineer — catching the problems you don't know to look for.

### `/vibe-tools:vibe-check`
The health inspection for your app. Finds things that are silently broken, half-finished, or will break soon: dead routes, missing environment variables, forms that don't submit, broken imports, orphaned files, database mismatches, and missing error handling. Outputs a vibe score so you know if you're demo-ready or need to keep cooking.

### `/vibe-tools:undo-map`
For when you've broken your app and don't know what changed. Traces all recent code changes, explains each one in plain English, identifies which change likely caused the problem, and gives you two options: a surgical fix or a safe rollback. Designed for people who don't think in git commits.

### `/vibe-tools:break-my-app`
Tries to break your app the way real users will. Tests every input for empty data, extreme values, special characters, and script injection. Checks for double-submit bugs, unauthorized URL manipulation, and missing error states. Outputs a toughness score and exact fixes for every weakness found.

### `/vibe-tools:will-it-scale`
Finds code that works fine now but will crash, crawl, or cost a fortune when real users show up. Catches database queries with no pagination, queries inside loops, missing rate limits, unthrottled AI calls, frontend performance bombs, and cost explosions. Shows the impact at 100, 1,000, and 10,000 users.

### `/vibe-tools:secret-doors`
Finds backdoors, debug leftovers, and test shortcuts you forgot to remove. Catches hardcoded test accounts, debug endpoints, disabled security checks, API keys in source code, exposed admin panels, and leftover test data. Everything that a bad actor could exploit or that would embarrass you in a demo.
