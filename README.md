# Peda - Personal Entertainment Device for Automation

Peda is a house automation software written in node.js and optimized to run on multiple Raspberry Pis.

The basic architecture is a Master-Slave concept. Each slaves finds the master via Bonjour. When a slave logs onto the master a list of the slaves capabilities (e.g. controlling the tv, switching sockets on/off, etc.) has to be transmitted to the master. The communication takes place via WebSockets and json. The capabilities of each slave are organized as node-modules.

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
    * Output
      * peda-output-sockets
      * peda-output-multimedia
      * peda-output-video
      * peda-output-weather
      * peda-output-time
  * Plugin Development


