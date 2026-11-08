# n8n-payment-api-healthcheck


# ⚡ Resilience-First Payment API Health-Check Workflow (n8n)

An enterprise-grade, event-driven API health monitoring workflow built in n8n. Designed for QA/Integration Engineering, this pipeline continuously validates payment endpoints against expected contract status codes while maintaining 100% loop resilience during downstream failures.

---

##  Architecture Overview

```text
                  ┌──────────────────────┐
                  │    Manual Trigger    │
                  └──────────┬───────────┘
                             │
                             ▼
                ┌──────────────────────────┐
                │ Get Payment API List     │ (Data Table)
                └──────────┬───────────────┘
                           │
                           ▼
             ┌──────────────────────────────┐
             │ Loop Through Payment APIs    │◄─────────────────┐
             └─────────────┬────────────────┘                  │
                           │                                   │
                           ▼                                   │
             ┌──────────────────────────────┐                  │
             │ Test Payment Endpoint        │                  │
             │ (HTTP GET, 10s Timeout)      │                  │
             └──────┬────────────────┬──────┘                  │
                    │                │                         │
            (Success Path)    (Timeout / Network Error)        │
                    │                │                         │
                    ▼                │                         │
        ┌───────────────────────┐    │                         │
        │ Status Code Match?    │    │                         │
        └───────┬───────┬───────┘    │                         │
             YES│       │NO          │                         │
                │       └────────────┼─────────────┐           │
                ▼                    ▼             ▼           │
     ┌────────────────────┐   ┌──────────────────────────┐     │
     │ Healthy Payment    │   │ Capture API Failure      │     │
     │ Integration Log    │   │ (Structured Record)      │     │
     └──────────┬─────────┘   └────────────┬─────────────┘     │
                │                          │                   │
                │                          ▼                   │
                │             ┌──────────────────────────┐     │
                │             │ Mock Engineering Alert   │     │
                │             └────────────┬─────────────┘     │
                │                          │                   │
                └──────────────────────────┴───────────────────┘
