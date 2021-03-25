summary: Step by step tutorial to understand how to use Data Methods on Freshworks Developer Platform
id: data-methods-freshdesk
categories:foundations
tags:data-methods
status:Draft
authors:Saif
Feedback Link: mailto:devrel@freshworks.com

# Data Methods

## Overview

Duration: 2

### What we’ll learn today

- How to access Data Methods
- What are available pages where Data methods can be used
- What are the resources available for the app to consume via data methods.

### What we’ll need

1. A Freshdesk trial account on Modern web browser
2. Freshworks CLI
3. A code editor
4. Basic knowledge of HTML, CSS, Javascript, CLI and Browser DevTools
5. The Sample Code (See Get Set Up)

### What we’ll build

You will explore this feature by writing some simple code to capture a few events like user clicks in app placeholders and explore writing code yourself

### Prerequisite Knowledge

1. How users use [Freshdesk](https://freshdesk.com/features).
2. [Available placeholders](https://developers.freshdesk.com/v2/docs/app-locations/) for apps.
3. Have [walked through code](https://developers.freshdesk.com/v2/docs/your-first-app/#code_walkthrough) of an Freshworks App
4. Awareness of [App Lifecycle Methods](https://developers.freshdesk.com/v2/docs/your-first-app/#code_walkthrough)

## Get Set Up

Duration: 5

Get the sample code ready

```sh
git clone https://github.com/freshworks-developers/data-methods-freshdesk.git
```

Alternatively,

<button>[Download the Zip] (https://github.com/freshworks-developers/data-methods-freshdesk/archive/main.zip) </button>

### Verify

1. Open the cloned repository in the code editor with ‘start’ branch.
2. Open Terminal and run `> fdk run`
3. You should notice the app is being served at `localhost:10001`
4. Login to Freshdesk trial account and pay attention to four placeholders after appending `?dev=true` at every page. You should see an app icon available to activate the app.
   1. [Global Sidebar](https://developers.freshdesk.com/v2/docs/app-locations/#sidebar)
   2. [Ticket Details page](https://developers.freshdesk.com/v2/docs/app-locations/#ticket_details_page)
   3. [New Email page](https://developers.freshdesk.com/v2/docs/app-locations/#new_email_page)
   4. [Contact Details page](https://developers.freshdesk.com/v2/docs/app-locations/#contact_details_page)

Note: The Data Methods can be invoked specific to the placeholders. See [documentation](https://developers.freshdesk.com/v2/docs/data-methods/#) for specific details.

## Data Methods

Duration: 4

Data Methods allow your app to access information on a given Freshdesk page. In this tutorial you’ll learn how to consume data methods in your freshworks apps.

`client.data.get("<argument>")` returns a promise where your callback would have access to data in the page which user is currently viewing.

Let’s see this in action.

```sh
❯ fdk run
Starting local testing server at http://*:10001/
Append 'dev=true' to your Freshdesk account URL to start testing
e.g. https://domain.freshdesk.com/a/tickets/1?dev=true
Quit the server with Control-C.
```

Freshworks CLI serves your app to the browser on port `:10001`. In the Freshdesk account, append the URL with `?dev=true` to find apps running on the global sidebar at CTI app location.

## Global Arguments

Duration: 3

Data Methods return the data that the app needs based on the app placeholders. Mostly giving the contextual information pertaining to the current page. However, the following arguments can be passed to the Data method that would return the payload given in any placeholder.

### loggedInUser

The following is how you can consume it in your app

```js
client.data
  .get('loggedInUser')
  .then(function renderUserBioData(payload) {
    var {
      loggedInUser: {
        contact: { name: name, job_title: job, email: email }
      }
    } = payload;
    displayArea.insertAdjacentHTML(
      'afterbegin',
      `<p>This is information received from data method:</p>
          <ul>
          <li>Name: <mark>${name}</mark></li>
          <li>Job Title: <mark> ${job} </mark></li>
          <li>Email Address: <mark> ${email} </mark></li>
          </ul>
          `
    );
  })
  .catch(console.error);
```

Let’s discuss this in detail.

1. `client` is an object that is brought into scope for the developer by the platform. See [Line 6 in app/scripts/fullpage.js](https://github.com/freshworks-developers/data-methods-freshdesk/blob/dce8d7cb33e2f32012f6b44c5dd6b41e78cbe9c5/app/scripts/fullpage.js#L6) is attaching it to the [global scope](https://developer.mozilla.org/en-US/docs/Glossary/Global_scope) of the window.
2. `client.data.get()` is a method that can interface with Freshdesk (via platform) and get the information for the app to consume.
3. In the above code snippet, an argument of type string `loggedInUser` is passed to the `get()` method. It returns a [promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)along with a payload. In this case, a payload containing the information of `loggedInUser.`
4. This payload becomes available within the scope of the callback function that is passed. Here, the callback is `renderUserBioData(payload){..}`
5. `The renderUserBioData(payload){..} `function does two things. It is assigning the payload data to `name`, `job` and `email` variables. Also, rendering the same `name`, `job` and `email` information to the DOM. [See [object destructuring](https://javascript.info/destructuring-assignment#object-destructuring) if you find the assignment confusing.]
6. That’s how the payload passed can be consumed by the app.

**Note**: Always refer to the [documentation](https://developers.freshdesk.com/v2/docs/data-methods/#) for latest payload structure and attribute definitions.

### domainName

```js
client.data
  .get('domainName')
  .then(function whatsDomain({ domainName }) {
    displayArea.insertAdjacentHTML('afterend', `<br> The domain name is <mark>${domainName}</mark>`);
  })
  .catch(console.error);
```

This snippet simply fetches the host Freshdesk’s domain name for the app.

See the [current state of source code on global-args branch](https://github.com/freshworks-developers/data-methods-freshdesk/tree/global-args) of sample code.

## Page Specific Arguments

Duration: 4

Each page has a couple of [placeholders](https://developers.freshdesk.com/v2/docs/app-locations/) for the app. Just like Global Arguments can be passed to the data method in any page or app placeholder. However, data is provisioned (via data methods) to the app based on the page app is placed in.

For this tutorial, let’s look at Ticket details page,

### Ticket Details Page

Open `app/scripts/sidebar.js` and try adding some methods consuming different valid arguments ticket details page.

Within the [`renderSidebar(){..}`](https://github.com/freshworks-developers/data-methods-freshdesk/blob/dce8d7cb33e2f32012f6b44c5dd6b41e78cbe9c5/app/scripts/sidebar.js#L15),

```js
// ticket
client.data
  .get('ticket')
  .then(function getDetails({ ticket: { description_text: desc, priority: priority } }) {
    space.insertAdjacentHTML(
      'afterbegin',
      `<li><i>"ticket"</i> priority: <mark>${priority}</mark> : desc: <mark>${desc}</mark></li>`
    );
  })
  .catch(console.error);

//contact
client.data
  .get('contact')
  .then(function getDetails({ contact: { address: address, name: name } }) {
    space.insertAdjacentHTML(
      'afterbegin',
      `<li><i>"contact"</i> address: <mark>${address}</mark> name: <mark>${name}</mark></li>`
    );
  })
  .catch(console.error);

// email_config
client.data
  .get('email_config')
  .then(function getDetails(payload) {
    let supportEmail = payload.email_config[0].replyEmail;
    space.insertAdjacentHTML('afterbegin', `<li><i>"email_config"</i> : <mark>${supportEmail}<mark></li>`);
  })
  .catch(console.error);

//requester
client.data
  .get('requester')
  .then(function getDetails(payload) {
    let name = payload.requester.name;
    space.insertAdjacentHTML('afterbegin', `<li><i>"requester"</i> : <mark>${name}<mark></li>`);
  })
  .catch(console.error);

//requester
client.data
  .get('company')
  .then(function getDetails(payload) {
    let name = payload.company.name;
    space.insertAdjacentHTML('afterbegin', `<li><i>"company"</i> : <mark>${name}<mark></li>`);
  })
  .catch(console.error);

//requester
client.data
  .get('group')
  .then(function getDetails(payload) {
    let name = payload.group.name;
    space.insertAdjacentHTML('afterbegin', `<li><i>"group"</i> : <mark>${name}<mark></li>`);
  })
  .catch(console.error);

//requester
client.data
  .get('company')
  .then(function getDetails(payload) {
    let name = payload.company.name;
    space.insertAdjacentHTML('afterbegin', `<li><i>"company"</i> : <mark>${name}<mark></li>`);
  })
  .catch(console.error);

//status_options
client.data
  .get('status_options')
  .then(function getDetails(payload) {
    let opt = payload.status_options[0];
    space.insertAdjacentHTML('afterbegin', `<li><i>"status options"</i> First option: <mark>${opt}<mark></li>`);
  })
  .catch(console.error);

//time_entry
client.data
  .get('time_entry')
  .then(function getDetails(payload) {
    let isTimerRunning = payload.time_entry.time_entries[0].time_spent;
    space.insertAdjacentHTML('afterbegin', `<li><i>"time entry"</i> First option: <mark>${isTimerRunning}<mark></li>`);
  })
  .catch(console.error);
```

The result would be information rendered to the DOM informing of a list in the ticket sidebar returned from different data methods.

At this point you have consumed data methods from Global Arguments and Ticket Details Page specific arguments. Take some time to go through the [final source code at this moment](https://github.com/freshworks-developers/data-methods-freshdesk/tree/finish).

## Wrapping Up

Duration: 1

Bunch of other locations where Data Methods can be accessed are

1. Ticket Details Page: `ticket`, `contact`, `email_config`, `requester`, `company`, `group`, `<fieldName>_options`, `time_entry`
2. New Ticket Page: `<fieldName>_options`
3. New Email Page: `email_cofig`, `<fieldName>_options`
4. Contact Details Page: `contact`, `company`

Why not try writing code yourself for `requesterpage.js` and `contactpage.js `?

1. Try to understand the existing code and it’s associated html and css files.
2. Surf through documentation to see valid arguments at these placeholders.
3. Try picking up a few of those payloads and writing to the DOM.

Above steps will give you more confidence using the Data Methods.

Congratulations! 💐

You have come so far walking throughout the tutorial.
