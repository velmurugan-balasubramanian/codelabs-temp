summary: This tutorial introduces you to the Instance methods available in the Freshworks developer platform, that helps you build data driven application that shares data among different app locations and instances. 
id: interface-methods
categories:freshdesk
tags:[instance, freshdesk]
status:Published
authors:Velmurugan Balasubramanian
Feedback Link: https://developers.freshworks.com

# Build data driven Apps using Instance methods

## Overview

### Instance method

In the Freshworks developer platform, the same app can render in multiple placeholders, though the app is the same in terms of the functionality and codebase, they are considered as different instances of the app.

Although all the instances belong to the same app, the instances do not share data or state, hence the client object in the freshworks developer platform exposes interface methods to share data and state among the different instances of the same app. 


The Instance methods can be used to achieve any of the following usecases,
* Resize an Instance (if the instance supports resizing, not all the instances can be resized)
* Close an Instance of the app (Close a modal for example)
* Get the context of the current instance of the app (for eg, the app's current location)
* Send data from one instance to another instance of the app
* Receive the data sent to the current instance of the app.


## Get Started

### What we’ll build today

In this tutorial we will build an application that explores all the functionalities of the most commonly used instance methods. 

### Prerequisites
*  Familiarity with Freshworks app development.
*  Latest version of the FDK installed locally. 
*  Knowledge of the Interface methods, if you are not familiar with the interface methods, please visit [this](/codelabs/introduction-to-interface-methods/index.html) tutorial. 
### Set up the boilerplate

To get started with this tutorial, let's clone the boilerplate app from the following repo.

```bash
git clone https://github.com/freshworks-developers/instance-method_freshdesk.git
```

The cloned repository contains two folders /Start and /completed.
	
You can navigate to the /completed folder to check how the finished app works.  

To continue with the tutorial navigate to the /start folder and open it on your favourite code editor.

## Resize and Close an Instance of the app

### Resize

To resize the current instance of the app, copy the following code snippet to the `resizeInstance(){..}` function.

```javascript
 client.instance.resize({ height: "400px" });
```

The above code snippet resizes the height of the app instance to 400px, let’s test it out.

![Resize instance](assets/resize-instance.gif)

From the above GIF we can see that the height of the app instance is increased to 400px from the default height.

### Close

The close instance method is used to close the instance that was opened using the interface method. In this section, we’ll open a modal using the interface method and close it programmatically using the instance method. 

Negative
: An instance method can be used to close the interface method, but it is not mandatory that all the interface methods should be closed programatically, If there is no need to close the interface method programatically, you can just close it through the ‘X’ in  the interface.

Copy the following snippet and paste it in the `closeModal(){..}` function of `app/scripts/modal.js` file. 

```javascript
// Snippet to close the modal
client.instance.close();
```

The above snippet closes the modal when the `Close this modal` button is clicked. Let’s test out the functionality. 

![Close Modal Using instance method](assets/close-modal.gif)

From the above GIF you can see that the modal closes successfully when the `Close this Modal` button is clicked on the modal.

## Send Data from parent to a modal

In a production application, it’s very important to share data between the instances of the app to avoid saving duplicate data or making duplicate API calls/requests, In the following sections we’ll learn about sharing data between parent and child instances.

### Send data to the Child(Modal) instance from Parent instance

We already have a modal in place, from our previous steps, let's send a sample data to the modal using the existing interface method.

Replace the existing interface method in the `showModal(){..}` function in the `app/scripts/app.js` file, with the following code snippet.

```javascript
// Interface method with data object, to pass it to the modal.
client.interface.trigger("showModal", {
   title: "A Modal to be closed",
   template: "modal.html",
   data:{
     name:' Steve rogers',
     Designation: 'Captain America'
   }
 }).then(function (data) {
   console.info(data);
 }).catch(function (error) {
   console.error(error);
 });
```

In the above snippet we are sending the object `{ name:' Steve rogers', Designation: 'Captain America'}` to the modal.

### Receive the data sent by Parent instance in the Child instance(Modal)

let’s fetch the data sent by the parent to the modal by Copy, Pasting the following code snippet inside the `fetchData(){..}` function in the `app/scripts/modal.js` file.

```javascript
 // Interface method to fetch data from parent
 client.instance.context().then(
   function(context)  {
        console.log(context.data);
   }
 );
```

Now that we have the logic in place, let’s check if our code snippet works as intended.

![Send to Modal](assets/send-data-to-modal.gif)

From the above GIF we can see that the data is being fetched and printed in the browser console successfully in the modal sent from the parent.

## Send Data from the Child Instance to the Parent Instance

Now that we have sent data from the ticket_sidebar app to the modal, let’s send some data back to the ticket_sidebar app from the modal.

### Send Data back to Parent Instance

To send data from the modal, copy the following snippet to `sendFromModal(){..}` inside `app/scripts/modal.js`

```javascript
 // Instance method to send data from modal to parent.
 client.instance.send({
   /* message can be a string, object, or array */
     message: 'Info received about captain America'
 });
```

The above snippet sends data from the modal to the parent location, the parent instance is the one from which the modal was opened in this case the `ticket_sidebar` app location.

### Receive the data sent by Modal in the ticket_sidebar location

To receive the data sent from the modal, copy the following snippet and paste it into the `fetchFromModal(){..}` in the `apps/scripts/app.js`

```javascript
// Instance method to get data from the modal
 client.instance.receive(
   function (event) {
     var data = event.helper.getData();
     console.info('Data from Modal')
     console.info(data.message);;
   }
 );
```

The above code snippet intercepts the data sent from the modal and prints it in the browser console.

Let’s test the functionality and see if it works as intended.

![Send Data from Modal to Parent](assets/modalToParent.gif)

You can see from the GIF above that the message from the modal is printed in the browser console.

## Send Data from one app location to another app location

Data can be transferred from location to another location by using `send()` and `receive()` instance methods. In the following example, we’ll send data from the `ticket_sidebar` app location to `ticket_requester_info` location.

In order to send data to another location of the app, we need to find out the instance ID of the app instance, to which we wish to send data.

Now, to find the Instance ID of the `ticket_requester_info` location and send the data, let's copy paste the following code snippet inside the `sendToAnother(){...}` in the 'app/scripts/app.js`

```javascript
  // Instance method to send data to the ticket_requester_info app location.
 client.instance.get().then(
   function(data)  {
      let locationToSend = data.find(x => x.location === "ticket_requester_info");
     client.instance.send({
       message: 'Hi From ticket_sidebar location',
       receiver: locationToSend.instanceId
    });
  });
```

The above code snippet uses the instance methods to fetch the instance ID of the ticket_requester_info location and send a message to the location.

Once the message is sent, it’s time to receive the message and process it in the ticket_requester_info location. In order to do that let’s copy the following snippet to `receiveData(){...}` in `app/scripts/ticket_requester_template.js` file.

```javascript
 // Receive the message and display it in the html
 client.instance.receive(
   function (event) {
     let data = event.helper.getData();
     document.getElementById('message').innerHTML = ''
     document.getElementById('message').innerHTML = data.message
   });
```

Let us test out the functionality

![Send From One location to Another location](assets/location-location.gif)

From the above GIF, we can see that the initial message in the `ticket_requester_info` app location is changed from *No message from Ticket sidebar yet* to  *Hi From ticket_sidebar location* after sending the message from `ticket_sidebar` app location. 

## Recap

In this Tutorial we learned,

✅  How to Resize an Instance using the Instance method. 
✅  How to close a modal programatically using the Instance Method. 
✅  How to send data from the Parent Instance of the app to the Child Instance of the same app. 
✅  How to send data from the Child Instance to the parent Instance of the same app. 
✅  How to send data from one app location to another app location.

### What's next?

Now that you are familiar with the Interface methods, you can explore all the advanced features of the Interface methods through the [documentation](https://developer.freshdesk.com/v2/docs/instance-method/#) and build more advanced apps. 



