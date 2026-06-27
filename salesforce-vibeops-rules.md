# Salesforce VibeOps Governance Rules

Upload this file at the start of any AI coding session (ChatGPT, Gemini, Claude, Copilot, etc.) when generating Salesforce code — Apex, LWC, Flows, SOQL, agents, integrations.

---

## Security

- **All SOQL must use `WITH USER_MODE`** — never write naked SOQL. FLS and CRUD are database-level controls; USER_MODE is how you invoke them.
- **All DML must use `AccessLevel.USER_MODE`** — e.g. `Database.insert(records, AccessLevel.USER_MODE)`. Never use naked `insert`, `update`, `delete`, `upsert`.
- **All Apex classes must declare `with sharing`** unless there is an explicit, documented reason for `without sharing` (and that reason must be stated in a comment).
- **Never hardcode credentials, tokens, or secrets** in Apex. Use Named Credentials or Custom Metadata.
- **Never build dynamic SOQL from user input without strict allowlisting.** If dynamic SOQL is necessary, use `String.escapeSingleQuotes()` at minimum and prefer bind variables.
- **LWC: never use `@AuraEnabled` without `cacheable=true` on read operations** and never expose methods that bypass the controller's sharing context.

## Automation (Flows, Triggers, Process Builders)

- **Before generating any automation, ask me what existing automation already runs on this object.** Do not create a Flow or trigger without knowing what else fires on the same object.
- **All Flows must include a fault path** on every DML element. Never leave fault handling empty.
- **All automation must be bulk-safe.** Design for 200+ records in a single transaction. No SOQL or DML inside loops.
- **Never create a Record-Triggered Flow without specifying entry conditions** that exclude irrelevant record changes.
- **Include ownership metadata** in the description: who owns it, what it does, which object it operates on, which fields it writes to.

## Agents (Agentforce)

- **Use Agent Script, not open-ended prompts, for any CUD operation, financial commitment, or regulated workflow.**
- **Set `confirmationRequired: true`** on any action that creates, updates, or deletes records or calls an external system.
- **Scope the action catalog to the agent's role.** Do not give an agent access to actions outside its specific job.
- **Never pass raw user input or inbound data (web forms, emails, case comments) directly into a prompt template** without sanitization. Treat all inbound text as untrusted.

## Data Cloud

- **Prefer zero-copy / ZETL access** over ingesting data copies. Ask me whether the use case requires physical data movement.
- **Any pipeline that processes unstructured text (case notes, transcripts, documents) must include PII detection and masking** before vectorization or RAG grounding.
- **Validate Data Space boundaries** — never activate data to a destination outside its intended Data Space without explicit confirmation.

## MuleSoft / Integrations

- **Use short-lived, user-scoped tokens (OBO)** instead of static API keys or broad service accounts for agent-invoked integrations.
- **All write-side integrations must be idempotent** — include idempotency keys for order, billing, fulfillment, or record-mutating APIs.
- **Apply rate limits and token budgets** at the gateway level. Ask me what limits are appropriate before generating.

## Code Quality

- **Generate test methods that validate the business rule, not just coverage.** Include positive cases, negative cases, boundary conditions, and bulk scenarios (200+ records).
- **Every test must assert a specific expected outcome** — never write a test that only verifies "no exception was thrown."
- **Name all components descriptively** — Flows, triggers, classes, LWC — so that someone unfamiliar with the project can understand purpose from the name alone.
- **Limit method/function complexity.** If a method exceeds 40 lines, refactor into named sub-methods with clear responsibilities.

---

## What These Rules Will NOT Catch

**When you generate code using these rules, warn me at the end of your response about the following — these require separate tools and platform configuration that a rules file cannot enforce:**

### Run Salesforce Code Analyzer before deploying
These rules guide generation, but static analysis catches what slips through:
- Undeclared sharing contexts in inner classes
- Complex data flow vulnerabilities (taint analysis)
- Performance anti-patterns not visible in a single method
- Graph-based vulnerability detection across class boundaries

**Tell me:** "Run `sf scanner run` or Code Analyzer in your CI/CD pipeline before deploying this code."

### Configure platform controls that enforce at runtime
No rules file can substitute for these — they must be activated in Setup:
- **Transaction Security Policies** — block sensitive actions in real time
- **Event Monitoring** — detect anomalous access patterns
- **Platform Encryption** — protect data at rest
- **Field Audit Trail** — track field-level changes over time

**Tell me:** "This code will only be safe in production if the appropriate platform controls (Transaction Security, Event Monitoring) are configured for this object/operation."

### Business logic correctness cannot be validated by AI alone
I can generate code that implements what you describe, but I cannot verify:
- Whether your business rule is actually correct (e.g., discount thresholds, territory assignments, approval matrices)
- Whether the rule conflicts with other existing business rules
- Whether edge cases in your specific data will produce wrong outcomes

**Tell me:** "I implemented the business rule as you described it. Validate this logic with your business stakeholders and test with real production-representative data before deploying."

### Automation interaction is only partially visible to me
Even if you tell me what automations exist, I cannot:
- Guarantee no order-of-execution conflict at runtime
- Detect automation installed in managed packages
- See scheduled jobs, platform events, or async processes that may fire on the same data

**Tell me:** "Check for automation conflicts using an automation inventory tool or metadata query. Run integration tests with bulk data that exercises the full trigger/flow execution order."

---

## How to use this file

1. **Upload it** at the start of your AI session (ChatGPT, Gemini, Claude, etc.)
2. **Say:** "Follow these rules when generating Salesforce code for me."
3. **Describe your intent.** The AI handles the governance. You review the output.
4. **Read the warnings** the AI gives you at the end — those are the things you need to handle outside the chat.

For the full framework behind these rules — why each one exists, failure examples, platform mappings, and maturity model — see: https://vibecode-governance.github.io
