Below is a rigorous set of objectives to use in a **financial system design interview** if you want to sound like someone who has worked at—or is ready to work at—top-tier fintech companies such as **Stripe, Adyen, Wise, Revolut, Square, PayPal, Nubank, Monzo, Plaid, Checkout.com**, etc.

The key mindset shift is this:

> In a top 1% fintech system design interview, you are not just designing a scalable API.  
> You are designing a system that can be trusted with money under **partial failure, retries, fraud, regulation, reconciliation, currency complexity, and adversarial behavior**.

---

# 1. The Prime Directive

For any financial system design interview, your top priority order should be:

```text
1. Money correctness
2. Auditability
3. Compliance and security
4. Operational recoverability
5. Availability
6. Latency
7. Scalability
8. Developer/product velocity
```

Do not optimize scale before correctness.  
Do not optimize availability before money safety.  
Do not optimize UX before fraud and compliance controls.

A strong interview answer explicitly says:

> “For this system, I will prioritize correctness, idempotency, reconciliation, and auditability over raw throughput. Money movement systems must tolerate retries, timeouts, partial failures, and external rail instability without losing or duplicating funds.”

---

# 2. The Top 1% Non-Negotiables

These are the objectives you should state early and repeatedly anchor your design to.

## Objective 1: No money is created, destroyed, lost, or duplicated

Every movement of money must be represented as a controlled, balanced, auditable ledger mutation.

### Rigorous standard

- Every debit has a corresponding credit.
- Every ledger journal entry balances to zero.
- No balance is mutated without an immutable ledger entry.
- No external payment state change can mutate the ledger without an idempotent, reconcilable event.

### What to say in the interview

> “The ledger is the source of truth. Balances are derived from immutable ledger entries. Payment state can be eventually consistent in the orchestration layer, but the ledger itself must be strongly consistent and balanced.”

---

## Objective 2: Every financial action must be idempotent

Retries are guaranteed. Clients retry. Networks fail. Queues redeliver. Webhooks repeat. Bank rails duplicate callbacks. Idempotency is not optional.

### Where idempotency is required

- Public API requests
- Payment creation
- Refund creation
- Payout creation
- FX quote execution
- Ledger posting
- Internal workflow steps
- Webhook processing
- Reconciliation corrections
- Batch settlement processing

### Rigorous standard

```http
POST /v1/payments
Idempotency-Key: 8f9b1c6a-3f2e-4a4b-9c2d-1f7e6a5b3c9d
```

If a client retries the same request with the same idempotency key and payload, return the original response.

If the same idempotency key is used with a different payload, return a conflict error.

### What to say

> “I would store the idempotency key, request hash, response code, response body, and processing state. Duplicate requests return the original result instead of creating a new payment.”

---

## Objective 3: Explicitly model the payment lifecycle as a state machine

Top fintech companies do not use vague statuses like `success` and `failed` only. They model the full lifecycle.

Example payment states:

```text
CREATED
PENDING
RISK_CHECKING
AUTHORIZED
CAPTURED
PROCESSING
SETTLED
FAILED
CANCELLED
REVERSED
REFUNDED
PARTIALLY_REFUNDED
DISPUTED
UNKNOWN
```

### Rigorous standard

- State transitions are atomic.
- Invalid transitions are rejected.
- Terminal states cannot be mutated.
- Unknown states are first-class.
- Timeouts are handled explicitly.
- External rail callbacks are validated against current state.

### Example state transitions

```text
CREATED -> PENDING
PENDING -> AUTHORIZED
AUTHORIZED -> CAPTURED
AUTHORIZED -> REVERSED
CAPTURED -> SETTLED
CAPTURED -> REFUNDED
PENDING -> FAILED
UNKNOWN -> SETTLED
UNKNOWN -> FAILED
UNKNOWN -> REVERSED
```

### What to say

> “External payment rails are unreliable, so I would not assume a timeout means failure. I would use an explicit UNKNOWN or PROCESSING state and resolve it through polling, reconciliation, or asynchronous rail notifications.”

---

## Objective 4: Separate the payment orchestration layer from the ledger

This is one of the biggest differentiators between average and top-tier answers.

### Payment orchestration

Responsible for:

- API requests
- Payment intent
- Risk checks
- Fraud scoring
- FX quotes
- Routing
- Retry logic
- External rail communication
- Status webhooks
- User-facing state

### Ledger

Responsible for:

- Double-entry accounting
- Account balances
- Immutable journal entries
- Fees
- refunds
- reversals
- settlements
- internal asset/liability tracking
- auditability

### Why this matters

The payment may be processing externally, but the ledger must only move when the business has enough certainty to post an accounting entry.

### What to say

> “I would separate the mutable workflow state from the immutable accounting ledger. The orchestrator manages the process and external rails. The ledger records the financial truth using double-entry bookkeeping.”

---

## Objective 5: Use double-entry bookkeeping for the ledger

A top-tier fintech design should mention double-entry accounting, even at a high level.

### Core ledger entities

```text
Account
LedgerEntry
JournalEntry
Transaction
Currency
Balance
Posting
ReversalEntry
```

### Example ledger posting

A customer pays $100 using a card.

Simplified double-entry:

```text
Debit:  Customer Receivable / Settlement Asset   $100
Credit: Customer Payable / Merchant Liability    $100
```

When fees are charged:

```text
Debit:  Fee Expense / Merchant Fee Account       $3
Credit: Fee Revenue                              $3
```

### Rigorous standard

Each journal entry must satisfy:

```text
sum(debits) - sum(credits) = 0
```

per currency.

### What to say

> “The ledger uses append-only double-entry postings. Corrections are made by reversal entries, not by updating or deleting ledger rows.”

---

## Objective 6: Make the ledger strongly consistent

For the ledger, eventual consistency is dangerous unless you have extremely strong reconciliation and compensation controls.

### Rigorous standard

- Ledger writes must be transactional.
- Journal entries must either fully post or not post at all.
- Balances must not drift from ledger entries.
- Multi-region ledger replication must avoid split-brain.

### Good approach

Use a strongly consistent relational database or distributed database with strong consistency guarantees for the ledger core.

Examples:

- PostgreSQL with logical replication and careful failover
- CockroachDB
- Spanner
- YugabyteDB
- Aurora with strong failover discipline
- Custom ledger service with consensus-based replication

### What to say

> “Read models and notifications can be eventually consistent, but the ledger itself should be strongly consistent. We should not tolerate unbalanced entries or double-posted money merely for scalability.”

---

## Objective 7: Treat external systems as unreliable and adversarial

In financial systems, external banks, card networks, ACH rails, SEPA rails, wallet providers, FX providers, and sanctions vendors will fail in messy ways.

### Failure modes to mention

- Timeouts
- Duplicate callbacks
- Out-of-order callbacks
- Unknown responses
- Rail maintenance windows
- Partial settlements
- Incorrect amounts
- Currency mismatches
- Duplicate settlement files
- Invalid webhook signatures
- Reversed payments
- Chargebacks
- Sanctions hits after initial approval
- Bank rejection after initial acceptance

### Rigorous standard

Design for:

```text
Unknown is a valid state.
Retry is normal.
External state may conflict with internal state.
Reconciliation resolves truth.
```

### What to say

> “I would not trust external rail responses blindly. I would validate signatures, compare amounts and currencies, enforce idempotency, and use reconciliation to resolve mismatches.”

---

## Objective 8: Use outbox and inbox patterns for reliable event processing

This is a very strong signal in fintech interviews.

## Outbox Pattern

Used when a local database transaction must also publish an event.

Example:

```text
1. Write payment record
2. Write ledger entry
3. Write event to outbox table
4. Commit transaction
5. Relay publishes event
```

This prevents:

```text
Database committed but event lost
Event published but database rolled back
```

## Inbox Pattern

Used to make event consumers idempotent.

Example:

```text
1. Receive event
2. Check inbox using event_id
3. If processed, return success
4. If not processed, process and store event_id
```

This prevents duplicate processing from queue redelivery.

### What to say

> “I would use the transactional outbox pattern to guarantee that state changes and events are committed atomically. On the consumer side, I would use an inbox table for idempotent event processing.”

---

## Objective 9: Use sagas and compensating actions, not naive distributed transactions

Do not say you will use two-phase commit across the payment gateway, ledger, risk engine, bank rail, notification service, and wallet service.

That is a red flag.

### Better approach

Use a saga or workflow orchestrator.

Example:

```text
Create payment intent
Reserve funds
Run risk check
Call external rail
If external rail fails:
  release funds
  record failure
If external rail times out:
  mark unknown
  reconcile later
If settlement succeeds:
  post ledger entry
  notify user
```

### Compensating actions

- Reverse authorization
- Release reserved balance
- Refund captured payment
- Post reversal ledger entry
- Mark dispute liability
- Move to manual review queue

### What to say

> “I would use a saga-based workflow with compensating actions. The system must recover from partial failure without corrupting financial state.”

---

## Objective 10: Reconciliation is a first-class product feature

Top fintech systems do not treat reconciliation as an afterthought.

### Types of reconciliation

| Type | Description |
|---|---|
| Internal reconciliation | Orchestrator state vs ledger state |
| External reconciliation | Internal payments vs bank/rail settlement files |
| Fee reconciliation | Expected fees vs charged fees |
| FX reconciliation | Expected FX rate vs settled FX rate |
| Settlement reconciliation | Gross volume vs net settlement |
| Balance reconciliation | Cached balances vs ledger-derived balances |
| Wallet reconciliation | Customer wallet balance vs ledger liability |
| Dispute reconciliation | Chargebacks vs original payments |

### Rigorous standard

- Reconcile near real-time where possible.
- Detect breaks automatically.
- Classify breaks by severity.
- Auto-match common cases.
- Route unknown breaks to ops queues.
- Never silently mutate ledger to fix a break.

### What to say

> “I would design reconciliation as a continuous system, not only an end-of-day batch. The system should detect breaks between internal payment records, ledger entries, external rail files, fees, and settlement balances.”

---

## Objective 11: Handle money precision, currencies, and rounding correctly

This is a subtle but powerful signal.

### Rules

- Do not use floats for money.
- Use decimal types or integer minor units.
- Use ISO 4217 currency codes.
- Respect currency exponent.
- Define rounding rules explicitly.
- Handle zero-decimal currencies like JPY.
- Handle high-precision FX rates separately from final settlement amounts.

### Examples

```json
{
  "amount": "100.00",
  "currency": "USD"
}
```

or integer minor units:

```json
{
  "amount_minor_units": 10000,
  "currency": "USD"
}
```

### FX rounding issue

If converting:

```text
100.00 USD -> EUR at 0.923456789
```

You must define:

- How many decimal places are used?
- When is rounding applied?
- Who absorbs fractional remainder?
- Are fees applied before or after FX?
- Does the user see the exact rate and final amount?

### What to say

> “I would avoid floating-point arithmetic. Amounts would be represented as decimals or integer minor units, and rounding rules would be explicit per currency and per product.”

---

## Objective 12: Design fees as a first-class financial domain

Do not treat fees as a random number.

### Fee dimensions

- Fixed fee
- Percentage fee
- FX spread
- Rail-specific fee
- Country-specific fee
- Currency-specific fee
- Interchange fees
- Scheme fees
- Processing fees
- Minimum fee
- Maximum fee
- Tax on fees
- Fee currency
- Who pays the fee
- When fee is recognized

### Example fee object

```json
{
  "fee_id": "fee_01J9XYZ",
  "currency": "USD",
  "gross_amount": "100.00",
  "fee_amount": "2.90",
  "net_amount": "97.10",
  "breakdown": [
    {
      "type": "processing_fee",
      "amount": "2.50"
    },
    {
      "type": "fixed_fee",
      "amount": "0.40"
    }
  ]
}
```

### Accounting implication

Fees need separate ledger postings for:

- Gross amount
- Fee revenue
- Net settlement
- Refunded fees
- Reversed fees

### What to say

> “Fees would be modeled as explicit, versioned, auditable objects. They would be calculated before ledger posting and reconciled against settlement files.”

---

## Objective 13: Build fraud, risk, and sanctions controls into the flow

Top fintech companies do not treat fraud as a bolt-on.

### Risk controls to mention

- Velocity checks
- Device fingerprinting
- IP/geolocation checks
- Transaction limits
- Daily/monthly limits
- Country risk
- Currency risk
- Sanctions screening
- AML monitoring
- KYC/KYB status
- 3DS / SCA
- Behavioral anomaly detection
- Card testing prevention
- Payout allowlists
- Beneficiary verification
- Cooling-off periods for new accounts
- Step-up authentication

### Placement in flow

```text
Create payment
  -> identity/auth
  -> limits
  -> risk scoring
  -> sanctions screening
  -> FX quote
  -> payment routing
  -> rail execution
  -> ledger posting
```

### Rigorous standard

Risk controls must have:

- Timeout behavior
- Fallback behavior
- Audit logs
- Decision reason codes
- Manual review queues
- Ability to freeze accounts
- Ability to reverse suspicious transactions

### What to say

> “Risk and sanctions checks would be synchronous for high-risk money-out flows, with explicit fail-closed behavior when required by policy.”

---

## Objective 14: Enforce limits and exposure controls

Financial systems need hard limits.

### Types of limits

- Per-transaction limit
- Daily limit
- Monthly limit
- Currency-specific limit
- Country-specific limit
- Account lifetime limit
- KYC-tier limits
- Merchant limits
- Payout limits
- FX exposure limits
- Liquidity limits
- Credit limits
- Settlement exposure limits

### Rigorous standard

Limits must be checked atomically with the transaction request.

Avoid race conditions where two concurrent payments both pass a limit check.

Use:

- database row locks,
- atomic counters,
- distributed lock-free ledgers,
- or ledger-based reservations.

### Example

```text
Available balance: 1000
Two concurrent withdrawals of 800 each
```

Bad system allows both.  
Good system allows only one.

### What to say

> “Limit checks must be transactionally consistent with balance reservations to prevent race-condition overdrafts.”

---

## Objective 15: Security and compliance are functional requirements

In fintech, compliance is not a footnote.

### Compliance and regulatory topics to mention

| Area | Examples |
|---|---|
| PCI DSS | Card data tokenization, no raw PAN storage |
| PSD2 / SCA | Strong Customer Authentication in Europe |
| AML | Anti-money-laundering monitoring |
| KYC / KYB | Customer and business identity verification |
| Sanctions | OFAC, EU, UN sanctions lists |
| GDPR | PII minimization, right to erasure constraints |
| Data residency | Country-specific data storage rules |
| SOC 2 | Audit controls, access reviews |
| Open Banking | Consent, account access, payment initiation |
| Local rails | SEPA, ACH, Faster Payments, PIX, UPI, etc. |
| Tax reporting | 1099-K, DAC7, local tax rules |

### Security controls

- TLS everywhere
- mTLS internally
- API authentication
- OAuth 2.0 / OIDC
- Short-lived tokens
- RBAC/ABAC
- KMS encryption
- Field-level encryption for PII
- Tokenization of card and bank account details
- Webhook signature verification
- Secret rotation
- Audit logging
- Privileged access approval workflows
- Break-glass controls

### What to say

> “I would assume card and bank details are tokenized and stored only in PCI-compliant vaults. The core payment system should operate on tokens, not raw sensitive data.”

---

## Objective 16: Design for auditability and forensic traceability

Financial systems must answer:

```text
Who did what?
When?
From where?
Why?
With what authorization?
What changed?
What was the state before and after?
```

### Audit log contents

```json
{
  "audit_id": "aud_123",
  "actor_id": "user_456",
  "actor_type": "customer",
  "action": "payment.created",
  "resource_type": "payment",
  "resource_id": "pay_789",
  "ip_address": "203.0.113.10",
  "user_agent": "App/2.0",
  "request_id": "req_abc",
  "idempotency_key": "idem_123",
  "before_state": {},
  "after_state": {},
  "created_at": "2026-07-11T14:30:00Z"
}
```

### Ledger corrections

Never do:

```sql
UPDATE ledger_entries SET amount = 90 WHERE id = 123;
```

Do:

```text
Post a reversal entry.
Post a correcting entry.
Reference the original entry.
Attach operator, reason, and approval metadata.
```

### What to say

> “The ledger must be append-only. Corrections are made with compensating ledger entries that preserve the full audit trail.”

---

## Objective 17: Define SLOs and failure modes explicitly

Top candidates do not just say “highly available.” They define service tiers.

### Example service tiers

| Service | Importance | Availability target | Notes |
|---|---:|---:|---|
| Ledger writes | Critical | 99.95%+ | Cannot lose data |
| Payment authorization | Critical | 99.95%+ | Latency-sensitive |
| External rail integration | Critical but externally constrained | Depends on rail | Need degradation |
| Webhooks | Important | 99.9%+ | At-least-once delivery |
| Reporting | Important | 99.9% | Can lag |
| Dashboard reads | Lower | 99.9% | Can use read replicas |
| Notifications | Lower | 99.5%+ | Async |

### Example latency targets

| Operation | Target |
|---|---:|
| API authentication | < 50 ms p99 |
| Risk decision | < 100–200 ms p99 |
| Payment intent creation | < 200 ms p99 internal |
| Card authorization | < 500 ms to 1s p99, rail-dependent |
| Ledger posting | < 100 ms p99 internal |
| Webhook delivery first attempt | < 5s after event |

### Degradation scenarios

- Risk engine down:
    - decline high-risk transactions
    - allow low-risk transactions
    - route to manual review
- FX provider down:
    - disable FX
    - use cached rate only if policy allows
- Bank rail down:
    - queue payments
    - show pending state
    - prevent duplicate submission
- Ledger read replica lag:
    - serve stale reads with warning
    - or route balance-critical reads to primary
- Webhook consumer down:
    - retain events
    - retry with backoff
    - preserve ordering where required

### What to say

> “I would define SLOs per critical user journey and build explicit degradation paths for dependencies like risk, FX, and external rails.”

---

## Objective 18: Design for scale without violating consistency

Financial systems scale by isolating domains, not by making the ledger eventually consistent.

### Scaling strategies

#### 1. Partition by customer/account

Shard or partition ledger by account ID, user ID, or merchant ID.

But beware:

- cross-account transfers
- global settlement accounts
- hot accounts
- platform fee accounts

#### 2. Separate reads from writes

Use CQRS:

```text
Write path:
  API -> orchestrator -> ledger

Read path:
  ledger events -> read model -> API/dashboard/reporting
```

#### 3. Use asynchronous fanout for non-critical work

After payment is safely committed:

- send webhook
- send notification
- update analytics
- update fraud features
- update reporting

#### 4. Use batching for settlement and reporting

Do not force every report query against the primary ledger.

#### 5. Avoid distributed locks where possible

Use local transactionality and idempotent workflows.

### What to say

> “I would scale reads using event-driven projections and keep the ledger write path strongly consistent. Non-critical side effects would be asynchronous.”

---

## Objective 19: Make webhooks reliable, signed, and idempotent

Fintech APIs depend heavily on webhooks.

### Webhook requirements

- HTTPS only
- Signed payloads
- Timestamp validation
- Replay protection
- Retry with exponential backoff
- At-least-once delivery
- Consumer-side idempotency
- Event versioning
- Dead-letter queue after repeated failures
- Dashboard replay capability

### Example event

```json
{
  "id": "evt_01J9XYZ",
  "type": "payment.succeeded",
  "created_at": "2026-07-11T14:30:00Z",
  "data": {
    "payment_id": "pay_123",
    "status": "succeeded",
    "amount": "100.00",
    "currency": "USD"
  }
}
```

### What to say

> “Webhooks would be at-least-once, signed, versioned, and replayable. Consumers must handle duplicate events using event IDs.”

---

## Objective 20: Design for operations, incidents, and manual intervention

A top-tier answer recognizes that financial systems need operational tooling.

### Ops capabilities

- Search payments by ID, user, reference, idempotency key
- View full event history
- View ledger entries linked to payment
- Freeze accounts
- Manually retry stuck workflows
- Force reconciliation breaks into review queue
- Approve manual corrections with four-eyes control
- Replay events safely
- Export regulator reports
- Generate settlement reports
- Simulate external rail failures in staging
- Run incident playbooks

### Four-eyes principle

Sensitive operations require two approvals:

```text
Ops engineer proposes correction
Compliance/risk officer approves
System records both approvals
```

### What to say

> “I would build an internal operations console with strict RBAC, audit logs, and approval workflows. Manual corrections should never be direct database mutations.”

---

# 3. A Strong Mental Model for the Interview

Use this architecture pattern for most financial system design questions.

```text
Clients / Merchants / Apps
        |
    API Gateway
        |
Identity / Authentication / Authorization
        |
Product API
  - Payments
  - Transfers
  - Payouts
  - Refunds
  - FX
  - Wallets
        |
Workflow Orchestrator / Saga Engine
        |
+----------------+----------------+----------------+
|                |                |                |
Risk Engine   FX Engine       Ledger Service   Limits Engine
|                |                |                |
+----------------+----------------+----------------+
        |
Rail Adapter Layer
  - Card network
  - ACH
  - SEPA
  - Faster Payments
  - Wire
  - Wallet
        |
External Banks / Networks / Providers
        |
Reconciliation Service
        |
Reporting / Settlement / Notifications / Webhooks
```

---

# 4. Core Domain Objects to Mention

Use language that shows domain fluency.

## Payment Domain

```text
PaymentIntent
PaymentAttempt
PaymentMethod
Authorization
Capture
Refund
Reversal
Dispute
Chargeback
Payout
Transfer
Quote
FXTrade
SettlementBatch
```

## Ledger Domain

```text
Account
JournalEntry
LedgerEntry
Posting
Balance
Currency
AccountingPeriod
ReversalEntry
AdjustmentEntry
```

## Risk Domain

```text
RiskDecision
SanctionsScreeningResult
AMLAlert
LimitCheckResult
FraudCase
ManualReviewQueue
```

## Operations Domain

```text
ReconciliationBreak
SettlementFile
OpsCase
AuditLog
Incident
ApprovalWorkflow
```

---

# 5. The Best High-Level Answer Structure

Use this structure in the interview.

## Step 1: Clarify the product

Ask:

```text
What are we building exactly?
- Payments?
- Wallet?
- Payouts?
- FX?
- Merchant settlement?
- Internal ledger?
- Card issuing?
- Bank transfer?

Who are the users?
- Consumers?
- Merchants?
- Businesses?
- Platform marketplaces?

Which geographies?
Which currencies?
Which rails?
What regulatory constraints?
What scale?
What latency expectations?
```

## Step 2: Define functional requirements

Example:

```text
- Create payment
- Authorize payment
- Capture payment
- Refund payment
- Handle disputes
- Store payment method tokens
- Support multiple currencies
- Generate settlement reports
- Send webhooks
```

## Step 3: Define non-functional requirements

Say:

```text
- No lost or duplicated money
- Idempotent APIs
- Strongly consistent ledger
- Auditability
- Reconciliation
- High availability
- Low-latency authorization
- Regulatory compliance
- Fraud controls
```

## Step 4: Define invariants

Example:

```text
- Ledger entries must balance.
- A payment cannot be captured unless authorized.
- A refund cannot exceed captured amount.
- A payout cannot exceed available balance.
- A duplicate request must not create a duplicate payment.
- Unknown external states must be resolvable.
```

## Step 5: Draw the architecture

Show:

```text
API layer
Orchestration layer
Ledger
Risk
FX
Rail adapters
Reconciliation
Webhooks
Reporting
```

## Step 6: Deep dive into one or two hard problems

Choose:

- Payment state machine under timeouts
- Ledger consistency
- Idempotency
- Reconciliation
- Fraud and limits
- Multi-currency settlement
- Webhook delivery
- Refund/dispute handling

## Step 7: Discuss failure modes

This is where top 1% candidates shine.

Mention:

```text
- External bank timeout
- Duplicate webhook
- Out-of-order rail notification
- Partial settlement file
- Risk engine down
- FX rate expired
- Database failover during payment
- Queue redelivery
- Double-spend race condition
- Reconciliation break
```

## Step 8: Discuss operations

Say:

```text
- Monitoring
- Alerting
- Replayability
- Manual review queues
- Audit logs
- Break resolution
- Incident response
```

---

# 6. The Most Rigorous Interview Objectives Checklist

You can use this as your actual checklist.

## A. Problem Framing

- [ ] Clarify whether the system is payments, ledger, wallet, payout, FX, issuing, or settlement.
- [ ] Identify users, currencies, geographies, rails, and regulatory constraints.
- [ ] Define critical user journeys.
- [ ] Define success metrics and failure budgets.
- [ ] State assumptions explicitly.

## B. Correctness and Money Safety

- [ ] Double-entry ledger.
- [ ] Immutable ledger entries.
- [ ] Balanced journal entries.
- [ ] No balance mutation without ledger posting.
- [ ] Idempotent API and workflow steps.
- [ ] No duplicate money movement.
- [ ] No lost money movement.
- [ ] Explicit rounding rules.
- [ ] Decimal or minor-unit money representation.
- [ ] Multi-currency handling.

## C. State Machines and Workflows

- [ ] Explicit payment/transfer state machine.
- [ ] Atomic state transitions.
- [ ] Terminal states.
- [ ] Unknown/pending state.
- [ ] Timeout handling.
- [ ] Retry handling.
- [ ] Compensation/reversal flows.
- [ ] Refund and dispute flows.
- [ ] Manual review states.

## D. Distributed System Reliability

- [ ] Outbox pattern.
- [ ] Inbox pattern.
- [ ] At-least-once event delivery.
- [ ] Idempotent consumers.
- [ ] Saga orchestration.
- [ ] No naive 2PC across external services.
- [ ] Dead-letter queues.
- [ ] Retry with backoff.
- [ ] Circuit breakers.
- [ ] Bulkheads.
- [ ] Timeouts.
- [ ] Graceful degradation.

## E. Reconciliation and Settlement

- [ ] Internal reconciliation.
- [ ] External rail reconciliation.
- [ ] Fee reconciliation.
- [ ] FX reconciliation.
- [ ] Settlement batching.
- [ ] Break detection.
- [ ] Ops review queues.
- [ ] Auto-matching rules.
- [ ] End-of-day and near-real-time reconciliation.
- [ ] Audit trail for corrections.

## F. Risk, Fraud, Compliance

- [ ] KYC/KYB status.
- [ ] AML monitoring.
- [ ] Sanctions screening.
- [ ] Fraud scoring.
- [ ] Velocity checks.
- [ ] Transaction limits.
- [ ] Country/currency restrictions.
- [ ] Strong Customer Authentication where required.
- [ ] PCI DSS tokenization.
- [ ] GDPR/data residency considerations.
- [ ] Manual review workflows.
- [ ] Account freezing.
- [ ] Suspicious activity reporting support.

## G. Security

- [ ] OAuth/OIDC authentication.
- [ ] RBAC/ABAC.
- [ ] mTLS internally.
- [ ] Encryption at rest and in transit.
- [ ] Tokenization of sensitive data.
- [ ] Webhook signatures.
- [ ] Secret rotation.
- [ ] Audit logs.
- [ ] Privileged access controls.
- [ ] Four-eyes approvals.

## H. Scalability and Performance

- [ ] Partitioning strategy.
- [ ] Read/write separation.
- [ ] Event-driven projections.
- [ ] Caching carefully used.
- [ ] Async processing for non-critical paths.
- [ ] Batch settlement/reporting.
- [ ] Hot-account mitigation.
- [ ] Load shedding.
- [ ] Rate limiting.
- [ ] Backpressure.
- [ ] Multi-region strategy.

## I. Observability and Operability

- [ ] Request IDs.
- [ ] Idempotency keys.
- [ ] Event tracing.
- [ ] Ledger-to-payment traceability.
- [ ] Reconciliation dashboards.
- [ ] Fraud dashboards.
- [ ] Rail error dashboards.
- [ ] Alerting on breaks.
- [ ] Replay tools.
- [ ] Manual correction tools.
- [ ] Incident runbooks.

## J. Testing and Verification

- [ ] Idempotency tests.
- [ ] Duplicate webhook tests.
- [ ] Timeout tests.
- [ ] Partial failure tests.
- [ ] Concurrency tests.
- [ ] Ledger invariant tests.
- [ ] Reconciliation simulation.
- [ ] Chaos testing.
- [ ] Regulatory scenario tests.
- [ ] Load testing.
- [ ] Failover testing.

---

# 7. Top 1% Differentiators

These are the things that make you sound senior/staff/principal.

## 1. “Unknown is a valid state”

Average candidate:

> “If the bank call times out, we mark the payment failed.”

Top candidate:

> “If the external rail times out, the payment is in an unknown state. We should not immediately fail it because the rail may have accepted it. We poll, reconcile, and use rail notifications to converge to a terminal state.”

---

## 2. “Ledger is append-only”

Average candidate:

> “We update the user balance.”

Top candidate:

> “We create an immutable ledger entry. Balance is derived from entries. Corrections are reversals, not updates.”

---

## 3. “Reconciliation is continuous”

Average candidate:

> “We run a nightly batch job.”

Top candidate:

> “We use both near-real-time event-based reconciliation and end-of-day settlement reconciliation. Breaks are detected, classified, routed to ops, and resolved with auditability.”

---

## 4. “Side effects are asynchronous and idempotent”

Average candidate:

> “After payment succeeds, we send webhook, email, analytics, and update balance.”

Top candidate:

> “The critical path commits the payment and ledger atomically. Then we use the outbox pattern to fan out webhooks, notifications, analytics, and reporting asynchronously and idempotently.”

---

## 5. “Compliance is part of the flow”

Average candidate:

> “We can add KYC later.”

Top candidate:

> “KYC, sanctions screening, limits, and AML checks are functional requirements that influence state transitions. Certain flows must fail closed if compliance checks cannot be completed.”

---

# 8. Red Flags That Will Hurt You

Avoid these answers.

## Red Flag 1: Using floats for money

Bad:

```json
{
  "amount": 10.07
}
```

Good:

```json
{
  "amount": "10.07",
  "currency": "USD"
}
```

or:

```json
{
  "amount_minor_units": 1007,
  "currency": "USD"
}
```

---

## Red Flag 2: Mutating ledger rows

Bad:

```sql
UPDATE ledger SET amount = 50 WHERE id = 1;
```

Good:

```text
Create reversal entry + corrected entry.
```

---

## Red Flag 3: Assuming external calls are reliable

Bad:

> “We call the bank and if it returns success, we mark success.”

Good:

> “We handle timeout, duplicate, delayed, and conflicting responses.”

---

## Red Flag 4: No idempotency

Bad:

> “The client can retry if the request fails.”

Good:

> “The client supplies an idempotency key. We store request and response state to return the original result safely.”

---

## Red Flag 5: Making ledger eventually consistent without a plan

Bad:

> “The ledger can be eventually consistent like other microservices.”

Good:

> “Read models can be eventually consistent, but the ledger should be strongly consistent or protected by very strong reconciliation and compensation controls.”

---

## Red Flag 6: No reconciliation

Bad:

> “The payment service and ledger will stay in sync because we use transactions.”

Good:

> “Even with transactions, external rails and distributed workflows create discrepancies, so reconciliation is required.”

---

## Red Flag 7: Ignoring compliance

Bad:

> “Compliance is out of scope.”

Good:

> “Compliance may be abstracted in the interview, but I would identify where KYC, AML, sanctions, PCI, and data residency affect the design.”

---

# 9. A Sample Top 1% Opening Statement

You can say something like this:

> “Before jumping into scaling, I want to establish the key invariants for a financial system: no lost or duplicated money, strongly consistent ledger entries, idempotent APIs, explicit payment state machines, reliable reconciliation, and full auditability.
>
> I would separate the orchestration layer from the accounting ledger. The orchestration layer handles payments, retries, external rails, risk, FX, and webhooks. The ledger is the source of financial truth and uses immutable double-entry postings.
>
> External systems like banks and card networks are unreliable, so I would treat unknown responses as a first-class state and resolve them using polling, reconciliation, and compensating actions.
>
> From there, we can discuss scaling, latency, and multi-region reliability without compromising money correctness.”

That immediately positions you at a high level.

---

# 10. Company-Specific Emphasis

Different companies may emphasize different parts.

## Stripe-like interview

Focus on:

- Developer experience
- API design
- Idempotency
- Webhooks
- PaymentIntent abstraction
- Retry safety
- Global payment methods
- Extensible state machines
- Error clarity

Important concepts:

```text
PaymentIntent
ConfirmToken
Idempotency-Key
Webhook event types
Delayed notification methods
Payment method reuse
```

---

## Adyen-like interview

Focus on:

- Global acquiring
- Payment routing
- Risk engine
- Settlement
- Reconciliation
- Merchant reporting
- Multi-currency
- Scheme/network fees
- Authorization optimization

Important concepts:

```text
Routing
Authorization rates
Settlement batches
Fee reconciliation
Multi-rail support
Local payment methods
```

---

## Wise-like interview

Focus on:

- Multi-currency balances
- FX transparency
- Mid-market rates
- Local rails
- Cross-border payments
- Liquidity management
- Regulatory licensing
- Currency-specific rails
- Real-time conversion

Important concepts:

```text
Multi-currency wallet
FX quote
Currency pair liquidity
Local account details
Cross-border settlement
Transparency of fees and rates
```

---

## Revolut-like interview

Focus on:

- Consumer app
- Real-time UX
- Card issuing
- FX
- Notifications
- Limits
- Fraud controls
- Multi-currency accounts
- Instant transfers
- High-scale mobile traffic

Important concepts:

```text
Real-time balance updates
Card authorization
FX rate locking
Spending limits
Fraud detection
Push notifications
Instant freezing/unfreezing
```

---

# 11. A Strong Example: Design a Payment System

If asked:

> “Design a payment processing system.”

You could answer with these objectives.

## Functional requirements

```text
- Accept payment creation
- Authorize payment
- Capture payment
- Refund payment
- Handle disputes
- Support cards and bank transfers
- Support multiple currencies
- Provide merchant dashboard
- Send webhooks
```

## Non-functional requirements

```text
- Idempotency
- No duplicate charges
- Strong ledger consistency
- Reconciliation
- PCI-compliant tokenization
- Fraud checks
- High availability
- Low-latency authorization
```

## High-level design

```text
Merchant API
  -> API Gateway
  -> Payment Service
  -> Risk Service
  -> Ledger Service
  -> Rail Adapter
  -> External Card Network/Bank
  -> Reconciliation Service
  -> Webhook Service
```

## Payment state machine

```text
CREATED
PENDING
AUTHORIZED
CAPTURED
SETTLED
FAILED
CANCELLED
REFUNDED
DISPUTED
UNKNOWN
```

## Key invariants

```text
- Cannot capture without authorization.
- Cannot refund more than captured.
- Cannot post ledger entry unless payment state justifies it.
- Duplicate request returns original payment.
- Unknown rail timeout goes to UNKNOWN, not FAILED.
```

## Failure handling

```text
Card network timeout:
  mark UNKNOWN
  poll or wait for async notification
  reconcile with settlement file

Duplicate webhook:
  use event_id inbox to ignore duplicate

Ledger posting failure:
  retry with idempotency key
  alert if repeated
  do not mutate payment state without ledger certainty
```

---

# 12. A Strong Example: Design a Wallet System

If asked:

> “Design a digital wallet.”

## Core objectives

```text
- Customer balances
- Deposits
- Withdrawals
- Transfers
- Holds/reservations
- FX balances
- Transaction history
- Ledger auditability
```

## Key invariants

```text
- Available balance = ledger balance - holds - pending debits
- No transfer can double-spend
- Concurrent transfers must not overdraw
- Wallet liability must reconcile against ledger
- Deposits from external rails must be matched to wallet credits
```

## Ledger model

```text
Customer wallet account
Platform liability account
Settlement account
Fee account
Suspense account
```

### Transfer example

User A sends $50 to User B:

```text
Debit User A wallet liability   $50
Credit User B wallet liability  $50
```

Or depending on accounting structure:

```text
Debit User A balance
Credit User B balance
```

The important point is that the entry is balanced, immutable, and currency-aware.

---

# 13. A Strong Example: Design FX Exchange

If asked:

> “Design a currency exchange feature.”

## Objectives

```text
- Transparent rate
- Locked quote
- Fee breakdown
- No stale-rate execution
- Idempotent execution
- Multi-currency ledger
- FX reconciliation
```

## Flow

```text
Get rate
  -> Create quote
  -> Lock quote with TTL
  -> Execute exchange
  -> Post ledger entries
  -> Reconcile FX provider settlement
```

## Important risks

```text
Rate expired
FX provider unavailable
Concurrent quote execution
Fractional rounding
Multiple currency balances
Regulatory disclosure requirements
```

## Quote object

```json
{
  "quote_id": "q_123",
  "sell_currency": "USD",
  "buy_currency": "EUR",
  "sell_amount": "1000.00",
  "buy_amount": "921.40",
  "exchange_rate": "0.92140",
  "fee": "5.00",
  "expires_at": "2026-07-11T14:30:30Z"
}
```

---

# 14. Metrics to Mention

Top candidates quantify their objectives.

## Correctness metrics

```text
Unbalanced ledger entries: 0
Duplicate payments caused by retries: 0
Lost payment events: 0
Reconciliation breaks detected: near real-time
Manual correction rate: trending down
```

## Reliability metrics

```text
Payment authorization success rate
Rail error rate
Timeout rate
Unknown state convergence time
Webhook delivery success rate
Dead-letter queue depth
```

## Latency metrics

```text
API p50/p95/p99
Risk decision latency
Ledger posting latency
External rail latency
Webhook first delivery latency
```

## Operational metrics

```text
Reconciliation break aging
Ops queue backlog
Fraud false-positive rate
Fraud loss rate
Manual review throughput
Incident mean time to detect
Incident mean time to recover
```

---

# 15. A Possible Final Architecture Blueprint

For many fintech interview questions, this blueprint is strong.

```text
                       Clients / Merchants / Internal Ops
                                      |
                                  API Gateway
                                      |
                         Authentication / Authorization
                                      |
                 +--------------------+--------------------+
                 |                    |                    |
          Payments API          Wallets API           FX API
                 |                    |                    |
                 +--------------------+--------------------+
                                      |
                           Workflow Orchestrator
                                      |
        +----------------+------------+------------+----------------+
        |                |                         |                |
   Risk Engine      Limits Engine              FX Engine       Ledger Service
        |                |                         |                |
        +----------------+------------+------------+----------------+
                                      |
                              Rail Adapter Layer
                                      |
             +----------------+-------+--------+----------------+
             |                |                |                |
          Card Rail        Bank Rail       Wallet Rail      Payout Rail
             |                |                |                |
             +----------------+-------+--------+----------------+
                                      |
                          External Providers / Banks
                                      |
                          Reconciliation Service
                                      |
         +----------------+-----------+-----------+----------------+
         |                |                       |                |
   Settlement Engine  Reporting Engine       Webhook Engine   Ops Console
```

---

# 16. What “Top 1%” Sounds Like

Here are phrases that signal senior fintech thinking.

### On correctness

> “I would rather degrade the user experience than allow an unbalanced ledger or duplicate money movement.”

### On external rails

> “External rails can return success, failure, timeout, or contradictory asynchronous notifications. The design must converge safely from all of these.”

### On idempotency

> “Every state-changing endpoint and every internal workflow step must be idempotent. Retries should be safe by design.”

### On reconciliation

> “Reconciliation is not an afterthought. It is a core subsystem that validates the consistency between orchestration, ledger, external rails, fees, and settlement.”

### On compliance

> “Compliance is not just a legal concern; it changes the architecture. KYC, AML, sanctions, data residency, and PCI constraints affect data flow and service boundaries.”

### On scale

> “Scale should be achieved by isolating the critical ledger write path and moving non-critical work into asynchronous, replayable event streams.”

---

# 17. The Final Checklist Before You Finish the Interview

Before wrapping up, say:

```text
To summarize, the design is optimized for:

1. Money correctness through double-entry ledger and atomic transitions.
2. Idempotency across APIs, workflows, and event consumers.
3. Explicit handling of unknown external states.
4. Reconciliation across internal and external systems.
5. Strong auditability and immutable corrections.
6. Fraud, limits, sanctions, and compliance controls.
7. Operational visibility and manual intervention tooling.
8. Scalability through asynchronous event-driven projections.
```

Then ask:

```text
Would you like me to go deeper into the ledger model, payment state machine, failure handling, or scaling strategy?
```

That is a very strong way to close.

---

# 18. The Most Rigorous One-Page Objective List

If you want a compact list to memorize, use this:

```text
1. No lost, duplicated, or unbalanced money.
2. Strongly consistent, immutable, double-entry ledger.
3. Idempotent APIs, workflows, and event consumers.
4. Explicit payment/transfer state machine.
5. Unknown external states handled explicitly.
6. Saga-based orchestration with compensating actions.
7. Outbox pattern for reliable event publishing.
8. Inbox pattern for idempotent event consumption.
9. Continuous and end-of-day reconciliation.
10. Fee and FX reconciliation.
11. Decimal or minor-unit money handling.
12. ISO currency and rounding rules.
13. Fraud, limits, sanctions, AML, KYC in the flow.
14. PCI-safe tokenization for card/bank data.
15. Signed, replayable, at-least-once webhooks.
16. Audit logs and append-only corrections.
17. Operational tooling and manual review queues.
18. SLOs and degradation paths.
19. Scale through async projections, not weak ledger consistency.
20. Clear tradeoffs, assumptions, and failure-mode analysis.
```

---

# 19. The Single Best Guiding Principle

If you remember only one thing:

> Design the system so that even if every network call times out, every queue redelivers, every webhook duplicates, every external rail disagrees, and every service restarts, the system can still recover to a state where **no money is lost, duplicated, or unaccounted for, and every mutation is explainable.**