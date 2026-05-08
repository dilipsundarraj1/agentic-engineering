# How Coding Has Changed with AI

## Where Did the Time Go?

Think about how a developer spent their day in 2015:

| Activity              | Time Spent |
|-----------------------|------------|
| Planning what to build | 10%       |
| Writing code           | 60%       |
| Reviewing my own code  | 20%       |
| Reviewing others' code | 10%       |

Now think about 2026:

| Activity              | Time Spent |
|-----------------------|------------|
| Planning what to build | 25%       |
| Writing code           | 5%        |
| Reviewing AI code      | 35%       |
| Reviewing others' code | 35%       |

**Writing code is no longer the job. Reviewing and thinking clearly is.**

---

## A Day in the Life — Before vs After AI

### Before AI (2019)

You get a ticket: *"Build a REST client to integrate with the Payments service."*

An experienced developer knows the drill:

- 30 min: Read and parse the API contract (OpenAPI spec, Postman collection, or a Wiki page)
- 45 min: Create record/DTO classes for every request and response shape
- 30 min: Decide on a REST client library — RestTemplate, WebClient, Feign, HttpClient?
- 1 hour: Implement the client — base URL config, headers, error handling
- 45 min: Add resilience — Retry logic, Circuit Breaker, timeouts
- 1 hour: Write tests — unit tests with mocked responses, integration tests
- 30 min: Open a PR, wait for review

**Total: 1–2 days. Most of it was mechanical, repetitive work.**

---

### After AI (2026)

Same ticket.

You paste the contract. That is the input.

Your day:
- 15 min: Read the contract and think about failure scenarios and SLA expectations
- 5 min: Share the contract with AI and ask it to generate the record classes, client implementation, resilience config, and tests
- 1 hour: Review what AI produced — does the retry policy fit your SLA? Is the circuit breaker threshold sensible? Are the tests actually testing the right things?
- 15 min: Open PR

**Total: ~1.5 hours. Most of it was thinking and reviewing.**

The mechanical work — record classes, boilerplate client code, retry wiring — disappeared. What remains is the judgment work that only you can do.

---

## What This Means for You as a Developer

The shift is not "AI does the work, you relax."

The shift is: **the bottleneck moved from your fingers to your brain.**

Before AI, you could get away with a vague idea because the act of writing code forced you to work out the details.

With AI, if your idea is vague, the AI will invent its own details — and they will probably be wrong.

### Practical Example

**Vague prompt:**
```
Build a REST client to integrate with the Payments service.
```

AI will invent a base URL, pick a random HTTP client library, decide how errors and timeouts are handled, and skip resilience entirely — all without telling you.

**Clear prompt (after thinking first):**
```
Build a REST client to integrate with the Payments service using the attached OpenAPI contract.
Use WebClient with a 3-second timeout.
Retry on 5xx responses up to 3 times with exponential backoff.
Open the circuit breaker after 5 consecutive failures.
Map 4xx responses to a PaymentClientException with the error body included.
Generate request/response record classes and unit tests with mocked HTTP responses.
```

Same AI. Completely different output.

**The quality of AI output is directly proportional to the clarity of your thinking.**

---

## The New Developer Skill Set

| Old Priority       | New Priority           |
|--------------------|------------------------|
| Typing speed       | Thinking speed         |
| Memorizing APIs    | Knowing what to ask    |
| Writing code       | Reviewing code         |
| Debugging syntax   | Spotting design flaws  |
| Finding examples   | Verifying AI output    |

---

## The One Thing That Did Not Change

**Bad code is still bad code.**

AI writes code faster. That means if you do not have good fundamentals, you end up with a bad codebase faster.

The developers who will thrive are not the ones who prompt the most. They are the ones who:
- Think before they prompt
- Review what comes back
- Know when AI got it wrong

## Software Fundamentals Are More Important Than Ever

In the age of AI, fundamentals are not less relevant — they are the one thing that separates a developer who uses AI well from one who just generates a mess faster.

AI can generate code in seconds. What it cannot do:

- Judge whether the code is correct for your specific system
- Know which edge cases matter in your domain
- Decide if a design is going to cause problems at scale
- Spot when a generated test is not actually testing anything
- Choose the right resilience pattern for your SLA

That judgment is yours — and it is only possible if your fundamentals are solid.

This is exactly what this course is about.

We are not just going to talk about these ideas in theory. We are going to learn them by building and implementing real use cases, hands-on. As we write real code, you will see what good fundamentals look like in practice, why they matter when working with AI, and how they help you catch what AI gets wrong before it becomes a problem.

---

## Quick Exercise

Think about the last feature you built with AI help.

1. How much time did you spend planning before prompting?
2. Did you review the AI output line by line, or just run it and hope?
3. Did you write or verify the tests?

If planning was short and reviewing was shallow — that is the most common pattern, and it is also where most AI-related bugs come from.

In the next lesson, we look at two different approaches to working with AI: Vibecoding vs Agentic Engineering.
