# Core
A really simple messaging and command dispatch centre.

## Usage guide

``npm i @mm83/core``

This library aims to provide a single, central point through which your whole application can communicate without breaking
data encapsulation, or any existing design pattern. It can be used to implement anything from a simple message dispatcher
to a full MVC system.

### API

#### Events

Events are messages which state that something has happened. They should be used to pass information to be acted on,
not as commands in themselves. Events use a near-identical API to the DOM, and the principle of dispatch/listen is the same.  

_NB If you wish to stop propagation of an event, make the handler return false._

``addEventListener(eventName, handler)``

Pass a function to be executed whenever an event happens. This function may be passed a relevant data object.

``addOnceEventListener(eventName, handler)``

As above, but the handler is automatically removed after a single execution. Handy for one-time loaders etc.

``removeEventListener(eventName, handler)``
``removeOnceEventListener(eventName, handler)``

Remove a handler which was assigned to a given event.

``listEventListeners(eventName)``

List any currently active events for an event. This appears in the console, and is primarily for debugging.

``dispatchEvent(eventName, data)``

Dispatch an event to the core. Any listeners or once-listeners which are expecting this will be executed. If a data object
is passed, it will be passed to each listener function. Be aware that, for speed, it is not a deep- or shallow- copy. You 
should read data from it, not modify it directly.

## Simple event example
```
Core.addEventListener("myEvent, (message)=>
{
  console.log("Event heard", message);
});

Core.dispatchEvent("myEvent", "Hello!");
```

#### Commands

Commands may be near-identical to events in how you set them up, but they are conceptually different. Events are, at
the simplest level, notifications about things which have happened. They can be acted on, but contain no instruction.
Commands, on the other hand, are explicit instructions. Their handlers are in charge of carrying these out, and dispatching
further commands/events, as necessary.

``addCommand(commandName, command)``

Pass a command function to be executed whenever invoked. This may seem identical to the eventListener, but there are
two differences - one in the code, one in how you should use it:

- You can only have a single command-per-commandName. You may have as many event listeners-per-event.
- Event listeners react to something which happens. Commands are the things that happen.

``removeCommand(commandName, command)``

Remove a command from the Core.

``exec(commandName, data)``

Execute a command, passing the handler a data object, if provided. Commands do not dispatch events upon execution, you should
call dispatchEvent at the end of your command method with an appropriate name/data object, if you wish anything to respond
to this.

``listAllCommands()``

List every command defined. Again, this appears in the console, and is meant for debugging.

## Simple command example
```
Core.addCommand("myCommand, (data)=>
{
  console.log("Command executed!");
  //Use an event to inform the application of successful completion
  Core.dispatchEvent("myCommandExecuted");
});

Core.exec("myCommand", myDataObject);
```

#### Queries

Queries are used when you want to pass data directly between parts of your application. A query is a simple request for
data - if any response handlers are bound to that query, they'll execute with the expectation that they'll return the
required information.

``query(queryName [, data])``

This method only requires a queryName. You may also pass data, which the response handlers will receive as their sole argument,
if required. This function will return:

- A single data object (whatever the response returned), if there's only on response bound to the query.
- An array of these, if more than one response was found.

``respond(queryName, handler)``

Set up a response to a query. The handler should return any data which is expected. If any data is passed with the query, it'll
receive it as arg0.

``respondOnce(queryName, handler)``

As above, though this response will be removed after it's been called for the first time.

``retract(queryName, handler)``

Remove a response from a query.

``retractOnce(queryName, handler)``

Remove a one-time response from a query.

## Simple query example
```
Core.respond("getAge", (name)=>
{
  switch(name)
  {
    case "Bob":
      return 50;
    case "Jane":
      return 20;
  }
});

let response = Core.query("getAge", "Bob");
```

#### Flags

Flags are simple, single values/variables which can be directly written and read arbitrarily. Whilst you can use any 
data type, they are ideally used to represent single, boolean values (eg is something loaded/ready?). You can also 
listen for changes to a particular flag.

``setFlag(flagName, value)``

Self-explanatory.

``getFlag(flagName,)``

Once again.

``setFlagListener(flagName, handler)``

Assign a handler to be activated when a flag value changes. The handler is sent the flag's new value as arg0.

``setOnceFlagListener(flagName, handler)``

As above, but the handler is removed from the listener list after its first execution.
