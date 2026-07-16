# SentinelOps AI

> An agentic IT incident responder that diagnoses server failures, automates safe restarts, and escalates complex incidents to engineers.

SentinelOps AI is a Python proof of concept for AI-assisted IT operations. Given a natural-language incident report, an OpenAI-powered agent investigates the affected server using health metrics and recent logs, then decides whether to restart the service or escalate the incident to an on-call SRE.

The project demonstrates practical agentic AI patterns: function calling, multi-step tool use, decision-making from operational signals, and human-in-the-loop escalation.

## What it does

The agent follows a simple incident-response playbook:

1. Receives an incident report, such as `payment-gateway-01 is timing out`.
2. Checks the server's CPU, memory, and health status.
3. Retrieves recent logs to determine the likely cause.
4. Restarts the service when CPU or memory usage exceeds 90%.
5. Escalates dependency failures or complex/unknown issues to an on-call engineer.
6. Returns a concise final response describing its investigation and action.

## Architecture

```text
Incident report
      │
      ▼
OpenAI agent (GPT-4.1 mini)
      │
      ├── get_server_health() ──► CPU, memory, service status
      ├── fetch_recent_logs() ─► Diagnostic log evidence
      ├── restart_service() ───► Automated remediation
      └── escalate_to_engineer() ► On-call SRE ticket
      │
      ▼
Final incident summary
```

## Tools available to the agent

| Tool | Purpose |
| --- | --- |
| `get_server_health(server_id)` | Retrieves CPU, memory, and health status. |
| `fetch_recent_logs(server_id, lines)` | Retrieves recent diagnostic log entries. |
| `restart_service(server_id)` | Performs an automated service restart. |
| `escalate_to_engineer(summary)` | Creates an escalation for an on-call SRE engineer. |

## Sample scenarios

| Server | Simulated issue | Expected response |
| --- | --- | --- |
| `payment-gateway-01` | 99% CPU, depleted thread pool, hung process | Investigate and restart service. |
| `auth-SSO-03` | 96% memory, Java heap out-of-memory error | Investigate and restart service. |
| `search-service-04` | Search-cluster connection refused | Escalate; restarting cannot resolve a failed dependency. |
| `db-SQL-02` | Healthy backup and replication activity | Confirm healthy state / no remediation. |

## Tech stack

- Python
- OpenAI Python SDK
- GPT-4.1 mini with function calling
- Google Colab Secrets (`userdata`) for API-key handling
- JSON-based simulated monitoring and log data

## Getting started

### Prerequisites

- Python 3.9+
- An OpenAI API key
- `openai` Python package

Install the dependency:

```bash
pip install openai
```

### Configure the API key

The supplied notebook-style code uses Google Colab Secrets:

```python
from google.colab import userdata
client = OpenAI(api_key=userdata.get("OPENAI_APIKEY"))
```

In Colab, add an `OPENAI_APIKEY` secret before running the notebook. For local development, replace this with environment-variable loading, for example:

```python
import os
client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
```

Then run your script or notebook and call the workflow:

```python
run_it_agent("payment-gateway-01 is timing out and users cannot complete payments")
```

## Design decisions

- **Tool-grounded investigation:** The model acts on supplied health and log data instead of guessing at infrastructure state.
- **Policy-driven remediation:** The system instruction limits automatic restarts to high CPU or memory conditions.
- **Human escalation:** Dependency errors and uncertain failures are routed to an engineer instead of being treated as safely automatable.
- **Extensible tool layer:** The `AVAILABLE_FUNCTIONS` map cleanly separates agent orchestration from tool implementations.

## Production considerations

This repository is a learning prototype: the current monitoring, log, restart, and escalation functions use simulated data. A production version should add:

- Real integrations with observability, ticketing, and service-management platforms.
- Authentication, role-based access control, and least-privilege credentials.
- Explicit approval gates before any production-changing action.
- Tool-call audit logs, traceability, retries, timeouts, and error handling.
- Evaluation tests for tool selection, escalation accuracy, and unsafe-action prevention.
- Redaction and retention controls for logs and incident data.

## Skills demonstrated

This project showcases applied AI engineering skills relevant to IT operations and platform teams:

- LLM function calling and agent/tool orchestration
- Incident triage and runbook automation
- Prompt design for policy-constrained decisions
- Human-in-the-loop escalation design
- Python API integration and structured JSON data handling

## Roadmap

- [ ] Replace simulated tools with live monitoring, logging, and ticketing integrations
- [ ] Add an approval workflow for service restarts
- [ ] Store incident timelines and generate post-incident reports
- [ ] Add automated tests and evaluation scenarios
- [ ] Build a dashboard for incidents, actions, and escalation outcomes
