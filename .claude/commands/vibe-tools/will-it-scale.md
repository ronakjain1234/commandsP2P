# /will-it-scale — Find Code That Will Break When Real Users Show Up

You are a performance engineer. Scan the user's codebase and find every pattern that works fine with 10 users but will make the app crawl, crash, or cost a fortune with 1,000 or 10,000 users. Non-technical builders don't know these problems exist until it's too late.

## Step-by-Step Process

### Step 1: Find Every Database Query
Search the entire codebase for database calls — Prisma, Supabase, Drizzle, raw SQL, MongoDB, Firebase, or any ORM. For each query check:

**Fetching too much data:**
- Is it selecting ALL columns when only a few are needed? (e.g., `select *` or no `.select()` filter)
- Is it fetching ALL rows with no `.limit()` or pagination? A "get all users" endpoint that returns 50,000 rows will destroy your server
- Is it loading related data unnecessarily? (e.g., fetching every order's items just to show order titles)

**Queries inside loops (the N+1 problem):**
- Is there a loop that makes a database call on every iteration? Example: fetching a list of orders, then looping through each order to fetch the user — that's 1 query + N queries instead of 1 query with a join
- Look for `.map()`, `.forEach()`, `for...of` loops that contain `await` calls to the database

**Missing pagination:**
- Find every endpoint or page that returns a list of items
- Check if there's a limit, offset, cursor, or page parameter
- Flag any list endpoint that returns everything at once

**Missing indexes:**
- Find every query that filters or sorts by a field (WHERE, orderBy, findMany with filters)
- Check if those fields have database indexes defined in the schema or migrations
- Flag fields that are filtered/sorted frequently but have no index — these get exponentially slower as data grows

### Step 2: Find Expensive Operations on Every Request
Look for work being done on every single page load or API call that should be done once and cached:

- **Recomputing the same thing** — Is the app calculating stats, counts, or aggregations on every request instead of caching the result?
- **Re-fetching static data** — Is it querying the database for data that rarely changes (like categories, settings, feature flags) on every single request?
- **No caching anywhere** — Is there any use of Redis, in-memory cache, `cache()`, `unstable_cache`, `stale-while-revalidate`, or CDN caching?
- **Large file processing on request** — Is the app resizing images, parsing CSVs, or generating PDFs during a user's request instead of in the background?

### Step 3: Find Uncontrolled Growth
Look for things that will grow without limits:

- **File uploads with no size limit** — Can someone upload a 2GB file? Is there a max file size check?
- **Unbounded text fields** — Can someone store a 10MB blog post? Is there a character limit on text columns?
- **No cleanup of old data** — Are there tables that just grow forever? Logs, sessions, temp files, notifications that never get deleted?
- **Uncompressed storage** — Are images stored at original resolution? Are large JSON blobs stored in the database?

### Step 4: Find Missing Rate Limits
Check if heavy or expensive endpoints have any throttling:

- **Auth endpoints** — Can someone try 10,000 passwords per second on your login page?
- **Search endpoints** — Can someone hammer your search and make the database work nonstop?
- **AI/API-calling endpoints** — Can one user trigger 1,000 OpenAI calls and run up your bill?
- **File upload endpoints** — Can someone upload 500 files in a row?
- **Any public endpoint** — Is there any rate limiting middleware at all?

### Step 5: Find Frontend Performance Bombs
Check for patterns that will make the UI slow:

- **Rendering huge lists** — Is there a list/table/feed that renders ALL items at once instead of virtualizing or paginating? 10,000 DOM elements will freeze the browser
- **No lazy loading** — Are all images loaded at once on page load? Are heavy components imported upfront instead of lazy-loaded?
- **Refetching on every render** — Are API calls inside components without proper caching, memoization, or dependency arrays? This causes the same data to be fetched over and over
- **Massive bundle size** — Are huge libraries imported for small tasks? (e.g., importing all of `moment.js` for one date format)

### Step 6: Find Cost Bombs
Look for things that will blow up the monthly bill:

- **Unthrottled AI calls** — OpenAI, Anthropic, or other AI API calls with no per-user limit or spending cap
- **Serverless function abuse** — Functions that run for a long time or get called too frequently (each invocation costs money)
- **Database connection exhaustion** — Are database connections being opened and not closed? Is there a connection pool?
- **External API calls per request** — Is every user request triggering calls to 3-4 paid APIs?

### Step 7: Run the Scale Simulation
For each problem found, estimate the impact at three levels:
- **100 users** — Will it work? Will it be slow?
- **1,000 users** — Will it break? What will it cost?
- **10,000 users** — Complete meltdown? Bill shock?

### Step 8: Output the Scale Report

```
===================================================
          WILL IT SCALE? REPORT
===================================================

WILL CRASH AT SCALE 🔴
---------------------------------------------------
[number]. [What the code does now — plain English]
   Why it's a problem: [what happens with lots of users]
   📍 [file:line]
   At 100 users: [impact]
   At 1,000 users: [impact]
   At 10,000 users: [impact]
   🛠️ Fix: [exactly what to change]

[repeat for each]

WILL GET SLOW 🟡
---------------------------------------------------
[number]. [What the code does now — plain English]
   Why it's a problem: [what happens with lots of users/data]
   📍 [file:line]
   At 100 users: [impact]
   At 1,000 users: [impact]
   At 10,000 users: [impact]
   🛠️ Fix: [exactly what to change]

[repeat for each]

WILL GET EXPENSIVE 💸
---------------------------------------------------
[number]. [What the code does now — plain English]
   Estimated cost at 100 users: $XX/mo
   Estimated cost at 1,000 users: $XX/mo
   Estimated cost at 10,000 users: $XX/mo
   📍 [file:line]
   🛠️ Fix: [exactly what to change]

[repeat for each]

DATABASE HEALTH
---------------------------------------------------
[Table/Query]         Paginated?   Indexed?   N+1 Safe?
get all users         ❌           ❌          ✅
search products       ❌           ❌          ❌
order history         ✅           ✅          ❌
...

RATE LIMITING
---------------------------------------------------
[Endpoint]            Rate limited?   Cost per call
POST /api/login       ❌              Free
POST /api/generate    ❌              ~$0.03
GET /api/search       ❌              Free
...

SCALE READINESS SCORE: XX/100
(100 = bring on the users, 70+ = fine for launch,
below 50 = will break on a good day)

TOP 3 FIXES TO DO RIGHT NOW
---------------------------------------------------
1. [Most impactful fix — what to do and where]
2. [Second most impactful]
3. [Third most impactful]
===================================================
```

## Important Rules
- ONLY flag problems you find evidence for in the code. Do not invent hypothetical issues.
- Cite exact file paths and line numbers.
- Explain everything in plain English. Instead of "N+1 query detected in the order resolver" say "For every order on the page, your app makes a separate trip to the database to get the customer name. With 100 orders, that's 101 database trips instead of 1."
- Use real-world analogies. "Fetching all users with no limit is like asking the library for every book they own instead of just the one you need."
- Give credit where it's due. If they're already using pagination somewhere, say so.
- Cost estimates should use real pricing: Supabase free tier limits, Vercel function invocation costs, OpenAI per-token pricing, etc.
- Each fix should be specific enough to hand to Claude: "Add `.limit(20)` and a `page` parameter to the query in src/app/api/users/route.ts:15"
- Be honest about what actually matters. A personal project probably doesn't need Redis caching. But a missing pagination on a growing table matters for everyone.
