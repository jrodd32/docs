---
nav-title: "NativeScript Events"
title: "Events"
description: "NativeScript Documentation: Events"
position: 12
---

#Events

##Overview

Event is a message sent from the event emitter for the occurrence of a specific action. This action could be generated by user action (such as finger tap), or by some program logic (indicates that downloading an image from a server is completed). The object that raises event is called **event sender** or just **sender**. The object that consumes the event is called **event listener** or just **listener**. Being a TypeScript (JavaScript) framework NativeScript cannot benefit from any language built-in event handling mechanism. Since events are very critical for any modern mobile application development NativeScript provides a class that powers the process of working with events. This class is called Observable ([API-Ref](./ApiReference/data/observable/observable.md)).

##How events work

Generally events are very similar to a radio station: **sender** plays music (fires an event) and **listener** dances (executes an action). As mentioned, NativeScript streamlines the events task with a class called Observable. It is one of the very basic classes within NativeScript framework so almost every NativeScript object (component) has an option for dealing with events.

##Adding an event listener

Adding an **event listener** means setting a function (method) which should be executed when event is raised. The example below shows how a function that prints a "Hello World!" message on the console is set to be executed when **button** is tapped.

* Adding an event handler with a short-hand syntax

This syntax is used to write an in-line function which should be executed when the **testButton** is tapped.

``` JavaScript
var buttonModule = require("ui/button");
var testButton = new buttonModule.Button();
testButton.text = "Test";

testButton.on(buttonModule.knownEvents.tap, function (eventData) {
  console.log("Hello World!");
});
```
``` TypeScript
import buttonModule = require("ui/button");
var testButton = new buttonModule.Button();
testButton.text = "Test";

testButton.on(buttonModule.knownEvents.tap, function (eventData) {
  console.log("Hello World!");
  });
```

Even though this short-hand syntax is very handy to assign some very simple functions it is missing some important features available in the original syntax:

* Adding an event handler

``` JavaScript
var buttonModule = require("ui/button");
var testButton = new buttonModule.Button();
testButton.text = "Test";

var onTap = function (eventData) {
  console.log("Hello World!");
};

testButton.addEventListener(buttonModule.knownEvents.tap, onTap, this);
```
``` TypeScript
import buttonModule = require("ui/button");
var testButton = new buttonModule.Button();
testButton.text = "Test";

var onTap = function (eventData) {
  console.log("Hello World!");
};

testButton.addEventListener(buttonModule.knownEvents.tap, onTap, this);
```
The only difference here is the third optional parameter which represents the **this** argument which will be used as **event handler** context (closure). In other words, if you need a **this** argument in your event handler function you have to use the long syntax, otherwise the **sender** object is used as the **this**.

* Adding an event handler with an xml declaration

Another option to set an event handler is via an xml declaration.

``` XML
<Page>
  <StackPanel>
    <Button tap="onTap" />
  </StackPanel>
</Page>
```
Of course we need a code behind file in order to write the body of the function. (Code behind file has same name as **xml** with **js** or **ts** extension).

``` JavaScript
function onTap(eventData) {
  console.log("Hello World!");
}
exports.listViewItemTap = listViewItemTap;
```
``` TypeScript
export function onTap(eventData) {
  console.log("Hello World!");
}
```

##Removing an event listener

Usually there is no need to remove the **event listener**. However there are some scenarios where we need to receive the event just once, or just remove the event listener in order to free resources.

* Remove an event listener with a short-hand syntax

``` JavaScript
testButton.off(buttonModule.knownEvents.tap);
```
``` TypeScript
testButton.off(buttonModule.knownEvents.tap);
```
This will remove all listeners for the tap event of the testButton instance. If there are more than one **event listener** objects, a second parameter with the name of the callback function could be set so that only the referenced event listener will be removed.

* Remove an event listener, long syntax

``` JavaScript
testButton.removeEventListener(buttonModule.knownEvents.tap, onTap);
```
``` TypeScript
testButton.removeEventListener(buttonModule.knownEvents.tap, onTap);
```
The difference between short and long syntax is the option of the long syntax to specify the **this** argument (which was used to add the event listener) if there are many event listeners with different **this** arguments.

> There is no syntax to remove an event listener via an xml declaration.

##PropertyChange event

Observable class has a built-in event called **propertyChange** which is called when any property has been changed. Subscribing to this event is very similar to previous examples.

``` JavaScript
var observableModule = require("data/observable");
var observableObject = new observableModule.Observable();

observableObject.on(observableModule.knownEvents.propertyChange, function(propertyChangeData){
  console.log(propertyChangeData.propertyName + " has been changed and new value is: " + propertyChangeData.value);
});
```
``` TypeScript
import observableModule = require("data/observable");
var observableObject = new observableModule.Observable();

observableObject.on(observableModule.knownEvents.propertyChange, function(propertyChangeData){
  console.log(propertyChangeData.propertyName + " has been changed and new value is: " + propertyChangeData.value);
});
```
What is important about this event is that it is very critical for the entire [data binding](./bindings.md) system to work. In order to take advantage of the data binding mechanism it is enough to simply make your business object **inherit** of the Observable class. The following example demonstrates how to do that:

``` JavaScript
var observableModule = require("data/observable");
var MyClass = (function (_super) {
  __extends(MyClass, _super);
  function MyClass() {
    _super.apply(this, arguments);
  }
  Object.defineProperty(MyClass.prototype, "myProperty", {
    get: function () {
      return this.get("myProperty");
      },
      set: function (value) {
        this.set("myProperty", value);
      },
      enumerable: true,
      configurable: true
    });
    return MyClass;
  })(observableModule.Observable);
exports.MyClass = MyClass;
```
``` TypeScript
import observableModule = require("data/observable");

export class MyClass extends observableModule.Observable {
  public get myProperty(): number {
    return this.get("myProperty");
  }

  public set myProperty(value: number) {
    this.set("myProperty", value);
  }
}
```
This code snippet will fire the **propertyChange** event when the value of the property is changed out-of-the-box.

##Creating a custom event

Using Observable, methods **get** and **set** works just great for the **propertyChange** event. In many cases, according to some business logic, there is a need to create a custom event. To fire (raise or emit) an event all we have to do is to Observable.notify() method with proper event data when the action is completed. By proper event data we refer to any **implementer** of the [EventData interface](./ApiReference/data/observable/EventData.md). This is the very basic information about an event (its name as **eventName** and instance of the **event sender** as **object**).

``` JavaScript
var eventData = {
  eventName: "myCustomEventName",
  object: this
};
this.notify(eventData);
```
``` TypeScript
var eventData: observableModule.EventData = {
  eventName: "myCustomEventName",
  object: this
}
this.notify(eventData);
```
The bare minimum needed to raise an event is the eventName - it will be used to execute all **event handlers** associated with this event.

Next step is to hook for this event.

``` JavaScript
var myCustomObject = new MyClass();
myCustomObject.on("myCustomEventName", function(eventData){
  console.log(eventData.eventName + " has been raised! by: " + eventData.object);
})
```
Almost the same logic is implemented for the **propertyChange** event, so if your business logic has some special requirements **propertyChange** could be emitted manually via **notify()** method (without using Observable.set() method which also fires the **propertyChange** event).

##Avoiding memory leaks

Though the radio station comparison is convenient for understanding the concept, the situation is a bit more complicated in the inside. In order to be able to notify the listener, the sender has a pointer to the listener. This means that even if that listener object gets set to `null` or `undefined`, it is not eligible for garbage collection, since the sender is alive and has a live reference to the listener object. This could result in a memory leak in some scenarios when object lifetimes of the sender and the listener are quite different. Here's an example scenario: a UI element creates a lot of child controls, where every control hooks for an event of the parent and then a child control gets released (very similar to list view scrolling). In order to prevent such memory leaks it is a good practice to remove your event listener handler before releasing the listener object. Unfortunately in some cases we could define the exact time when we should call (off or removeEventListener) functions. Here comes another option in NativeScript framework **weakEvents**.

##Working with weak events

* Adding a weak event listener

Weak events as its name suggests creates an weak reference to the **listener** object, which allows this listener object to be released without removing the **event listener** pointer. Using weak event listeners is very similar to *normal* events. Here is the example with comments for each line.

``` JavaScript
var weakEventListenerModule = require("ui/core/weakEventListener");
var buttonModule = require("ui/button");
var observableModule = require("data/observable");

var testButton = new buttonModule.Button();
testButton.text = "Test";
testButton.on(buttonModule.knownEvents.tap, function () {
  source.set("testProperty", "change" + counter);
  });

  var source = new observableModule.Observable();

  var counter = 0;
  var handlePropertyChange = function () {
    counter++;
    this.text = counter + "";
  };

  var weakEL = weakEventListenerModule.WeakEventListener;
  var weakEventListenerOptions: weakEventListenerModule.WeakEventListenerOptions = {
    // a weak reference of the event listener object
    targetWeakRef: new WeakRef(this),
    // a weak reference if the event sender object
    sourceWeakRef: new WeakRef(this.source),
    // the name of the event
    eventName: observable.knownEvents.propertyChange,
    // the handler of the event
    handler: handlePropertyChange,
    // the context which will be used to execute the handler (optional)
    handlerContext: testButton,
    // a special property which is used for a extra event recognition (optional)
    key: this.options.targetProperty
  }
  weakEL.addWeakEventListener(this.weakEventListenerOptions);
```
``` TypeScript
import weakEventListenerModule = require("ui/core/weakEventListener");
import buttonModule = require("ui/button");
import observableModule = require("data/observable");

var testButton = new buttonModule.Button();
testButton.text = "Test";
testButton.on(buttonModule.knownEvents.tap, function () {
  source.set("testProperty", "change" + counter);
});

var source = new observableModule.Observable();

var counter = 0;
var handlePropertyChange = function () {
  counter++;
  this.text = counter + "";
};

var weakEL = weakEventListenerModule.WeakEventListener;
var weakEventListenerOptions: weakEventListenerModule.WeakEventListenerOptions = {
  // a weak reference of the event listener object
  targetWeakRef: new WeakRef(this),
  // a weak reference if the event sender object
  sourceWeakRef: new WeakRef(this.source),
  // the name of the event
  eventName: observable.knownEvents.propertyChange,
  // the handler of the event
  handler: handlePropertyChange,
  // the context which will be used to execute the handler (optional)
  handlerContext: testButton,
  // a special property which is used for a extra event recognition (optional)
  key: this.options.targetProperty
}
weakEL.addWeakEventListener(this.weakEventListenerOptions);
```

Previous example shows how to attach a **weak event listener** to an observable object instance. A closer look to the **handlePropertyChange** function shows that **text** property of the **this** object is changed when the **propertyChange** event is raised (via the button tap event). It demonstrates how to use the **handlerContext** property - the value of this property is taken as a **this** argument inside the **event handler** function.
Generally **targetWeakRef** and **key** properties are not needed to invoke a function when an event is raised. However these properties support another useful option of the weak event listener (a possibility to remove an event listener) both properties are used as keys for a key-value pair that stores weak event listeners.

* Removing weak event listener

``` JavaScript
weakEL.removeWeakEventListener(this.weakEventListenerOptions);
```
``` TypeScript
weakEL.removeWeakEventListener(this.weakEventListenerOptions);
```
