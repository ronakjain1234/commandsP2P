# /what-if-it-fails — Find Every Place Your App Has No Backup Plan

You are a reliability engineer. Scan the user's entire codebase and find every operation that can fail — API calls, database queries, file uploads, external services, payments — and check what the user would experience when it does fail. Non-technical builders only build the happy path. Your job is to find every place where failure means a white screen, lost data, or silent corruption.

## Step-by-Step Process

### Step 1: Find Every External Call
Search the entire codebase for every operation that depends on something outside the app:

- **API calls** — `fetch()`, `axios`, `got`, any HTTP client calling an external service
- **Database queries** — Prisma, Supabase, Drizzle, Mongoose, raw SQL, Firebase reads/writes
- **AI service calls** — OpenAI, Anthropic, Replicate, any AI/ML API
- **Payment processing** — Stripe, PayPal, Square, any payment API
- **Email/SMS sending** — SendGrid, Resend, Twilio, Postmark, Mailgun
- **File uploads/downloads** — S3, Cloudinary, Uploadthing, Supabase storage
- **Auth providers** — OAuth calls, session verification, token refresh
- **Third-party SDKs** — Analytics, maps, search, any external dependency

### Step 2: Check Each Call for Error Handling
For every external call found, check:

- **Is it wrapped in try/catch?** If not, any error crashes the entire request or page
- **Is there a `.catch()` on the promise?** Unhandled promise rejections cause silent failures
- **What happens in the catch block?**
  - Empty catch `catch(e) {}` — error is swallowed, nobody knows it happened
  - Console.log only `catch(e) { console.log(e) }` — developer sees it in logs, user sees nothing useful
  - Generic error thrown — user gets a cryptic error message
  - Proper handling — user gets a helpful message, data is preserved, app stays functional

### Step 3: Trace the User Impact
For each unhandled or poorly handled failure, trace what the user actually experiences:

- **White screen** — The page crashes entirely because an error bubbled up with no error boundary
- **Infinite loading** — A spinner that never stops because the fetch failed and the loading state was never set to false
- **Silent data loss** — The user clicked "Save" and it looked like it worked but nothing was actually saved
- **Partial state** — Half the page loaded, half is broken because one of several parallel requests failed
- **Stuck UI** — A button that stays disabled after a failed submit because the loading state was never reset
- **Stale data shown** — An error occurred on refresh so the app shows old cached data as if it's current
- **Broken redirect** — Auth check fails and the app doesn't know what to do with the user

### Step 4: Check for Missing Retry Logic
For important operations, check if there's any retry mechanism:

- **Payment processing** — If a charge fails due to network issues (not card decline), does it retry or just fail?
- **Critical data saves** — If saving user data fails, is there any retry or queue?
- **File uploads** — If an upload fails midway, can the user retry without starting over?
- **Email sending** — If an email fails to send (verification, password reset), can the user trigger it again?

### Step 5: Check for Missing Timeouts
Find operations that could hang forever:

- **API calls with no timeout** — If an external service is slow, does the request wait forever?
- **Database queries with no timeout** — A complex query on a growing table could take minutes
- **File uploads with no timeout** — A large file on a slow connection could hang the UI
- **AI calls with no timeout** — LLM calls can take 30+ seconds, does the UI handle this?

### Step 6: Check for Missing Fallbacks
For each external dependency, check what happens if the entire service is down:

- **Database down** — Does the app crash entirely or can it show a maintenance page?
- **AI service down** — Does the feature that uses AI break the whole app or just that feature?
- **Auth provider down** — Can anyone use the app at all or is everyone locked out?
- **CDN/storage down** — Do missing images break the layout or show a placeholder?
- **Payment provider down** — Can users still browse, or does the entire checkout flow crash?

### Step 7: Check for Data Preservation on Failure
Find places where user effort is lost when something fails:

- **Long forms** — If submitting a 10-field form fails, does the user have to fill it all out again?
- **Multi-step flows** — If step 3 of 5 fails, does the user go back to step 1?
- **Text editors** — If saving a long post fails, is the content still in the editor or gone?
- **File uploads** — If the upload fails after the user selected files, do they have to pick them again?
- **Shopping carts** — If adding to cart fails, does the user know it failed or think it worked?

### Step 8: Check for Error Boundaries (Frontend)
For React/Next.js/similar apps:

- **Is there a root error boundary?** Without one, any component error crashes the entire app
- **Are there section-level error boundaries?** One broken widget shouldn't take down the whole page
- **Do error boundaries show a useful message?** Or just a blank space where content should be

### Step 9: Output the Failure Report

```
===================================================
       WHAT IF IT FAILS? — FAILURE MAP
===================================================

YOUR APP DEPENDS ON THESE SERVICES
---------------------------------------------------
[Service]              Used in              Failure handling
Supabase DB            12 files             ⚠️ 4/12 have no error handling
OpenAI API             3 files              🚨 0/3 have error handling
Stripe                 2 files              ✅ 2/2 handled
Resend email           1 file               🚨 0/1 handled
...

WILL CRASH THE APP 🚨
---------------------------------------------------
[number]. [What fails + what the user sees — plain English]
   Example: "If the database is slow for even 1 second when loading
   the dashboard, the entire page goes white because there's no
   loading timeout and no error boundary."
   📍 [file:line]
   Happens when: [realistic scenario that triggers this]
   User sees: [exactly what appears on screen]
   🛠️ Fix: [specific code to add]

[repeat for each]

USER LOSES THEIR WORK ⚠️
---------------------------------------------------
[number]. [What fails + what data is lost]
   Example: "If saving a profile update fails, the form resets and
   everything the user typed is gone. They won't know it failed
   because there's no error message."
   📍 [file:line]
   Happens when: [realistic scenario]
   🛠️ Fix: [specific code to add — preserve form data, show error]

[repeat for each]

FAILS SILENTLY (nobody knows) 🔇
---------------------------------------------------
[number]. [What fails + why nobody notices]
   Example: "If sending the welcome email fails, the catch block
   logs to console and moves on. The user never gets the email
   and has no way to request it again."
   📍 [file:line]
   🛠️ Fix: [specific code to add — user feedback, retry option]

[repeat for each]

MISSING TIMEOUTS ⏰
---------------------------------------------------
[Operation]              File:line          Timeout set?
OpenAI chat completion   src/api/chat:15    ❌ Could hang 60s+
Supabase query           src/api/users:8    ❌ No timeout
Image upload             src/upload:22      ❌ No timeout
...

MISSING FALLBACKS
---------------------------------------------------
[Service]         If it's down...         Fallback?
Database          Entire app crashes      ❌
OpenAI            AI features break       ❌
Image CDN         Broken images           ❌
Auth provider     Everyone locked out     ❌
...

ERROR BOUNDARY COVERAGE
---------------------------------------------------
Root error boundary:          [YES/NO]
Page-level error boundaries:  [X of Y pages]
Component error boundaries:   [X of Y risky components]

RESILIENCE SCORE: XX/100
(100 = graceful under fire, 70+ = handles common failures,
below 50 = one hiccup and the whole thing goes down)

TOP 3 FIXES
---------------------------------------------------
1. [Most impactful — what and where]
2. [Second most impactful]
3. [Third most impactful]
===================================================
```

## Important Rules
- ONLY report issues you actually find in the code. Do not invent failure scenarios for services the app doesn't use.
- Cite exact file paths and line numbers.
- Explain everything in plain English. Instead of "unhandled promise rejection in the async middleware" say "If the database is down when someone tries to log in, the app crashes because nobody told it what to do when that happens."
- Be realistic about what failures actually happen. A database being slow for a few seconds is common. A database being down for an hour is rare. Prioritize accordingly.
- Give credit for good error handling. If Stripe checkout has proper error handling, say so.
- For each fix, provide the actual code pattern to add — not just "add error handling" but the specific try/catch with the user-facing error message and state cleanup.
- Distinguish between errors the user caused (wrong password, invalid input) and errors that aren't the user's fault (network down, service outage). Both need handling but with different messages.
- If the app has no error handling anywhere, say that upfront and recommend starting with the 3 most critical paths rather than overwhelming with 50 fixes.
