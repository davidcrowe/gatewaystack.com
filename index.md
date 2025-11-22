---
layout: default
title: gatewaystack
---

# gatewaystack

**user-scoped authentication and governance for llm and agentic apps (apps sdk + mcp)**
  
  
until now, most AI systems have been built around model access, not **user identity**.

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
- transform requests before they hit the model e.g., to remove pii
- route, rate-limit, and log every call at the user level

## why now?

as organizations adopt agentic systems and user-specific AI workflows, identity, policy, and governance become mandatory. shared api keys cannot support enterprise-grade access control or compliance.

a new layer is required — centered around users, not models.

> **the user-scoped trust and governance gateway**

## who gatewaystack is for

gatewaystack is built for teams that need **user-scoped, auditable, policy-enforced access to AI models** — not just raw model access behind a shared API key.

it is especially useful for:

### platform & AI infrastructure teams

teams rolling out internal copilots, agent runtimes, or AI-powered tools across the organization. they need to ensure:

- every model call is tied to a specific employee
- roles and scopes determine exactly which tools they can use
- requests never reach external models unless policy allows

### security, governance & compliance teams

organizations where regulatory or internal controls matter:

- finance and legal workflows
- healthcare and life sciences
- enterprise data governance
- SOC 2 / HIPAA / PCI environments

they need “who did what” audit trails, per-user enforcement, PII handling, and policy guarantees.

### saas platforms serving many tenants

multi-tenant products that want to:

- differentiate features by plan or tier  
- enforce per-user and per-tenant quotas  
- separate customer data paths  
- prevent shared “one big OpenAI key” usage  

gatewaystack turns every model call into a **user-bound, tenant-aware interaction**.

### teams building apps sdk and mcp-powered agents

developers integrating their apps directly into ChatGPT or exposing data/tools via MCP:

- validate and map Apps SDK tokens to real identities
- thread identity through all downstream tools
- attach user/org/tenant metadata without custom plumbing

gatewaystack becomes the unified **identity → policy → routing → audit** path for all agent interactions.

## designing a user-scoped AI trust & governance gateway

**the user-scoped trust and governance gateway** sits between applications (or agents) and model providers, ensuring that **every model interaction is authenticated, authorized, observable, and governed.**

**gatewaystack** defines this layer.

gatewaystack sits between your application and the llm provider. it receives identity-bound requests (via oidc or apps sdk tokens), applies governance, and then forwards the request to the model.

it provides the foundational primitives every agent ecosystem needs — starting with secure user authentication and expanding into full lifecycle governance.

## the core modules

**1. [`identifiabl`](https://identifiabl.com) – user identity & authentication**

every model call must be tied to a real user, tenant, and context. identifiabl verifies identity, handles oidc/app sdk tokens, and attaches identity metadata to each request.

**2. [`transformabl`](https://transformabl.com) — content transformation & safety pre-processing**

before a request can be validated or routed, its content must be transformed into a safe, structured, model-ready form.

transformabl handles pre-model transformations, including:

- PII detection and redaction  
- content classification (legal, sensitive, unsafe)  
- jailbreak / harmful-content detection  
- anonymization and pseudonymization  
- metadata extraction (topics, intent, sentiment, risk)  
- routing hints for validatabl and proxyabl  

this layer ensures that the request entering validatabl and proxyabl is clean, safe, and structured, enabling fine-grained governance and more intelligent routing decisions.

**3. [`validatabl`](https://validatabl.com) – access & policy enforcement**

once a request is tied to a user, validatabl ensures it follows your rules:

- tool- and model-level permissions
- org-level policies
- schema & safety checks
- tenant boundaries
- scopes & roles

this is where most governance decisions happen.

**4. [`proxyabl`](https://proxyabl.com) – in-path routing & enforcement**

proxyabl is the gateway itself — the in-path request processor that:

- forwards traffic to an llm
- chooses providers or models
- injects identity metadata
- applies routing rules
- enforces rate limits and policies in-path

**5. [`limitabl`](https://limitabl.com) – rate limits, quotas, and spend controls**

every user, org, or agent needs usage constraints:

- per-user rate limits
- per-org quotas
- budget ceilings
- cost anomaly detection
- model/provider throttling

limitabl enforces these constraints consistently for all calls.

**6. [`explicabl`](https://explicabl.com) – observability & audit**

the control plane must record:

- who did what
- what model they used
- what policy decisions were triggered
- what the model cost
- what happened internally

explicabl provides the audit logs, traces, and metadata needed for trust, security, debugging, and compliance.

## end to end flow

```text
user
   → identifiabl       (who is calling?)
   → transformabl      (prepare, clean, classify, anonymize)
   → validatabl        (is this allowed?)
   → proxyabl          (where does it go?)
   → limitabl          (how much can they use?)
   → explicabl         (what happened?)
   → llm provider
   → response
```

each module intercepts the request, adds or checks metadata, and guarantees that the call is:

identified, transformed, validated, routed, constrained, and audited.

this is the foundation of user-scoped AI.

## how teams adopt gatewaystack

most teams don’t roll out every module on day one. a common path looks like:

1. **start with identifiabl + proxyabl**  
   - front your existing LLM provider with a simple gateway  
   - validate OIDC / Apps SDK tokens and inject `x-user-id` / `x-org-id`  
   - route requests to the same models you already use

2. **add limitabl and explicabl**  
   - configure per-user and per-tenant quotas  
   - emit identity-level audit logs and cost metrics  
   - give security and finance a clear view of “who is using which model and what it costs”

3. **layer in transformabl and validatabl**  
   - enforce content and safety policies before the model  
   - apply fine-grained, role-based access to tools and datasets  
   - route sensitive traffic to compliant models or on-prem deployments

over time, gatewaystack becomes the **shared trust and governance layer** for all of your AI workloads — internal copilots, SaaS features, and Apps SDK / MCP-based agents.

## example user flow

**example**

a finance analyst uses an internal copilot to summarize a contract.

- identifiabl verifies OIDC credentials and attaches user/org/role
- transformabl classifies the request as “contains sensitive financial data”
- validatabl checks policy: external models not allowed for this role
- proxyabl routes to the enterprise’s internal model
- limitabl applies the user’s quota
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
      <div class="arch-node-sub">user-scoped trust & governance gateway</div>

      <div class="arch-pill-row">
        <span class="arch-pill">identifiabl</span>
        <span class="arch-pill">transformabl</span>
        <span class="arch-pill">validatabl</span>
        <span class="arch-pill">proxyabl</span>
        <span class="arch-pill">limitabl</span>
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
    every request flows from your app through gatewaystack’s modules before it reaches an llm provider –
    <strong>identified, transformed, validated, routed, constrained, and audited.</strong>
  </p>
</div>


## inputs

- oidc tokens / app sdk identity tokens
- user context (user_id, org_id, roles, scopes)
- model request (messages, params, attachments)
- configured policies
- routing rules
- quota & budget settings

## outputs

- authenticated, user-bound request
- policy decision (allow, deny, modify)
- routing decision (provider/model/tool)
- enforced limits (rate, cost, budgets)
- full audit log + runtime metadata

## integrates with your existing stack

gatewaystack works natively with:

- chatgpt apps sdk (identity tokens)
- model context protocol (mcp)
- oauth2 / oidc identity providers
- any llm provider (openai, anthropic, google, and internal models)

it acts as a drop-in governance layer without requiring changes to your application logic.

## links

want to explore the modules in detail?  
→ [view the gatewaystack github repo](https://github.com/davidcrowe/gatewaystack)

want to contact us for enterprise deployments? 
→ [reducibl applied ai studio](https://reducibl.com)
