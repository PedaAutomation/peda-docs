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

First, the Slave has to identify itself with a name. This name will be known to all the other slaves and can also be used in commands:
```
{ 
  "message": "name",
  "data": "Livingroom"
}
```

After that is done, the server has to know the slave's capabilities. 

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


