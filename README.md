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


Key Technical Features & Engineering Decisions
Non-Blocking Resilience: Prevents total workflow halts when single services drop out. All iterations complete regardless of failure counts.

Dual-Path Error Handling:

Contract Failures: Catches non-2xx status codes or expected vs. actual status code mismatches.

Network/Transport Failures: Handles network timeouts (10s threshold) and DNS/connection drops via dedicated n8n error outputs.

Structured Incident Schema: Captures normalized JSON metadata (health_status, failure_type, service_name, endpoint_url, expected_status, actual_status, error_details, and checked_at) ready for downstream ingestion.

Mock Engineering Dispatcher: Formats actionable, high-severity incident notifications ready for Slack/PagerDuty webhook integration.

How to Import and Run
1. Clone this repository:
