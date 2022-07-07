---
title: Custom Events
meta_title: Add custom metadata to conversations by recording data alongside your text or audio events
description: This topic provides an overview of the custom events use via the Vonage Client SDK. 
---

# Custom Events

Custom events allow you to add custom metadata to conversations by recording data alongside your text or audio events. You can add events [using the REST API](/conversation/code-snippets/event/create-custom-event) or any of the Client SDKs.

## Creating a custom event

Each custom event consists of a unique `type` and a `data`. The `type` has the following restrictions:

* Must not exceed 100 characters
* Must only contain alphanumeric, `-` and `_` characters

In addition, the event `data` must not exceed 4096 bytes.

```tabbed_content
source: _tutorials_tabbed_content/client-sdk/custom-events/creating
```

## Listening to custom events

In addition to adding custom events to the conversation, you can listen for custom events using the Client SDK. Register an event handler that listens for your custom event name:

```tabbed_content
source: _tutorials_tabbed_content/client-sdk/custom-events/listening
```

## Complete example

```tabbed_content
source: _tutorials_tabbed_content/client-sdk/custom-events/complete
```
