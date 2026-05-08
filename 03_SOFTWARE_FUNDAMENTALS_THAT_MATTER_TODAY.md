# Software Fundamentals That Matter More Than Ever

## Why Fundamentals? I Have AI Now.

Here is a quick experiment.

Give AI a messy codebase — scattered files, inconsistent naming, no clear boundaries — and ask it to add a feature.

Then give AI a clean codebase — clear modules, consistent naming, good tests — and ask it to add the same feature.

The output quality is dramatically different.

**AI does not fix a bad codebase. It inherits it.**

If your codebase is hard for a human to understand, it is hard for AI to understand too. And AI working in a bad codebase will produce bad code — fast.

This is why fundamentals matter *more* now, not less.

---

## Fundamental 1: Consistent Naming

### The Problem

Look at this real scenario. A Kafka project has:

- `LibraryEvent` (the domain object)
- `LibraryEventMessage` (what is sent to Kafka — same thing, different name)
- `KafkaPayload` (used in one consumer — same thing, third name)
- `EventDTO` (used in tests — same thing, fourth name)

When AI reads this codebase, it does not know which is the "real" thing. It invents a fifth name.

### The Fix: One Term, One Concept

Pick one name for each concept and use it *everywhere* — in class names, method names, variable names, comments, and AI prompts.

**Bad:**
```java
public void sendMessage(LibraryEventMessage message) { ... }
public KafkaPayload buildPayload(EventDTO dto) { ... }
```

**Good:**
```java
public void publishEvent(LibraryEvent event) { ... }
public LibraryEvent buildEvent(LibraryEvent event) { ... }
```

### Practical Action

Create a `GLOSSARY.md` file in your project root. List every core concept with its one canonical name. Use that file when prompting AI.

```markdown
| Concept         | Name in Code     | Meaning                              |
|-----------------|------------------|--------------------------------------|
| Domain event    | LibraryEvent     | An event representing a book action  |
| Event category  | EventType        | Enum: NEW or UPDATE                  |
| Failed event    | Dead Letter      | Event that failed all retries        |
```

---

## Fundamental 2: Clear Module Boundaries

### The Problem

AI loves generating lots of small files. After a few sessions, you end up with:

```
src/
  KafkaConfig.java
  KafkaProducerConfig.java
  KafkaConsumerConfig.java
  KafkaTopicConfig.java
  KafkaRetryConfig.java
  KafkaSerializerConfig.java
  KafkaErrorHandlerConfig.java
```

Seven files. All related to Kafka setup. To understand how Kafka is configured, you must read all seven.

When something breaks, where do you look? When AI needs to change the retry behavior, which file does it touch?

### The Fix: Group Related Code Behind One Boundary

Instead of seven scattered files, one well-named class that owns all Kafka infrastructure:

```java
@Configuration
public class KafkaInfrastructureConfig {

    @Bean
    public KafkaTemplate<Integer, String> kafkaTemplate() { ... }

    @Bean
    public ConsumerFactory<Integer, String> consumerFactory() { ... }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<Integer, String> listenerFactory() { ... }

    @Bean
    public NewTopic libraryEventsTopic() { ... }
}
```

One file. One place to look. One place to change.

### Why This Helps AI

When you ask AI to "change the retry settings", it now knows exactly where to look. It does not have to guess across seven files. It changes one thing in one place. The risk of touching something unrelated drops to near zero.

### Practical Action

Find any package in your project with more than 4 small, single-purpose config or utility classes that are all related to the same concept. Merge them. Give the merged class one clear name.

---

## Fundamental 3: Tests as a Safety Net

### The Problem

You ask AI to refactor the consumer logic. It does. You run the app. It seems to work.

Two weeks later, you get a bug report: retry events are no longer being sent to the dead letter topic.

The refactor broke it. Nobody caught it because there was no test for that behavior.

### The Fix: Test the Behavior You Care About

You do not need 100% code coverage. You need tests for the behaviors that matter.

For a Kafka consumer, the behaviors that matter are:

1. A valid event is processed successfully
2. A failing event is retried the correct number of times
3. After max retries, the event lands in the dead letter topic
4. An invalid event does not crash the consumer

These four scenarios, tested, give you a safety net. When AI makes a change, you run the tests. If they pass, the key behaviors are intact.

**Example — testing retry behavior:**

```java
@Test
void shouldSendEventToDeadLetterTopicAfterMaxRetries() {
    // Arrange: simulate a consumer that always throws
    doThrow(new RuntimeException("processing failed"))
        .when(libraryEventService).processLibraryEvent(any());

    // Act: send a library event
    kafkaTemplate.send("library-events", libraryEvent);

    // Assert: event lands in DLT after retries exhausted
    ConsumerRecord<Integer, String> record =
        KafkaTestUtils.getSingleRecord(deadLetterConsumer, "library-events.DLT");

    assertThat(record.value()).contains("bookId");
}
```

This test does not care *how* retries work internally. It just verifies that the end result — event in DLT — happens.

### The Key Insight

Good tests let you hand the implementation to AI and stay confident about the outcome. You define what correct looks like. AI figures out how to make it happen.

### Practical Action

Pick one consumer or service in your project. Write three tests for its three most important behaviors. Run them. They should all pass today. Now every time AI changes this code, run these three tests. If any fail — something broke.

---

## Fundamental 4: Small, Focused Changes

### The Problem

You ask AI: *"Refactor the entire consumer package and also add retry logic and also improve error handling."*

AI produces 400 lines of changes across 8 files.

Something breaks. You have no idea where.

### The Fix: One Change at a Time

Break every task into the smallest possible step that produces a verifiable result.

**Instead of:** *"Refactor the consumer and add retry logic"*

**Do this:**
```
Step 1: Show me only the current consumer class. Do not change anything yet.
Step 2: What would you change to make this more testable? Explain before changing.
Step 3: Make only that change. Nothing else.
Step 4: Now add the retry configuration. Show me what you plan before writing it.
Step 5: Write the retry logic.
Step 6: Write a test for the retry behavior.
```

Six steps. Each one small. Each one reviewable. If something breaks, you know exactly which step caused it.

### Practical Action

The next time you prompt AI for a change, add this line to your prompt:

```
Make the smallest possible change that achieves this goal.
Do not refactor unrelated code. Do not add features I did not ask for.
```

You will be surprised how much cleaner the output becomes.

---

## Fundamental 5: Understand What You Ship

### The Problem

AI writes a class. You read the first few lines. It looks fine. You accept it.

Later, a senior developer asks: *"Why are we using a ConcurrentKafkaListenerContainerFactory here instead of a regular one?"*

You do not know. AI wrote it.

### The Fix: Never Ship Code You Cannot Explain

This does not mean you need to understand every internal detail. It means you need to understand:

- What does this code do?
- Why was this approach chosen over alternatives?
- What would break if this were removed?

If you cannot answer these three questions, ask AI before accepting the code:

```
Before I accept this — explain why you used ConcurrentKafkaListenerContainerFactory
instead of a regular factory. What are the tradeoffs?
```

AI will explain it. You will learn something. And you will be able to defend the decision in a code review.

### Practical Action

For one week, add this rule: before accepting any AI-generated code, read it out loud in plain English. If you stumble, you do not understand it well enough. Ask AI to explain the part where you stumbled.

---

## Putting It All Together

These five fundamentals are not new ideas. They have been best practices for decades.

What is new is the cost of ignoring them.

| Fundamental          | What AI Does Without It       | What AI Does With It           |
|----------------------|-------------------------------|--------------------------------|
| Consistent naming    | Invents new terms constantly  | Uses your language correctly   |
| Clear boundaries     | Touches unrelated files       | Changes only what is needed    |
| Tests                | Breaks things silently        | Failures caught immediately    |
| Small changes        | Produces unrevinable blobs    | Produces focused, clear diffs  |
| Understanding output | You lose control of the code  | You stay in charge             |

---

## Final Thought

AI is an incredibly powerful tool. But a powerful tool in a messy workshop causes more damage than in a clean one.

These fundamentals are how you keep your workshop clean. They are how you stay in control as AI writes more and more of your code.

The developer who will do best in this era is not the one who prompts the most. It is the one who thinks the clearest, reviews the most carefully, and understands what they are shipping.

That is the job now.
