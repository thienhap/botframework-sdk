# Skills: Validating skill functionality (DRAFT) <!-- omit in toc -->

This document outlines the set of functional tests required to ensure skills and skill consumers function correctly across the breadth of the Bot Framework.

## Goals <!-- omit in toc -->

Creating a solid set of test needs to enable the following purposes:

1. Existing functionality can be validated, and any issues identified, fixed and documented.
2. New functionality can be easily tested, without the need to recreate the complex topologies required when working with skills.
3. Test infrastructure can be used either directly or as a template for support scenarios to repro customer issues.
4. Test the most common complex scenarios. Simple scenarios like Echo Bot or protocol tests are not in scope for functional tests.

To support these goals, the testing infrastructure used to validate the functional tests derived from this document must be carefully considered.

This document includes:

- [Scenarios](#scenarios)
  - [1. Single turn interaction with a skill](#1-single-turn-interaction-with-a-skill)
  - [2. Multi turn interaction with a skill](#2-multi-turn-interaction-with-a-skill)
  - [3. The skill needs to authenticate the user with an OAuthCard](#3-the-skill-needs-to-authenticate-the-user-with-an-oauthcard)
  - [4. The consumer authenticates the user and passes OAuth credentials to the skill using SSO](#4-the-consumer-authenticates-the-user-and-passes-oauth-credentials-to-the-skill-using-sso)
  - [5. Skill sends proactive message to consumer](#5-skill-sends-proactive-message-to-consumer)
  - [6. Skill calls another skill](#6-skill-calls-another-skill)
  - [7. A skill provides a teams task module](#7-a-skill-provides-a-teams-task-module)
  - [8. A skill uses team specific APIs](#8-a-skill-uses-team-specific-apis)
  - [Draft scenarios](#draft-scenarios)
  - [Things a skill might want to do](#things-a-skill-might-want-to-do)
- [Variables](#variables)
- [Consumer/Skill architecture](#consumerskill-architecture)
  - [Simple](#simple)
  - [Multiple skills](#multiple-skills)
  - [Multiple consumers](#multiple-consumers)
  - [Skill chaining](#skill-chaining)
  - [Complex](#complex)
  - [Circular](#circular)
- [Implementation notes](#implementation-notes)
- [Glossary](#glossary)

## Scenarios

There are four components necessary to create the full list of testing scenarios.

- **[Consumer/Skill architecture](#consumerskill-architecture).** The topology used to deploy the consumer and the skills.
- **State of the skill.** The current state of the skill, based on the *variables* list below.
- **State of the consumer.** The current state of the consumer, based on the *variables* list below.
- **What the skill wants to do.** This is the primary pivot for the testing scenarios, given in the *things a skill might want to do* list below.

Given this, one can create scenarios like:

### 1. Single turn interaction with a skill

> A consumer bot sends an activity to a skill (e.g.: GetWeather) and the skill responds to the user and ends.

**Variables**
Consumer type: Composer, VA Bot, PVA.
Skill type: Composer, Coded
Topology: Simple
Languages: C#, JS, Python
Auth context: Public Cloud, Gov Cloud, Sandboxed
Delivery mode: Normal, ExpectReplies

### 2. Multi turn interaction with a skill

> A consumer bot starts a multi turn interaction with a skill (e.g.: book a flight) and handles multiple turns (2 or more) until the skill completes the task.

**Alternate flows**
2.1 The consumer cancels the skill (sends EndOfConversation)
2.2 The consumer sends parameters to the skill
2.3 The skill sends a result to the consumer
2.4 The skill sends an event to the consumer (GetLocation) and the consumer sends an event back to the skill.
2.5 The skill throws and exception and fails (the consumer gets a 500 error)

**Variables**
Consumer type: Composer, VA Bot, PVA.
Skill type: Composer, Coded
Topology: Simple
Languages: C#, JS, Python
Auth context: Public Cloud, Gov Cloud, Sandboxed
Delivery mode: Normal, ExpectReplies

### 3. The skill needs to authenticate the user with an OAuthCard

> A consumer bot starts a multi turn interaction with a skill (e.g.: how does my day look like) and the skill renders an OAuthPrompt to allow the user to log in, once the skill obtains a token it performs an operation, returns a response to the user and logs it out.

### 4. The consumer authenticates the user and passes OAuth credentials to the skill using SSO

> A consumer bot starts a multi turn interaction with a skill (e.g.: how does my day look like) and the skill renders an OAuthPrompt to allow the user to log in, once the skill obtains a token it performs an operation, returns a response to the user and logs it out.

### 5. Skill sends proactive message to consumer

> TODO

### 6. Skill calls another skill

> TODO

### 7. A skill provides a teams task module

> TODO

### 8. A skill uses team specific APIs

> Currently not supported but it would involve things like:
>
> - Retrieve list of channels in a team
> - Get team info


### Draft scenarios

This section contains raw ideas to be incorporated in the scenarios enumerated above

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
  - OAuth input

## Variables

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
- Authentication context, the skill and consumer are deployed to the public cloud, gov cloud, or a sandboxed environment.
- Network protocol: the consumer is accessed over straight HTTP (webchat) or Web Sockets (streaming clients)
- BotFramework version for the skill: 4.x or 3.x.
- Bot runtime: Composer bot, PVA or SDK coded bot.
- Channel: Emulator, Teams, DirectLine
- Bot programming language: C#, JS, Python or Java.
- Bot Adapter: OOTB, custom channel

## Consumer/Skill architecture

This section describes the most common consumer/skill topologies that can exist. The topologies given below are further complicated based on the variables above, as well as the SDK language of any particular bot (consumer or skill) in the topology.

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

## Implementation notes

Based on the scenarios described above we will need to build the following artifacts to implement and run functional tests on skills:

- Composer consumer bot (C# only for now)
- VA Scaler consumer bot (C# and JS)
- PVA consumer bot (C#)
- Composer GetWeather skill (C#)
- Coded GetWeather skill (C#, JS, Python)
- Travel skill (C#, JS, Python)
- OAuth skill (C#, JS, Python)
- Teams skill (C#, JS, Python)
- Proactive service (C#)
- Transcript based test runner (C#)

## Glossary

- **Consumer:** A bot that passes Activities to another bot (a skill) for processing.
- **Skill:** A bot that accepts Activities from another bot (a consumer), and passes Activities to users through that consumer.
- **Active skill:** The consumer is currently forwarding Activities to the skill.
- **Inactive skill:** The consumer is not currently forwarding Activities to the skill.
