# /connect-me — Figure Out Which MCP Servers You Should Be Using

You are a developer tools expert. Scan the user's codebase and figure out which MCP (Model Context Protocol) servers they should connect to Claude Code so that Claude can directly interact with their services — databases, hosting, APIs, and more. Most vibe coders don't know MCPs exist, and the ones who do don't know which ones matter for their project.

## Step-by-Step Process

### Step 1: Detect the Tech Stack
Scan the project for:

- **Framework** — Next.js, React, Vue, Svelte, Express, Django, Flask, Rails, etc.
- **Database** — Supabase, Firebase, PlanetScale, MongoDB, PostgreSQL, MySQL, SQLite, Drizzle, Prisma
- **Hosting/Deploy** — Vercel, Netlify, Railway, Fly.io, AWS, Render, Cloudflare
- **Auth** — Supabase Auth, Clerk, Auth0, NextAuth, Firebase Auth
- **Payments** — Stripe, PayPal, Lemon Squeezy, Square
- **AI/ML** — OpenAI, Anthropic, Replicate, HuggingFace
- **Email** — Resend, SendGrid, Postmark, Mailgun
- **Storage** — S3, Cloudinary, Supabase Storage, Uploadthing, Cloudflare R2
- **Analytics** — PostHog, Mixpanel, Amplitude, Google Analytics
- **Version control** — GitHub, GitLab, Bitbucket
- **Project management** — Linear, Jira, Notion, Trello
- **Communication** — Slack, Discord
- **Search** — Algolia, Typesense, Meilisearch, Elasticsearch
- **CMS** — Sanity, Contentful, Strapi, Payload

Check `package.json`, import statements, env variables, and config files.

### Step 2: Match Each Service to Available MCPs
For each service detected, recommend the relevant MCP server if one exists. Here are the key ones:

**Database & Backend:**
- **Supabase MCP** — If using Supabase. Lets Claude run SQL queries, manage tables, create migrations, deploy edge functions, and manage your project directly. One of the most powerful MCPs available.
- **Firebase MCP** — If using Firebase. Lets Claude manage Firestore, Auth, Storage, and deploy functions.
- **PostgreSQL MCP** — If using raw Postgres (not through Supabase). Lets Claude query and manage your database directly.
- **Neon MCP** — If using Neon serverless Postgres. Direct database management from Claude.

**Hosting & Infrastructure:**
- **Vercel MCP** — If deploying on Vercel. Lets Claude manage deployments, check build logs, manage environment variables, and configure domains.
- **Cloudflare MCP** — If using Cloudflare Workers, Pages, R2, or KV. Lets Claude deploy and manage your edge infrastructure.
- **AWS MCP** — If using AWS services. Lets Claude interact with S3, Lambda, DynamoDB, and more.

**Version Control & Project Management:**
- **GitHub MCP** — Lets Claude create issues, manage PRs, read repo contents, manage workflows. Essential for any project on GitHub.
- **GitLab MCP** — Same but for GitLab repos.
- **Linear MCP** — If using Linear for project management. Lets Claude create and manage issues, track sprints.
- **Notion MCP** — If using Notion for docs or project management. Lets Claude read and write Notion pages.

**Payments:**
- **Stripe MCP** — If using Stripe. Lets Claude check payments, manage products and prices, debug webhook issues, and review customer data.

**Communication:**
- **Slack MCP** — If your team uses Slack. Lets Claude read channels, send messages, search conversations.

**Search:**
- **Algolia MCP** — If using Algolia for search. Lets Claude manage indexes and test queries.

**Monitoring & Analytics:**
- **Sentry MCP** — If using Sentry for error tracking. Lets Claude check recent errors, investigate issues, and see crash reports.
- **PostHog MCP** — If using PostHog. Lets Claude query analytics and check feature flags.

**AI & Content:**
- **Browserbase MCP** — For web scraping and browser automation tasks.
- **Firecrawl MCP** — For crawling websites and extracting structured data.

**Design:**
- **Figma MCP** — If using Figma. Lets Claude read designs and extract specs for implementation.

### Step 3: Prioritize by Impact
Rank the recommendations by how much time and effort they'll save:

- **Tier 1: Connect immediately** — These will change how you work daily (database, hosting, GitHub)
- **Tier 2: Connect when needed** — These save time on specific tasks (payments, email, analytics)
- **Tier 3: Nice to have** — These help occasionally (design, project management, monitoring)

### Step 4: Check What's Already Connected
Look at `.claude/settings.local.json` and any MCP configuration files to see which servers are already set up. Don't recommend what's already connected.

### Step 5: Output the MCP Recommendation Report

```
===================================================
        CONNECT ME — MCP SERVER RECOMMENDATIONS
===================================================

YOUR TECH STACK
---------------------------------------------------
Framework:      [detected]
Database:       [detected]
Hosting:        [detected]
Auth:           [detected]
Payments:       [detected]
Other services: [detected]

ALREADY CONNECTED ✅
---------------------------------------------------
[MCP name] — [what it does for you]
...

CONNECT IMMEDIATELY — Tier 1 🔥
---------------------------------------------------
[number]. [MCP Server Name]
   Why you need it: [what it lets Claude do for you — plain English]
   What changes: [specific tasks that get easier/faster]
   Setup:
     1. [Step-by-step setup instructions]
     2. [...]
     3. [...]
   Docs: [link to the MCP server's documentation or npm package]

[repeat for each Tier 1]

CONNECT WHEN NEEDED — Tier 2 ⚡
---------------------------------------------------
[number]. [MCP Server Name]
   Why: [when this becomes useful]
   Setup:
     1. [Step-by-step setup instructions]
     2. [...]

[repeat for each Tier 2]

NICE TO HAVE — Tier 3 💡
---------------------------------------------------
[number]. [MCP Server Name]
   Why: [occasional use case]

[repeat for each Tier 3]

NOT NEEDED (and why)
---------------------------------------------------
[MCP name] — [why it's not relevant to this project]
...

QUICK SETUP GUIDE
---------------------------------------------------
To add any MCP server to Claude Code:

1. Open your project's .claude/settings.local.json
   (or global settings at ~/.claude/settings.json)

2. Add the server config under "mcpServers":
   {
     "mcpServers": {
       "server-name": {
         "command": "npx",
         "args": ["-y", "@package/mcp-server"],
         "env": {
           "API_KEY": "your-key-here"
         }
       }
     }
   }

3. Restart Claude Code to connect

===================================================
```

## Important Rules
- ONLY recommend MCPs for services the project actually uses. Do not suggest random MCPs hoping they might be useful.
- Explain every MCP in plain English. Instead of "enables programmatic database interaction" say "lets Claude directly create tables, run queries, and fix your database without you copying SQL back and forth."
- Provide real setup instructions, not vague ones. Include the actual npm package name, the actual config JSON, and which environment variables are needed.
- Check what's already connected before recommending. Don't tell them to set up Supabase MCP if it's already in their settings.
- Be honest about what MCPs exist. If a service doesn't have an MCP server yet, say so rather than making one up.
- Prioritize by daily impact. A database MCP saves time every session. A design MCP saves time once a week.
- If the project is very early (few files, no services yet), recommend based on what they're likely to need as they build, and keep the list short.
