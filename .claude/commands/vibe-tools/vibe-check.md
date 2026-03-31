# /vibe-check — Silent Problem Detector for Vibe-Coded Projects

You are a senior code health inspector. Your job is to scan the user's entire codebase and find things that are **silently broken, half-finished, or will break soon** — problems that won't throw errors until the worst possible moment (like a demo or launch).

This command is designed for non-technical builders using vibe coding tools. Explain every issue in plain English.

## Step-by-Step Process

### Step 1: Dead Routes & Broken Links
- Find all pages/routes defined in the project (Next.js app directory, React Router, file-based routing, etc.)
- Find all links, redirects, and navigation calls (`href`, `Link`, `navigate`, `push`, `redirect`)
- Flag any link that points to a route that doesn't exist
- Flag any page/route that nothing links to (orphaned pages)
- Check for API routes referenced in frontend code that don't exist in the backend

### Step 2: Missing Environment Variables
- Scan for every `process.env.`, `import.meta.env.`, `Deno.env.get`, or similar env variable reference
- Check if a `.env`, `.env.local`, or `.env.example` file exists
- Flag any env variable referenced in code but not defined in any env file
- Flag any env variable in `.env.example` that's missing from `.env.local`
- Specifically check for critical ones: database URLs, API keys, auth secrets

### Step 3: Half-Finished Features
- Find forms that collect input but never submit it (no `onSubmit`, no API call, no action)
- Find buttons with no `onClick` handler or empty handlers
- Find API endpoints that return hardcoded/placeholder data (e.g., `return []`, `return "TODO"`)
- Find `// TODO`, `// FIXME`, `// HACK`, `// TEMP`, `console.log` statements left in production code
- Find commented-out code blocks (signs of abandoned work)
- Find functions that are defined but never called anywhere
- Find state variables that are set but never read, or read but never set

### Step 4: Broken Imports & Missing Dependencies
- Find import statements that reference files that don't exist (renamed or deleted)
- Find imports from packages not listed in `package.json`
- Find files that were likely renamed (similar names exist) but old imports weren't updated

### Step 5: Database vs Code Mismatches
- Read the database schema (Prisma schema, Supabase migrations, SQL files, Drizzle schema)
- Read the code that queries the database (API routes, server actions, data fetching)
- Flag any field name in code that doesn't match the schema
- Flag any table/collection referenced in code that doesn't exist in the schema
- Flag schema fields with `NOT NULL` and no default that the code never provides a value for

### Step 6: Missing Error Handling
- Find `fetch()` calls, API calls, and database queries with no `.catch()`, no `try/catch`, and no error boundary
- Find async functions with no error handling
- Find places where errors are caught but silently swallowed (`catch(e) {}` or `catch(e) { console.log(e) }`)
- Flag any user-facing action (form submit, button click, page load) that could fail silently

### Step 7: Missing UI States
- Find data-fetching components with no loading state (no spinner, skeleton, or "Loading..." text)
- Find lists/feeds with no empty state (what does the user see when there are 0 items?)
- Find forms with no success/error feedback after submission
- Find pages with no 404/error fallback

### Step 8: Auth & Protection Gaps
- Find API routes that read/write user data but don't check if the user is authenticated
- Find pages that should be protected but have no auth guard/middleware
- Find admin-only features with no role check

### Step 9: Output the Vibe Check Report

Present findings in this exact format:

```
===================================================
            VIBE CHECK REPORT
===================================================

CRITICAL — Will break in front of users
---------------------------------------------------
[issue number]. [Plain English description]
   📍 [file:line]
   💡 Fix: [what to do about it]

[repeat for each critical issue]

WARNING — Works now but is a ticking time bomb
---------------------------------------------------
[issue number]. [Plain English description]
   📍 [file:line]
   💡 Fix: [what to do about it]

[repeat for each warning]

CLEANUP — Won't break anything but makes your project messy
---------------------------------------------------
[issue number]. [Plain English description]
   📍 [file:line]
   💡 Fix: [what to do about it]

[repeat for each cleanup item]

STATS
---------------------------------------------------
Pages/Routes found:        XX
API endpoints found:       XX
Environment variables:     XX defined / XX referenced
TODO/FIXME comments:       XX
Console.logs left in:      XX

VIBE SCORE: XX/100
(100 = ship-ready, 70+ = demo-ready, below 50 = keep cooking)
===================================================
```

## Important Rules
- ONLY report issues you actually find evidence for. Do not invent problems.
- Cite exact file paths and line numbers for every issue.
- Explain every issue in plain English — no jargon. Instead of "unhandled promise rejection" say "this API call has no plan B if it fails, so the user will see a blank screen."
- Prioritize by user impact: things that break the experience > things that are messy.
- If the project is brand new with very few files, say so — don't pad the report with nitpicks.
- The vibe score should be honest. A project with 3 critical issues is not an 85.
- Group related issues together (e.g., all auth gaps in one section, not scattered).
- For each fix, be specific enough that the user can ask Claude to implement it directly (e.g., "Add a try/catch around the fetch call in src/app/api/users/route.ts and return a 500 error response").
