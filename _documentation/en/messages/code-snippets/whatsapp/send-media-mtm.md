---
title: Send a Media Message Template
meta_title: Send a WhatsApp Media Message Template using the Messages API
navigation_weight: 2
---

# Send a WhatsApp Media Message Template

In this code snippet you learn how to send a WhatsApp Media Message Template using the Messages API.

To send the Media Message Template you need to use the Messages custom object. The custom object takes a partial section of the original WhatsApp API request and sends it directly through to WhatsApp.

> **IMPORTANT:** If a customer messages you, you have 24 hours to respond to the customer with a free form message. After this period you must use a template message (MTM). If a customer has not messaged you first, then the first time you send a message to a user, WhatsApp requires that the message contains a template. This is explained in more detail in the [Understanding WhatsApp topic](/messages/concepts/whatsapp).

## Message format and length restrictions

WhatsApp Media Message Templates consist of a Header, Body and Footer structure. The Header contains the media, which can be text, location, video, image or file. The Body contains the text message. **This is currently limited to 1024 characters when displayed to the end user, so as to avoid the need for scrolling.** The Footer is optional and contains static text only.

> **NOTE:** The Header and Footer lengths are currently limited to 60 characters each and the Message body length is limited to 1024 characters.

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
source: '_examples/messages/whatsapp/send-media-mtm'
application:
  type: messages
  name: 'Send a WhatsApp media message template'
```

## Try it out

When you run the code a WhatsApp Media Message Template is sent to the destination number.

## Further information

-   [WhatsApp documentation for Media Message Templates](https://developers.facebook.com/docs/whatsapp/api/messages/message-templates/media-message-templates)
