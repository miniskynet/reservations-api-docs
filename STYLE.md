# Documentation style guide

The rules this spec and guide follow. They keep hundreds of descriptions consistent, and they make review faster because "does it follow the guide?" has a yes-or-no answer.

## Operations

- **Summary:** sentence case, starts with a verb, five words or fewer. *List reservations*, not *Reservations List Endpoint*.
- **Description:** starts with what the operation returns or does, in the third person. *Returns a paginated list of reservations.*
- Then say what a developer can't see from the schema: side effects, default ordering, what happens on conflict, and anything that can't be undone.

## Fields and parameters

- Start with what the value **is**, not its type. The schema already shows the type.
- State the **default** and the **unit** whenever there is one. *Number of results per page.* plus `default: 25` and `maximum: 100` in the schema.
- Say what `null` means for every nullable field. *`null` unless `status` is `cancelled`.*
- Mark fields the API sets as `readOnly` and say *Set by the API.* so nobody tries to send them.
- Explain every enum value in a list or table, not just its name.

## Errors

- Every 4xx response says what triggered it **and** what to do next.
- Error examples use real-looking messages, so readers recognise them in their logs.

## Examples

- Every request and response has an example, and examples are consistent across the spec: `cus_310` is always Jordan Lee, `prd_12` is always the window table.
- Use `example.com` domains and dates in 2030, so nothing looks like real data.
- Examples must validate against their schema. CI checks this.

## Voice

- Second person, present tense, active voice. *The API returns…*, *You can…*
- Plain words over jargon. No *simply*, *just*, *easy* or *obviously*: if it were obvious, it wouldn't need documenting.
- One idea per sentence. Lists for three or more parallel items.
- UK spelling in prose, but field names and values exactly as the API uses them.
