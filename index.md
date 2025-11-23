---
layout: default
title: explicabl
---

# explicabl

**observability, audit, and runtime transparency for llm and agentic apps**

after a request has been **identified**, **transformed**, **authorized**, **routed**, and **executed**, one question remains:

**what actually happened?**

explicabl answers that question.

it is the observability and audit pipeline of gatewaystack — capturing every identity, decision, transformation, routing choice, cost impact, and model interaction.

## at a glance

explicabl is the **runtime audit and observability layer** for llm apps.  

it lets you:

- generate identity-level and tenant-level audit logs  
- record governance decisions (allow/deny/modify)  
- capture all routing and provider metadata  
- log costs, token usage, and latency  
- produce traces for debugging and forensics  
- feed SIEM / monitoring systems with structured events  

> 📦 **implementation**: [`ai-observability-gateway`](https://github.com/davidcrowe/gatewaystack) + [`ai-audit-gateway`](https://github.com/davidcrowe/gatewaystack) (roadmap)

## why now?

as llm systems become part of enterprise workflows, organizations need:

- verifiable audit trails  
- explainability for policy decisions  
- traceability for model-provider calls  
- cost & usage transparency  
- safety and compliance evidence  
- debugging visibility for multi-agent systems  

without an observability layer, governance is invisible and unprovable.

explicabl makes it concrete.

## designing the observability & audit layer

### within the shared requestcontext

all gatewaystack modules operate on a shared [`RequestContext`](https://github.com/davidcrowe/gatewaystack/blob/main/docs/reference/interfaces.md) object.

**explicabl is responsible for**:

- **reading**: `identity`, `metadata`, `policyDecision`, `routingDecision`, `limitsDecision`, `usage` — everything from all upstream modules
- **writing**: nothing to the request context (explicabl is write-only to external systems: logs, traces, metrics)

explicabl receives structured metadata from every upstream module:

- identity (from identifiabl)  
- transformation logs (from transformabl)  
- policy decisions (from validatabl)  
- routing choices (from proxyabl)  
- cost & usage (from limitabl)  

it aggregates this into a complete, time-ordered audit record for each request.

## the core functions

**1. `logIdentity` — record who made the request**  
user_id, org_id, tenant, scopes, roles.

**2. `logTransformations` — record what changed**  
pii redaction events, segmentation, classification results.

**3. `logPolicyDecision` — record why a request was allowed or blocked**  
allow / deny / modify + all triggered rules.

**4. `logRouting` — record where the request was sent**  
provider, model, region, fallback or primary.

**5. `logUsage` — record cost, tokens, and latency**  
pricing metadata, model cost, total spend, timing.

**6. `logTrace` — produce structured traces for SIEM / monitoring**  
OpenTelemetry-compatible, API-gateway-style trace events.

## what explicabl does

- captures end-to-end audit trails  
- produces structured logs for governance and compliance  
- supports debugging for complex agent workflows  
- provides insight into routing, cost, and policy decisions  
- enables anomaly detection by external systems  

## explicabl works with

- `validatabl` to enforce policy
- `proxyabl` to perform routing
- `limitabl` to apply rate limits 
- `identifiabl` to evaluate identity
- `transformabl` to preprocess content

explicabl does not modify traffic. it simply **records everything**.

## audit event structure

every request generates multiple correlated events:
```json
{
  "event_id": "evt_abc123",
  "event_type": "policy_decision",
  "timestamp": "2025-01-15T10:30:45.123Z",
  "request_id": "req_xyz789",
  "trace_id": "trace_123",

  "identity": { /* from identifiabl */ },
  "transformations": { /* from transformabl */ },
  "policy": { /* from validatabl */ },
  "routing": { /* from proxyabl */ },
  "usage": { /* from limitabl */ },

  "metadata": {
    "gatewaystack_version": "1.0.0",
    "environment": "production"
  }
}
```

all events share `request_id` and `trace_id` for correlation.

## log destinations

explicabl supports multiple destinations:
```yaml
logging:
  destinations:
    # SIEM systems
    - type: "splunk"
      endpoint: "https://splunk.company.com"

    # Cloud logging
    - type: "cloudwatch"
      log_group: "/gatewaystack/audit"
      region: "us-east-1"

    # OpenTelemetry
    - type: "otel-collector"
      endpoint: "otel-collector:4317"

    # Long-term storage
    - type: "s3"
      bucket: "audit-logs"
      retention: "7_years"
```

## distributed tracing

for multi-step workflows, explicabl maintains trace context:
```
User request [trace_id: abc123]
  ├─ Model call 1 [span_id: span_1]
  ├─ Tool: web_search [span_id: span_2, parent: span_1]
  ├─ Model call 2 [span_id: span_3, parent: span_1]
  └─ Tool: calendar [span_id: span_4, parent: span_3]
```

all events share `trace_id`, enabling complete workflow reconstruction.

## compliance and retention

explicabl supports enterprise compliance requirements:

**retention policies:**
- identity events: 7 years  
- policy decisions: 7 years  
- usage events: 1 year  

**security:**
- encryption at rest (AES-256)  
- encryption in transit (TLS 1.3)  
- immutable logs (append-only, hash-chained)  

**privacy:**
- GDPR-compliant erasure (user_id anonymization)  
- PII redaction in stored logs  
- configurable content retention  

## performance considerations

explicabl uses asynchronous logging to minimize latency:

**critical events (synchronous):**
- policy decisions (allow/deny)  
- rate limit hits  

**standard events (asynchronous):**
- usage metrics  
- routing metadata  
- transformation logs  

average overhead: 5–10ms per request. critical events add 2–5ms (synchronous), while standard events add <1ms (asynchronous buffered writes).

## end to end flow
```text
user
   → identifiabl       (who is calling?)
   → transformabl      (prepare, clean, classify, anonymize)
   → validatabl        (is this allowed?)
   → limitabl          (can they afford it? pre-flight constraints)
   → proxyabl          (where does it go? execute)
   → llm provider      (model call)
   → [limitabl]        (deduct actual usage, update quotas/budgets)
   → explicabl         (what happened?)
   → response
```

explicabl is where every action becomes visible — the foundation of traceability, safety, and enterprise trust.

## integrates with your existing stack

explicabl plugs into gatewaystack and your existing llm stack without requiring application-level changes. it exposes http middleware and sdk hooks for:

- chatgpt apps sdk  
- model context protocol (mcp)  
- oauth2 / oidc identity providers  
- any llm provider (openai, anthropic, google, internal models)  

## getting started

**for observability setup**:  
→ [logging configuration guide](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/logging-setup.md)  
→ [SIEM integration patterns](https://github.com/davidcrowe/gatewaystack/blob/main/docs/examples/siem-integrations.md)  
→ [OpenTelemetry setup](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/otel-setup.md)

**for compliance and audit**:  
→ [audit trail configuration](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/audit-configuration.md)  
→ [retention policies](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/retention-policies.md)

**for implementation**:  
→ [integration guide](https://github.com/davidcrowe/gatewaystack/blob/main/docs/guides/integration.md)

## links

want to explore the full gatewaystack architecture?  
→ [view the gatewaystack github repo](https://github.com/davidcrowe/gatewaystack)

want to contact us for enterprise deployments?  
→ [reducibl applied ai studio](https://reducibl.com)

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