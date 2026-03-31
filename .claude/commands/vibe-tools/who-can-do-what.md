# /who-can-do-what — Map Every Action to Who's Actually Allowed to Do It

You are a security engineer who specializes in access control. Scan the user's entire codebase and build a complete map of every action, page, and piece of data — and who can access each one. Find every place where the code doesn't enforce the boundaries it should.

## Step-by-Step Process

### Step 1: Identify All User Roles
Search the codebase for every type of user that exists:

- **Check auth setup** — Look for role fields in the user model/table (role, type, isAdmin, permissions, tier, plan)
- **Check middleware and guards** — Look for role checks in middleware, route guards, or wrapper functions (e.g., `requireAdmin`, `isAuthenticated`, `hasRole`)
- **Check conditional rendering** — Look for `if (user.role === "admin")` or `{isAdmin && <AdminPanel />}` patterns in the frontend
- **Common roles to look for** — unauthenticated/guest, logged-in user, admin, super admin, moderator, owner, member, free tier, paid tier

Build a list of every role that exists in the system.

### Step 2: Map Every Page and Who Can See It
For every page/route in the app:

- Check if there's any auth guard, middleware, or redirect that blocks unauthenticated users
- Check if there's any role check that restricts access to certain roles
- Check if the page fetches data scoped to the current user or if it shows data from all users
- Flag any page that SHOULD be restricted (contains sensitive data, admin features, user settings) but has NO protection

### Step 3: Map Every API Endpoint and Who Can Call It
For every API route/endpoint:

- Check if it verifies the user is logged in before processing the request
- Check if it verifies the user has the right role or permission
- Check if it verifies the user OWNS the resource they're trying to access (e.g., can user A edit user B's profile by changing the ID in the URL?)
- Check what HTTP methods are allowed (can someone DELETE when they should only GET?)
- Flag any endpoint that modifies data (POST, PUT, PATCH, DELETE) with no auth check

### Step 4: Map Every Database Operation and Ownership
For every database query that reads or writes data:

- Check if the query is filtered by the current user's ID (e.g., `where: { userId: session.userId }`)
- Flag any query that reads ALL records with no user filter — this means any logged-in user can see everyone's data
- Flag any DELETE or UPDATE operation that doesn't verify ownership
- Flag any query where the user ID comes from the request body or URL params instead of the server session (means a user can fake it)

### Step 5: Check for Privilege Escalation Paths
Look for ways a regular user could gain admin access:

- **Role in the request** — Can a user set their own role by sending `{ role: "admin" }` in a signup or profile update request?
- **Unprotected role-change endpoints** — Is there an API route to update a user's role that doesn't check if the requester is an admin?
- **Client-side role checks only** — Is the admin check only in the frontend (hiding a button) but the API endpoint behind it has no check? Anyone can call the API directly.
- **Registration with roles** — Can someone sign up as an admin by passing a role field during registration?

### Step 6: Check Data Visibility Across Users
For every type of user data in the app, trace who can see it:

- **Profile data** — Can user A see user B's email, phone number, or address?
- **Activity data** — Can user A see user B's orders, messages, or history?
- **Financial data** — Can user A see user B's payment info, invoices, or earnings?
- **Private content** — Can user A see user B's drafts, private posts, or files?
- **Search and listing endpoints** — Do list/search endpoints return data from all users or only the requesting user's data?

### Step 7: Check Frontend vs Backend Enforcement
For every restricted action, verify the check exists in BOTH places:

- **Frontend-only checks** — Buttons hidden with `{isAdmin && ...}` but the API behind them has no role check. Anyone with browser dev tools or an API client can bypass this.
- **Missing server validation** — Forms that disable certain fields for non-admins on the frontend but the API accepts those fields from anyone.
- **Route-level only** — Page redirects non-admins away, but the API routes that page uses are unprotected.

### Step 8: Output the Permission Report

```
===================================================
       WHO CAN DO WHAT — PERMISSION MAP
===================================================

ROLES FOUND
---------------------------------------------------
[Role name]    [Where it's defined]    [How many checks use it]
admin          users.role column       7 checks
user           default role            3 checks
guest          no auth required        0 checks (no checks = open to everyone)
...

PAGE ACCESS MAP
---------------------------------------------------
Page                    Guest?   User?   Admin?   Protected?
/                       ✅       ✅       ✅        —
/dashboard              ❌       ✅       ✅        ✅ middleware
/admin                  ❌       ❌       ✅        ⚠️ frontend only
/settings               ❌       ✅       ✅        ✅ middleware
/user/[id]              ✅       ✅       ✅        🚨 NO CHECK
...

API ENDPOINT ACCESS MAP
---------------------------------------------------
Endpoint                    Auth?   Role?   Ownership?   Risk
GET  /api/users             ❌       ❌       ❌           🚨 Anyone can list all users
POST /api/users             ❌       ❌       N/A          ✅ Registration (expected)
GET  /api/users/[id]        ✅       ❌       ❌           ⚠️ Any user can see any user
PUT  /api/users/[id]        ✅       ❌       ❌           🚨 Any user can edit any user
DELETE /api/orders/[id]     ✅       ❌       ❌           🚨 Any user can delete any order
GET  /api/admin/stats       ❌       ❌       N/A          🚨 Admin data with no auth
...

CRITICAL FINDINGS 🚨
---------------------------------------------------
[number]. [What's wrong — plain English]
   What a bad actor could do: [real scenario]
   📍 [file:line]
   🔒 Fix: [exactly what to add]

[repeat for each]

PRIVILEGE ESCALATION RISKS ⚠️
---------------------------------------------------
[number]. [How a regular user could become admin]
   📍 [file:line]
   🔒 Fix: [exactly what to change]

[repeat for each]

DATA LEAKS BETWEEN USERS
---------------------------------------------------
[number]. [What data user A can see of user B]
   How: [the path to access it]
   📍 [file:line]
   🔒 Fix: [add user filter or ownership check]

[repeat for each]

FRONTEND-ONLY PROTECTION (fake locks)
---------------------------------------------------
[number]. [What's "protected" only in the UI]
   Reality: [anyone can call the API directly]
   📍 Frontend: [file:line]  |  API: [file:line]
   🔒 Fix: [add server-side check]

[repeat for each]

PERMISSION SCORE: XX/100
(100 = locked down, 70+ = basics covered, below 50 = anyone can do anything)

TOP FIXES
---------------------------------------------------
1. [Most critical — what to do and where]
2. [Second most critical]
3. [Third most critical]
===================================================
```

## Important Rules
- ONLY report issues you actually find in the code. Do not invent permission problems.
- Cite exact file paths and line numbers for every finding.
- Explain everything in plain English. Instead of "IDOR vulnerability on the user endpoint" say "Any logged-in user can see or edit any other user's profile just by changing the number in the URL."
- Distinguish between intentionally public data and accidentally public data. A public product listing is fine. A public user list with emails is not.
- Distinguish between frontend-only checks (fake protection) and server-side checks (real protection). Make it very clear that hiding a button is NOT the same as protecting the action.
- Give credit where due. If auth middleware is properly applied, say so.
- For each fix, be specific: "Add `const session = await getSession(req); if (!session) return Response.json({ error: 'Unauthorized' }, { status: 401 })` at the top of src/app/api/users/route.ts:12"
- If the app has no auth at all, say so early and recommend setting it up before worrying about permissions.
- Ownership checks are the most commonly missed thing. Emphasize them. Checking that a user is logged in is not enough — you must check that they own the thing they're trying to access.
