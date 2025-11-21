---
layout: default
title: gatewaystack
---

# gatewaystack

**Secure user-scoped authentication for LLM apps using the Apps SDK and Model Context Protocol (MCP)**

Until now, most AI systems have been built around model access, not **user identity**.

Access typically happens through a single API key — often shared across a team, department, or entire organization.

That makes it impossible to answer basic questions:

- *who made this request?*
- *what are they allowed to do?*
- *did it follow policy?*
- *what did it cost?*

## Why Now?

As organizations adopt agentic systems and user-specific AI workflows, identity, policy, and governance become mandatory. Shared API keys cannot support enterprise-grade access control or compliance.

A new layer is required—centered around users, not models.

## Designing a User-Scoped AI Trust & Governance Gateway

A new layer is emerging in the modern AI stack:

> **the user-scoped trust and governance gateway**

It sits between applications (or agents) and model providers, ensuring that **every request is authenticated, authorized, observable, and governed.**

**gatewaystack** defines this layer.

gatewaystack sits between your application and the LLM provider. It receives identity-bound requests (via OIDC or Apps SDK tokens), applies governance, and then forwards the request to the model.

It provides the foundational primitives every agent ecosystem needs — starting with secure user authentication and expanding into full lifecycle governance.

## The Core Modules

**1. identifiabl – User identity & authentication**

Every model call must be tied to a real user, tenant, and context. identifiabl verifies identity, handles OIDC/App SDK tokens, and attaches identity metadata to each request.

**2. transformabl — Content transformation & safety pre-processing**

Before a request can be validated or routed, its content must be transformed into a safe, structured, model-ready form.

transformabl handles all pre-model transformations, including:

- PII detection and redaction
- content classification (legal, sensitive, unsafe)
- safety pre-processing (jailbreak detection, harmful content)
- input segmentation (safe vs unsafe regions)
- anonymization and pseudonymization
- formatting normalization and cleanup
- metadata extraction (topics, intent, sentiment, risk)
- generating routing hints based on detected patterns

This layer ensures that the request entering validatabl and proxyabl is clean, safe, and structured, enabling fine-grained governance and more intelligent routing decisions.

**3. validatabl – Access & policy enforcement**

Once a request is tied to a user, validatabl ensures it follows your rules:

- tool- and model-level permissions
- org-level policies
- schema & safety checks
- tenant boundaries
- scopes & roles

This is where most governance decisions happen.

**4. proxyabl – In-path routing & enforcement**

proxyabl is the gateway itself — the in-path request processor that:

- forwards traffic to an LLM
- chooses providers or models
- injects identity metadata
- applies routing rules
- enforces rate limits and policies in-path

**5. limitabl – Rate limits, quotas, and spend controls**

Every user, org, or agent needs usage constraints:

- per-user rate limits
- per-org quotas
- budget ceilings
- cost anomaly detection
- model/provider throttling

limitabl enforces these constraints consistently for all calls.

**6. explicabl – Observability & audit**

The control plane must record:

- who did what
- what model they used
- what policy decisions were triggered
- what the model cost
- what happened internally

explicabl provides the audit logs, traces, and metadata needed for trust, security, debugging, and compliance.

## End to End Flow

```text
User
   → identifiabl       (who is calling?)
   → transformabl      (prepare, clean, classify, anonymize)
   → validatabl        (is this allowed?)
   → proxyabl          (where does it go?)
   → limitabl          (how much can they use?)
   → explicabl         (what happened?)
   → LLM provider
   → response
```

Each module intercepts the request, adds or checks metadata, and guarantees that the call is:

identified, transformed, validated, routed, constrained, and audited.

This is the foundation of user-scoped AI.

## Inputs

- OIDC tokens / App SDK identity tokens
- User context (user_id, org_id, roles, scopes)
- Model request (messages, params, attachments)
- Configured policies
- Routing rules
- Quota & budget settings

## Outputs

- Authenticated, user-bound request
- Policy decision (allow, deny, modify)
- Routing decision (provider/model/tool)
- Enforced limits (rate, cost, budgets)
- Full audit log + runtime metadata

## Integrates with Your Existing Stack

gatewaystack works natively with:

- ChatGPT Apps SDK (identity tokens)
- Model Context Protocol (MCP)
- OAuth2 / OIDC identity providers
- Any LLM provider (OpenAI, Anthropic, Google, and internal models)

It acts as a drop-in governance layer without requiring changes to your application logic.


## Links

Want to explore the modules in detail? 
[gatewaystack GitHub repo](https://github.com/davidcrowe/gatewaystack)

Or contact us to discuss enterprise deployments:
[reducibl applied ai studio](https://reducibl.com)
