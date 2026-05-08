# Vibecoding vs Agentic Engineering

## Two Developers, Same AI, Very Different Results

Meet two developers. Both are building a Spring Boot application that needs to integrate with an external Payments service.

They both use a coding assistant (GitHub Copilot, Claude Code, Cursor, etc.). They both have the same ticket.

> **Ticket:** Build a REST client to integrate with the Payments service. Use the provided OpenAPI contract. The client must handle retries, circuit breaking, and proper error mapping.

---

## Developer A: The Vibecoder

Developer A opens their AI tool and types:

```
Build a REST client for the Payments service
```

AI generates:
- A client class using whatever HTTP library it prefers, ignoring what the project already uses
- Hardcoded base URL in the client class itself
- A retry mechanism that retries on every exception including 4xx errors
- 3 new utility classes nobody asked for
- No tests

Developer A runs it. It compiles. They push it.

Two days later:
- The client retries on 400 Bad Request, flooding the Payments service with duplicate calls
- The hardcoded URL breaks in staging because it points to localhost
- The circuit breaker was never added, so one slow Payments response hangs the entire thread pool
- Nobody knows which class does what

Developer A asks AI to fix it. AI adds more code. Things break further.

**This is vibecoding.** It feels productive in the moment. It accumulates hidden damage over time.

---

## Developer B: The Agentic Engineer

Developer B stops before prompting. They think through the ticket:

- What already exists? A WebClient bean is already configured in the project. Resilience4j is on the classpath.
- What changes? A new `PaymentsClient` class that wraps WebClient calls to the Payments API.
- What must NOT change? The existing WebClient config. The project's error handling conventions.
- What are the edge cases? Retry only on 5xx — never on 4xx. Circuit breaker threshold. Timeout per call.

Then Developer B prompts:

```
I need a REST client to integrate with the Payments service using the attached OpenAPI contract.

Rules:
- Use the existing WebClient bean — do NOT create a new one
- Retry on 5xx responses only, up to 3 times with exponential backoff
- Do NOT retry on 4xx — map those to a PaymentClientException with the error body
- Add a circuit breaker using Resilience4j that opens after 5 consecutive failures
- Use a 3-second timeout per request

Start by showing me only the record classes generated from the contract.
Do not write the client yet.
```

AI generates a small, focused set of record classes. Developer B reviews them against the contract. Then moves to the next step.

**This is agentic engineering.** Slower to start. Much faster to finish correctly.

---

## Side-by-Side Comparison

| | Vibecoding | Agentic Engineering |
|---|---|---|
| **Starting point** | Open AI, start typing | Think first, then prompt |
| **Prompt style** | Vague, high-level | Specific, constrained |
| **Scope of AI output** | As much as AI decides | Only what you asked for |
| **Review** | Run it, if it compiles it's done | Read it, test it, verify it |
| **Tests** | Optional, added later (or never) | Written alongside or before code |
| **When something breaks** | Ask AI to fix it blindly | Understand why, then fix |
| **Codebase over time** | Gets harder to change | Stays clean and navigable |

---

## The Vibecoding Trap

Vibecoding means asking AI to write code without thinking about the end goal. You describe something loosely, accept what comes back, and keep going. No upfront design, no constraints, no review.

This is not inherently wrong — it is actually a great fit for proof of concepts and prototyping, where speed matters more than structure and the code will likely be thrown away.

But for production code, it is a problem.

Vibecoding feels faster — and it is, for the first 20 minutes.

But here is what happens over time:

```
Week 1: Ship fast. Everything works.
Week 2: New feature. AI touches old code. One thing breaks.
Week 3: Fix breaks something else. Add a workaround.
Week 4: Nobody wants to touch the codebase.
Week 6: Rewrite discussion begins.
```

This is called **software entropy** — the natural drift of a codebase toward disorder. AI accelerates this drift because it produces code fast without caring about the overall design.

---

## What Agentic Engineering Looks Like in Practice

### Step 1: Think Before You Prompt

Before opening any AI tool, write down:
- What is the exact behavior I want?
- What already exists that I must not break?
- What are the boundaries of this change?

For the Payments client:
```
Behavior: PaymentsClient calls POST /payments on the Payments service and returns a PaymentResponse
Must not break: existing WebClient bean configuration
Boundaries: only create PaymentsClient, the record classes, and Resilience4j config for this client
```

### Step 2: Prompt in Small Chunks

Do not ask for everything at once.

Instead of: *"Build the full Payments client"*

Do this:
```
Step 1: Generate only the request and response record classes from the contract
Step 2: Now show me only the PaymentsClient class using the existing WebClient bean
Step 3: Now add the Resilience4j retry and circuit breaker configuration
Step 4: Now write unit tests with mocked HTTP responses for success, 4xx, and 5xx cases
```

Each step is reviewable. Each step is testable. You stay in control.

### Step 3: Review With Intent

When AI returns code, do not just run it. Read it.

Ask yourself:
- Does this match what I asked for?
- Did it change anything I told it not to touch?
- Does the naming match the rest of the codebase?
- Are there any obvious edge cases not handled?

If something looks off — ask the AI to explain it before accepting it.

### Step 4: Verify With Tests

Every change should have at least one test that proves the behavior works.

If AI did not write a test — ask for one.
If AI wrote a test — read it. A test that always passes is worse than no test.

---

## Practical Exercise

Take a feature you built recently using AI.

1. Open the code.
2. Find one class that AI generated.
3. Answer these questions without running the code:
   - What does this class do?
   - What would break if I deleted it?
   - Does its name describe what it does?

If you struggle to answer — that is a sign of vibecoding. The code was generated, not understood.

Now pick one small piece you do not fully understand. Ask the AI to explain it to you. Then ask if there is a simpler way to write it.

**Understanding what AI produces is not optional. It is the job.**

---

## Summary

Vibecoding is not a strategy. It is what happens when you use AI without a process.

Agentic Engineering is a set of habits:
- Think before prompting
- Prompt in small, constrained steps
- Review every output
- Test every behavior
- Stay in control of the design

In the next lesson, we look at the specific software fundamentals that make agentic engineering possible — because without them, even the best prompts produce a mess.
