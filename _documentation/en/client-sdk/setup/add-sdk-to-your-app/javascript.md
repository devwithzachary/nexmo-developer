---
title: JavaScript
language: javascript
---

# Overview

In this guide you learn how to add the Client SDK to your client-side JavaScript app.

## Prerequisites

The Client SDK requires [Node.js](https://nodejs.org) and [NPM](https://www.npmjs.com/).

## To add the Client SDK to your project

### Navigate to your app

Open your terminal. If you have an existing app, navigate to its root. Otherwise, create a new directory for your project.

### Add the Client SDK to your project

```tabbed_content
source: '_tutorials_tabbed_content/client-sdk/setup/add-sdk/javascript'
``` 

## Using the Client SDK in your app

### Creating Users and JWTs

A JSON Web Token (JWT) is necessary to log in to your Vonage Application. The Client SDK cannot manage users nor generate JWTs, so you must choose a method of handling it on the backend:

- For onboarding or testing purposes, you can get your client-side app working before setting up a backend by [generating a test JWT from the command line](/tutorials/client-sdk-generate-test-credentials) and hard-coding it in your client-side JavaScript.
- For real world usage, you can deliver JWTs from the server using the Node or PHP [backend SDKs](/tools), and set the `jwt` variable in your code by fetching that data:

    ```javascript
    fetch('/getJwt')
      .then(results => results.json())
      .then(data => {
        jwt = data.token;
      })
      .catch(err => console.log(err));
    ```
- Read more on generating JWTs [in this article](/client-sdk/concepts/jwt-acl)

### Instantiate and log in the NexmoClient

No arguments are necessary to instantiate a new `NexmoClient`, but you will need to pass your JWT as the argument to `login()`.

```javascript
let nexmo = new NexmoClient()
  .createSession(jwt)
  .then(app => console.log('Logged in to app', app))
  .catch(err => console.log(err));
```

### Client SDK analytics and usage data

To provide Vonage with more information to enable us to fix bugs and build features, you can _optionally_ opt-in to our Client SDK analytics and usage data programme.

To enable analytics and data usage reporting, please set the `enabled` parameter of `log_reporter` to `true`. The following code provides an example of this:

```javascript
new NexmoClient({debug:true, log_reporter: {enabled: true}})
```

> **NOTE:** This is opt-in only and turned off by default.

## Conclusion

You added the Client SDK to your client-side JavaScript app and logged in to a `NexmoClient` instance, which returned an `Application` object. You can now use `Application.newConversation()` to create a conversation, and then access the functionality of a `Conversation`.

## See also

* [Data Center Configuration](/client-sdk/setup/configure-data-center) - this is an advanced optional configuration you can carry out after adding the SDK to your application.
