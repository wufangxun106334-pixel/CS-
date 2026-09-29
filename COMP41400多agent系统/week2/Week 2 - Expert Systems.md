---
course: COMP41400 Multi-Agent Systems
week: 2
topic: Expert Systems
source: Expert Systems.pptx
---

# Week 2 - Expert Systems

> Source: `Expert Systems.pptx` (17 slides).  
> Core idea: an **Expert System** combines facts about a current case with general **rules**, then lets an **Inference Engine** derive new conclusions.

## 1. The central idea

An Expert System is a **rule-based** AI system. It does not learn a conclusion from a large dataset during each query. Instead, a human expert or **knowledge engineer** writes domain knowledge explicitly.

```text
Facts + Rules + Inference Engine = Conclusions
```

Example weather facts:

```text
LOW_PRESSURE
CLOUDY
COLD
```

Example rules:

```text
IF LOW_PRESSURE AND CLOUDY THEN RAIN_LIKELY
IF HIGH_PRESSURE AND NOT CLOUDY THEN RAIN_UNLIKELY
```

Because `LOW_PRESSURE` and `CLOUDY` are facts, the system derives:

```text
RAIN_LIKELY
```

`COLD` stays in the database but does not affect this conclusion because no rule uses it. This illustrates **relevance**: a fact only contributes when it matches a rule condition.

## 2. Anatomy of an Expert System (slides 2-6)

```text
Database / Working Memory  <-->  Inference Engine  <-->  Rule Base
                                      ^
                               Execution Loop
```

### Database / Working Memory

The **Database** or **Working Memory** contains the current state of one problem instance, usually as **facts**.

Medical example:

```text
FEVER(patient_1)
COUGH(patient_1)
MUSCLE_ACHE(patient_1)
```

### Rule Base

The **Rule Base** contains reusable domain rules. A rule typically has an **antecedent** (IF condition) and a **consequent** (THEN result).

```text
IF FEVER(x) AND COUGH(x) AND MUSCLE_ACHE(x)
THEN FLU_RISK(x)
```

Rules express general knowledge. Facts express what is currently known about a specific case.

### Inference Engine

The **Inference Engine** is the procedural part. It matches facts against rules and adds derived facts to Working Memory.

### Execution Loop

The system repeatedly selects a rule and tries to apply it:

1. Check whether the IF conditions are true.
2. Add the THEN conclusion if it is new.
3. Ignore rules that cannot add a new fact.
4. Stop when no rule can add new state.

This makes the system a simple **closed reasoning loop**.

## 3. Forward Chaining (slides 5-8)

**Forward Chaining** starts from known facts and derives every reachable conclusion.

```text
known facts -> matching rule -> new fact -> more matching rules
```

Weather example:

```text
Initial Working Memory:
{ LOW_PRESSURE, CLOUDY, COLD }

Matching rule:
LOW_PRESSURE AND CLOUDY -> RAIN_LIKELY

Updated Working Memory:
{ LOW_PRESSURE, CLOUDY, COLD, RAIN_LIKELY }
```

### Case: vehicle diagnosis

```text
Facts:
ENGINE_WONT_START
BATTERY_VOLTAGE_LOW

Rules:
IF ENGINE_WONT_START AND BATTERY_VOLTAGE_LOW
THEN BATTERY_PROBLEM

IF BATTERY_PROBLEM
THEN RECOMMEND_CHARGE_OR_REPLACE
```

Forward Chaining derives `BATTERY_PROBLEM`, then `RECOMMEND_CHARGE_OR_REPLACE`.

### Strengths and limits

- Useful for **query-intensive** systems: generate conclusions once, answer many queries quickly.
- Useful for monitoring: new sensor facts can trigger new alerts.
- Can create excessive **overhead** when a large Rule Base generates many conclusions that no user asks for.

## 4. Backward Chaining (slides 9-12)

**Backward Chaining** begins with a query or goal and works backwards through rules.

```text
goal -> rule that could prove the goal -> required subgoals -> facts
```

Weather proof:

```text
Query: Is RAIN_LIKELY true?

Rule found:
LOW_PRESSURE AND CLOUDY -> RAIN_LIKELY

Subgoals:
Is LOW_PRESSURE true?
Is CLOUDY true?

Both facts are in the Database.
Therefore RAIN_LIKELY is true.
```

### Case: medical triage

```text
Query:
FLU_RISK(patient_1)?

Rule:
FEVER(x) AND COUGH(x) AND MUSCLE_ACHE(x) -> FLU_RISK(x)

Known facts:
FEVER(patient_1)
COUGH(patient_1)
MUSCLE_ACHE(patient_1)
```

The system proves only facts relevant to `FLU_RISK(patient_1)`.

### Strengths and limits

- **On-demand inference** avoids unrelated computation.
- Well suited to diagnosis and interactive question answering.
- Can repeat reasoning when multiple queries require the same intermediate conclusion.

## 5. Forward vs Backward Chaining (slide 13)

| Question | Forward Chaining | Backward Chaining |
|---|---|---|
| Starting point | Facts | Query / goal |
| Direction | Facts to conclusions | Goal to supporting facts |
| Main advantage | Reuse derived conclusions | Compute only what the query needs |
| Main cost | Wasted inferences | Repeated intermediate reasoning |
| Typical use | Monitoring, many queries | Diagnosis, one focused query |

Memory aid:

> Forward Chaining asks: **What else follows from what I know?**  
> Backward Chaining asks: **What must be true for this goal to hold?**

## 6. Knowledge Representation and explanation

Expert Systems are a practical use of **Knowledge Representation**:

- Facts represent the present environment.
- Rules represent general expert knowledge.
- Inference turns the two into a decision.

They are often **explainable** because the system can expose its proof:

```text
CREDIT_RISK_HIGH because:
NO_INCOME was true
HIGH_DEBT was true
Rule R17 fired
```

This is valuable in domains such as credit decisions, compliance, troubleshooting, and clinical decision support.

## 7. Limits of Expert Systems (slide 13)

### Tacit knowledge

**Tacit knowledge** is expertise that people use but cannot easily state as explicit rules: intuition, visual judgement, or years of practical experience.

### Rule explosion

A simple rule quickly accumulates exceptions.

```text
IF FEVER THEN INFECTION
```

Real use may require age, medication, test reliability, history, and many exceptions. Maintaining all rules becomes a **knowledge engineering** problem.

### Human reasoning is not purely logical

Humans often form an intuition first and justify it afterwards. A rule engine can only use the knowledge someone has made explicit.

## 8. SHRDLU (slides 14-16)

**SHRDLU** was developed by Terry Winograd in 1968-70. It understood a restricted subset of English and manipulated blocks in a small **blocks world**.

It combined:

- natural-language parsing;
- a world model;
- logical reasoning;
- action execution;
- reference resolution.

Example:

```text
Person: Pick up a big red block.
Person: Find a block taller than the one you are holding and put it into the box.
```

The word `it` requires **reference resolution**: the system must infer that `it` means the newly found taller block, not the block already held.

SHRDLU is more precisely a symbolic AI and natural-language-understanding prototype than a classic diagnosis-style Expert System. It demonstrates how rules, world state, language, and action can work together.

Historical links from the slides:

- **Lisp** (John McCarthy, 1958) supplied procedural support.
- **Micro Planner** (Carl Hewitt, 1969), a precursor of Prolog, supplied logical reasoning.
- **Resolution** (Robinson, 1965) supported theorem-proving-style reasoning.

## 9. Shakey: from Expert System to embodied Agent (slide 17)

**Shakey** (SRI, 1966-72) was an early fully embodied AI robot. It connected symbolic reasoning to action in a physical environment.

```text
Perception -> state representation -> STRIPS planning -> A* navigation
          -> action execution -> monitoring
```

Important contributions noted by the slides:

- early **layered architecture**;
- line / edge detection;
- **STRIPS** (Stanford Research Institute Problem Solver);
- **A*** Search, a heuristic shortest-path algorithm.

This connects directly to Multi-Agent Systems: an Agent needs knowledge about its environment, a way to reason over it, a way to select actions, and monitoring that links actions back to the world.

## 10. Expert Systems and LLMs

| Aspect | Rule-based Expert System | LLM |
|---|---|---|
| Knowledge source | Human-authored rules | Learned statistical patterns |
| Reasoning trace | Usually explicit | Often difficult to inspect |
| New situations | Limited by written rules | Can generalize language patterns |
| Main risk | Missing / conflicting rules | Hallucination |

A practical **hybrid system** can let an LLM understand the user's natural language, then use a Rule Engine for verifiable decisions.

## Exam checklist

1. Define the roles of Database, Rule Base, Inference Engine, and Execution Loop.
2. Explain the difference between a fact and a rule.
3. Manually run Forward Chaining over a small rule set.
4. Prove a query with Backward Chaining and subgoals.
5. Compare computation trade-offs between the two chaining directions.
6. Explain why tacit knowledge and rule explosion limit Expert Systems.
7. Relate SHRDLU and Shakey to symbolic AI and intelligent Agent architectures.
