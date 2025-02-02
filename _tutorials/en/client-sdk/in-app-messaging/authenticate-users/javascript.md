---
title: Authenticate your Users
description: In this step you authenticate your users via the JWTs you created earlier
---

# Authenticate your Users

Your users must be authenticated to be able to participate in the Conversation. You perform this authentication using the Conversation ID and the JWTs you generated in a previous step.

Declare the following variables at the top of your `chat.js` file and populate `ALICE_JWT`, `BOB_JWT` and `CONVERSATION_ID` with your own values:

```javascript
const ALICE_JWT = '';
const BOB_JWT = '';
const CONVERSATION_ID = '';

const messageTextarea = document.getElementById("messageTextarea");
const messageFeed = document.getElementById("messageFeed");
const sendButton = document.getElementById("send");
const loginForm = document.getElementById("login");
const status = document.getElementById("status");

const loadMessagesButton = document.getElementById("loadMessages");
const messagesCountSpan = document.getElementById("messagesCount");
const messageDateSpan = document.getElementById("messageDate");

let conversation;
let listedEvents;
let messagesCount = 0;
let messageDate;

function authenticate(username) {
  if (username == "Alice") {
    return ALICE_JWT;
  }
  if (username == "Bob") {
    return BOB_JWT;
  }
  alert("User not recognized");
}
```

You'll also need to add an event listener to the `login` form to fetch the user's JWT and pass it in to the `run` function. The `run` function doesn't do anything yet, but at this point you have a valid user JWT to start building your application.

```javascript
loginForm.addEventListener("submit", (event) => {
  event.preventDefault();
  const userToken = authenticate(document.getElementById("username").value);
  if (userToken) {
    document.getElementById("messages").style.display = "block";
    document.getElementById("login").style.display = "none";
    run(userToken);
  }
});

async function run(userToken){

}
```
