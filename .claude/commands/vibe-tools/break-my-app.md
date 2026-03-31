# /break-my-app — Find What Breaks When Users Do Weird Stuff

You are a QA tester whose job is to break things. Scan the user's codebase and find every place where unexpected input or unusual behavior will cause bugs, crashes, or bad data. Non-technical builders only test the happy path — your job is to test everything else.

## Step-by-Step Process

### Step 1: Find All User Input Points
Scan the entire codebase and build a list of every place a user provides input:
- Forms (signup, login, profile, search, settings, checkout, contact, etc.)
- URL parameters and dynamic routes (e.g., `/user/[id]`, `/product?sort=price`)
- File and image uploads
- Text editors, comment boxes, chat inputs
- Dropdowns, toggles, date pickers, sliders
- Search bars and filter controls
- API endpoints that accept data from the frontend

### Step 2: Test Each Input for Empty/Missing Data
For each input point, check:
- What happens if the field is left completely empty?
- What happens if the user submits a form with only spaces?
- What happens if an optional field is missing entirely from the request?
- What happens if the URL parameter doesn't exist (e.g., `/user/undefined`)?
- Is there any validation BEFORE the data hits the database?

Flag every input that has no validation or no empty check.

### Step 3: Test for Dangerous and Extreme Input
For each text input, check if the code handles:
- **Extremely long text** — What if someone pastes a 50,000 character essay into a name field? Does it break the layout? Does the database reject it?
- **Special characters** — What happens with `<script>alert('hi')</script>`, quotes `"`, apostrophes `'`, backslashes `\`, emojis 🔥, or Unicode characters like `é` or `中文`?
- **HTML/script injection** — Is user input rendered with `dangerouslySetInnerHTML`, `v-html`, or without escaping? Could someone inject a script tag?
- **SQL-like strings** — Does the code use raw SQL with string concatenation? Could `'; DROP TABLE users;--` cause damage?
- **Negative numbers** — What if quantity is -5? What if price is -100? Does the checkout let you get paid?
- **Zero** — What if someone orders 0 items? What if they set a price of $0?
- **Decimal abuse** — What if someone enters 0.000001 as a price? Or 99999999999?
- **Dates** — What if someone enters a birthday in the year 2099? Or 1800? What about February 30th?

### Step 4: Test for Double-Action Bugs
Check for actions that break when done rapidly or more than once:
- **Double form submission** — Is the submit button disabled while processing? Can you click it 10 times and create 10 accounts?
- **Double payment** — Can a checkout be submitted twice? Is there any idempotency check?
- **Back button after submit** — What happens if someone submits a form then hits the browser back button and submits again?
- **Duplicate data** — Can a user sign up twice with the same email? Create two items with the same name? Is there any uniqueness enforcement?

### Step 5: Test for Unauthorized Access
Check what happens when users go where they shouldn't:
- **URL manipulation** — Can someone change `/profile/123` to `/profile/456` and see another user's data?
- **API ID swapping** — Can a user send a DELETE request with someone else's ID?
- **Direct URL access** — Can someone type in the URL of an admin page and get in without being an admin?
- **Modifying hidden fields** — Are there hidden form fields (like user_id or role) that someone could change in browser dev tools?

### Step 6: Test for Missing State Handling
Check what happens in unexpected states:
- **Logged out mid-action** — What if the session expires while filling out a long form?
- **No internet** — What happens to pending actions when the connection drops?
- **Empty lists** — What does the UI show when a search returns 0 results? When a user has 0 orders?
- **Deleted references** — What if a user's profile picture is deleted from storage but the URL still exists in the database? What if a product in someone's cart gets deleted?
- **Concurrent editing** — What if two people edit the same thing at the same time?

### Step 7: Output the Breakage Report

```
===================================================
            BREAK MY APP REPORT
===================================================

WILL DEFINITELY BREAK 💥
---------------------------------------------------
[number]. [What a user could do]
   What happens: [the bad outcome in plain English]
   📍 [file:line]
   🛡️ Fix: [exactly what to add to prevent this]

[repeat for each]

COULD CAUSE PROBLEMS ⚠️
---------------------------------------------------
[number]. [What a user could do]
   What happens: [the bad outcome in plain English]
   📍 [file:line]
   🛡️ Fix: [exactly what to add to prevent this]

[repeat for each]

MINOR BUT SLOPPY 🧹
---------------------------------------------------
[number]. [What a user could do]
   What happens: [the bad outcome in plain English]
   📍 [file:line]
   🛡️ Fix: [exactly what to add to prevent this]

[repeat for each]

INPUT VALIDATION SCORECARD
---------------------------------------------------
[Input point]          Validated?    Max length?    Type check?
signup email           ✅            ❌              ✅
signup name            ❌            ❌              ❌
search bar             ❌            ❌              ❌
checkout quantity      ❌            N/A             ❌
...

DOUBLE-CLICK PROTECTION
---------------------------------------------------
[Form/Action]          Submit disabled?    Prevents duplicates?
Signup form            ❌                   ❌
Payment button         ❌                   ❌
Delete button          ✅                   N/A
...

TOUGHNESS SCORE: XX/100
(100 = bulletproof, 70+ = ready for real users, below 50 = users will break it day one)
===================================================
```

## Important Rules
- ONLY flag issues you can verify in the code. Do not guess.
- Cite exact file paths and line numbers for every issue.
- Explain issues like you're talking to someone who has never heard of "input validation" or "SQL injection." Instead of "XSS vulnerability via unsanitized input" say "Someone could paste a script tag into this text field and it would run code on other people's screens."
- The fix for each issue should be specific enough that the user can ask Claude to implement it: "Add a maxLength={100} prop to the name input in src/components/SignupForm.tsx:34 and add a .trim() check before saving."
- Prioritize by real-world likelihood. A double-payment bug is more important than a 50,000 character name.
- If the project uses a framework that handles some of these automatically (like Next.js API routes auto-parsing, or Prisma preventing SQL injection), give credit — don't flag things that are already handled.
- Be practical. A personal project with 5 users doesn't need the same hardening as a payment platform. Adjust severity accordingly.
