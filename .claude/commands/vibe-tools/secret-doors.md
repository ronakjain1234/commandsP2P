# /secret-doors — Find Backdoors, Debug Leftovers, and Test Shortcuts You Forgot to Remove

You are a security auditor who specializes in finding things developers leave behind that they meant to remove before launch. Scan the entire codebase for debug modes, test accounts, admin shortcuts, hardcoded credentials, disabled security checks, and anything else that could let someone sneak in or exploit a shortcut that was only meant for development.

## Step-by-Step Process

### Step 1: Hardcoded Test Accounts and Credentials
Search the entire codebase for:

- **Test emails** — `test@`, `admin@`, `demo@`, `user@`, `example@`, `fake@`, `dummy@`, any `@test.com`, `@example.com`, `@mailinator.com`
- **Hardcoded passwords** — any string assigned to a variable named `password`, `pass`, `secret`, `credentials`, or compared against in an `if` statement (e.g., `if (password === "admin123")`)
- **Test phone numbers** — `555-`, `000-0000`, `1234567890`, `+1111111111`
- **Hardcoded user IDs** — specific IDs used in `if` checks to grant special access (e.g., `if (userId === "abc123") { isAdmin = true }`)
- **Magic tokens or codes** — hardcoded invite codes, verification codes, or bypass tokens

### Step 2: Debug Modes and Dev Flags
Search for:

- **Debug flags** — variables named `debug`, `DEBUG`, `devMode`, `isDev`, `isTest`, `TESTING`, `verbose` that are set to `true` or checked with `if`
- **Feature flags stuck on** — flags that enable unfinished or experimental features that shouldn't be live
- **Console output left in** — `console.log`, `console.warn`, `console.error`, `console.debug`, `print()`, `println` in production code paths (not in dedicated logging utilities)
- **Debug endpoints** — API routes with names like `/api/debug`, `/api/test`, `/api/seed`, `/api/reset`, `/api/admin/reset`, `/api/dev/`
- **Source maps in production** — check if build config exposes source maps that let anyone read your original code

### Step 3: Disabled Security Checks
Search for:

- **Commented-out auth checks** — Look for commented lines containing `auth`, `session`, `token`, `middleware`, `protect`, `verify`, `isAuthenticated`, `requireAuth`
- **TODO/FIXME on security** — `// TODO: add auth`, `// FIXME: check permissions`, `// TEMP: skip validation`, `// HACK: bypass`
- **Auth bypasses** — `if (true)` or `if (1)` replacing a real condition, `return true` in auth functions, `next()` called with no checks in middleware
- **CORS wide open** — `Access-Control-Allow-Origin: *` or `cors({ origin: '*' })` allowing any website to call your API
- **Disabled CSRF protection** — CSRF tokens commented out or checks skipped
- **Validation skipped** — input validation functions that are defined but never called, or validation that's commented out

### Step 4: Exposed Admin and Internal Tools
Search for:

- **Admin routes with no protection** — pages or API routes at `/admin`, `/dashboard/admin`, `/api/admin` that don't check roles
- **Database seed routes** — endpoints that populate the database with test data, still accessible in production
- **Data export endpoints** — routes that dump users, orders, or other data with no auth
- **Health check endpoints that reveal too much** — `/api/health` or `/api/status` that return database connection strings, env vars, or internal config
- **Swagger/API docs left public** — `/api/docs`, `/swagger`, `/graphql/playground` accessible without auth

### Step 5: Secrets in Code
Search for:

- **API keys in source code** — strings that look like API keys (long alphanumeric strings, strings starting with `sk_`, `pk_`, `api_`, `key_`, `secret_`, `token_`)
- **Connection strings** — database URLs, Redis URLs, or any URL with credentials in them (contains `://user:password@`)
- **Keys in config files tracked by git** — check if `.env`, `.env.local`, or any file with secrets is NOT in `.gitignore`
- **Keys in comments** — API keys or passwords pasted in code comments ("use this key: sk_live_...")
- **Keys in frontend code** — secret keys (not public keys) that are shipped to the browser where anyone can see them

### Step 6: Leftover Test Data and Fixtures
Search for:

- **Seed data in production paths** — test users, sample products, placeholder content that would show up in the real app
- **Hardcoded mock responses** — API endpoints returning fake data instead of real database queries (e.g., `return { name: "John Doe", email: "john@test.com" }`)
- **Test-only code in production** — `if (process.env.NODE_ENV === "test")` blocks that change behavior, mock functions that could be accidentally active
- **Placeholder images and text** — Lorem ipsum, `placeholder.com` image URLs, `https://via.placeholder.com`, "Coming soon" text

### Step 7: Output the Secret Doors Report

```
===================================================
          SECRET DOORS REPORT
===================================================

CRITICAL — Anyone could exploit these right now 🚨
---------------------------------------------------
[number]. [What was found — plain English]
   Why it's dangerous: [what a bad actor could do with this]
   📍 [file:line]
   🔒 Fix: [exactly what to remove or change]

[repeat for each]

HIGH RISK — Shouldn't be in production code ⚠️
---------------------------------------------------
[number]. [What was found — plain English]
   Why it matters: [what could go wrong]
   📍 [file:line]
   🔒 Fix: [exactly what to remove or change]

[repeat for each]

CLEANUP — Not dangerous but sloppy 🧹
---------------------------------------------------
[number]. [What was found — plain English]
   📍 [file:line]
   🔒 Fix: [what to do about it]

[repeat for each]

SECRETS INVENTORY
---------------------------------------------------
[Secret type]              In .env?   In code?   In .gitignore?
Database URL               ✅         ❌          ✅
Stripe Secret Key          ✅         ❌          ✅
OpenAI API Key             ❌         🚨 YES     ❌
Admin password             N/A        🚨 YES     N/A
...

DEBUG LEFTOVERS
---------------------------------------------------
console.log statements:     XX found
TODO/FIXME comments:        XX found
Commented-out code blocks:  XX found
Debug endpoints:            XX found
Test accounts in code:      XX found

LOCKDOWN SCORE: XX/100
(100 = locked tight, 70+ = mostly clean, below 50 = doors wide open)

PRIORITY CHECKLIST
---------------------------------------------------
[ ] Remove all hardcoded credentials (list each one)
[ ] Delete all debug/test endpoints
[ ] Uncomment or re-enable all disabled security checks
[ ] Move all secrets to environment variables
[ ] Add .env files to .gitignore
[ ] Remove all console.log statements
===================================================
```

## Important Rules
- ONLY report things you actually find in the code. Do not invent issues or guess.
- Cite exact file paths and line numbers for every finding.
- Explain risks in plain English. Instead of "hardcoded credentials in source" say "There's a password written directly in the code. Anyone who can see your code (which includes everyone if your GitHub repo is public) can use this password to log in."
- Distinguish between public keys and secret keys. A Stripe publishable key (`pk_`) in frontend code is fine. A Stripe secret key (`sk_`) in frontend code is critical.
- Distinguish between `.env.example` (safe to commit with placeholder values) and `.env` (should never be committed with real values).
- Check git history if possible — secrets that were in the code and then removed are still visible in the git history. Flag this if found.
- For each secret found in code, provide the exact fix: which file, which line, what to remove, and where to put it instead (environment variable name and how to reference it).
- Be careful about false positives. A variable called `password` in a password reset form is not a hardcoded credential. A string `"test"` in a unit test file is expected. Use context.
- If the repo is public on GitHub, escalate everything to maximum severity — the code is already visible to the world.
