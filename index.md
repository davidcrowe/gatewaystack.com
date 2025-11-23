---
layout: default
title: gatewaystack
---

# gatewaystack

**user-scoped authentication and governance for llm and agentic apps (apps sdk + mcp)**
  
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

## why now?

as organizations adopt agentic systems and user-specific ai workflows, identity, policy, and governance become mandatory. shared api keys cannot support enterprise-grade access control or compliance.

a new layer is required — centered around users, not models.

> **the user-scoped trust and governance gateway**

## who gatewaystack is for

gatewaystack is built for teams that need **user-scoped, auditable, policy-enforced access to ai models** — not just raw model access behind a shared api key.

it is especially useful for:

### platform & ai infrastructure teams

teams rolling out internal copilots, agent runtimes, or ai-powered tools across the organization. they need to ensure:

- every model call is tied to a specific employee  
- roles and scopes determine exactly which tools they can use  
- requests never reach external models unless policy allows  

### security, governance & compliance teams

organizations where regulatory or internal controls matter:

- finance and legal workflows  
- healthcare and life sciences  
- enterprise data governance  
- soc 2 / hipaa / pci environments  

they need "who did what" audit trails, per-user enforcement, pii handling, and policy guarantees.

### saas platforms serving many tenants

multi-tenant products that want to:

- differentiate features by plan or tier  
- enforce per-user and per-tenant quotas  
- separate customer data paths  
- prevent shared "one big openai key" usage  

gatewaystack turns every model call into a **user-bound, tenant-aware interaction**.

### teams building apps sdk- and mcp-powered agents

developers integrating their apps directly into chatgpt or exposing data/tools via mcp:

- validate and map apps sdk tokens to real identities  
- thread identity through all downstream tools  
- attach user/org/tenant metadata without custom plumbing  

gatewaystack becomes the unified **identity → policy → routing → audit** path for all agent interactions.

## designing a user-scoped ai trust & governance gateway

**the user-scoped trust and governance gateway** sits between applications (or agents) and model providers, ensuring that **every model interaction is authenticated, authorized, observable, and governed.**

**gatewaystack** defines this layer.

gatewaystack sits between your application and the llm provider. it receives identity-bound requests (via oidc or apps sdk tokens), applies governance, and then forwards the request to the model.

it provides the foundational primitives every agent ecosystem needs — starting with secure user authentication and expanding into full lifecycle governance.

## the core modules

**CHANGE: Reordered to match execution flow - moved limitabl before proxyabl**

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

**CHANGE: Added note about limitabl's two-phase operation**

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

## error handling

**CHANGE: NEW SECTION - Critical for production deployments**

gatewaystack modules use a **fail-safe strategy** based on the security/availability tradeoff:

| module | on failure | reason |
|--------|-----------|--------|
| **identifiabl** | deny (fail closed) | no anonymous requests allowed |
| **transformabl** | continue (fail open) | allow untransformed content with warning |
| **validatabl** | deny (fail closed) | safety first — block when policy engine fails |
| **limitabl (pre-flight)** | continue (fail open) | favor availability over limit enforcement |
| **proxyabl** | fallback → error | try alternative providers, then return error |
| **limitabl (accounting)** | log error | accounting failure shouldn't block response delivery |
| **explicabl** | log error | audit failure logged but doesn't block request |

**circuit breakers**: modules that make external calls (jwks validation, policy stores, redis) implement circuit breakers with configurable thresholds.

**degraded mode**: when upstream dependencies fail, gatewaystack can operate in degraded mode with reduced governance guarantees (configurable per environment).

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

## inputs

- oidc tokens / apps sdk identity tokens  
- user context (user_id, org_id, roles, scopes)  
- model request (messages, params, attachments)  
- configured policies  
- routing rules  
- quota & budget settings  

## outputs

- authenticated, user-bound requests  
- policy decisions (allow, deny, modify)  
- routing decisions (provider/model/tool)  
- enforced limits (rate, cost, budgets)  
- full audit logs and runtime metadata  

## shared requestcontext

**CHANGE: Added note pointing to GitHub for full reference**

all modules operate on a shared `RequestContext` object that flows through the pipeline. this context carries identity, content, metadata, and decisions from each module.

> 📘 **full type definitions and integration examples**: see [`docs/reference/interfaces.md`](https://github.com/davidcrowe/gatewaystack/blob/main/docs/reference/interfaces.md) in the repo

**core types** (simplified view):
```ts
// shared across modules
export type Identity = {
  user_id: string;
  org_id?: string;
  tenant?: string;
  roles: string[];
  scopes: string[];
};

export type ContentMetadata = {
  contains_pii?: boolean;
  classification?: string[];   // ["sensitive", "financial"]
  risk_score?: number;         // 0–1
  topics?: string[];
  // extensible bag
  [key: string]: unknown;
};

export type ModelRequest = {
  model: string;               // "gpt-4", "smart-model"
  tools?: string[];
  max_tokens?: number;
  messages: any[];
  attachments?: any[];
};

export type PolicyDecision = {
  effect: 'allow' | 'deny' | 'modify';
  reasons: string[];
  modifications?: Partial<ModelRequest>;
};

export type LimitsDecision = {
  effect: 'ok' | 'throttle' | 'deny' | 'fallback' | 'degrade';
  reasons: string[];
  constraints?: {
    max_tokens?: number;
    max_cost?: number;
  };
};

export type RoutingDecision = {
  provider: string;           // "openai", "azure-openai"
  model: string;              // concrete provider model
  region?: string;
  alias?: string;             // "smart-model"
};

export type UsageRecord = {
  input_tokens: number;
  output_tokens: number;
  total_cost: number;
  latency_ms: number;
};

export type RequestContext = {
  request_id: string;
  trace_id?: string;

  identity?: Identity;
  content?: {
    messages: any[];
    attachments?: any[];
  };
  metadata?: ContentMetadata;
  modelRequest: ModelRequest;

  policyDecision?: PolicyDecision;
  limitsDecision?: LimitsDecision;
  routingDecision?: RoutingDecision;
  usage?: UsageRecord;

  // arbitrary module extensions
  [key: string]: unknown;
};
```

## module boundaries

**identifiabl**  
- **input**: `RequestContext` with `modelRequest` + http auth header  
- **reads**: authorization header, jwks endpoint  
- **writes**: `identity`, `trace_id`, identity headers (`x-user-id`, `x-org-id`, etc.)  

**transformabl**  
- **input**: `RequestContext` with `identity`, `content`  
- **reads**: `content` (messages, attachments)  
- **writes**: `metadata` (pii flags, classification, risk score), redacted `content`, transformation logs  

**validatabl**  
- **input**: `identity`, `content`, `metadata`, `modelRequest`  
- **reads**: all above + policy definitions  
- **writes**: `policyDecision` (allow/deny/modify + reasons)  

**limitabl** (two-phase)  
- **pre-flight phase**:  
  - **reads**: `identity`, `modelRequest`, `policyDecision`  
  - **writes**: `limitsDecision` (constraints, budget availability)  
- **accounting phase**:  
  - **reads**: provider response (tokens, cost)  
  - **writes**: `usage`, updates quota/budget stores, emits usage events  

**proxyabl**  
- **input**: `modelRequest`, `identity`, `metadata`, `policyDecision`, `limitsDecision`  
- **reads**: routing rules, provider configurations, secrets  
- **writes**: `routingDecision`, normalized provider response  

**explicabl**  
- **input**: full `RequestContext` + events from all modules  
- **reads**: everything (for audit trail construction)  
- **writes**: external logs and traces only (no modification of request context)  

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

want to explore the modules in detail?  
→ [view the gatewaystack github repo](https://github.com/davidcrowe/gatewaystack)

want to contact us for enterprise deployments?  
→ [reducibl applied ai studio](https://reducibl.com)