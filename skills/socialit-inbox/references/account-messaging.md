# Messenger and Instagram entry points

These settings shape how a conversation starts on an account. Every tool takes the account as `accountId` (`sa_...`). A set tool replaces the whole setting, so read the current value first with the matching get tool, show the user the complete new value, and set it after a yes. Delete tools remove the setting for everyone.

| Setting | Tools | Account |
| --- | --- | --- |
| Persistent menu | `get-messenger-menu`, `set-messenger-menu`, `delete-messenger-menu` | Facebook or Instagram |
| Get Started button | `get-messenger-get-started`, `set-messenger-get-started`, `delete-messenger-get-started` | Facebook |
| Greeting text | `get-messenger-greeting`, `set-messenger-greeting`, `delete-messenger-greeting` | Facebook |
| Messenger ice breakers | `get-messenger-ice-breakers`, `set-messenger-ice-breakers`, `delete-messenger-ice-breakers` | Facebook |
| Instagram ice breakers | `get-instagram-ice-breakers`, `set-instagram-ice-breakers`, `delete-instagram-ice-breakers` | Instagram |

## Persistent menu

```json
{
  "accountId": "sa_...",
  "persistent_menu": [
    {
      "locale": "default",
      "composer_input_disabled": false,
      "call_to_actions": [
        { "type": "postback", "title": "Track my order", "payload": "TRACK_ORDER" },
        { "type": "web_url", "title": "Shop", "url": "https://example.com/shop" }
      ]
    }
  ]
}
```

Item `type` is `postback`, `web_url`, or `nested` (with its own `call_to_actions`, up to 5). Titles are up to 30 characters. `composer_input_disabled: true` hides the text box, so people can only tap the menu; confirm that is wanted.

## Get Started button

`{ "accountId": "sa_...", "payload": "GET_STARTED" }`. The payload is sent back when a new person taps the button.

## Greeting text

```json
{ "accountId": "sa_...", "greeting": [{ "locale": "default", "text": "Hi! Ask us anything about your order." }] }
```

Up to 160 characters per locale.

## Ice breakers

Messenger, per locale, 1 to 4 questions:

```json
{
  "accountId": "sa_...",
  "ice_breakers": [
    { "locale": "default", "call_to_actions": [{ "question": "Where is my order?", "payload": "WHERE_ORDER" }] }
  ]
}
```

Instagram, 1 to 4 questions, no locale:

```json
{ "accountId": "sa_...", "ice_breakers": [{ "question": "What are your prices?", "payload": "PRICES" }] }
```

Questions are up to 80 characters.

## Handling the payloads

When someone taps a postback, a Get Started button, or an ice breaker, the payload arrives in the conversation. Workflows can branch on it with the `postback.payload` and `quick_reply.payload` run variables; see the `socialit-automations` skill.
