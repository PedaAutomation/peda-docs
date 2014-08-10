# Peda - Personal Entertainment Device for Automation

Peda is a house automation software written in node.js and optimized to run on multiple Raspberry Pis.

The basic architecture is a Master-Slave concept. Each slave finds the master via Bonjour. When a slave logs onto the master a list of the slaves capabilities (e.g. controlling the tv, switching sockets on/off, etc.) has to be transmitted to the master. The communication takes place via WebSockets and json. The capabilities of each slave are organized as node-modules.

## Master/Slave-Communication
As noted before, Slaves find the Master via Bonjour. When they find the Master, they connect to it. Communication is served via WebSockets (because they are simple to use with Node.js). The Messages consist of simple JSON. The look like this:
```
{
  "message": "<MESSAGE>",
  "data": "<ADDITIONAL_JSON_DATA>"
}
```
The Slave connects to the Master. Then Communication begins.

### Init

First, the Slave has to identify itself with a name. This name will be known to all the other slaves and can also be used in commands:
```
{ 
  "message": "name",
  "data": "Livingroom"
}
```

After that is done, the server has to know the slave's capabilities. There are 3 different types of capabilities: input, logic and output. At the moment, those only consist of a name. Additionally the logic capability has a regex, so it can evaluate inputs:
```
{
  "message": "capabilities",
  "data": [
    {
      "type": "input",
      "name": "input-tts"
    },
    {
      "type": "input",
      "name": "input-keyboard"
    },
    {
      "type": "logic",
      "name": "logic-weather:weather",
      "regex": "/what is the weather like/g"
    },
    {
      "type": "logic",
      "name": "logic-sockets:turnon",
      "regex": "/turn on the tv/g"
    },
    {
      "type": "output",
      "name": "output-tts"
    },
    {
      "type": "output",
      "name": "output-display"
    }
  ]
}
```

After this message was sent, initialization is done. Now the slave and master are registered successfully and the fun can begin.

### Sending Input:
When the Slave wants to send input, he simply has to send an "input" message:
```
{
  "message": "input",
  "data": {
    "command": "<PARSED_INPUT>"
  }
}
```
The slave can also add additional parameters to the data object which will get sent to the logic parsing the command.

### Sending Output:
If one of the slaves logic-plugins wants to send output, an "outputForward" message has to get sent. You can target a specific slave and/or a specific output-plugin-type, or you can simply output to all available slaves and output types:

Broadcast:
```
{
  "message": "forwardOutput",
  "data": {
    "data": "<OUTPUT_DATA>"
  }
}
```

Target a specific Slave:
```
{
  "message": "forwardOutput",
  "data": {
    "data": "<OUTPUT_DATA>",
    "targetDevice": "<SLAVE_NAME>"
  }
}
```

Target a specific Output-Capability (`tts` in this case):
```
{
  "message": "forwardOutput",
  "data": {
    "data": "<OUTPUT_DATA>",
    "targetCapability": "tts"
  }
}
```

`targetCapability` and `targetDevice` can be mixed.

### Receiving Output:
When the Master thinks that this slave should output something, it sends an "output" message. It may send a targetCapability. In this case, the output should only be forwarded to that plugin.

```
{
  "message": "handleOutput",
  "data": {
    "data": "<OUTPUT>",
    "targetCapability": "tts"
}
```

### Handling Logic:
When the Master is of the opinion that the Slave should handle some Logic, it sends a "handleLogic" message:
```
{
  "message": "handleLogic",
  "data": {
    "command": "what is the weather like",
    "capability": "logic-weather:weather"
  }
}
```

## Writing plugins
Writing plugins is quite simple. They are just NPM-Modules. The Main file (specified in the modules package.json) has to look like this:

```
module.exports = function(slave) {
  slave.setType("input"); // or logic or output
  slave.setName("keyboard"); // The name of this plugin (keyboard, speech, etc.)
  // Insert stuff here
}
```

You have to decide what kind of plugin you want to write: Input, Output or Logic.
Input and Logic plugins are both really easy to implement:

** Input: **

```
module.exports = function(slave) {
  slave.setType("input");
  slave.setName("modernart");
  slave.sendInput("Modern Art is in the house"); // Send the command here. You probably want to call this in a callback or somewhere else, depending on your plugin.
}
```

** Output: **

```
module.exports = function(slave) {
  slave.setType("output");
  slave.setName("console");
  slave.on('output', function(data) {
  	var string = data.data;
    console.log(string);
    console.log("Language: " + data.language); // There is also a language parameter, useful for tts etc.
  });
}
```

** Logic: **

Logic-Plugins are a bit more difficult.
A Logic Plugin can listen for input, using regex. But because peda is made to be multi-language, it has to localize everything. Here's an example:

```
var germanData = {
  helloRegex: "/hallo/i",
  world: "Hallo Welt"
};

var englishData = {
  helloRegex: "/hello/i",
  world: "Hello World"
};

module.exports = function(slave) {
  slave.setType("logic");
  slave.setName("hello");
 
  
  slave.registerLanguage("en", englishData, true); // The true makes this language the fallback language.
  slave.registerLanguage("de", germanData);

  slave.registerLogic("helloworld" /* This is a unique identifier for the logic call */, slave.__("helloRegex") /* the regex from the translation", function(data, slave) {
    // data contains the whole input at "data.command".
    // for now, we'll just answer in a simple way
    slave.sendOutput(slave.__("world"));
    // You can also target a specific device and/or a specific output capability:
    // slave.sendOutputToSlave("hi", "Livingroom"); -> goes to the livingroom
    // slave.sendOutputToCapability("hi", "tts"); -> only output via tts
    // slave.sendOutputToCapabilityAndSlave("hi", "tts", "Livingroom"); -> output via tts only in livingroom
  });

}
```

That's the whole deal.

## TODO
* Setup
  * Hardware
  * Software
* Plugins/ Modules
  *  Basic Plugins
    * Input
      * peda-input-voice
      * peda-input-keyboard
      * peda-input-mobile
      * Other
    * Logic
      * peda-logic-weather
      * peda-logic-time
      * Other
    * Output
      * peda-output-tts
      * peda-output-display
      * Other
  * Plugin Development


