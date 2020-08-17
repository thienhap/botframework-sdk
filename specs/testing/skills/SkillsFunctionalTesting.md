# Skills: Functional Testing Matrix

This document outlines the set of functional tests required to ensure skills and skill consumers function correctly across the breadth of the Bot Framework.

## Scenarios

There are three components necessary to create the full list of testing scenarios.

- **What the skill wants to do.** This is the primary pivot for the testing scenarios, given in the *things a skill might want to do* list below.
- **State of the skill.** The current state of the skill, based on the *variables* list below.
- **State of the consumer.** The current state of the consumer, based on the *variables* list below.

Given this, you end up with scenarios like:

> A skill sends a proactive message when the skill is not currently active and the consumer is actively engaged in a waterfall dialog with a different skill.

### Things a skill might want to do

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

## Glossary

- **Consumer:** A bot that passes Activities to another bot (a skill) for processing.
- **Skill:** A bot that accepts Activities from another bot (a consumer), and passes Activities to users through that consumer.
- **Active skill:** The consumer is currently forwarding Activities to the skill.
- **Inactive skill:** The consumer is not currently forwarding Activities to the skill.
