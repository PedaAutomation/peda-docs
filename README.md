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
The slave can also add additional parameters to the data object which will get send to the logic parsing the command.



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


