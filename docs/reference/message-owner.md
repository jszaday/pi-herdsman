# `message_owner` API

[Documentation index](../README.md)

`message_owner` is mandatory managed agent infrastructure for sending the
direct owner a non-blocking message while the agent keeps working.

Use it for timely information, warnings, emergencies, or status the owner
should see now. Do not use it for routine progress polling. Use
[`ask_owner`](ask-owner.md) instead when the agent cannot proceed without an
owner decision and wants to stop and wait for the answer.

## Availability

Every valid managed agent receives the `message_owner` tool at launch, with
the same infrastructure status as `ask_owner`: `tools`, `excludeTools`, and
`noTools` cannot remove it. Ordinary Leads and Managers do not receive it; they
use `message_supervisor`.

## Request

```json
{
  "message": "The migration is destructive; I paused before running it.",
  "files": ["plan.md", "result:researcher#1"]
}
```

`message` must contain non-whitespace text. `files` follows the same rules,
UTF-8 text embedding, and rejection behavior as [`ask_owner`](ask-owner.md#request).
Message text is delivered complete.

## Semantics

`message_owner` is fire-and-forget on the ownership edge only.

- It may be called alongside other tool calls and at any time, including while
  the agent's own direct agents are running.
- It does not block the assignment, does not set a pending ask, and creates no
  reply obligation. The tool result is an ordinary result and the agent keeps
  working.
- It requires only a valid active assignment (not settling) and a matching
  agent identity. Otherwise it is rejected with the cause and the action to
  take.
- At most eight messages may await delivery per agent. A further call is
  rejected until the owner session has received earlier ones; fold related
  content into one message.

## Owner delivery

The agent writes one durable record per message in its mailbox. The exact
direct owner session re-checks the agent identity and receives the message as a
display message of type `pi-herdsman-agent-owner-message`, shown as
`message (no reply pending)`. Unlike an ask, it does not wait for the owner to
be idle: a working owner receives it at its next steering boundary and an idle
owner is woken. The record is removed once delivery is confirmed, and a message
already present in the owner session is not delivered twice. Undelivered
messages are flushed before the agent's final result message.

The owner may respond with `steer_agent`. There is no reply tool for these
messages; `reply_agent` answers only a correlated pending `ask_owner` question.

Messages are not forwarded through an ownership chain. Undelivered messages are
discarded when the agent mailbox is reset or removed.

## See also

- [`ask_owner`](ask-owner.md)
- [Delegation](../concepts/delegation.md)
