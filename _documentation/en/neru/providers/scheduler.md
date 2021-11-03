---
title: Scheduler
description: The Scheduler Provider
navigation_weight: 5
---

# Scheduler Provider

The Scheduler provider allows you to schedule functions to be run at a specific time or after a time out.

## Functions

* [`startAt`](/neru/code-snippets/scheduler-provider/schedule-callback.md)

* [`cancel`](/neru/code-snippets/scheduler-provider/cancel-callback.md)

## Initializing the Scheduler Provider

To use the Scheduler Provider you need to create an instance of the provider using a session:

```javascript
const session = neru.createSession();
const scheduler = new Scheduler(session);
```

Requests made from NeRu to your application via the Scheduler provider will contain the session information from the session which set up the scheduled callback. Meaning you can access state from the originating session using `getSessionFromRequest` to create a session object. 

## Usage

For example, to use the Scheduler provider to send a reminder text in five minutes time after an incoming message:

```javascript
router.post('/onMessage', async (req, res, next) => {
    try {
        const session = neru.createSession();
        const scheduler = new Scheduler(session);

        const reminderTime = new Date(new Date().setMinutes(new Date().getMinutes() + 5)).toISOString();

        await scheduler.startAt({
            startAt: reminderTime,
            callback: 'textReminder',
            payload: {
                from: req.body.from,
            }
        }).execute();


    } catch (error) {
        next(error);
    }
});
```

After five minutes have passed the `textReminder` route will get called with the payload:

```javascript
router.post('/textReminder', async (req, res, next) => {
    try {
        const session = neru.createSession();
        const messaging = new Messages(session);

        const from = req.body.from;

        const to = { type: "sms", number: from };
        const vonageNumber = { type: "sms", number: "447000000000" }; 

        await messaging.sendText(
            vonageNumber,
            to,
            `Reminder from Vonage!`
        ).execute();

        res.sendStatus(200);
    } catch (error) {
        next(error);
    }
});
```