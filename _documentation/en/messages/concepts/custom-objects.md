---
title: Custom objects
navigation_weight: 8
description: Understanding custom objects
---

# Custom objects

The Messages API has the concept of a custom object. The custom object allows you to leverage advanced features of the supported messaging platforms, such as Facebook Messenger, WhatsApp and Viber.

The custom object takes a partial section of the underlying messaging platform's API request format, and sends it directly to that platform.

Using WhatsApp by way of example, for the following Messages API call:

``` json
{
  "to": "447700900000",
  "from": "447700900001",
  "channel": "whatsapp",
  "message_type": "custom",
  "custom": {$CUSTOM_OBJECT}
}
```

Vonage would send the following to WhatsApp:

``` json
{
  "recipient_type": "individual",
  "to": "$TO_NUMBER",
  {$CUSTOM_OBJECT}
}
```

The JSON in `CUSTOM_OBJECT` depends on the messaging platform's format for the type of request you want to make.

If the WhatsApp request was:

``` json
{
  "recipient_type": "individual",
  "to": "447700900000",
  "type": "template",
  "template": {
    "namespace": "whatsapp:hsm:technology:nexmo",
    "name": "parcel_location",
    "language": {
      "policy": "deterministic",
      "code": "en"
    },
    "components": [
      {
        "type": "location",
        "location": {
          "longitude": -122.425332,
          "latitude": 37.758056,
          "name": "Facebook HQ",
          "address": "1 Hacker Way, Menlo Park, CA 94025"
        }
      },
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "Value 1"
          },
          {
            "type": "text",
            "text": "Value 2"
          },
          {
            "type": "text",
            "text": "Value 3"
          }
        ]
      }
    ]
  }
}
```

The Messages API request format would be:

``` json
{
  "to": "447700900000",
  "from": "447700900001",
  "channel": "whatsapp",
  "message_type": "custom",
  "custom": {
    "type": "template",
    "template": {
      "namespace": "whatsapp:hsm:technology:nexmo",
      "name": "parcel_location",
      "language": {
        "policy": "deterministic",
        "code": "en"
      },
      "components": [
        {
          "type": "header",
          "parameters": [
            {
              "type": "location",
              "location": {
                "longitude": -122.425332,
                "latitude": 37.758056,
                "name": "Facebook HQ",
                "address": "1 Hacker Way, Menlo Park, CA 94025"
              }
            }
          ]
        },
        {
          "type": "body",
          "parameters": [
            {
              "type": "text",
              "text": "Value 1"
            },
            {
              "type": "text",
              "text": "Value 2"
            },
            {
              "type": "text",
              "text": "Value 3"
            }
          ]
        }
      ]
    }
  }
}
```

The custom object, `CUSTOM_OBJECT`, defined in the original request format is:

``` json
  "type": "template",
  "template": {
    "namespace": "whatsapp:hsm:technology:nexmo",
    "name": "parcel_location",
    "language": {
      "policy": "deterministic",
      "code": "en"
    },
    "components": [
      {
        "type": "location",
        "location": {
          "longitude": -122.425332,
          "latitude": 37.758056,
          "name": "Facebook HQ",
          "address": "1 Hacker Way, Menlo Park, CA 94025"
        }
      },
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "Value 1"
          },
          {
            "type": "text",
            "text": "Value 2"
          }
        ]
      }
    ]
  }
```

Note that *when using custom objects*, the format for properties such as language code must match that of the target platform. For example, in the underlying WhatsApp message format, language codes are of the form `en_GB`, not `en-GB`.

## Facebook Messenger Message Templates

You can use custom objects for sending Facebook Messenger message templates. For example:

``` json
{
  "to": $FB_RECIPIENT_ID,
  "from": $FB_SENDER_ID,
  "channel": "messenger",
  "message_type": "custom",
  "custom": {$CUSTOM_OBJECT}
}
```

The Messages API message would be transformed to something similar to the following:

``` json
{
  "recipient":{
    "id":"<PSID>"
  },
  "message":{
    $CUSTOM_OBJECT
  }
}
```

Where the custom object, `$CUSTOM_OBJECT`, can be anything from within the [Messenger Message Object](https://developers.facebook.com/docs/messenger-platform/send-messages/templates/).

## See more examples

* [WhatsApp send contact](/messages/code-snippets/whatsapp/send-contact)
* [WhatsApp send location](/messages/code-snippets/whatsapp/send-location)
* [Facebook Messenger custom object](/messages/code-snippets/messenger/send-template)
