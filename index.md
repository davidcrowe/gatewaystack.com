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
- *what are they allowed to do?*
- *did it follow policy?*
- *what did it cost?*

## at a glance

gatewaystack is a user-scoped **trust and governance gateway** for llm apps.  
it lets you:

- plug openai's apps sdk / mcp into your identity provider
- enforce per-user and per-tenant model/tool access
- route, rate-limit, and log every call at the user level

## why now?

as organizations adopt agentic systems and user-specific AI workflows, identity, policy, and governance become mandatory. shared api keys cannot support enterprise-grade access control or compliance.

a new layer is required—centered around users, not models.

## designing a user-scoped AI trust & governance gateway

a new layer is emerging in the modern AI stack:

> **the user-scoped trust and governance gateway**

it sits between applications (or agents) and model providers, ensuring that **every request is authenticated, authorized, observable, and governed.**

**gatewaystack** defines this layer.

gatewaystack sits between your application and the llm provider. it receives identity-bound requests (via oidc or apps sdk tokens), applies governance, and then forwards the request to the model.

it provides the foundational primitives every agent ecosystem needs — starting with secure user authentication and expanding into full lifecycle governance.

## the core modules

**1. `identifiabl` – user identity & authentication**

every model call must be tied to a real user, tenant, and context. identifiabl verifies identity, handles oidc/app sdk tokens, and attaches identity metadata to each request.

**2. `transformabl` — content transformation & safety pre-processing**

before a request can be validated or routed, its content must be transformed into a safe, structured, model-ready form.

transformabl handles all pre-model transformations, including:

- pii detection and redaction
- content classification (legal, sensitive, unsafe)
- safety pre-processing (jailbreak detection, harmful content)
- input segmentation (safe vs unsafe regions)
- anonymization and pseudonymization
- formatting normalization and cleanup
- metadata extraction (topics, intent, sentiment, risk)
- generating routing hints based on detected patterns

this layer ensures that the request entering validatabl and proxyabl is clean, safe, and structured, enabling fine-grained governance and more intelligent routing decisions.

**3. `validatabl` – access & policy enforcement**

once a request is tied to a user, validatabl ensures it follows your rules:

- tool- and model-level permissions
- org-level policies
- schema & safety checks
- tenant boundaries
- scopes & roles

this is where most governance decisions happen.

**4. `proxyabl` – in-path routing & enforcement**

proxyabl is the gateway itself — the in-path request processor that:

- forwards traffic to an llm
- chooses providers or models
- injects identity metadata
- applies routing rules
- enforces rate limits and policies in-path

**5. `limitabl` – rate limits, quotas, and spend controls**

every user, org, or agent needs usage constraints:

- per-user rate limits
- per-org quotas
- budget ceilings
- cost anomaly detection
- model/provider throttling

limitabl enforces these constraints consistently for all calls.

**6. `explicabl` – observability & audit**

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

or contact us to discuss enterprise deployments:  
→ [reducibl applied ai studio](https://reducibl.com)
