---
summary: "Adaptive Cards v4.1.0 plugin — rendering, host profiles, accessibility scoring, persistence, and verb routing"
read_when:
  - Working with Adaptive Cards in MS Teams or other hosts
  - Building interactive card UIs
  - Understanding card accessibility or host compatibility
title: "Adaptive Cards"
---

# Adaptive Cards

Last updated: 2026-03-19

The Adaptive Cards plugin (v4.1.0) lets OpenClaw render, validate, and route
interactive cards across messaging channels. Cards follow the
[Adaptive Cards v1.6 specification](https://adaptivecards.io/) and work with
MS Teams, Outlook, Web Chat, and other supported hosts.

If you want channel-specific setup, see
[MS Teams channel](/channels/msteams).

## Element reference

Supported elements in v4.1.0:

| Element            | Category   | Notes                                      |
| ------------------ | ---------- | ------------------------------------------ |
| `TextBlock`        | Content    | Markdown subset, wrapping, color, size     |
| `Image`            | Content    | URL-based, size constraints, select action |
| `Icon`             | Content    | Host-resolved icon set (v4.1.0)            |
| `List`             | Content    | Ordered/unordered item lists (v4.1.0)      |
| `RichTextBlock`    | Content    | Inline text runs with formatting           |
| `Media`            | Content    | Audio/video with poster image              |
| `ActionSet`        | Content    | Inline action buttons                      |
| `FactSet`          | Content    | Key-value label pairs                      |
| `Container`        | Layout     | Vertical grouping with style, bleed        |
| `ColumnSet`        | Layout     | Horizontal columns with width ratios       |
| `Column`           | Layout     | Single column within a ColumnSet           |
| `Table`            | Layout     | Row/column grid with headers               |
| `ImageSet`         | Layout     | Horizontal image gallery                   |
| `Action.OpenUrl`   | Action     | Opens a URL in the host browser            |
| `Action.Submit`    | Action     | Posts data back to the bot (legacy)        |
| `Action.Execute`   | Action     | Universal Action with verb routing         |
| `Action.ShowCard`  | Action     | Reveals an inline sub-card                 |
| `Action.ToggleVisibility` | Action | Shows/hides target elements          |
| `Input.Text`       | Input      | Single/multi-line text input               |
| `Input.Number`     | Input      | Numeric input with min/max                 |
| `Input.Date`       | Input      | Date picker                                |
| `Input.Time`       | Input      | Time picker                                |
| `Input.Toggle`     | Input      | Boolean toggle (checkbox)                  |
| `Input.ChoiceSet`  | Input      | Dropdown or radio/checkbox group           |

## Host compatibility

v4.1.0 ships 7 host profiles that control which features and styles are
available at render time. The plugin selects a profile automatically based on
the target channel, or you can specify one explicitly.

| Profile    | Channel / surface           | AC spec level | Notes                          |
| ---------- | --------------------------- | ------------- | ------------------------------ |
| `Teams`    | MS Teams                    | v1.6          | Full feature set               |
| `Outlook`  | Outlook Actionable Messages | v1.4          | No media element               |
| `WebChat`  | Bot Framework Web Chat      | v1.6          | Full feature set               |
| `Windows`  | Windows Widgets / Shell     | v1.5          | Limited action types           |
| `Viva`     | Viva Connections             | v1.4          | Dashboard card subset          |
| `Webex`    | Cisco Webex                 | v1.3          | No Table, limited inputs       |
| `Generic`  | Fallback / custom hosts     | v1.5          | Safe baseline for unknown apps |

When rendering a card, the plugin validates all elements against the selected
host profile and strips unsupported features with a warning rather than failing
the entire card.

## WCAG accessibility scoring

Every card rendered through the plugin receives an accessibility score from 0
to 100 based on WCAG 2.1 guidelines. The scorer checks:

- **Color contrast** between text and background (AA minimum 4.5:1 ratio)
- **Alt text** presence on all Image elements
- **Input labels** on form fields (Input.Text, Input.ChoiceSet, etc.)
- **Tap target size** for actions (minimum 44x44 dp)
- **Reading order** and logical heading structure
- **Keyboard navigability** of interactive elements

Cards scoring below 50 emit a warning in the gateway logs. Cards below 30 are
blocked by default (configurable via `adaptiveCards.minAccessibilityScore` in
gateway config).

```
# Set minimum accessibility score (0-100, default 30)
openclaw config set adaptiveCards.minAccessibilityScore 50
```

## Persistence and preview

Cards can be stored and retrieved using a `cardId`. When you send a card with a
`cardId`, the gateway persists the card payload so it can be:

- **Retrieved** later via `GET /api/cards/:cardId`
- **Updated** in place (refresh the card in the host without resending)
- **Previewed** via a shareable URL: `https://<gateway-host>/cards/:cardId/preview`

Preview URLs render the card using the Generic host profile and are useful for
debugging or sharing card designs outside a messaging channel.

```
# Send a card with persistence
openclaw message send --channel msteams --card ./invoice.json --card-id inv-2026-001

# Retrieve a stored card
curl https://localhost:18789/api/cards/inv-2026-001
```

## Verb routing

`Action.Execute` is the recommended action type for interactive cards. When a
user taps an execute action, the host sends the action verb and data back to
the gateway. The plugin routes each verb to the corresponding gateway method:

| Verb pattern         | Gateway method         | Description                         |
| -------------------- | ---------------------- | ----------------------------------- |
| `doApprove`          | `card.action.approve`  | Approval workflow actions            |
| `doReject`           | `card.action.reject`   | Rejection workflow actions           |
| `doUpdate`           | `card.action.update`   | Card refresh / data update           |
| `doSubmit`           | `card.action.submit`   | Generic form submission              |
| `custom.*`           | `card.action.custom`   | Custom verbs with wildcard matching  |

Verb routing is configured in the card payload:

```json
{
  "type": "Action.Execute",
  "title": "Approve",
  "verb": "doApprove",
  "data": {
    "requestId": "req-123",
    "approver": "user@example.com"
  }
}
```

The gateway dispatches the verb to registered handlers. Unrecognized verbs
fall through to `card.action.custom` if a handler is registered, or return a
400 error to the host.

## Ecosystem

| Metric             | v4.1.0 |
| ------------------ | ------ |
| Test cases          | 86     |
| Bridge functions    | 25     |
| Exported types      | 20     |
| Host profiles       | 7      |
| AC spec version     | v1.6   |
