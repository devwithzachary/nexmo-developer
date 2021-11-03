---
title: NeRu Training
description: To follow along with in the NeRu Training session
meta_title: NeRu Training
---

# NeRu Training

Thanks for coming to the NeRu training session. 

Session outline:

* Overview of NeRu (+ Questions)
* Demo (+ Questions)
* Set up an example project
* Debugging
* Add a feature to the example project
* Open questions/feedback

## NeRu Overview

* NeRu
* Providers
* Deploying
    * Config file
    * Sessions
    * Instance
* Debugging

## Demo

NeRu demo from Max.

## Example project

If you haven't installed the [NeRu CLI](https://vonage-neru.herokuapp.com/neru/cli) yet please do so first. Check if you are on the latest version and configure the NeRu CLI if you have not already:

```
neru version
neru configure
```

Create a folder and change directory into it:

```
mkdir neru-training
cd neru-training
```

### Project Configuration

* Create a new Vonage application:

```
neru app create --name "Test app"
```

NeRu will handle your public/private key pair for you, do not create a new key pair via the dashboard.

* **Buy and/or link a number** to the new application via the API dashboard or using the Vonage CLI:

```
vonage apps:link APPLICATION_ID --number=YOUR_VONAGE_NUMBER
```

* **Update your Vonage application's webhooks** to enable Voice:

```
neru app configure --voice=true --messages=true --rtc=false --app-id=APPLICATION_ID
```

> **Double check your project on the API Dashboard has Voice and Messages enabled with your LVN linked before continuing. Messages should be set to the v0.1 webhook version.**

Create a NeRu project:

```
neru init
```

Choose the `VonagePark` template. When it completes, install the dependencies:

```
npm install
```

Open the folder in VSCode and edit the `neru.yml` file to include the contact configuration:

```yml
project:
    name: nerutraining
instance:
    name: dev
    runtime: nodejs
    region: aws.euw1
    application-id: 0613a987-cbef-46ab-b9ac-ca5b8c260345
    configurations:
        contact:
            number: "$YOUR_VONAGE_NUMBER"
```

Debug the project locally:

```
neru debug
```

Now you are ready to call your Vonage number.

### Debug

Attach the VSCode debugger to your [debug](https://vonage-neru.herokuapp.com/neru/debugging) session Set breakpoints in your code and call your Vonage number again.

> If you are not getting an incoming call please make sure you have covered all the steps in the [project configuration](/neru/training#project-configuration) section. If you have, ask for help.

### Deploy

Now you are ready to deploy:

```
neru deploy
```

This will deploy your project to the NeRu platform. Once it is complete, visit the [NeRu dashboard](https://dashboard.serverless.vonage.com/) and view your project.

### Add a Feature

To build on top of the sample, we will add a feature to project, to send an SMS reminder to the user when their parking is about to run out. To do so we are going to use the Scheduler provider.

In the `pay` case of the `onEvent` route before the `res.json`, create an instance of the Scheduler provider:

```
const scheduler = new Scheduler(session);
```

Then calculate the time to send the message, we will send a reminder 10 minutes before. For testing we will send a reminder in 20 seconds:

```javascript
const testTime = new Date(new Date().setSeconds(new Date().getSeconds() + 20)).toISOString();
```

Now make a call to the scheduler:

```javascript
await scheduler.startAt({
    startAt: testTime,
    callback: 'parkingReminder',
    payload: {
        number: from,
    }
}).execute();
```

So, when the time is reached, the scheduler will make a call to the `/parkingReminder` route:

```javascript
router.post('/parkingReminder', async (req, res, next) => {
    try {
 
        
    } catch (error) {
        next(error);
    }
});
```

Inside the route, create a session from the request and create an instance of the Messages scheduler:

```javascript
const session = neru.getSessionFromRequest(req)
const state = session.getState();
const messaging = new Messages(session);
```

Below, get the data we need. Note how we are able to access the state from before because we created the session from the request:

```javascript
const from = req.body.number;
const data = await state.get("calldata");

const to = { type: "sms", number: from };
const vonageNumber = { type: "sms", number: contact.number }; 
```

Finally, send the message:

```javascript
await messaging.sendText(
    vonageNumber,
    to,
    `Your parking at ${data.parkingID} for is about to run out.`
).execute();

res.sendStatus(200);
```

Now if you call back and go through the flow, you will receive a text message 20 seconds later. 

## Feedback

Thanks for attending the training, we would love your feedback on the training and NeRu. Please post in the training slack channel. Any feedback is welcome, but here are some talking points to consider:

* What applications do you think will be easier to build now with NeRu?

* How did you find using NeRu compared to writing a similar application (Voice API/Messages API) normally?

* Can you think of any projects for clients you worked on recently that could have benefited from using NeRu?

* What features would you like to see added to NeRu?

* Any NeRu concepts you didn't understand?

* Did you find the NeRu debugging useful?

* Are you able to volunteer some time to develop a sample use case with NeRu, to build on what you’ve learned?