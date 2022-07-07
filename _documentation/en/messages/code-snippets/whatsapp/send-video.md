---
title: Send a Video Message
meta_title: Send a Video message with WhatsApp
---

# Send a Video Message

In this code snippet you learn how to send a video message through WhatsApp using the Messages API. For WhatsApp the maximum outbound media size is 64MB.

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
- VIDEO_URL.WHATSAPP.MESSAGES
- VIDEO_CAPTION
```

```code_snippets
source: '_examples/messages/whatsapp/send-video'
application:
  type: messages
  name: 'Send a video message'
```

## Try it out

When you run the code a video message is sent to the WhatsApp recipient.
