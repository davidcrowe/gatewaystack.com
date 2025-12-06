---
layout: default
title: gatewaystack
---

# gatewaystack

**agentic control plane for user-scoped AI governance**

an open-source control plane that makes AI agents **enterprise-ready** by enforcing user-scoped identity, policy, and audit trails on every model call.

> ⚠️ early-stage: gatewaystack is under active development. the first layer, `identifiabl`, is live and published (`npm i identifiabl`). Additional modules (`transformabl`, `validatabl`, `limitabl`, `proxyabl`, `explicabl`) are on the roadmap.

until now, most ai systems have been built around model access, not **user identity**.

access typically happens through a single api key — often shared across a team, department, or entire organization.

that makes it impossible to answer basic questions:

- *who made this request?*
- *does it contain sensitive info like pii that needs to be removed?*
- *what are they allowed to do?*
- *what did it cost?*
- *what model should it be sent to?*
- *did it follow policy?*

## at a glance

gatewaystack is a **user-scoped trust and governance gateway** for llm apps.  

it lets you:

- plug your identity provider into your model context protocol (mcp) server and openai's apps sdk  
- enforce per-user and per-tenant model/tool access and policies  
- transform requests before they hit the model (for example, to remove pii)  
- route, rate-limit, and log every call at the user level  

→ [view the gatewaystack github repo](https://github.com/davidcrowe/gatewaystack)  
→ [read the architecture overview](#designing-a-user-scoped-ai-trust--governance-gateway)  
→ [contact reducibl for enterprise deployments](https://reducibl.com)  

## why now?

as organizations adopt agentic systems and user-specific ai workflows, identity, policy, and governance become mandatory. shared api keys cannot support enterprise-grade access control or compliance.

a new layer is required — centered around users, not models.

> **the user-scoped trust and governance gateway**


### the three-party problem

modern AI apps are really **three-party systems**:

**the user** — a real human with identity, roles, and permissions  
**the llm** — a model acting on their behalf (chatgpt, claude)  
**your backend** — the trusted data and tools the model needs to access  

these three parties all talk to each other, but they don’t share a common, cryptographically verified identity layer.

**the gap:** The llm knows who the user is (they logged into chatgpt). your backend doesn't. So it can't:
- filter data per-user (*"show me my calendar"* → returns everyone's calendar)
- enforce per-user policies (*"only doctors use medical models"* → anyone can)
- audit by user (*"who made this query?"* → can't answer)

**without a unifying identity layer, you get:**
- shared API keys (everyone sees everything, or no one sees anything)
- no enforcement ("who can use which models for what")
- no audit trail (can't prove compliance)
- enterprises block AI access entirely (too risky)

this instability across user ↔ LLM ↔ backend is what Gatewaystack calls the **Three-Party Problem**. 

it shows up in two directions:

### when the three-party problem goes wrong

while gatewaystack doesn't necessarily solve the root cause of each issue, it would have prevented these resources from being accessed without a cryptographically verified user identity on the request. 

user-scoped requests act as a safety net for all kinds of common mistakes:  

→ [xAI Dev Leaks API Key for Private SpaceX, Tesla LLMs](https://krebsonsecurity.com/2025/05/xai-dev-leaks-api-key-for-private-spacex-tesla-llms/?utm_source=chatgpt.com)  
→ [DeepSeek database left user data, chat histories exposed for anyone to see](https://www.theverge.com/news/603163/deepseek-breach-ai-security-database-exposed?utm_source=chatgpt.com)  
→ [How I Reverse Engineered a Billion-Dollar Legal AI Tool and Found 100k+ Confidential Files](https://alexschapiro.com/security/vulnerability/2025/12/02/filevine-api-100k)  
→ [Leading AI Companies Accidentally Leak Their Passwords and Digital Keys on GitHub - What You Need to Know](https://www.fortra.com/blog/ai-companies-accidentally-leak-passwords-digital-keys-github?utm_source=chatgpt.com)  


### direction 1: enterprises controlling who can use which models and tools
*"how do i ensure only **licensed doctors** use medical models, only **analysts** access financial data, and **contractors** can't send sensitive prompts?"*

> user ↔ backend ↔ llm

**without gatewaystack:**
```typescript
app.post('/chat', async (req, res) => {
  const { model, prompt } = req.body;
  const response = await openai.chat.completions.create({
    model, // Anyone can use gpt-4-medical
    messages: [{ role: 'user', content: prompt }]
  });
  res.json(response);
});
```

**with gatewaystack:**
```typescript
app.post('/chat', async (req, res) => {
  const userId = req.headers['x-user-id'];
  const userRole = req.headers['x-user-role']; // "doctor", "analyst", etc.
  const userScopes = req.headers['x-user-scopes']?.split(' ') || [];
  
  // Gateway already enforced: only doctors with medical:write can reach here
  const response = await openai.chat.completions.create({
    model: req.body.model,
    messages: [{ role: 'user', content: req.body.prompt }],
    user: userId // OpenAI audit trail
  });
  res.json(response);
});
```

**gateway policy:**
```json
{
  "gpt-4-medical": {
    "requiredRoles": ["doctor", "physician_assistant"],
    "requiredScopes": ["medical:write"]
  }
}
```

the gateway enforces role + scope checks **before** forwarding to your backend. If a nurse tries to use `gpt-4-medical`, they get `403 Forbidden`.

---

### direction 2: users accessing their own data via AI
*"how do i let chatgpt read **my** calendar without exposing **everyone's** calendar?"*

> user ↔ llm ↔ backend

**without gatewaystack:**
```typescript
app.get('/calendar', async (_req, res) => {
  const events = await getAllEvents(); // Everyone sees everything
  res.json(events);
});
```

**with gatewaystack:**
```typescript
app.get('/calendar', async (req, res) => {
  const userId = req.headers['x-user-id']; // Verified by gateway
  const events = await getUserEvents(userId);
  res.json(events);
});
```

the gateway validates the oauth token, extracts the user identity, and injects `X-User-Id` — so your backend can safely filter data per-user.

---

### why both directions matter
attaching a cryptographically confirmed user identity to a shared request context is the key that makes request level governance possible:

**without solving the three-party problem, you can't:**
- filter data per-user (direction 1: everyone sees everything)
- enforce "who can use which models" (direction 2: no role-based access)
- audit "who did what" (compliance impossible)
- rate limit per-user (shared quotas get exhausted)
- attribute costs (can't charge back to teams/users)

**gatewaystack solves both** by binding cryptographic user identity to every AI request:

* oauth login per user (rs256 jwt, cryptographic identity proof)
* per-user / per-tenant data isolation by default
* deny-by-default authorization (scopes per tool/model/role)
* immutable audit trails (who, what, when, which model)
* rate limits & spend caps (per user/team/org)
* drop-in between AI clients and your backend (no sdk changes)

gatewaystack is composed of modular packages that can run **standalone** or as a cohesive **six-layer pipeline** for complete AI governance.

## designing a user-scoped ai trust & governance gateway

**the user-scoped trust and governance gateway** sits between applications (or agents) and model providers, ensuring that **every model interaction is authenticated, authorized, observable, and governed.**

**gatewaystack** defines this layer.

gatewaystack sits between your application and the llm provider. it receives identity-bound requests (via oidc or apps sdk tokens), applies governance, and then forwards the request to the model.

it provides the foundational primitives every agent ecosystem needs — starting with secure user authentication and expanding into full lifecycle governance.

## the core modules

**1. [`identifiabl`](https://identifiabl.com) — user identity & authentication**

every model call must be tied to a real user, tenant, and context. identifiabl verifies identity, handles oidc/apps sdk tokens, and attaches identity metadata to each request.

> 📦 **implementation**: [`ai-auth-gateway`](https://github.com/davidcrowe/gatewaystack/tree/main/packages/ai-auth-gateway) (published)

**2. [`transformabl`](https://transformabl.com) — content transformation & safety pre-processing**

before a request can be validated or routed, its content must be transformed into a safe, structured, model-ready form.

transformabl handles pre-model transformations, including:

- pii detection and redaction  
- content classification (legal, sensitive, unsafe)  
- jailbreak / harmful-content detection  
- anonymization and pseudonymization  
- metadata extraction (topics, intent, sentiment, risk)  
- routing hints for validatabl and proxyabl  

this layer ensures that the request entering validatabl and proxyabl is clean, safe, and structured, enabling fine-grained governance and more intelligent routing decisions.

> 📦 **implementation**: `ai-content-gateway` (roadmap)

**3. [`validatabl`](https://validatabl.com) — access & policy enforcement**

once a request is tied to a user, validatabl ensures it follows your rules:

- tool- and model-level permissions  
- org-level policies  
- schema & safety checks  
- tenant boundaries  
- scopes & roles  

this is where most governance decisions happen.

> 📦 **implementation**: [`ai-policy-gateway`](https://github.com/davidcrowe/gatewaystack) (roadmap)

**4. [`limitabl`](https://limitabl.com) — rate limits, quotas, and spend controls**

every user, org, or agent needs usage constraints:

- per-user rate limits  
- per-org quotas  
- budget ceilings  
- cost anomaly detection  
- model/provider throttling  

limitabl enforces these constraints in two phases — **pre-flight checks** before execution and **usage accounting** after the model responds.

> 📦 **implementation**: [`ai-rate-limit-gateway`](https://github.com/davidcrowe/gatewaystack) + `ai-cost-gateway` (roadmap)

**5. [`proxyabl`](https://proxyabl.com) — in-path routing & execution**

proxyabl is the gateway execution layer — the in-path request processor that:

- chooses providers or models  
- forwards traffic to an llm  
- injects identity and policy metadata  
- applies routing rules  
- respects constraints returned by limitabl  

> 📦 **implementation**: [`ai-routing-gateway`](https://github.com/davidcrowe/gatewaystack) (roadmap)

**6. [`explicabl`](https://explicabl.com) — observability & audit**

the control plane must record:

- who did what  
- what model they used  
- what policy decisions were triggered  
- what the model cost  
- what happened internally  

explicabl provides the audit logs, traces, and metadata needed for trust, security, debugging, and compliance.

> 📦 **implementation**: [`ai-observability-gateway`](https://github.com/davidcrowe/gatewaystack) + `ai-audit-gateway` (roadmap)

## who gatewaystack is for

gatewaystack is built for teams that need **user-scoped, auditable, policy-enforced access to ai models** — not just raw model access behind a shared api key.

- platform & ai infra teams
- security / governance / compliance
- multi-tenant saas products
- teams building apps sdk- and mcp-powered agents

**example use cases:**

### healthcare saas — hipaa-compliant AI diagnostics

a platform with 10,000+ doctors needs to ensure every AI-assisted diagnosis is tied to the licensed physician who requested it, with full audit trails for hipaa and internal review.

**before gatewaystack:** all AI calls run through a shared openai key — impossible to prove *which physician* made *which request*.

**with gatewaystack:** 
- `identifiabl` binds every request to a verified physician (`user_id`, `org_id` = clinic/hospital)
- `validatabl` enforces `role:physician` and `scope:diagnosis:write` per tool
- `explicabl` emits immutable audit logs with the physician's identity on every model call

**result:** user-bound, tenant-aware, fully auditable AI diagnostics.

---

### enterprise copilot — per-employee policy enforcement

a global company rolls out an internal copilot that can search confluence, jira, google drive, and internal apis. employees authenticate with sso (okta / entra / auth0), but the copilot calls the llm with a shared api key.

**before gatewaystack:** security teams can't enforce "only finance analysts can run this tool" or audit which employee triggered which action.

**with gatewaystack:**
- `identifiabl` binds the copilot session to the employee's SSO identity (`sub` from okta)
- `validatabl` enforces per-role tool access ("legal can see these repos, not those")
- `limitabl` applies per-user rate limits and spend caps
- `explicabl` produces identity-level audit trails for every copilot interaction

**result:** full identity-level governance without changing the copilot's business logic.

---

### multi-tenant saas — per-tenant cost tracking

a saas platform offers AI features across free, pro, and enterprise tiers. today, all AI usage runs through a single openai key per environment — making it impossible to answer "how much did org x spend?" or "which users hit quota?"

**before gatewaystack:** one big shared key. no tenant-level attribution. cost overruns are invisible until the bill arrives.

**with gatewaystack:**
- `identifiabl` attaches `user_id` and `org_id` to every request
- `validatabl` enforces tier-based feature access (`plan:free`, `plan:pro`, `feature:advanced-rag`)
- `limitabl` enforces per-tenant quotas and budgets
- `explicabl` produces per-tenant usage reports

**result:** per-tenant accountability without changing app logic.

---

## what's different from traditional api gateways?

**identity providers (auth0, okta, cognito, entra id)**  
Handle login and token minting, but stop at the edge of your app. They don't understand model calls, tools, or which provider a request is going to — and they don't enforce user identity inside the AI gateway.

**api gateways and service meshes (kong, apigee, aws api gateway, istio, envoy)**  
great at path/method-level auth and rate limiting, but they treat llms like any other http backend. **you can build AI governance on top of them** (kong plugins, istio policies, lambda authorizers), **but it requires significant custom development** to replicate what gatewaystack provides out-of-the-box: user-scoped identity normalization, per-tool scope enforcement, pre-flight cost checks, apps sdk / mcp compliance, and AI-specific audit trails.

**cloud AI gateways (cloudflare AI gateway, azure openai + api management, vertex AI, bedrock guardrails)**  
Focus on provider routing, quota, and safety filters at the tenant or API key level. User identity is usually out-of-band or left to the application.

**hand-rolled middleware**  
many teams glue together jwt validation, headers, and logging inside their app or a thin node/go proxy. it works... until you need to support multiple agents, providers, tenants, and audit/regulatory requirements.

**gatewaystack is different:**
- **user-scoped by default** — every request is tied to a verified user, not a shared key
- **model-aware** — understands tools, scopes, and provider semantics (apps sdk, mcp, openai, anthropic)
- **composable governance** — each layer (identity, policy, limits, routing, audit) can run standalone or as part of the full control plane
- **built for agents** — prevents runaway loops, enforces per-workflow budgets, and tracks multi-step traces

**example: kong + openai**

to get user-scoped AI governance with kong, you'd need to:
1. install `jwt` plugin (validate tokens)
2. install `request-transformer` plugin (inject headers)
3. write custom Lua script to normalize identity claims
4. write custom Lua script for scope-to-tool mapping
5. write custom plugin for pre-flight cost estimation
6. build separate service for Protected Resource Metadata
7. configure DCR flow manually
8. build custom audit log forwarding

**signficant development + ongoing maintenance.**

**with gatewaystack:** configure `.env` file, deploy, done.

you can still run gatewaystack alongside traditional api gateways — it's the **user-scoped identity and governance slice** of your AI stack.

## end to end flow
```text
user
   → identifiabl       (who is calling?)
   → transformabl      (prepare, clean, classify, anonymize)
   → validatabl        (is this allowed?)
   → limitabl          (how much can they use? pre-flight constraints)
   → proxyabl          (where does it go? execute)
   → llm provider      (model call)
   → [limitabl]        (deduct usage - optional accounting phase)
   → explicabl         (what happened?)
   → response
```

each module intercepts the request, adds or checks metadata, and guarantees that the call is:

**identified, transformed, validated, constrained, routed, and audited.**

this is the foundation of user-scoped ai.

## how teams adopt gatewaystack

most teams don't roll out every module on day one. a common path looks like:

1. **start with identifiabl + proxyabl**  
   - front your existing llm provider with a simple gateway  
   - validate oidc / apps sdk tokens and inject `x-user-id` / `x-org-id`  
   - route requests to the same models you already use  

2. **add limitabl and explicabl**  
   - configure per-user and per-tenant quotas  
   - emit identity-level audit logs and cost metrics  
   - give security and finance a clear view of "who is using which model and what it costs"  
   - note: limitabl runs in two phases (pre-flight + accounting) to both prevent and track usage

3. **layer in transformabl and validatabl**  
   - enforce content and safety policies before the model  
   - apply fine-grained, role-based access to tools and datasets  
   - route sensitive traffic to compliant models or on-prem deployments  

over time, gatewaystack becomes the **shared trust and governance layer** for all of your ai workloads — internal copilots, saas features, and apps sdk / mcp-based agents.

## example user flow

a finance analyst uses an internal copilot to summarize a contract.

- identifiabl verifies oidc credentials and attaches user/org/role  
- transformabl classifies the request as "contains sensitive financial data"  
- validatabl checks policy: external models not allowed for this role  
- limitabl confirms the user is within quota and budget (pre-flight)  
- proxyabl routes to the enterprise's internal model  
- limitabl records actual token usage and updates spend counters (accounting)  
- explicabl records a full audit event for compliance  

every step is user-bound, governed, and auditable.

## architecture diagram

<div class="arch-diagram">
  <div class="arch-row">
    <div class="arch-node">
      <div class="arch-node-title">app / agent</div>
      <div class="arch-node-sub">chat ui · internal tool · agent runtime</div>
    </div>

    <div class="arch-arrow">→</div>

    <div class="arch-node arch-node-gateway">
      <div class="arch-node-title">gatewaystack</div>
      <div class="arch-node-sub">user-scoped trust &amp; governance gateway</div>

      <div class="arch-pill-row">
        <span class="arch-pill">identifiabl</span>
        <span class="arch-pill">transformabl</span>
        <span class="arch-pill">validatabl</span>
        <span class="arch-pill">limitabl</span>
        <span class="arch-pill">proxyabl</span>
        <span class="arch-pill">explicabl</span>
      </div>
    </div>

    <div class="arch-arrow">→</div>

    <div class="arch-node">
      <div class="arch-node-title">llm providers</div>
      <div class="arch-node-sub">openai · anthropic · internal models</div>
    </div>
  </div>

  <p class="arch-caption">
    every request flows from your app through gatewaystack's modules before it reaches an llm provider —
    <strong>identified, transformed, validated, constrained, routed, and audited.</strong>
  </p>
</div>

## error handling, inputs, outputs, and shared requestcontext
→ [architecture.md](https://github.com/davidcrowe/gatewaystack/blob/main/docs/architecture.md)

## integrates with your existing stack

gatewaystack works natively with:

- **chatgpt apps sdk** — validates apps sdk identity tokens  
- **model context protocol (mcp)** — threads identity through mcp tool calls  
- **oauth2 / oidc providers** — auth0, okta, cognito, entra id  
- **any llm provider** — openai, anthropic, google, azure, internal models  

it acts as a drop-in governance layer without requiring changes to your application logic.

## getting started

**for concepts and architecture**: continue reading the module white papers linked above.

**for implementation and deployment**:  
→ [quickstart guide](https://github.com/davidcrowe/gatewaystack#quick-start)  
→ [deployment guide](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/deployment.md)  
→ [migration guide](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/migration.md)  

**for policy examples and reference code**:  
→ [policy examples](https://github.com/davidcrowe/gatewaystack/blob/main/docs/examples/policy-examples.md)  
→ [reference implementations](https://github.com/davidcrowe/gatewaystack/tree/main/examples)  

## links

want to contact us for enterprise deployments?  
→ [reducibl applied ai studio](https://reducibl.com)