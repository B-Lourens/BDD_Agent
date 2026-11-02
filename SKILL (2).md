---
name: bdd-authoring
description: Use when creating, reviewing, correcting, improving, or extending Behaviour-Driven Development scenarios, Gherkin scenarios, acceptance criteria, Scenario Outlines, business-rule examples, negative scenarios, or boundary scenarios from requirements or user stories.
user-invocable: true
disable-model-invocation: false
---

# BDD Authoring

Apply these standards whenever creating or reviewing BDD scenarios.

## Objective

Create concise, business-readable BDD scenarios that describe observable behaviour rather than implementation details.

Before writing scenarios:

1. Identify the business behaviour.
2. Identify each distinct business rule.
3. Determine the relevant initial context.
4. Determine the triggering event or action.
5. Determine the observable business result.
6. Identify meaningful alternative, rejection, missing-data, or boundary cases.
7. Do not invent business rules that were not supplied.

If information is missing and it materially changes the expected behaviour, ask the user for clarification.

Minor assumptions may be made only when they do not alter the underlying business rule. Clearly state such assumptions.

---

# Gherkin Structure

Use:

- `Feature` for the capability being described.
- `Rule` when several scenarios illustrate the same explicit business rule.
- `Scenario` for one concrete behavioural example.
- `Scenario Outline` only when several examples exercise exactly the same rule.
- `Examples` to provide values for a Scenario Outline.
- `Given` for relevant initial context.
- `When` for the event or action.
- `Then` for observable business results.
- `And` and `But` only to extend the meaning of the preceding keyword.

Example:

```gherkin
Feature: Process selected customers

  Rule: Only type A customers are processed

    Scenario: Selected type A customer is processed
      Given a selected customer exists
      And the customer has type "A"
      When the selected-customer task runs
      Then the customer is marked as processed
```

---

# Core Rules

## Describe behaviour, not implementation

BDD scenarios should describe business-visible behaviour.

Prefer:

```gherkin
Given an active customer exists
```

Avoid:

```gherkin
Given CUSTOMER_TABLE contains STATUS = 'ACTIVE'
```

Do not reference implementation details unless the user explicitly requests technical or integration-level scenarios.

Implementation details include:

- database tables;
- database columns;
- Java classes;
- methods;
- internal service names;
- message queues;
- internal APIs;
- UI selectors;
- CSS selectors;
- technical status flags.

---

## Use one main When per scenario

Each scenario should normally contain one main business event or action.

Prefer:

```gherkin
When the selected-customer task runs
```

Avoid:

```gherkin
When the customer is selected
And the task runs
And the user confirms processing
```

Do not use `And` after `When` to hide additional independent actions.

If several independent business actions occur, consider separating them into different scenarios.

---

## Keep scenarios focused

Each scenario should demonstrate one meaningful behaviour or business rule.

Do not combine unrelated rules simply to reduce the number of scenarios.

Prefer several clear scenarios over one large scenario containing multiple behaviours.

---

## Keep scenarios independent

A scenario must not depend on another scenario running first.

Each scenario should establish all context required to understand its behaviour.

Avoid scenarios such as:

```gherkin
Scenario: Process the customer
  Given the customer from the previous scenario exists
```

---

## Describe observable results

`Then` should describe something that can be observed from the business or externally visible system behaviour.

Prefer:

```gherkin
Then the customer is marked as processed
```

Avoid:

```gherkin
Then the processCustomer method returns true
```

unless the user explicitly requests component-level or technical BDD.

---

## Use concrete examples

Examples should make business rules easier to understand.

Prefer:

```gherkin
Given the customer has type "A"
```

over:

```gherkin
Given the customer has an eligible type
```

when `"A"` is the actual business example supplied by the requirement.

Do not invent example values when those values could change the meaning of the business rule.

---

## Avoid OR conditions representing different behaviours

Do not write:

```gherkin
Given the customer has type "A" or "B"
```

when type A and type B result in different behaviour.

Create separate scenarios instead.

For example:

```gherkin
Scenario: Type A customer is processed
  Given a selected customer exists
  And the customer has type "A"
  When customer processing runs
  Then the customer is processed

Scenario: Type B customer is skipped
  Given a selected customer exists
  And the customer has type "B"
  When customer processing runs
  Then the customer is skipped
```

---

## Avoid conjunctions that combine behaviours

Avoid steps containing `and` or `or` when those words combine distinct business behaviours.

For example, avoid:

```gherkin
Then the customer is blocked and a notification is sent
```

when blocking and sending a notification represent separately important business outcomes.

Split these where doing so improves clarity or separates independently testable behaviour.

It is acceptable to use `And` when it merely extends the same context or outcome.

---

## Do not use And to hide another action

`And` after `Given` may extend context.

Example:

```gherkin
Given a customer has an active loan
And the customer has missed a payment
```

`And` after `Then` may describe closely related observable consequences.

Do not use it after `When` to conceal another independent action.

---

# Scenario Naming

Scenario names should express the behaviour and preferably its expected outcome.

Preferred:

```gherkin
Scenario: Type B customer is skipped
```

Avoid:

```gherkin
Scenario: Test customer processing
```

Also avoid generic names such as:

```gherkin
Scenario: Test 1
Scenario: Test 2
Scenario: Customer test
```

A reader should understand the purpose of the scenario from its name.

---

# Feature

Use `Feature` to describe the business capability or behaviour being specified.

Example:

```gherkin
Feature: Process selected customers
```

Avoid feature names that describe technical components.

Prefer:

```gherkin
Feature: Send overdue-payment reminders
```

over:

```gherkin
Feature: ReminderService
```

unless the requested scope is explicitly technical.

---

# Rule

Use `Rule` when multiple scenarios demonstrate the same explicit business rule.

Example:

```gherkin
Feature: Process selected customers

  Rule: Only type A customers can be processed

    Scenario: Selected type A customer is processed
      Given a selected customer exists
      And the customer has type "A"
      When the selected-customer task runs
      Then the customer is marked as processed

    Scenario: Selected type B customer is skipped
      Given a selected customer exists
      And the customer has type "B"
      When the selected-customer task runs
      Then the customer is not marked as processed
```

Do not add `Rule` merely for formatting.

Do not create a separate Rule for every individual scenario unless each represents a genuinely separate business rule.

---

# Background

Use `Background` only when the same meaningful context applies to most or all scenarios in a Feature or Rule.

Example:

```gherkin
Feature: Manage an active loan

  Background:
    Given a customer has an active loan
```

Do not use `Background` simply to remove repeated text.

Do not place important scenario-specific conditions in Background.

Prefer scenarios that remain understandable when read independently.

Avoid very long Background sections.

---

# Scenario Outline

Use `Scenario Outline` when multiple concrete examples demonstrate exactly the same business behaviour and follow the same flow.

Example:

```gherkin
Scenario Outline: Reminder is sent at an overdue milestone
  Given a payment is <days_overdue> days overdue
  When the reminder process runs
  Then an overdue-payment reminder is sent

  Examples:
    | days_overdue |
    | 5            |
    | 10           |
    | 20           |
```

Use Scenario Outline when:

- the Given/When/Then structure remains identical;
- only example values differ;
- all values demonstrate the same business rule;
- the expected type of behaviour is identical.

Do not use Scenario Outline merely to reduce the number of scenarios.

If different examples produce materially different business behaviour, use separate scenarios.

For example, if:

- 5 days sends Reminder 1;
- 10 days sends Reminder 2;
- 20 days sends a Final Notice;

these may represent different business behaviours and should normally be expressed separately unless the actual business rule treats them uniformly.

---

# Examples Tables

Keep `Examples` tables concise.

Only include columns that affect the scenario behaviour.

Prefer:

```gherkin
Examples:
  | days_overdue |
  | 5            |
  | 10           |
  | 20           |
```

Avoid adding irrelevant data simply because it is available.

Do not put many unrelated combinations into one Examples table.

---

# Given

`Given` establishes the relevant business context before the action occurs.

Examples:

```gherkin
Given a selected customer exists
Given the customer has type "A"
Given a payment is 5 days overdue
```

Avoid describing actions in `Given`.

Avoid excessive setup that is irrelevant to the business rule.

---

# When

`When` describes the event or action that triggers the behaviour.

Examples:

```gherkin
When the selected-customer task runs
When the customer submits the payment
When the contract is cancelled
```

Use one main `When` per scenario.

Do not use implementation-specific actions unless technical BDD is explicitly requested.

---

# Then

`Then` describes the observable expected business result.

Examples:

```gherkin
Then the customer is marked as processed
Then an overdue-payment reminder is sent
Then the contract remains active
```

Expected results must be clear and testable.

Avoid vague results such as:

```gherkin
Then the customer is handled correctly
```

or:

```gherkin
Then processing succeeds
```

unless the meaning of success is explicitly defined.

---

# Coverage Analysis

Before creating scenarios, identify the business rules represented in the requirement.

Consider whether meaningful scenarios exist for:

- successful behaviour;
- alternative valid behaviour;
- rejection behaviour;
- invalid eligibility;
- missing required information;
- boundary conditions;
- duplicate or repeated actions;
- permissions;
- state transitions;
- already-completed actions;
- conflicting conditions.

Do not automatically create one scenario for every category.

Only create scenarios that represent meaningful business behaviour supported by the requirement.

Do not invent business rules for the sake of coverage.

---

# Happy Path

Include the successful or expected behaviour when one exists.

Example:

```gherkin
Scenario: Eligible customer is processed
  Given a selected customer exists
  And the customer is eligible for processing
  When customer processing runs
  Then the customer is marked as processed
```

---

# Alternative Behaviour

Create separate scenarios when valid inputs produce a different legitimate outcome.

Example:

```gherkin
Scenario: Type B customer is skipped
```

This is not necessarily an error scenario if skipping type B customers is expected business behaviour.

---

# Rejection Behaviour

Create rejection scenarios when the requirement defines circumstances in which an action is not permitted.

Example:

```gherkin
Scenario: Ineligible customer is not processed
```

Describe the observable rejection result.

Do not assume rejection rules that were not supplied.

---

# Missing Information

Consider scenarios involving missing data only when the requirement indicates that absence is meaningful.

For example:

```gherkin
Scenario: Selected customer does not exist
```

Do not invent handling such as error messages, retries, or notifications unless specified.

---

# Boundary Conditions

Pay special attention when requirements contain:

- dates;
- ages;
- monetary amounts;
- limits;
- thresholds;
- ranges;
- counts;
- percentages;
- time periods;
- maximums;
- minimums.

For a rule such as:

> A reminder is sent after 5 days overdue.

Potentially relevant examples may include:

- before 5 days;
- exactly 5 days;
- after 5 days.

However, generate these scenarios only when the actual business rule makes those boundaries meaningful.

Never invent threshold values.

---

# Duplicate and Repeated Actions

Consider repeated actions when repeating an action could change the expected result.

Examples include:

- sending a reminder twice;
- submitting the same payment twice;
- cancelling an already-cancelled contract;
- processing an already-processed customer.

Only create such scenarios if this behaviour is relevant to the supplied requirement.

---

# Business-Level vs Technical BDD

Default to business-level BDD.

Business-level example:

```gherkin
Scenario: Eligible customer is processed
  Given an eligible customer has been selected
  When customer processing runs
  Then the customer is marked as processed
```

Technical/integration example:

```gherkin
Scenario: Customer processing request succeeds
  Given an eligible customer exists
  When a processing request is submitted to the customer API
  Then the API responds with status 200
```

Use technical wording only when:

- the user explicitly requests API-level scenarios;
- integration behaviour is itself the subject of the requirement;
- the technical contract is the observable behaviour being specified.

Do not automatically convert all business scenarios into technical tests.

---

# Clarification Rules

Ask for clarification when missing information materially affects the expected business behaviour.

For example, if the user says:

> Create scenarios for cancelling a contract.

Important missing information may include:

- which contracts may be cancelled;
- who can cancel them;
- what happens after cancellation;
- whether effective dates matter.

Do not invent these rules.

When possible, ask concise questions focused on the missing business decision.

Do not ask for unnecessary detail that does not affect the scenarios.

---

# Assumptions

Avoid assumptions where possible.

When a minor assumption is required:

1. state it clearly;
2. ensure it does not introduce a new material business rule;
3. allow the user to correct it easily.

Never silently invent:

- thresholds;
- eligibility conditions;
- statuses;
- time periods;
- error handling;
- notifications;
- permissions.

---

# Reviewing Existing BDD Scenarios

When the user provides existing scenarios, perform the following review.

## 1. Understand intent

Determine the business behaviour the existing scenario is attempting to describe.

Preserve that intent when improving the scenario.

## 2. Check scenario naming

Determine whether the scenario name describes meaningful behaviour and outcome.

## 3. Check Given

Ensure Given describes relevant context rather than implementation or actions.

## 4. Check When

Ensure there is one main action or event.

Identify additional hidden actions.

## 5. Check Then

Ensure expected results are observable, specific and testable.

## 6. Check implementation details

Flag unnecessary:

- tables;
- columns;
- method names;
- classes;
- internal APIs;
- internal services;
- technical flags.

## 7. Check combined behaviours

Identify steps or scenarios containing several distinct business behaviours.

## 8. Check Scenario Outline use

Verify that all Examples genuinely demonstrate the same rule.

## 9. Check independence

Verify that scenarios do not rely on execution order or previous scenarios.

## 10. Check duplication

Identify scenarios that test the same behaviour without adding useful coverage.

## 11. Check missing behaviour

Identify obvious missing scenarios only when they are supported by the supplied requirements.

Do not invent missing rules.

## 12. Provide an improved version

Return corrected Gherkin while preserving the original business intent.

Explain significant corrections when useful.

---

# Creating New BDD Scenarios

Use the following workflow.

## Step 1 — Understand the requirement

Extract:

- actor or subject;
- initial context;
- triggering event;
- expected outcome;
- business rules;
- exceptions;
- thresholds;
- states;
- relevant data examples.

## Step 2 — Identify distinct business rules

Separate each rule before writing scenarios.

For example:

Requirement:

> Type A customers are processed. Type B customers are skipped.

Identify:

- Rule/example 1: Type A → processed.
- Rule/example 2: Type B → skipped.

Do not merge these into one ambiguous scenario.

## Step 3 — Identify examples

Choose concrete examples that demonstrate each rule.

Prefer examples supplied by the user.

## Step 4 — Analyse coverage

Consider:

- successful behaviour;
- alternative behaviour;
- rejection;
- missing information;
- boundaries;
- repeated actions.

Only retain cases that add meaningful coverage.

## Step 5 — Write scenarios

Use:

```text
Given → context
When  → event
Then  → observable result
```

## Step 6 — Review the scenarios

Verify:

- each scenario represents one behaviour;
- scenario names express intent;
- there is one main When;
- scenarios are independent;
- steps describe business behaviour;
- Then statements are observable;
- no supplied business rule is omitted;
- no new business rule has been invented;
- scenarios are not redundant;
- Scenario Outlines represent a single rule.

---

# Default Output Format

Unless the user asks for another format, structure the response as follows.

## Behaviour understood

Briefly summarise the business behaviour and identified rules.

Keep this concise.

## Questions or assumptions

Only include this section when clarification or assumptions are genuinely necessary.

## BDD scenarios

Return scenarios in a fenced Gherkin block.

Example:

```gherkin
Feature: Process selected customers

  Rule: Only type A customers are processed

    Scenario: Selected type A customer is processed
      Given a selected customer exists
      And the customer has type "A"
      When the selected-customer task runs
      Then the customer is marked as processed

    Scenario: Selected type B customer is skipped
      Given a selected customer exists
      And the customer has type "B"
      When the selected-customer task runs
      Then the customer is not marked as processed
```

## Coverage

Briefly explain which supplied business rules or important paths are covered.

Do not produce a lengthy testing essay unless the user requests one.

---

# Language

Respond in the same language as the user.

Use standard English Gherkin keywords:

- Feature
- Rule
- Background
- Scenario
- Scenario Outline
- Examples
- Given
- When
- Then
- And
- But

unless the user explicitly requests localised Gherkin keywords.

---

# Quality Checklist

Before returning scenarios, verify:

- [ ] Business behaviour is clear.
- [ ] No material business rules were invented.
- [ ] Each scenario covers one meaningful behaviour.
- [ ] Scenario names describe behaviour.
- [ ] Context is expressed with Given.
- [ ] There is one main When.
- [ ] Results are expressed with Then.
- [ ] Results are observable and testable.
- [ ] Implementation details are avoided unless requested.
- [ ] Scenarios are independent.
- [ ] OR conditions do not hide separate behaviours.
- [ ] And does not conceal additional independent actions.
- [ ] Scenario Outline is used only for one consistent rule.
- [ ] Boundary scenarios use actual supplied boundaries.
- [ ] Duplicate scenarios are removed.
- [ ] Coverage is useful rather than exhaustive for its own sake.