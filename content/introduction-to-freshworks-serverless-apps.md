summary: Freshworks apps can be built with a Serverless platform that will let you attach desired handler functions. These functions can run in a Serverless Node.js environment.
id: introduction-to-freshworks-serverless-apps
categories: Freshdesk, Serverless
tags: freshdesk, serverless, backend-events
status: Published
authors: Freshworks

# Introduction to Freshworks Serverless Apps

<!-- ------------------------ -->

## Overview

Freshworks apps can be built with a Serverless platform that will let you attach desired handler functions. These functions can run in a Serverless Node.js environment.

## What are we going to learn?

* Quick notes about Serverless in general
* Serverless apps in Freshworks platform
* Limitations and boundaries
* Testing Serverless events

## Quick notes about Serverless in general

Cloud simply means someone else’s computer. Similarly, Serverless means a server infrastructure maintained by someone else.

It’s a way of running code without worrying  about the underlying infrastructure or hardware. They are also priced based on usage. Isn’t both making the best pair to go with this model for the integration use-cases.

Serverless applications are headless applications that can run on it’s own when a specific function is invoked. Let’s see how it works in the Freshworks Developer Platform.

## Serverless apps in Freshworks Platform

Freshworks Developer Platform has provided a similar Serverless environment to address some specific use-case that cannot be addressed only with a frontend app or would be better to do it behind a secure wall. Freshworks Developer Platform implements this technology to the developers and their apps at no cost. This will let your apps react to desired events and solve problems elegantly.

Freshworks Serverless apps support only Node.js runtime. It is deployed as part of the Freshworks app that can also contain other components such as frontend and configurations page. So, no special steps or configurations required to publish serverless apps.

Let’s start with how the structure of the Serverless app would look like in the Freshworks platform.

### Skeleton of Serverless app

Create a serverless app in an empty folder.

```shell
fdk create --products freshdesk --template your_first_serverless_app
```

Navigate to the application root folder by running the following command.

```shell
cd your_first_serverless_app
```

The tree structure of a Serverless application would look like this. We will build the application on top of this application under the `server` directory for all the Serverless component changes.

```text
.
├── README.md
├── config
│   └── iparams.json
├── manifest.json
└── server
    └── server.js

3 directories, 4 files
```

The Serverless application contains,

* README.md - To document the code repository in markdown format.
* config/iparams.json - To configure the Installation parameters that will be filled by the account admin while installing the app.
* manifest.json - To capture the information about the app. It also acts as the configuration of the overall app.
* server/server.js - To write and expose the Serverless functions. All the serverless files that can be imported in this file will also go under the same directory.

With the information about the overall application, let’s jump to the server.js file to see how the events are registered with a function and how to actually write the Serverless function.

## Serverless events and functions

When a function needs to be invoked from outside the Serverless component, it needs to be exported. After exporting a function, it needs to be registered with a Serverless event that will be triggered in a specific case. For example, when a ticket is created in Freshdesk, an event is triggered which can be registered with a Serverless function. Whenever the same event occurs, the particular Serverless function is executed. These events are called Product events in Freshworks Developer Platform.

This function can have the business logic to execute upon the event trigger for ticket creation. Similarly, there are some other [Product Events](https://developer.freshdesk.com/v2/docs/product-events) available which will be triggered upon receiving respective events from the Freshdesk product.

```javascript
exports = {

  events: [
    { event: "onTicketCreate", callback: "onTicketCreateHandler" }
  ],

  // args is a JSON block containing the payload information.
  // args["iparam"] will contain the installation parameter values.
  onTicketCreateHandler: function(args) {
    console.log("Hello" + args["data"]["requester"]["name"]);
  }
};
```

Here, the `onTicketCreateHandler` function is executed when a ticket is created in the Freshdesk product.

* Some data about the ticket will be sent in the function arguments automatically for every product event.
* All the Serverless events will have some metadata passed along in the function arguments.
* The product events do not expect any response in return or have any callback function attached. So, execution stops with the custom business logic written in the function.

Freshworks platform has some boundaries with intention to make use of the Serverless functions effectively without abusing or misusing it.

Since these events get triggered after the event occurs in Freshdesk, there cannot be any response from the functions expected in the product. If there are errors, it has to be handled within this Serverless function. Let’s see how it can be done.

## Error handling

It is recommended to handle every error and logging to the console to troubleshoot the issues in the Serverless function logic. Unhandled errors will automatically result in the Serverless logs which can later be used for troubleshooting the function.

```shell
ReferenceError: variableA is not defined
 at Object.onTicketCreateHandler (server.js:137:51)
 at server.js:1:68
 at server.js:1:1549
 at Script.runInContext (vm.js:133:20)
 at ProductEvent.sandboxExecutor
```

An unhandled error is handled as shown in the above shown snippet. One of my functions expects variableA to be defined already which actually doesn’t exist or defined. This results in an error in the terminal along with the stack trace to help troubleshoot the issue.

The Serverless logs will not include only the errors from the app. It will also include any errors that could happen due to dependencies that you may import and use in these functions. Let’s see how.

## Dependencies

Serverless applications can also use dependencies from popular package manager, [NPM](http://npmjs.com). The dependencies are defined in the manifest.json file with the following syntax.

```json
"dependencies": {
  "underscore": "1.8.3"
}
```

Those dependencies that are defined will automatically be installed in the Serverless environment to import and use it in any function. The imported dependencies can be availed throughout the file where it is imported based on the scope.

```javascript
import * as underscore from 'underscore';

exports = {
 events: [
   { event: 'onTicketCreate', callback: 'onTicketCreateHandler' }
 ],
 onTicketCreateHandler: function(args) {
   const uniqueNumbers = underscore.uniq([1, 2, 1, 4, 1, 3]);
   console.log(uniqueNumbers); // Prints [1, 2, 3, 4]
 }
};
```

In this function, the underscore library from NPM has been imported and used to filter out the unique numbers from an array with duplicate numbers. Any package can be installed as a dependency in the Serverless app.

We have seen how the Serverless events can be registered, functions can be written and what features are available. Like any application, the Serverless functions can be tested individually just like how it will be run. Let’s see how the Serverless functions can be tested in the local environment as soon as the app development is started.

## Simulation of Serverless Events

Serverless events cannot be tested with the Freshdesk product when the app is run locally since the product lives in the cloud and the app is not exposed to the internet.

To test the events, we are going to use simulation to trigger the events that we would like to test. All the Serverless events can be tested in this way.

Let’s test the `onTicketCreate` product event with the following steps:

1. Run the app locally by executing `fdk run` command.
2. Go to the simulation page at [http://localhost:10001/web/test](http://localhost:10001/web/test).
3. Select `onTicketCreate` event. A payload data will be shown that will be sent in the parameter for this event handler function. This test data is stored under the `/server/test_data` directory of the app which can be modified in the file or in this page to receive the required data for your test scenario. This file is relevant only for simulating the event.
4. Click on the `Simulate` button. Now, the event will be triggered to the app, the log message from the event can be found in the Terminal or Command Prompt where the app is run.

Now, we have tested a particular event and the relevant function has been executed. When the app development is complete, there could be many events used and they are written in many functions or files for modular code structure. Some utility functions could be reused in multiple event handlers.

When the app grows complex, it would be difficult to track which of the areas of the app is tested and which are yet to be tested and whether a utility function is covered as part of the tested events.

Freshworks platform has got your back and track the test coverage when running the app locally. It will also provide a report summarising the test coverage. Let’s see it in detail.

### Test coverage

Test coverage helps to track and generate reports on which sections of the code is covered in the testing and which are not. This will help to cover all the sections of the code for a complete testing and a reliable application build.

* Now, go back to the terminal where the app is run and terminate the running app with the command, `Control` + `C`. The terminal will show the following test coverage report.
* The coverage report is available in the  /.reports directory in the app and the index HTML file can be opened in the browser to view the detailed report to see code coverage line by line.

When the app is published as a custom app in the Freshdesk portal, the event simulation is not required since the event will automatically trigger the installed apps. Let’s see how the Serverless event works in a custom app.

## Publish as a Custom app

Refer to [the documentation](https://developer.freshdesk.com/v2/docs/custom-apps) to publish and install a custom app in your Freshdesk portal using the same app that has been created.

Let’s make an action in Freshdesk that will trigger the onTicketCreate event in the installed Serverless app.

* Create a ticket in the Freshdesk portal by filling in the required fields. The requester creating the ticket will be logged to the Serverless logs.
* To check the Serverless logs for this event, find the steps by referring to [this wiki](https://community.developers.freshworks.com/t/how-to-obtain-serverless-logs/527) article.
* The requester/contact which is selected to create the ticket can be viewed in the Serverless logs of the app.
* For troubleshooting an issue, the caught errors can be logged to the console or the uncaught errors can be found in the Serverless logs.

When the business logic code runs in a Serverless app, it can only be triggered from the same installed Freshdesk account. Security is not a concern but a feature of the Serverless apps.

## Security

By default, the Serverless environment is secure since it is run on a private cloud environment where other services cannot reach without proper address and authentication for the function.

Also, it cannot be read from the frontend component of the application and exposed any information in the browser. With the same strength, all the installation parameters are sent to the Serverless functions in the arguments in plain text.

So, it is also a great option to run any secure operation in the Serverless function rather than in the frontend itself. For example, generating a secret token for external API services or encoding a particular text before sending it over a HTTP request.

## Recap

Voila! You have learnt how this new area of Serverless environment works in Freshworks Developer Platform and how it can be utilised best for specific use-cases.

We have covered the following topics:

* Serverless environment - How does this new pattern of applications work.
* Serverless apps in Freshworks platform - How the function is defined in a Freshworks serverless app, error is handled, dependencies are used, and tested with simulation.
* Publish as a custom app - How the Serverless apps are published as a custom app and how to troubleshoot.

To learn beyond, take a look at the [Serverless features](https://developer.freshdesk.com/v2/docs/overview) offered by the Freshworks developer platform.
