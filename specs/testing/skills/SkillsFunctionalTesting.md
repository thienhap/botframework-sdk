# Skills: Functional Testing Matrix

This document outlines the set of functional tests required to ensure skills and skill consumers function correctly across the breadth of the Bot Framework.

## Scenarios

There are three components necessary to create the full list of testing scenarios.

- **State of the skill.** The current state of the skill, based on the *variables* list below.
- **State of the consumer.** The current state of the consumer, based on the *variables* list below.
- **What the skill wants to do.** This is the primary pivot for the testing scenarios, given in the *things a skill might want to do* list below.

Given this, one can create scenarios like:

> The consumer is actively engaged with a skill in an adaptive dialog, the skill uses adaptive dialogs, and the skill wants to send a proactive message.

> The consumer is actively engaged with a _different_ skill in an adaptive dialog, the skill uses adaptive dialogs, and the skill wants to send a proactive message.

> The consumer is not engaged with a skill, the skill uses adaptive dialogs, and the skill wants to send a proactive message.

> The consumer is not engaged with a skill, the skill uses adaptive dialogs, and the skill wants to call `createConversation` in order to send a new proactive message.

Using those examples, we can extrapolate a template for creating a realistic test scenario:

> The consumer is in `someVariableState`, the skill is in `someVariableState`, and the skill wants to `performSomeAction`.

### Things a skill might want to do

- Perform multi-turn dialogs, with child dialog/prompts
- Send proactive messages
- Receive and respond to invoke Activities
- Send cards and respond to card actions
  - Adaptive Card
    - Action.Submit
    - Action.Execute (AC 2.0)
  - Suggested Actions
  - Non-invoke actions (ImBack)
- Retrieve conversation members
  - Single member
  - All members
  - Paged members
- Update messages
- Delete messages
- Create a new conversation
- Use channel-specific functionality
  - Retrieve list of channels in a team
  - Get team info
- Authentication
  - SSO
  - OAuth prompt
  - OAuth card

### Variables

- Activity Handling (applies to both the skill and the consumer)
  - Waterfall
  - Adaptive
  - Prompts
  - Raw activity handling
- Consumer sent the Activity to the skill with "expectReplies"
- Skill is currently active
- Skill is currently inactive
- Some _other_ skill is currently active
- Parent bot is engaged in a _different_ dialog
- Skill or consumer uses a custom adapter

## Consumer/Skill architecture

This section attempts to describe the most common consumer/skill topologies that can exist. The topologies given below are further complicated based on the variables above, as well as the SDK language of any particular bot (consumer or skill) in the topology.

One of the most important things to keep in mind here is that any bot can act as a stand-alone bot, a consumer, or a skill, and may very well fulfill all three models at different times.

### Simple

In the simplest case there is a single consumer and a single skill.

```
C -----> S
```

### Multiple skills

A single consumer with multiple skills.

```
      ----> S1
C --<
      ----> S2
```

### Multiple consumers

A single skill is consumed by multiple consumers.

```
C1 --\
      ------> S
C2 --/
```

### Skill chaining

A consumer uses a skill, which in turn consumes another skill.

```
C1 -----> C2/S1 ----> S2
```

### Complex

Combining multiple skills, multiple consumers, and skill chaining.

```
C1 --\                              ----> S3
      ------> C3/S1 ----> C4/S2 --<
C2 --<              \               ----> S4
      ------> S5     -----> S6
```

### Circular

A consumer uses a skill, which in turn consumes another skill, which in turn consumes the original consumer as a skill. In practice, this topology should probably be avoided, however nothing directly prevents it from occurring.

```
C1/S1 ----> C2/S2 --
   ^                \----> C3/S3 --
   |                               |
   |------------------------------/

```

## Glossary

- **Consumer:** A bot that passes Activities to another bot (a skill) for processing.
- **Skill:** A bot that accepts Activities from another bot (a consumer), and passes Activities to users through that consumer.
- **Active skill:** The consumer is currently forwarding Activities to the skill.
- **Inactive skill:** The consumer is not currently forwarding Activities to the skill.
