---
name: Install and configure the Milana SDK in a codebase
description: >-
  Provider-published coding-agent prompt that installs milana-js, initializes
  session recording, wires user identification, applies privacy controls, and
  mirrors analytics events — saved verbatim from Milana's docs.
api: https://docs.getmilana.ai/llms.txt
operations: [init, identify, updateUser, updateSession, trackEvent, stopRecording]
generated: '2026-07-21'
method: searched
source: https://docs.getmilana.ai/quickstart/coding-agent
---

# Coding Agent Setup (provider-published prompt, verbatim)

Milana publishes this single prompt for Claude Code, Codex, Cursor, or any modern
coding agent. It analyzes a codebase, presents a setup plan for approval, then
handles the full Milana integration.

---

Set up Milana in this codebase. Milana is an AI-powered session recording and product analytics SDK — it records user sessions via a lightweight browser SDK and uses AI to analyze the recordings so product teams can understand user behavior.

The goal is to produce an easy-to-review pull request that is under 200 lines of code. Keep changes minimal and focused.

## Reference

Fetch [https://docs.getmilana.ai/llms.txt](https://docs.getmilana.ai/llms.txt) and use it as your SDK reference throughout this task. It contains the full API surface, initialization options, and framework-specific integration patterns.

## Step 1: Explore the codebase

Find the frontend application code. In a monorepo, identify the package that serves the user-facing UI — look for a React DOM render call, a framework entry point (e.g. Next.js `app/layout.tsx`, Remix `root.tsx`, Vite `main.tsx`), or an HTML template that loads the app.

Determine:

* **Framework and rendering model**: React (client-side), Next.js (SSR/RSC), Remix (SSR), vanilla JavaScript, or static HTML. This determines the integration method — React provider (`MilanaProvider`), programmatic init (`init()`), or CDN script tag — and whether you need a `"use client"` boundary.
* **User object**: Find where my primary user object is loaded. This might be from an auth provider (Clerk, Auth0, NextAuth/Auth.js, Supabase Auth, Firebase Auth, better-auth, or a custom system), or from a user context, store, or API call elsewhere in the app. You need access to the user's ID, email, and name.
* **Existing analytics**: PostHog, Segment, Amplitude, Mixpanel, or others. Note whether there is a shared analytics wrapper function, and find calls that track activation events, conversion/purchase events, cancellation/churn events, and AI agent usage events.
* **Environment variable pattern**: Are other client-side API keys stored in environment variables (e.g. `NEXT_PUBLIC_*`, `VITE_*`, `REACT_APP_*`)? Note the naming convention used.
* **Content Security Policy**: Search for any `Content-Security-Policy` header set in middleware, server config, framework config (e.g. `next.config.js`), or `<meta http-equiv>`. If one exists, note that `connect-src` (for all installs) and `script-src` (for script-tag installs) will need to allow Milana origins — surface this in the plan in Step 2.
* **Environment and version**: How does the app detect its current environment (e.g. `process.env.NODE_ENV`, `import.meta.env.MODE`)? Is there an existing version constant or build-time injection from `package.json`?
* **Session context**: Does the app read UTM parameters from the URL, track referrer, store A/B test or experiment variants, or maintain feature flag state? Are there specific fields that track where users are coming from (e.g. entry point, traffic source, invite origin)? Note where this context is already available on the client.
* **Sensitive UI elements**: Scan for forms that collect credit card numbers, payment information, government IDs, social security numbers, or other PII beyond standard email, password, and phone fields (those input types are masked automatically by the SDK). Note: third-party payment UIs that render in iframes or popups (Stripe Elements, Stripe Link, Braintree hosted fields, etc.) are already invisible to the recorder — only flag payment forms built with native HTML inputs.
* **Feature flag system**: Is there a feature flag provider (LaunchDarkly, PostHog, Statsig, etc.)? Note this for the rollout recommendation.

**Important**: Only use data that is already loaded and available on the client. Do not add new server calls, API fetches, or data loading to support Milana — work with what the app already has.

## Step 2: Present your plan

Before making any changes, present a plan that covers the following. For each item, name the specific file path and show surrounding code context so I can verify placement.

**SDK integration**

* Which integration method you'll use (React provider, JS init, or script tag) and why.
* The exact file where you'll add the initialization code, and where in the component tree or module it will go.
* For SSR frameworks: any `"use client"` directives or layout file considerations.

**User identification**

* Which auth provider or user object loader you found and where you'll call `identify()`.
* The `identify()` call must go inside a `useEffect` (React) or equivalent lifecycle hook that fires when the authenticated user changes — not on every render. For vanilla JS, call it after auth resolves.
* `identify()` takes `userId`, `email`, optional `name`, and optional `metadata`. Populate `metadata` with fields already available on the client (`createdAt`, `plan`, `orgId`, `role`, or equivalents). Do not fetch additional data from the server.
* For later partial refreshes of user fields during the same session (e.g. plan upgrade, role switch), use `updateUser()` — it accepts the same fields with `email` optional.

**Session metadata**

* How you'll set `environment` (dynamically from the app's existing env detection).
* How you'll set `version` (from an existing version constant or build-time injection; if none exists, use `"0.0.0"`).
* Any session-level context already available on the client (UTM parameters, referrer, experiment variants, feature flag state, entry point or traffic source fields) that you'll include via `sessionInfo.metadata` at init, or a subsequent `updateSession({ metadata: {...} })` call as context becomes available.

**Privacy controls**

* List each sensitive element you found, its file path, and whether you recommend `milana-block` (contents never recorded) or `milana-mask` (text replaced with masked placeholders).
* If you recommend `privacy.maskingLevel: "high"` or `"xhigh"`, also identify known-safe UI or product copy that should stay readable with `milana-unmask` or a custom `privacy.unmaskClass`.
* If the application masks long text, especially with `high` or `xhigh` masking, recommend `privacy.shouldUseLayoutPreservingMasking: true`. This helps masked replays preserve the original text layout more closely.
* Ask me which ones I want you to apply. Do not add privacy classes without my explicit approval.

**Credentials**

* Whether you'll store the product ID and client key in environment variables (matching the existing pattern) or hardcode them. The client key is safe to include in client-side code.

**Content Security Policy**

* If you found an existing CSP, list the directive changes needed: add `https://in.getmilana.ai` to `connect-src`. For script-tag installs, also add `https://cdn.getmilana.ai` to `script-src`, plus `'unsafe-inline'` (or a nonce) so the inline queue shim can run.

**Analytics events**

* If a shared analytics wrapper function exists, you'll add `trackEvent()` to it.
* Otherwise, list up to 5 specific events you found — prioritizing activation, conversion/purchase, cancellation/churn, and AI agent usage events — that you'll mirror with `trackEvent()`.

**Rollout**

* If you found a feature flag system, mention that Milana supports deferred initialization gated behind a feature flag. Link to the Feature-flagged Rollout guide from the SDK reference for details.

Wait for me to approve the plan before proceeding.

## Step 3: Implement

Once I approve the plan, implement it:

1. **Install the SDK**: Add `milana-js` using the project's package manager (`npm`, `yarn`, or `pnpm`). For non-bundled apps, add the CDN script tag instead.

2. **Initialize recording**: Add the initialization code in the location specified in the plan.
   * **React / Next.js / Remix**: Wrap the app with `MilanaProvider` from `milana-js/react`. Place it below the auth provider so user data is available. For SSR frameworks, add a `"use client"` directive to the component that renders `MilanaProvider`.
   * **Vanilla JS**: Call `init()` from `milana-js` in the application entry point.
   * **Script tag**: Add the queue snippet and async script tag to the HTML template.

3. **Set session info**: Set `environment` dynamically from the app's existing environment detection. Set `version` from an existing version constant or `"0.0.0"`. Include any session-level context already available on the client (UTM params, referrer, experiment variants, entry point) in `sessionInfo.metadata`.

4. **Identify users**: Add an `identify()` call inside a `useEffect` (React) or lifecycle hook that fires when the authenticated user changes. Include `userId`, `email`, optional `name`, and populate `metadata` with fields already available on the client like `createdAt`, `plan`, `orgId`, and `role`. If you need to refresh user fields later in the session, reach for `updateUser()` instead of calling `identify()` again.

5. **Apply privacy controls**: For each element I approved, add the `milana-block` or `milana-mask` CSS class directly to the element in the HTML or JSX.

6. **Store credentials**: Ask me for my Milana **product ID** (starts with `prd_`) and **client key** (starts with `key_`). These are found on the [Settings page](https://app.getmilana.ai/settings). Store them using the existing env var pattern if one exists, otherwise hardcode them directly.

7. **Mirror analytics events**: If a shared analytics wrapper exists, add a `trackEvent()` call inside it. Otherwise, add `trackEvent()` calls for each approved event — import `trackEvent` from `milana-js` or use the `useMilana()` hook in React components.

## Step 4: Next steps

After implementation, tell me to:

* Run the dev server and open the application in a browser
* Open the browser developer console (you may need to change the log level from **Default levels** to **Verbose**) and look for **"Milana: Ready"** — this confirms the SDK initialized successfully
* Navigate through a few pages and interact with the app to generate a test session. Check the [Integration page](https://app.getmilana.ai/integration) to see the recorded session
* Milana waits for **30 minutes of inactivity** before finalizing a session, and processing takes up to **10 minutes** after that

If something isn't working, consult the Troubleshooting section of the SDK reference (from the llms.txt you fetched earlier). Enabling debug mode is particularly helpful — it logs detailed SDK state to the browser console. You may need to change the console log level from **Default levels** to **Verbose** to see all messages.
