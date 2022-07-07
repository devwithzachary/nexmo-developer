---
title: Private Voice Communication
products: voice/voice-api
description: Enable users to call each other, keeping their real numbers private.
languages:
    - Node
navigation_weight: 7  
---

# Private Voice Communication

This use case shows you how to implement the idea described in [Private Voice Communication use case](https://www.nexmo.com/use-cases/private-voice-communication/). It teaches you how to build a voice proxy using Vonage's [Node Server SDK](https://github.com/Nexmo/nexmo-node), using virtual numbers to hide the real phone numbers of the participants. Full source code is also available in our [GitHub repo](https://github.com/Nexmo/node-voice-proxy).

## Overview

Sometimes you want two users to be able to call each other without revealing their private phone numbers.

For example, if you are operating a ride sharing service, then you want your users to be able to speak to each other to coordinate pick-up times and locations. But you don't want to give out your customers' phone numbers - after all, you have an obligation to protect their privacy. And you don't want them to be able to arrange ride shares directly without using your service because that means lost revenue for your business.

Using Vonage's APIs, you can provide each participant in a call with a temporary number that masks their real number. Each caller sees only the temporary number for the duration of the call. When there is no further need for them to communicate, the temporary number is revoked.

You can download the source code from our [GitHub repo](https://github.com/Nexmo/node-voice-proxy).

## Prerequisites

In order to work through this use case you need:

* A [Vonage account](https://dashboard.nexmo.com/sign-up?icid=tryitfree_api-developer-adp_nexmodashbdfreetrialsignup_nav)
* The [Vonage CLI](https://github.com/vonage/vonage-cli) installed and configured

## Code repository

There is a [GitHub repository containing the code](https://github.com/Nexmo/node-voice-proxy).

## Steps

To build the application, you perform the following steps:

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Code repository](#code-repository)
- [Steps](#steps)
- [Configuration](#configuration)
- [Create a Voice API application](#create-a-voice-api-application)
- [Create the web application](#create-the-web-application)
- [Provision virtual numbers](#provision-virtual-numbers)
- [Create a call](#create-a-call)
  - [Validate the phone numbers](#validate-the-phone-numbers)
  - [Map phone numbers to real numbers](#map-phone-numbers-to-real-numbers)
  - [Send a confirmation SMS](#send-a-confirmation-sms)
- [Handle inbound calls](#handle-inbound-calls)
- [Reverse map real phone numbers to virtual numbers](#reverse-map-real-phone-numbers-to-virtual-numbers)
- [Proxy the call](#proxy-the-call)
- [Conclusion](#conclusion)
- [Further information](#further-information)

## Configuration

You need to create a `.env` file containing configuration. Instructions on how to do that are explained in the [GitHub README](https://github.com/Nexmo/node-voice-proxy#configuration). As you work through this use case you can populate your configuration file with the required values for variables such as API key, API secret, Application ID, debug mode, and provisioned numbers.

## Create a Voice API application

A Voice API Application is a Vonage construct and should not be confused with the application you are going to write. Instead, it's a "container" for the authentication and configuration settings you need to work with the API.

You can create a Voice API Application with the Vonage CLI. You must provide a name for the application and the URLs of two webhook endpoints: the first is the one that Vonage's APIs will make a request to when you receive an inbound call on your virtual number and the second is where the API can post event data.

Replace the domain name in the following Vonage CLI command with your ngrok domain name ([How to run ngrok](https://developer.nexmo.com/tools/ngrok/)) and run it in your project's root directory:

``` shell
vonage apps:create "Voice Proxy" --voice_answer_url=https://example.com/proxy-call --voice_event_url=https://example.com/event
```

This command creates a file called `voice_proxy.key` that contains authentication information and returns a unique application ID. Make a note of this ID because you'll need it in subsequent steps.

## Create the web application

This application uses the [Express](https://expressjs.com/) framework for routing and the [Vonage Node Server SDK](https://github.com/vonage/vonage-node-sdk) for working with the Voice API. `dotenv` is used so that the application can be configured using a `.env` text file.

In `server.js` the code initializes the application's dependencies and starts the web server. A route handler is implemented for the application's home page (`/`) so that you can test that the server is running by running `node server.js` and visiting `http://localhost:3000` in your browser:

``` javascript
"use strict";

const express = require('express');
const bodyParser = require('body-parser');

const app = express();
app.set('port', (process.env.PORT || 3000));
app.use(bodyParser.urlencoded({ extended: false }));

const config = require(__dirname + '/../config');

const VoiceProxy = require('./VoiceProxy');
const voiceProxy = new VoiceProxy(config);

app.listen(app.get('port'), function() {
  console.log('Voice Proxy App listening on port', app.get('port'));
});
```

Note that the code instantiates an object of the `VoiceProxy` class to handle the routing of messages sent to your virtual number to the intended recipient's real number. The proxying process is described in [proxy the call](#proxy-the-call), but for now be aware that this class initializes the Vonage Server SDK using the API key and secret that you configure in the next step. This enables your application to make and receive voice calls:

``` javascript
const VoiceProxy = function(config) {
  this.config = config;
  
  this.nexmo = new Nexmo({
      apiKey: this.config.VONAGE_API_KEY,
      apiSecret: this.config.VONAGE_API_SECRET
    },{
      debug: this.config.VONAGE_DEBUG
    });
  
  // Virtual Numbers to be assigned to UserA and UserB
  this.provisionedNumbers = [].concat(this.config.PROVISIONED_NUMBERS);
  
  // In progress conversations
  this.conversations = [];
};
```

## Provision virtual numbers

Virtual numbers are used to hide real phone numbers from your application users.

The following workflow diagram shows the process for provisioning and configuring a virtual number:

```sequence_diagram
Participant App
Participant Vonage
Participant UserA
Participant UserB
Note over App,Vonage: Initialization
App->>Vonage: Search Numbers
Vonage-->>App: Numbers Found
App->>Vonage: Provision Numbers
Vonage-->>App: Numbers Provisioned
App->>Vonage: Configure Numbers
Vonage-->>App: Numbers Configured
```

To provision a virtual number you search through the available numbers that meet your criteria. For example, a phone number in a specific country with voice capability:

```code
source: '_code/voice_proxy.js'
from_line: 2
to_line: 47
```

Then rent the numbers you want and associate them with your application.

> **NOTE:** Some types numbers require you have a postal address in order to rent them. If you are not able to obtain a number programmatically, visit the [Dashboard](https://dashboard.nexmo.com/buy-numbers) where you can rent numbers as required.

When any event occurs relating to each number associated with an application, Vonage sends a request to your webhook endpoint with information about the event. After configuration you store the phone number for later use:

```code
source: '_code/voice_proxy.js'
from_line: 48
to_line: 79
```

To provision virtual numbers, visit `http://localhost:3000/numbers/provision` in your browser.

You now have the virtual numbers you need to mask communication between your users.

> **NOTE:** In a production application you choose from a pool of virtual numbers. However, you should keep this functionality in place to rent additional numbers on the fly.

## Create a call

The workflow to create a call is:

```sequence_diagram
Participant App
Participant Vonage
Participant UserA
Participant UserB
Note over App,Vonage: Conversation Starts
App->>Vonage: Basic Number Insight
Vonage-->>App: Number Insight response
App->>App: Map Real/Virtual Numbers\nfor Each Participant
App->>Vonage: SMS to UserA
Vonage->>UserA: SMS
App->>Vonage: SMS to UserB
Vonage->>UserB: SMS
```

The following call:

* [Validates the phone numbers](#validate-phone-numbers)
* [Maps phone numbers to real numbers](#map-phone-numbers)
* [Sends an confirmation SMS](#send-confirmation-sms)

```code
source: '_code/voice_proxy.js'
from_line: 89
to_line: 103
```

### Validate the phone numbers

When your application users supply their phone numbers use Number Insight to ensure that they are valid. You can also see which country the phone numbers are registered in:

```code
source: '_code/voice_proxy.js'
from_line: 104
to_line: 124
```

### Map phone numbers to real numbers

Once you are sure that the phone numbers are valid, map each real number to a [virtual number](#provision-virtual-voice-numbers) and save the call:

```code
source: '_code/voice_proxy.js'
from_line: 125
to_line: 159
```

### Send a confirmation SMS

In a private communication system, when one user contacts another, the caller calls a virtual number from their phone.

Send an SMS to notify each conversation participant of the virtual number they need to call:

```code
source: '_code/voice_proxy.js'
from_line: 160
to_line: 181
```

The users cannot SMS each other. To enable this functionality you need to setup [Private SMS communication](/use-cases/private-sms-communication).

In this use case each user has received the virtual number in an SMS. In other systems this could be supplied using email, in-app notifications, or a predefined number.

## Handle inbound calls

When Vonage receives an inbound call to your virtual number it makes a request to the webhook endpoint you set when you [created a Voice application](#create-a-voice-application):

```sequence_diagram
Participant App
Participant Vonage
Participant UserA
Participant UserB
Note over UserA,Vonage: UserA calls UserB's\nVonage Number
UserA->>Vonage: Calls virtual number
Vonage->>App:Inbound Call(from, to)
```

Extract `to` and `from` from the inbound webhook and pass them on to the voice proxy business logic:

``` javascript
app.get('/proxy-call', function(req, res) {
  const from = req.query.from;
  const to = req.query.to;

  const ncco = voiceProxy.getProxyNCCO(from, to);
  res.json(ncco);
});
```

## Reverse map real phone numbers to virtual numbers

Now you know the phone number making the call and the virtual number of the recipient, reverse map the inbound virtual number to the outbound real phone number:

```sequence_diagram
Participant App
Participant Vonage
Participant UserA
Participant UserB
UserA->>Vonage: 
Vonage->>App: 
Note right of App:Find the real number\n for UserB
App->>App:Number mapping lookup
```

The call direction can be identified as:

* The `from` number is UserA real number and the `to` number is UserB Vonage number
* The `from` number is UserB real number and the `to` number is UserA Vonage number

```code
source: '_code/voice_proxy.js'
from_line: 182
to_line: 216
```

With the number lookup performed all that's left to do is proxy the call.

## Proxy the call

Proxy the call to the phone number the virtual number is associated with. The `from` number is always the virtual number, the `to` is a real phone number.

```sequence_diagram
Participant App
Participant Vonage
Participant UserA
Participant UserB
UserA->>Vonage: 
Vonage->>App: 
App->>Vonage:Connect (proxy)
Note right of App:Proxy Inbound\ncall to UserB's\nreal number
Vonage->>UserB: Call
Note over UserA,UserB:UserA has called\nUserB. But UserA\ndoes not have\n the real number\nof UserB, nor\n vice versa.
```

In order to do this, create an [NCCO (Nexmo Call Control Object)](/voice/voice-api/ncco-reference). This NCCO uses a `talk` action to read out some text. When the `talk` has completed, a `connect` action forwards the call to a real number.

```code
source: '_code/voice_proxy.js'
from_line: 217
to_line: 252
```

The NCCO is returned to Vonage by the web server.

``` javascript
app.get('/proxy-call', function(req, res) {
  const from = req.query.from;
  const to = req.query.to;

  const ncco = voiceProxy.getProxyNCCO(from, to);
  res.json(ncco);
});
```

## Conclusion

You have learned how to build a voice proxy for private communication. You provisioned and configured phone numbers, performed number insight, mapped real numbers to virtual numbers to ensure anonymity, handled an inbound call and proxied the call to another user.

## Further information

* [Voice API](/voice/voice-api/overview)
* [NCCO reference](/voice/voice-api/ncco-reference)
* [GitHub repo](https://github.com/Nexmo/node-voice-proxy)
