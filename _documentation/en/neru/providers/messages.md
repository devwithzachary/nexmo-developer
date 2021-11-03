---
title: Messages
description: The Messages Provider
navigation_weight: 3
---

# Messages Provider

The Messages provider gives you an interface for the [Messages API](/messages/overview).

## Functions

* [`sendText`](/neru/code-snippets/messages-provider/send-message)
* [`sendImage`](/neru/code-snippets/messages-provider/send-image)
* [`listenMessages`](/neru/code-snippets/messages-provider/listen-messages)
* [`listenEvents`](/neru/code-snippets/messages-provider/listen-events)
* [`unsubscribeEvents`](/neru/code-snippets/messages-provider/unsubscribe-events)

## Initializing the Messages Provider

To use the Messages Provider you need to create an instance of the provider using a session:

```javascript
const router = neru.Router();
const session = neru.createSession();
const messaging = new Messages(session);
```

## Usage

For example, to use the use the Messages provider to send a text message:

```javascript
const session = neru.createSession();
const messaging = new Messages(session);

const to = { type: "sms", number: "447700900000" };
const from = { type: "sms", number: "447700900001" }; 

await messaging.sendText(from, to, "hello world").execute();
```