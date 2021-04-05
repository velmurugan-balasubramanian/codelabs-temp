summary: How does a few events features works in Serverless apps
id: deep-dive-into-serverless
categories: Intermediate
tags: freshdesk, serverless, backend-events
status: Published
authors: Freshworks

# Deep-dive into Serverless

<!-- ------------------------ -->

## Overview

Freshworks Serverless apps have a variety of features that can be used for different purposes. We are going to see how Product Events, Scheduled Events, and Server Method Invocation (SMI) works.

## What are we going to learn?

* [Product events](https://developers.freshdesk.com/v2/docs/product-events) - React to events from Freshdesk product
* [Scheduled events](https://developers.freshdesk.com/v2/docs/scheduled-events) - Schedule a function to execute at the set time
* [Server Method Invocation](https://developer.freshdesk.com/v2/docs/server-method-invocation) - Invoke a Serverless function from the frontend component of the app

## Prerequisites

* Knowledge of Serverless basics - Go through the tutorial

## Setup

Clone [this application repository](https://github.com/freshworks-developers/serverless-events-freshdesk) to get started with this tutorial,

```shell
git clone https://github.com/freshworks-developers/serverless-events-freshdesk.git

```

* Navigate to the branch, `start` to begin with the boilerplate application.
* The branch `finish` will have the complete code that can be referred to at the end of the tutorial or when stuck in any of the steps.

## Product Events

Product events are getting triggered whenever a specific event occurs in the Freshdesk product. If a specific event is registered with a function to execute when the event occurs, Serverless function will automatically be executed when the specific event occurs in the Freshdesk product. For example, a product event called onTicketCreate is triggered when a ticket is created in your Freshdesk account.

In `server.js` file under `/server` directory, the events are subscribed with a function that is exported to run upon event occurrence.

### Let’s register a product event

The syntax for subscribing for events can be noted on how `onTicketCreate` event and its respective function is mentioned in the `events` attribute of the export object in `server.js` file.

```shell
events: [
   { event: 'onTicketCreate', callback: 'onTicketCreateHandler' }
 ]
```

Let’s check the callback function code which only has a console log statement. We will develop further upon it to schedule a function whenever receiving an `onTicketCreate` event.

```shell
onTicketCreateHandler: function (args) {
   console.log('Hello');
 }
```

## Scheduled Events

This is a special feature available in Freshworks platform to schedule a function execution at a later point in time.

Sometimes, there will be a need to execute a function at a specific time or periodically at the same time. For example, send a reminder email at a specific time or generate an analytical report every day at a specific time. Scheduled events are a right choice for them.

### Let’s schedule

Scheduled events are registered similar to the product events. Add the following object where the events are subscribed in the `events` attribute of the exports object.

```shell
{ event: "onScheduledEvent", callback: "onScheduledEventHandler" }
```

Now the events object should look like it. Don’t forget to add the comma between two objects in the array.

```shell
events: [
   { event: 'onTicketCreate', callback: 'onTicketCreateHandler' },
   { event: "onScheduledEvent", callback: "onScheduledEventHandler" }
 ]
```

Let’s write a function, `onScheduledEventHandler` for this scheduled event to execute the function when triggered. Copy and paste the following function definition within the exports object.

```shell
onScheduledEventHandler: function (args) { $request.post(`https://${args.domain}/api/v2/tickets/${args.data.ticket_id}/notes`, {
  headers: {
    Authorization: "Basic <%= encode(iparam.api_key) %>"
  },
  json: {
    private: false,
    notify_emails: [args.iparams.notification_to],
    body: "Guess what! You forgot to reply to this ticket",
  }
   }).then(data => {
  console.info('Successfully added private note for reminder');
  console.info(data);
   }, error => {
  console.error('Error: Failed to add private note for reminder');
  console.error(JSON.stringify(error));
   });
 }
```

What are we doing in this function:

* We are making an API to Freshdesk to remind the watcher to look at the ticket.
* The reminder notes in the API contains the text, `Guess what! You forgot to reply to this ticket`.
* We have handled the error and logged to the console for troubleshooting when things go wrong in the production.

The function has been defined on what to do upon the execution of the scheduled event. But, this scheduled event has not been created yet.

### Let’s create the schedule

Let’s create the ticket reminder schedule in the `onTicketCreateHandler` function. Replace the content of the function with the following snippet. The `onTicketCreateHandler` function should enclose this code.

```shell
const newDate = new Date();
   $schedule.create({
  name: "ticket_reminder_" + args.data.ticket.id,
  data: { ticket_id: args.data.ticket.id},
  schedule_at: new Date(newDate.setMinutes(newDate.getMinutes() + 6)).toISOString(),
   }).then(function (data) {
    console.log('successfully scheduled the reminder')
    console.log(data);
  }, function (error) {
    console.error('Error: Failed to schedule the event');
    console.error(error);
  });
```

What are we doing in this function:

* We are creating a schedule with the `$schedule.create()` syntax.
* In the schedule creation option, the name of schedule provided as `ticket_reminder_123` (In this sample name, 123 is the ticket ID).
* Any data can be passed while creating the schedule to utilise within the respective function of the scheduled event. This data is passed in the `data` attribute of the options.
* The `schedule_at` option is passed with a time in ISO format. This time should be at least 5 minutes away from the schedule creation time. So, let’s add 6 minutes to the time when this schedule is created.

## Simulation

If you face issues during the simulation, refer to the code in the `finish` branch of the repository that has the complete code.

Let’s test out the schedule create process with the following steps:

1. Run the app locally by executing the `fdk run` command.
2. Fill the Installation Parameters by opening the Configurations page at [http://localhost:10001/custom_configs](http://localhost:10001/custom_configs)
3. To simulate the event triggers in the local environment, go to [http://localhost:10001/web/test](http://localhost:10001/web/test)
4. Select `onTicketCreate` event and verify or modify the parameters data to be passed to the event according to your testing criteria. The `id` attribute inside the `ticket` object is the only data interesting for us for the schedule name.
5. Click on the `Simulate` button and the event will be triggered to the app, relevant logs can be found in the Terminal/Command Prompt where the app is run.

The schedule has been created now if the logs exposed that the function executed in the positive flow.

Keep the app running. The function registered with the scheduled event will be triggered at the set time.

**Note:** If the app is published as a [Custom App](https://developer.freshdesk.com/v2/docs/custom-apps), the events can be triggered by creating a ticket in the Freshdesk portal and referring to [the documentation](https://developers.freshdesk.com/v2/docs/troubleshooting) to check the Serverless logs.

### How to verify the scheduled event creation?

1. Open the `localstore` file in the `/.fdk` directory in the root directory of the application.
2. If the scheduled event’s name is searched in the file, there will be an object containing the details about the schedule. This is where the scheduled events are stored to run by the server that runs the app.
3. After the execution of the scheduled event, this object with the information of the scheduled event will disappear from this file.

#### Let’s delete schedules when not necessary

When a Freshdesk agent wishes to delete the reminder schedule for a ticket, our app can enable them to do it. It is possible to delete the schedule before it’s execution with its name. But there’s a flaw to it.

#### Problem

An agent can only use the frontend app to do any actions and the schedule is accessible only from the Serverless application. There’s a solution for it.

#### Solution

There’s a feature called, Server Method Invocation. It is invoked from the frontend application. Let’s see how it can be implemented.

## Server Method Invocation

A Serverless function is defined in the regular way. But, it’s not registered to any event. This function can be called directly from the frontend application. So, when the frontend application calls this function, the function will be executed in the Serverless environment and response will be sent back to the frontend application. This is the flow of the Server Method Invocation (SMI).

Let’s try to understand it better by doing it. There’s a scenario where this scheduled reminder may not be needed. In this case, the scheduled event should be deleted before the execution. Let’s see how this case can be solved by using SMI.

Add the following function within the `export` object so that it can be called from the frontend application.

```shell
 deleteSchedule: function (args) {
   $schedule.delete({
     name: "ticket_reminder_" + args.ticket_id,
   }).then(function (data) {
     console.info('Successfully deleted schedule');
     console.info(data);
     renderData(null, data)
   }, function (error) {
     if (error.status === 404) {
       console.error('Schedule does not exist for the ticket:', args.data.ticket.id);
     } else {
       console.error('Error: Failed to delete the schedule');
     }
     console.error(error);
     renderData({ status: error.status, message: error.message });
   });
 }
```

What are we doing in this function:

* The schedule is deleted with `$schedule.delete()` syntax.
* The `ticket_id` is expected in the function arguments to find the name of the schedule for the particular ticket.
* When the schedule deletion succeeds or fails, the response is sent back to the frontend application with the `renderData(error, data)` syntax. This is a mandatory callback function for SMI. The error object should have status and message attributes and data is optional for error condition.
* We have handled the error and caught an error with 404 error code which would be thrown if the schedule is already been deleted. If the schedule is already executed, it would be deleted automatically.

The defined server method is called from the frontend with the following syntax. It can be found in the `app/app.js` file.

```shell
client.data.get("ticket").then(
  function (data) {
    client.request.invoke('deleteSchedule', { ticket_id: data.ticket.id })
  }
```

* The Server method has been invoked with the `client.request.invoke()` syntax.
* Though parameters for this invocation are optional, the ticket ID is fetched from the Data method and sent in the parameters to this server method. If you could remember from the server method, it was required to find the schedule’s name.

### Testing SMI

Let’s test the SMI with the following steps:

1. Run the app with `fdk run` command if not already running.
2. Log in to your Freshdesk account.
3. To the Freshdesk account URL, append `?dev=true`. Example URL: https://subdomain.freshdesk.com/a/tickets/1?dev=true
4. In the Tickets page of a ticket, the app is rendered on the right side of the page in one of the widgets.
5. Open the app by clicking on it’s name. The accordion will expand to load the content of the app.
6. Click the `Delete Schedule` button to call the server method.
7. If a schedule is created for this ticket, the respective reminder schedule will be deleted immediately.

#### How to verify the scheduled event deletion?

1. Open the `localstore` file in the `/.fdk` directory in the root directory of the application.
2. If the complete name of the scheduled event is searched in the file, there will be no result of occurrence of the scheduled event’s name as it has been removed. If there are any occurrences, the schedule has not been deleted properly.
3. Also, after the execution of the scheduled event, this object with the information of the scheduled event will disappear from this file.

## Wrap up

Voila! You have learnt a bunch of features in the Serverless environment of Freshworks Developer Platform. Refer to the complete code in the `finish` branch of the same code repository.

These are the areas which are covered in this tutorial:

1. [Product Events](https://developer.freshdesk.com/v2/docs/product-events): We have used onTicketCreate and onTicketUpdate events to decide when to create the schedule and when to remove if not necessary anymore.
2. [Scheduled Events](https://developer.freshdesk.com/v2/docs/scheduled-events): We have used it to add a reminder note to the ticket after 6 minutes.
3. [Server Method Invocation](https://developer.freshdesk.com/v2/docs/server-method-invocation): We have used it to delete the schedule from the frontend component of the application.
4. [Request Method](https://developer.freshdesk.com/v2/docs/request-method): We have additionally used it to make API for adding the note to a ticket.
5. [Data Method](https://developer.freshdesk.com/v2/docs/data-methods): We have used it to fetch the ticket ID in the frontend application.

## What next?

There are some of the Serverless features that are not covered in this tutorial. Go through the documentation and other resources to get your hands dirty with these areas to master the Freshworks Serverless apps.

* [App Setup Events](https://developers.freshdesk.com/v2/docs/app-setup-events) - They are similar to the life-cycle hooks and get triggered when the app is installed or uninstalled.
* [External Events](https://developers.freshdesk.com/v2/docs/external-events) - This will be handy when an app needs to listen to events from third-party applications by registering a webhook on the other platforms.
