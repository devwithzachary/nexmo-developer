---
title: Send an Image Message
meta_title: Send an image message on Viber using the Messages API
navigation_weight: 2
---

# Send an image Message

In this code snippet you will see how to send a Viber image message using the Messages API.

> **NOTE:** It is important that you read [this information about Viber accounts and messaging](/messages/concepts/viber) before trying this code snippet.

For a step-by-step guide to this topic, you can read our tutorial [Sending Viber Business Messages with the Messages API](/tutorials/sending-viber-service-messages-with-messages-api).

## Example

Find the description for all variables used in each code snippet below:

```snippet_variables
- VONAGE_APPLICATION_PRIVATE_KEY_PATH
- BASE_URL.MESSAGES
- MESSAGES_API_URL
- VONAGE_APPLICATION_ID
- VIBER_SERVICE_MESSAGE_ID
- VONAGE_VIBER_SERVICE_MESSAGE_ID
- VONAGE_NUMBER.MESSAGES
- TO_NUMBER.MESSAGES
- IMAGE_URL
```

> **NOTE:** Don't use a leading `+` or `00` when entering a phone number, start with the country code, for example, 447700900000.

```code_snippets
source: '_examples/messages/viber/send-image'
application:
  type: messages
  name: 'Send a Viber image message'
```

## Try it out

When you run the code a Viber image message is sent to the destination number.
