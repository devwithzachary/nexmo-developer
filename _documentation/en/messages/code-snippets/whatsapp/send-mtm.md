---
title: Send a Message Template (MTM)
meta_title: Send a WhatsApp Message Template (MTM) using the Messages API
navigation_weight: 1
---

# Send a WhatsApp Message Template (MTM)

In this code snippet you learn how to send a WhatsApp Message Template Message (MTM) using the Messages API.

> **IMPORTANT:** If a customer messages you, you have 24 hours to respond to the customer with a free-form message. After this period you must use a message template (MTM). If a customer has not messaged you first, then the first time you send a message to a user, WhatsApp requires that the message contains a template. This is explained in more detail in the [Understanding WhatsApp topic](/messages/concepts/whatsapp).

## Example

Find the description for all variables used in each code snippet below:

```snippet_variables
- VONAGE_APPLICATION_ID
- VONAGE_APPLICATION_PRIVATE_KEY_PATH
- BASE_URL.MESSAGES
- MESSAGES_API_URL
- WHATSAPP_NUMBER
- VONAGE_WHATSAPP_NUMBER
- VONAGE_NUMBER.WHATSAPP
- TO_NUMBER.MESSAGES
- WHATSAPP_TEMPLATE_NAMESPACE
- WHATSAPP_TEMPLATE_NAME
```

> **NOTE:** Don't use a leading `+` or `00` when entering a phone number, start with the country code, for example, 447700900000.

```code_snippets
source: '_examples/messages/whatsapp/send-mtm'
application:
  type: messages
  name: 'Send a WhatsApp template'
```

## Try it out

When you run the code a WhatsApp message template (MTM) is sent to the destination number.

## Further information

* [WhatsApp documentation for Message Templates](https://developers.facebook.com/docs/whatsapp/api/messages/message-templates)
