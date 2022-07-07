---
title: Kotlin
language: kotlin
---

```kotlin
private val messageListener = object : NexmoMessageEventListener {
    override fun onTypingEvent(typingEvent: NexmoTypingEvent) {}

    override fun onAttachmentEvent(attachmentEvent: NexmoAttachmentEvent) {}

    override fun onTextEvent(textEvent: NexmoTextEvent) {}

    override fun onSeenReceipt(seenEvent: NexmoSeenEvent) {}

    override fun onEventDeleted(deletedEvent: NexmoDeletedEvent) {}

    override fun onDeliveredReceipt(deliveredEvent: NexmoDeliveredEvent) {
        val userName = deliveredEvent.getEmbeddedInfo.user.name

        Log.d("TAG", "Event ${deliveredEvent.initialEventId()} delivered to User $userName")
    }
}

conversation.addMessageEventListener(messageListener)
```
