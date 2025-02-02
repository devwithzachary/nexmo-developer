---
title: Vonage Video Express with Ruby on Rails Part 2
description: A ruby on rails tutorial that implements the Video Express
  javascript library for fast and easy WebRTC video conferencing applications.
thumbnail: /content/blog/vonage-video-express-with-ruby-on-rails-part-2/video-express-ruby_part_2.png
author: benjamin-aronov
published: true
published_at: 2022-08-10T09:29:42.442Z
updated_at: 2022-08-10T09:29:43.528Z
category: tutorial
tags:
  - video-api
  - video-express
  - ruby-on-rails
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
This is the second part of a two-part series on creating a video watch party application using Ruby on Rails with Vonage Video API and the Video Express library.

In the [Part 1](https://developer.vonage.com/blog/22/08/09/vonage-video-express-with-ruby-on-rails-part-1), we went through the steps of building the Rails app, showed how to use a few Vivid components, and got the Video Express video chat to run. If you have not read that post yet, it would be a good place to start.

Once we are done, we will have a watch party app that we can use to chat with our friends and watch sports or videos together!

## What The App Will Do

A quick reminder, we are building a video-conferencing application that gives a toolbar to users for different audio/video controls. Additionally, the application gives the moderator the ability to send the Watch Party into different viewing modes.

At this point, we have a working Video Express [Room](https://tokbox.com/developer/video-express/reference/room.html). This object gives us the ability to call different functions that perform the actions in our toolbar. We want to give the user a way to trigger this functionality, we'll do that with Vivid components. We will organize both our HTML and JS into components. With Webpack, we'll then  `import` our Modules and `require` our components into `application.js` which will expose our Javascript to the client-side.

## Building Out Helper Components

The rest of this tutorial will be building the components that allow users to control their Video Express room. Each component will follow a similar structure: HTML with Vivid components and Javascript to trigger Video Express functions.

### Organizing the HTML

Let's build out our partials where the HTML will live. From the root of your application, using the command line, run:

`mkdir app/views/components`

`touch app/views/components/_header.html.erb`
`touch app/views/components/_toolbox.html.erb`

And update the `party.html.erb` file to render the partials:

```ruby
<header>
  <%= render partial: 'components/header' %>
</header>

<main class="app">
  <div id="roomContainer"></div>
  <toolbar>
    <%= render partial: 'components/toolbar' %>
  </toolbar>
```

### Organizing the Javascript

Just as we have a components folder in our Views, let's create a components folder in our Javascript folder to house our corresponding component logic.

`mkdir app/javascript/components`

Here we'll add our component files:
`touch app/javascript/components/header.js`
`touch app/javascript/components/toolbar.js`

To require them for our clientside via Webpack, add the following lines in Application.js below our module imports.

```ruby
require("components/header");
require("components/toolbar");
```

Because our Javascript will respond to user actions in the DOM, we want to make sure that Javascript is loaded by Rails after the DOM is loaded. So we need to make a small addition and add `defer:true` to the `javascript_pack_tag` in `application.html.erb`:

`<%= javascript_pack_tag 'application', 'data-turbolinks-track': 'reload', defer: true  %>`

Now we're ready to build out our components.

## Building the Header

### Building the HTML

A reminder of the header we want to build:

![The Header in Moderator View](/content/blog/vonage-video-express-with-ruby-on-rails-part-2/group-10-4-.png "The Header in Moderator View")

Some great news is that Vivid has exactly what we need, a [Top App Bar](https://vivid.vonage.com/?path=/story/components-top-app-bar-fixed--dense&args=dense:) component.
The top app bar comes with a few slot options but two that we care about: `title` and `actionItems`. The title is great for a logo or in our case title. And the `actionitems` can be used as the content of the app bar. This is where we will add the toggler for the moderator to change modes between chill mode and party mode. We can accomplish this with the `vwc-switch` component.

Inside `app/view/watch_party/_header.html.erb` will look like this:

```ruby
<vwc-top-app-bar-fixed alternate="true">
  <span slot="title" id="title">Big Game Chill Zone</span>
  <% if @name == @moderator_name  %>
    <span slot="actionItems" id="mode-name">Chill Mode</span>
    <vwc-switch slot="actionItems"></vwc-switch>
  <% else %>
    <span slot="actionItems"><%= @moderator_name %> is the host</span>
  <% end %>
</vwc-top-app-bar-fixed>
```

### Building The Header Javascript

Inside the `components/header.js` file we'll have 3 essential parts: listening for a toggle, toggling the sharescreen, and toggling the accompanying visuals.

**Important Note About Video Express**

In Video Express you have two set layouts: "Grid" and "Active Speaker". Video Express gives you a standarized video call running fast! But if you want to deviate from the default behaviour, tread lightly and you will probably be better off using the full Video API.

In this example, the moderator is the only screen sharer. We might think that the "Active Speaker" layout would allow us to make the shared screen the dominant screen. However, in standardized video conferencing, the expected behaviour is that the screen sharer presents from another tab. So by default, Video Express does not make their shared screen window the dominant view.

In our use case, the moderator doesn't need to control the screen and wants to see their screenshare big, just like everyone else. This is not the default behaviour. We will need to build it out ourselves. Thankfully, Video Express has the `[screenSharingContainer](https://tokbox.com/developer/video-express/reference/room.html)` option which will allow us to build what we want.

We can see from the documentation that we just need to add an empty DIV with id of `screenSharingContainer`. But we need to make it look nice and still be able to take advantage of VideoExpress LayoutManager for responsiveness. And, we only want to have this applied for the moderator in the screensharing mode. So our solution is to append the `screenSharingContainer` beside the `layoutContainer` and write some CSS to make the moderator's view as close as possible to what everyone else sees!

First let's create the base logic of the listener for the toggle:

```ruby
const switch_btn = document.querySelector('vwc-switch');

if (switch_btn !== null){
  switch_btn.addEventListener('change', (event) => {
    if (event.target.checked){
      <!-- Custom Styles To Scope Moderator View --->
      <!-- Start Screen Share -->
    }
    else if (!event.target.checked){
      <!-- Stop Screen Share -->
      <!-- Remove Custome Styles From Moderator --->
    } else{
        console.log("Error in Switch Button Listener");
      }
  });
}
```

Now before we can trigger the screenshare in Video Express, we need to prepare the Moderator's view so that the custom styling, which will touch the `layoutContainerWrapper` and `layoutContainer` doesn't affect the view of everyone else. We'll call this function `addModeratorCustomStyles`.

```ruby
let addModeratorCustomStyles = () => {
  mode_name.innerHTML = "Watch Mode"
  layoutContainerWrapper.firstElementChild.classList.add("moderator-screenshare");
  layoutContainerWrapper.classList.add("moderator-screenshare");
  screenShare.setAttribute("id", "screenSharingContainer");
  layoutContainer.appendChild(screenShare);
}
```

We can see this function does two things: update the label of the toggler and adds an id of `screenSharingContainer`. This added id helps us scope CSS to only be applied for the Moderator.

When the screenShare stops, we'll need to remove the custom styling so the Moderator's view is not messed up. So we have a function `removeModeratorCustomStyles` to undo everything from before:

```ruby
let removeModeratorCustomStyles = () => {
  mode_name.innerHTML = "Chill Mode";
  layoutContainerWrapper.firstElementChild.classList.remove("moderator-screenshare");
  layoutContainerWrapper.classList.remove("moderator-screenshare");
  screenShare.removeAttribute("id");
  layoutContainer.removeChild(layoutContainer.lastChild);
}
```

Now our toggle header is basically complete. We just need to query our elements and call the Video Express screensharing functions. The full header looks like this:

```ruby
const switch_btn = document.querySelector('vwc-switch');
const layoutContainer = document.querySelector('#layoutContainerWrapper');
const screenShare = document.createElement('div');
const layoutContainerWrapper = document.querySelector('#layoutContainerWrapper');
const mode_name = document.querySelector('#mode-name');

let addModeratorCustomStyles = (layoutContainer, screenShare, layoutContainerWrapper, mode_name) => {
  mode_name.innerHTML = "Watch Mode";
  layoutContainerWrapper.firstElementChild.classList.add("moderator-screenshare");
  layoutContainerWrapper.classList.add("moderator-screenshare");
  screenShare.setAttribute("id", "screenSharingContainer");
  layoutContainer.appendChild(screenShare);
}

let removeModeratorCustomStyles = (layoutContainer, screenShare, layoutContainerWrapper, mode_name) => {
  mode_name.innerHTML = "Chill Mode";
  layoutContainerWrapper.firstElementChild.classList.remove("moderator-screenshare");
  layoutContainerWrapper.classList.remove("moderator-screenshare");
  screenShare.removeAttribute("id");
  layoutContainer.removeChild(layoutContainer.lastChild);
}

if (switch_btn !== null){
  switch_btn.addEventListener('change', (event) => {
    if (event.target.checked){
      addModeratorCustomStyles(layoutContainer, screenShare, layoutContainerWrapper, mode_name);
      room.startScreensharing('screenSharingContainer');
    } else if (!event.target.checked){
        room.stopScreensharing('screenSharingContainer');
        removeModeratorCustomStyles(layoutContainer, screenShare, layoutContainerWrapper, mode_name);
    } else{
        console.log("Error in Switch Button Listener");
    }
  });
}
```

### Building The Toolbar HTML

Now we can add the last and most complicated component we have: the toolbar. But it won't be so bad, just building out the HTML with Vivid components and then adding some Javascript to trigger Video Express functions. You know the drill!

A reminder of the toolbar we want to build:

![The Toolbar To Control The Video Call](/content/blog/vonage-video-express-with-ruby-on-rails/screen-shot-2022-07-01-at-13.25.36.png "The Toolbar To Control The Video Call")

#### Building The Toggle Buttons

We can see in the toolbar that there are 3 groups of buttons that will toggle on/off some features in the room: mute/unmute all, disable/enable microphone, and disable/enable video camera. For all three we will use two buttons from Vivid and then use Javascript to hide the inactive button.

```ruby
<!-- Mute all / Unmute all -->
<vwc-icon-button icon="audio-max-solid" shape="circled" layout="filled" id="mute-all" class="white-border"></vwc-icon-button>
<vwc-icon-button icon="audio-off-solid" shape="circled" layout="filled" id="unmute-all" class="hidden white-border"></vwc-icon-button>
```

```ruby
<!-- Mute self / Unmute self -->
<vwc-icon-button icon="mic-mute-solid" shape="circled" layout="ghost" id="mute-self" class="vvd-scheme-alternate" ></vwc-icon-button>
<vwc-icon-button icon="microphone-2-solid" shape="circled" layout="ghost" id="unmute-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
```

```ruby
<!-- Disable camera / Enable camera -->
<vwc-icon-button icon="video-off-solid" shape="circled" layout="ghost" id="hide-self" class="vvd-scheme-alternate" ></vwc-icon-button>
<vwc-icon-button icon="video-solid" shape="circled" layout="ghost" id="unhide-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
```

#### Building Dropdown Selects

We can see that the second and third sets of buttons are a little different though. They also should be accompanied by a dropdown which will allow the user to select the associated input: microphone or camera. This is possible with Vivid's [`<vwc-action-group` element](https://vivid.vonage.com/?path=/story/alpha-components-actiongroup--split-button). The left side of the action group comes straight from documentation with a button and a separator. On the right side, we'll make use of the Vivid `vwc-select` component to generate a `select` element which we can target with the different options we receive from VideoExpress. We also pass the `vwc-select` two options: selected and disabled to tell it to show and disable the default `vwc-list-item` which will act as a label.

Our two action groups with buttons now look like this:

```ruby
<!-- Mute Self / Unmute Self -->
<!-- Select Mic Input -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate">
  <vwc-icon-button icon="mic-mute-solid" shape="circled" layout="ghost" id="mute-self" class="vvd-scheme-alternate" ></vwc-icon-button>
  <vwc-icon-button icon="microphone-2-solid" shape="circled" layout="ghost" id="unmute-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
  <span role="separator"></span>
  <vwc-select appearance="ghost" id="audio-input" class="select-max-width">
    <vwc-list-item
      disabled
      selected
    >
    Mic
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>
```

```ruby
<!-- Disable Camera / Enable Camera -->
<!-- Select Camera Input -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate">
  <vwc-icon-button icon="video-off-solid" shape="circled" layout="ghost" id="hide-self" class="vvd-scheme-alternate" ></vwc-icon-button>
  <vwc-icon-button icon="video-solid" shape="circled" layout="ghost" id="unhide-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
  <span role="separator"></span>
  <vwc-select appearance="ghost" class="select-max-width" id="video-input">
    <vwc-list-item
      disabled
      selected
    >
    Camera
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>
```

We can see that we have a third action group: the audio inputs. We can use the same structure:

```ruby
<!-- Select Audio Output -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate" id="audio-output-target">
  <vwc-icon-button icon="headset-solid" shape="circled" layout="ghost" class="vvd-scheme-alternate"></vwc-icon-button>
  <span role="separator"></span>
  <vwc-select appearance="ghost" id="audio-output" class="select-max-width">
    <vwc-list-item
      disabled
      selected
    >
    Audio
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>
```

#### Building The ToolTips HTML

Let's give our users a little bit more information about these components. We can do so elegantly using [Vivid's tooltips](https://vivid.vonage.com/?path=/story/alpha-components-tooltip--introduction).

There are three parts to the tooltips we care about: the `anchor`, the `corner`, and the `id`. The anchor tells the tooltip which HTML element to hook onto. The [corner](https://vivid.vonage.com/?path=/story/alpha-components-tooltip--introduction) tells the tooltip in which direction to display, in relation to the anchor. And the `id` is something we will use in the Javascript to trigger displaying the tooltip when a user hovers over the anchor.

With the Tooltips added, our full `_toolbar.html.erb` looks like this:

```ruby
<!-- Mute all / Unmute all -->
<vwc-icon-button icon="audio-max-solid" shape="circled" layout="filled" id="mute-all" class="white-border"></vwc-icon-button>
<vwc-icon-button icon="audio-off-solid" shape="circled" layout="filled" id="unmute-all" class="hidden white-border"></vwc-icon-button>
<vwc-tooltip anchor="mute-all" text="Mute All" corner="top" id="mute-all-tooltip"></vwc-tooltip>
<vwc-tooltip anchor="unmute-all" text="Un-Mute All" corner="top" id="unmute-all-tooltip"></vwc-tooltip>

<!-- Mute Self / Unmute Self -->
<!-- Select Mic Input -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate">
  <vwc-icon-button icon="mic-mute-solid" shape="circled" layout="ghost" id="mute-self" class="vvd-scheme-alternate" ></vwc-icon-button>
  <vwc-icon-button icon="microphone-2-solid" shape="circled" layout="ghost" id="unmute-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
  <vwc-tooltip anchor="mute-self" text="Disable Mic" corner="top" id="mute-self-tooltip"></vwc-tooltip>
  <vwc-tooltip anchor="unmute-self" text="Enable Mic" corner="top" id="unmute-self-tooltip"></vwc-tooltip>
  <span role="separator"></span>
  <vwc-select appearance="ghost" id="audio-input" class="select-max-width">
    <vwc-list-item
      disabled
      selected
    >
    Mic
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>



<!-- Disable Camera / Enable Camera -->
<!-- Select Camera Input -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate">
  <vwc-icon-button icon="video-off-solid" shape="circled" layout="ghost" id="hide-self" class="vvd-scheme-alternate" ></vwc-icon-button>
  <vwc-icon-button icon="video-solid" shape="circled" layout="ghost" id="unhide-self" class="hidden vvd-scheme-alternate"></vwc-icon-button>
  <vwc-tooltip anchor="hide-self" text="Disable Camera" corner="top" id="hide-self-tooltip"></vwc-tooltip>
  <vwc-tooltip anchor="unhide-self" text="Enable Camera" corner="top" id="unhide-self-tooltip"></vwc-tooltip>
  <span role="separator"></span>
  <vwc-select appearance="ghost" class="select-max-width" id="video-input">
    <vwc-list-item
      disabled
      selected
    >
    Camera
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>

<!-- Select Audio Output -->
<vwc-action-group layout="outlined" shape="pill" class="vvd-scheme-alternate" id="audio-output-target">
  <vwc-tooltip anchor="audio-output-target" text="Select Audio Output" corner="top" id="audio-output-tooltip"></vwc-tooltip>
  <vwc-icon-button icon="headset-solid" shape="circled" layout="ghost" class="vvd-scheme-alternate"></vwc-icon-button>
  <span role="separator"></span>
  <vwc-select appearance="ghost" id="audio-output" class="select-max-width">
    <vwc-list-item
      disabled
      selected
    >
    Audio
    </vwc-list-item>
  </vwc-select>
</vwc-action-group>
```

#### Styling The Toolbar

Before we build out the Javascript for the components, let's make the toolbar look like our mockup:

```ruby
// toolbar styles
toolbar {
  display: flex;
  justify-content: space-around;
  margin: 0 auto;
  width: 650px;
  background-color: var(--vvd-color-primary);
  padding: 10px;
  border-radius: 8px 8px 0 0;
}

.hidden {
  display: none;
}

.white-border {
  border: 2px solid white;
  border-radius: 50%;
}

.select-max-width {
  max-width: 130px;
}

vwc-tooltip {
  --tooltip-inline-size: 100px;
  text-align: center;
}
```

## Building The Toolbar Javascript

Going left to the right, the first component we need to build is the "Mute All" button. We want this to be a toggler which toggles on and off the audio of all other participants. But how can we have a toggle button? We really need two buttons: a "mute all" and an "unmute all" button. This pattern will be replicated several times in the toolbar so let's write a helper function for it.

```ruby
// toggle hide/display of buttons
let toggleButtonView = (buttonToHide, buttonToShow) => {
  buttonToHide.style.display = "none";
  buttonToShow.style.display = "block";
}
```

### Building The Mute Others Button

To mute all the participants we need to listen for the user to trigger the action and then cycle through all the participants in this user's instance of the room. *This is important to note that the room is local to each user.*

So to disable the audio, we need to iterate through all the participants in the room and use the `.camera.disableAudio()` function. You'll notice that we call this on participant\[1] because the participant object returns an array: [`id`, participantObject].

```ruby
// toggle Mute All / Unmute All
let toggleMuteAllButton = (button, state, participants) =>{
  button.addEventListener("click", function(){
    Object.entries(participants).forEach(participant => {
      if (state === "mute"){
        toggleButtonView(mute_all_btn, unmute_all_btn)
        participant[1].camera.disableAudio();
      } else if (state === "unmute") {
        toggleButtonView(unmute_all_btn, mute_all_btn)
        participant[1].camera.enableAudio();
      } else {
        console.log("Error in toggleMuteAll")
      }
    })
  })
}
```

```ruby
const mute_all_btn = document.querySelector('#mute-all');
const unmute_all_btn = document.querySelector('#unmute-all');
```

```ruby
toggleMuteAllButton(mute_all_btn, "mute", room.participants);
toggleMuteAllButton(unmute_all_btn, "unmute", room.participants);
```

### Building The Mute Self And Disable Camera Buttons

The Mute/Unmute Self and Disable/Enable Camera buttons follow the same logic as the MuteAll button. They will listen for a user action, check a state and call an action in VideoExpress. However they are much simpler because VideoExpress gives us functions to check the state. The two functions are `room.camera.isVideoEnabled` and `room.camera.isAudioEnabled`. Also we only need to trigger the action on a single user.

We can create this function `toggleInputButton` which will accept a condition, the boolean we receive from our Video Express functions, and then call the corresponding Video Express action. It will also update the view with `toggleButtonView`.

```ruby
// toggle button (display and functionality) of any audio and video input devices
let toggleInputButton = (condition, defaultBtn, altBtn, action) => {
  if (condition()){
    toggleButtonView(defaultBtn, altBtn);
    action();
  } else if (!condition()){
    toggleButtonView(altBtn, defaultBtn);
    action();
  } else {
    console.log(`Error in toggleInputButton. Condition: ${condition}`);
  }
}

The toggle will need to be triggered on a user click, so we can wrap this in the `listenForToggle` function.

// listen for clicks to trigger toggle of input buttons
let listenForToggle = (condition, defaultBtn, altBtn, defaultAction, altAction) => {
    defaultBtn.addEventListener("click", function(){
      console.log("inside listenfortoggle listener")
      toggleInputButton(condition, defaultBtn, altBtn, defaultAction)
    })
    altBtn.addEventListener("click", function(){
      toggleInputButton(condition, defaultBtn, altBtn, altAction)
    })
}
```

We'll need the specific DOM elements to listen for and pass `listenForToggle`, so we create some query selectors.

```ruby
const mute_self_btn = document.querySelector('#mute-self');
const unmute_self_btn = document.querySelector('#unmute-self');

const hide_self_btn = document.querySelector('#hide-self');
const unhide_self_btn = document.querySelector('#unhide-self');
```

Finally, all together we can call our code by passing the conditions, the buttons, and the actions:

```ruby
listenForToggle(room.camera.isVideoEnabled, hide_self_btn, unhide_self_btn, room.camera.disableVideo, room.camera.enableVideo);
listenForToggle(room.camera.isAudioEnabled, mute_self_btn, unmute_self_btn, room.camera.disableAudio, room.camera.enableAudio);
```

### Building The Select Device Inputs

Our select dropdowns for audio and video inputs are currently empty. Let's fill them up with each user's devices. Video Express gives us this functionality out of the box with `VideoExpress.getDevices()`. This returns an array of both audio and video inputs for the local VideoExpress instance.

We'll need to retrieve this list and then sort it so we can add audio devices to the microphone input and video devices to the camera input. We do so with this asynchronous function, which waits for the Promise to return from querying VideoExpress and then iterates on the devices and appends them to the select element.

```ruby
// Retrieve available input devices from VideoExpress
// add retrieved input devices to select options
async function getDeviceInputs(audioTarget, videoTarget){
  const audio = document.querySelector(`${audioTarget}`);
  const video = document.querySelector(`${videoTarget}`);
  const availableDevices = VideoExpress.getDevices();
  availableDevices.then(devices=> {
    devices.forEach(device => {
      if (device.kind === "audioInput"){
        let opt = document.createElement('vwc-list-item');
        opt.value = device.deviceId;
        opt.innerHTML = device.label;
        audio.appendChild(opt);
      }
      else if (device.kind === "videoInput"){
        let opt = document.createElement('vwc-list-item');
        opt.value = device.deviceId;
        opt.innerHTML = device.label;
        video.appendChild(opt);
      }
      else{
        console.log("Error in retrieveDevices");
      }
    })
  })
}
```

We'll also want to update the room when a user changes their selection. We can trigger this by listening to changes in the select menus and then calling either `room.camera.setAudioDevice()` or `room.camera.setVideoDevice()`.

```ruby
// // listen for changes to selected audio/video inputs
// // update room when inputs are changed
let listenInputChange = (target) => {
  const targetSelect = document.querySelector(`${target}`);
  targetSelect.addEventListener('change', (inputOption) => {
    if (target === "vwc-select#audio-input"){
      room.camera.setAudioDevice(inputOption.target.value);
    }
    else if (target === "vwc-select#video-input"){
      room.camera.setVideoDevice(inputOption.target.value);
    }
    else{
      console.log("Error in listenInputChange");
    }
  })
}
```

Putting it all together, calling our functions looks like this:

```ruby
getDeviceInputs("vwc-select#audio-input", "vwc-select#video-input");
listenInputChange("vwc-select#audio-input");
listenInputChange("vwc-select#video-input");
```

### Building The Select Audio Output

Retrieving the list of Audio Outputs from VideoExpress will look almost identical to the inputs. However, here we don't have any devices for Video Output so our code is almost identical except without the filtering. Just like we had `VideoExpress.getDevices()`, now we have `VideoExpress.getAudioOutputDevices()`. And similar to `room.camera.setAudioDevice()` or `room.camera.setVideoDevice()`,  now we have `VideoExpress.setAudioOutputDevice()`.

Because we are only listening for a single change, we can tie it all up nicely in a single function:

```ruby
// Retrieve lists of auidoOutput
// add audioOutputs to select menu
// On user select new option, update audio input
async function audioOutputs() {
  var audioOutputs = await VideoExpress.getAudioOutputDevices();
  const audioOutputSelect = document.querySelector('vwc-select#audio-output');
  audioOutputs.forEach(output => {
    let opt = document.createElement('vwc-list-item');
    opt.value = output.deviceId;
    opt.innerHTML = output.label;
    audioOutputSelect.appendChild(opt);
  })

  audioOutputSelect.addEventListener('change', (audioOutputOption) => {
    VideoExpress.setAudioOutputDevice(audioOutputOption.target.value);
  });
}
```

And don't forget to call it in your code!

```ruby
audioOutputs();
```

### Building The Tooltips

Our very last step in the toolbar is to add the behaviour of tooltips to appear and disappear. We do this with vanilla.js by targeting the anchor which we'll call the `target` and the associated tooltip which we call `targertToolTip`. The `mouseover` and `mouseout` allow us to listen when a user hovers on an anchor and when they stop.

Our `addToolTipListeners` looks like this:

```ruby
// toggle tooltips on hover
let addToolTipListeners = (toolTipsToListen) => {
  toolTipsToListen.forEach(toolTipToListen => {
    const target = document.querySelector(`#${toolTipToListen.targetId}`);
    const targetToolTip = document.querySelector(`#${toolTipToListen.toolTipId}`);
    target.addEventListener('mouseover', (event) => targetToolTip.open = !targetToolTip.open);
    target.addEventListener('mouseout', (event) => targetToolTip.open = !targetToolTip.open);
  })
}
```

Then we'll define our tooltips to listen on:

```ruby
const toolTipsToListen = [
  {targetId: "hide-self", toolTipId:"hide-self-tooltip"},
  {targetId: "unhide-self", toolTipId: "unhide-self-tooltip"},
  {targetId: "mute-self", toolTipId: "mute-self-tooltip"},
  {targetId: "unmute-self", toolTipId: "unmute-self-tooltip"},
  {targetId: "mute-all", toolTipId: "mute-all-tooltip"},
  {targetId: "unmute-all", toolTipId: "unmute-all-tooltip"},
  {targetId: "audio-output-target", toolTipId: "audio-output-tooltip"},
]
```

And lastly, call our code.

```ruby
addToolTipListeners(toolTipsToListen);
```

Our final code for `toolbar.js` looks like this:

```ruby
// Start of Toolbar Code

// toggle hide/display of buttons
let toggleButtonView = (buttonToHide, buttonToShow) => {
  buttonToHide.style.display = "none";
  buttonToShow.style.display = "block";
}

// toggle Mute All / Unmute All
let toggleMuteAllButton = (button, state, participants) =>{
  button.addEventListener("click", function(){
    Object.entries(participants).forEach(participant => {
      if (state === "mute"){
        toggleButtonView(mute_all_btn, unmute_all_btn)
        // why need both here???
        participant[1].camera.disableAudio();
      } else if (state === "unmute") {
        toggleButtonView(unmute_all_btn, mute_all_btn)
        participant[1].camera.enableAudio();
      } else {
        console.log("Error in toggleMuteAll")
      }
    })
  })
}



// toggle button (display and functionality) of any audio and video input devices
let toggleInputButton = (condition, defaultBtn, altBtn, action) => {
  if (condition()){
    toggleButtonView(defaultBtn, altBtn);
    action();
  } else if (!condition()){
    toggleButtonView(altBtn, defaultBtn);
    action();
  } else {
    console.log(`Error in toggleInputButton. Condition: ${condition}`);
  }
}

// listen for clicks to trigger toggle of input buttons
let listenForToggle = (condition, defaultBtn, altBtn, defaultAction, altAction) => {
    defaultBtn.addEventListener("click", function(){
      toggleInputButton(condition, defaultBtn, altBtn, defaultAction)
    })
    altBtn.addEventListener("click", function(){
      toggleInputButton(condition, defaultBtn, altBtn, altAction)
    })
}

// Retrieve available input devices from VideoExpress
// Add retrieved input devices to select options
async function getDeviceInputs(audioTarget, videoTarget){
  const audio = document.querySelector(`${audioTarget}`);
  const video = document.querySelector(`${videoTarget}`);
  const availableDevices = VideoExpress.getDevices();
  availableDevices.then(devices=> {
    devices.forEach(device => {
      if (device.kind === "audioInput"){
        let opt = document.createElement('vwc-list-item');
        opt.value = device.deviceId;
        opt.innerHTML = device.label;
        audio.appendChild(opt);
      }
      else if (device.kind === "videoInput"){
        let opt = document.createElement('vwc-list-item');
        opt.value = device.deviceId;
        opt.innerHTML = device.label;
        video.appendChild(opt);
      }
      else{
        console.log("Error in retrieveDevices");
      }
    })
  })
}


// listen for changes to selected audio/video inputs
// update room when inputs are changed
let listenInputChange = (target) => {
  const targetSelect = document.querySelector(`${target}`);
  targetSelect.addEventListener('change', (inputOption) => {
    if (target === "vwc-select#audio-input"){
      room.camera.setAudioDevice(inputOption.target.value);
    }
    else if (target === "vwc-select#video-input"){
      room.camera.setVideoDevice(inputOption.target.value);
    }
    else{
      console.log("Error in listenInputChange");
    }
  })
}


// Retrieve lists of auidoOutput
// Add audioOutputs to select menu
// On user select new option, update audio input
async function audioOutputs() {
  var audioOutputs = await VideoExpress.getAudioOutputDevices();
  const audioOutputSelect = document.querySelector('vwc-select#audio-output');
  audioOutputs.forEach(output => {
    let opt = document.createElement('vwc-list-item');
    opt.value = output.deviceId;
    opt.innerHTML = output.label;
    audioOutputSelect.appendChild(opt);
  })

  audioOutputSelect.addEventListener('change', (audioOutputOption) => {
    VideoExpress.setAudioOutputDevice(audioOutputOption.target.value);
  });
}


// toggle tooltips on hover
let addToolTipListeners = (toolTipsToListen) => {
  toolTipsToListen.forEach(toolTipToListen => {
    const target = document.querySelector(`#${toolTipToListen.targetId}`);
    const targetToolTip = document.querySelector(`#${toolTipToListen.toolTipId}`);
    target.addEventListener('mouseover', (event) => targetToolTip.open = !targetToolTip.open);
    target.addEventListener('mouseout', (event) => targetToolTip.open = !targetToolTip.open);
  })
}


// define all our DOM elements which we'll want to listen to
const mute_all_btn = document.querySelector('#mute-all');
const unmute_all_btn = document.querySelector('#unmute-all');

const mute_self_btn = document.querySelector('#mute-self');
const unmute_self_btn = document.querySelector('#unmute-self');

const hide_self_btn = document.querySelector('#hide-self');
const unhide_self_btn = document.querySelector('#unhide-self');

const toolTipsToListen = [
  {targetId: "hide-self", toolTipId:"hide-self-tooltip"},
  {targetId: "unhide-self", toolTipId: "unhide-self-tooltip"},
  {targetId: "mute-self", toolTipId: "mute-self-tooltip"},
  {targetId: "unmute-self", toolTipId: "unmute-self-tooltip"},
  {targetId: "mute-all", toolTipId: "mute-all-tooltip"},
  {targetId: "unmute-all", toolTipId: "unmute-all-tooltip"},
  {targetId: "audio-output-target", toolTipId: "audio-output-tooltip"},
]


// Call all of our functions!
toggleMuteAllButton(mute_all_btn, "mute", room.participants);
toggleMuteAllButton(unmute_all_btn, "unmute", room.participants);
listenForToggle(room.camera.isVideoEnabled, hide_self_btn, unhide_self_btn, room.camera.disableVideo, room.camera.enableVideo);
listenForToggle(room.camera.isAudioEnabled, mute_self_btn, unmute_self_btn, room.camera.disableAudio, room.camera.enableAudio);
getDeviceInputs("vwc-select#audio-input", "vwc-select#video-input");
listenInputChange("vwc-select#audio-input");
listenInputChange("vwc-select#video-input");
audioOutputs();
addToolTipListeners(toolTipsToListen);
```

## Finale

And....that's it! With a little bit of Ruby and a bit more Javascript manipulation, you have a full video conferencing application.

We can run the app and try it out with some friends!

### Running ngrok

The easiest way is with ngrok. [Install ngrok](https://developer.vonage.com/tools/ngrok) if you don't have it.

Then use ngrok to run a publicly accessible server. From the command line run:
`ngrok http 3000`

In your terminal you'll see a window that looks like this:

![ngrok server screenshot](/content/blog/vonage-video-express-with-ruby-on-rails-part-2/screen-shot-2022-07-01-at-16.32.41.png "ngrok server screenshot")

You'll want to copy the line that ends in ngrok.io. This is the temporarily accessible URL that ngrok will forward your Rails server to. We need Rails to give permission to ngrok to be a host. In our `config/environments/development.rb` file, inside the `Rails.application.configure do` we need to add the following line:

`config.hosts << "[ngrok url]"`

For example, in the above instance of running ngrok I would add:

`config.hosts << "c3b9-146-185-57-50.ngrok.io"`

Now we can run our rails server:

`rails s`

And now our app is up and running and you can invite friends to join your watch party by sending them the ngrok URL. And the password!

![GIF Preview Of Finished Video Conferencing App Built With Vonage Video Express](https://s3.eu-west-1.amazonaws.com/developer.vonage.com/blog/blogposts/vonage-video-express-with-ruby-on-rails-part-1/ezgif.com-gif.gif "GIF Preview Of Finished Video Conferencing App Built With Vonage Video Express")

### Get In Touch

How did you like this tutorial? Have questions or feedback about Video Express or Vivid? Join us on the [Vonage Developer Slack](https://developer.vonage.com/community/slack). And follow us on [Twitter](https://twitter.com/VonageDev) to keep up with the latest Vonage Developer updates.
