Repo created to store smartthings smartapps and device handlers I find. Also to play with and create my own

#Kodi Manager (Smartapp) & Kodi Client (Device Handler)
I wanted the ability to interact with and trigger events from kodi. Having looked round, there did not seem to be many options that did what I was looking to do. The closest was <a href="https://github.com/Toliver182/SmartThings-Kodi">Toliver182's</a>, bit doesnt seem to be in active development. So I forked it.

##Key Features

Communicate to and from Kodi
Get current state (Playing, Paused, Stopped, Shutdown, Startup)
Get current type - this is taken directly from Kodi
Get current category (Movie, TV Show, Sports)
All above configurable in preferences within the DTH

Interact with Kodi, both ways, to and from Smartthings. Trigger lights etc from events on Kodi i.e. When a 'Movie' plays turn on Movie Lights. Alternatively, when the doorbell rings, pause Kodi. One of the key things I wanted was to switch off my TV when I leave the house. This is why I added the shutdown capability, I have Kodi running on a RPi and set CEC apater to turn the TV off when Kodi shuts down, perfect. There is no startup capability, as once off there is no way to start up. So I use a smart outlet to re power the RPi, which fires up Kodi and the TV.
 NB if you are using a different system to run Kodi (i.e. media centre) you may be able to use WOL to wake it.
 
 ###Installation
 Kodi needs to be enabled for control over HTTP. So enable the following:  
 system>services>Web Server>Allow remote control via HTTP  
 **NB** You will need to know the port (required), username (optional) and password (optional) from this screen to set up Kodi Manager
 
 You will also need to set up the <a href="http://kodi.wiki/view/Add-on:Kodi_Callbacks">Callbacks</a> plugin. This is how Kodi talks to Smartthings, meaning you hget real time updates. Once installed you need to create the tasks for play, pause, stop, resume, shutdown, startup and then add them to the appropriate events (read the Wiki in the link for help)  
 **NB** You only need this for two way comms and you can install this after installing the smartthings app & device handler
 
 
 For current stable: 
 
 Add the following to your IDE:  
 Owner:     north3221  
 Name:      north3221SmartThings  
 Branch:    master  
 
 Manual (copy and past the code into your IDE):  
 Device Handler:    <a href="https://raw.githubusercontent.com/north3221/north3221SmartThings/master/devicetypes/north3221/kodi-client.src/kodi-client.groovy">Copy and paste this code into a new 'Device Handler'</a>  
 Smartapp:          <a href="https://raw.githubusercontent.com/north3221/north3221SmartThings/master/smartapps/north3221/kodi-manager-cbs.src/kodi-manager-cbs.groovy">Copy and paste this code into a new 'Smartapp'</a>
 
 For the Beta version add the following to your IDE:  
 Owner:     north3221  
 Name:      north3221SmartThings  
 Branch:    beta  
 
 Manual (copy and past the code into your IDE):  
 Device Handler:    <a href="ADD ONCE CREATED">Copy and paste this code into a new 'Device Handler'</a>  
 Smartapp:          <a href="ADD ONCE CREATED">Copy and paste this code into a new 'Smartapp'</a>
 

####Version 1.0

Initial release with a few bits added ontop of the forked version I took.  
I've also added some custom attributes:
```groovy
attribute "currentPlayingType", "string"
attribute "currentPlayingCategory", "enum", ["Movie", "TV Show", "Sports", "None", "Unknown"]
attribute "currentPlayingName", "string"
```

**currentPlayingType**: This is taken directly from Kodi 'type' on now playing metadata. So if its your library you are
playing, its likely it will know its 'Movie' or 'TV show'. However in other cases it will be 'unknown' so I added...

**currentPlayingCategory**: I have added some code to try and work out what is playing. It uses some logic in order to
try and work it out. This is because I want to be able to trigger different lights etc from different playing types.
```
Logic:
    1   -   Check the kodi 'type' if its set then it knows best, so 'movie' = 'Movie' and 'episode' = 'TV Show'
    2   -   Then if kodi type is 'unknown' then I try and work it out in this order (stop at first that matches):
            i   Movie = 'Movie Label' check kodi label for any of the words in Movie label
            ii  Sports = 'Sports Label' check kodi label for any of the words in Sports labels
            iii TV Show = 'TV Show label' check kodi label for any of the words in TV Show labels
            iv  Movie = 'Minimum Movie Runtime' check if the runtime is longer than min movie runtime
            v   TV Show = Runtime is > 0 but less than minimum movie runtime
            vi  Movie = a 'plot' has been added by kodi
```
**NB** The above labels and runtime is exposed in a device manager preference. So you can update the lists to your
  own liking. Please **BE AWARE** that the default values on a preference do not really exist - see the <a href="http://docs.smartthings.com/en/latest/device-type-developers-guide/device-preferences.html#additional-notes">Smartthings Docs</a> so if you
  update **ANY** of them then you have to update them all (you can update to the same thing obviously)

 **currentPlayingName**: Calculated current playing title

With these attributes please also be aware that kodi tells the device handler that its changed (from the call backs
 you configure) then the device handler makes a call to ask kodi what's
 playing, then works out what it is (using above logic). So this may take some time (dependant on networks and traffic I
 guess a few ms to a couple of secs). Therefore, don't trigger on a state change then expect to be able to read
 the attribute, it may not be updated yet, I think I can fix this as part of many improvements I want to make, but
 for now be careful.
 I have got round this by adding the custom attribute as the trigger in my CoRE Poston- NB you need to turn on
 'expert mode' in CoRE to do that.

**Custom Command - Shutdown**  
This allows you to call the shutdown command from Smartthings, which will shutdown Kodi



####Beta Release (Target Release 1.1)
I've done quite and overhaul to the both smartapp and device handler. I've updated all the tiles to create a media player and full Kodi control inside yoru smarthings mobile app and unlocked full capability for interaction with Kodi.

Because I made the app into a remote control, I've exposed all the commands it uses:
```groovy

command "shutdown"
command "up"
command "down"
command "left"
command "right"
command "back"
command "info"
```
So you can call any of these from Smartthings

The key one that I have added on top of this though is:
```groovy
command "executeAction" , ["string"]
```
This lets you send almost any command to Kodi. Here is a list of what you can send:

```"left", 
   "right", 
   "up", 
   "down", 
   "pageup", 
   "pagedown", 
   "select", 
   "highlight", 
   "parentdir", 
   "parentfolder", 
   "back", 
   "previousmenu", 
   "info", 
   "pause", 
   "stop", 
   "skipnext", 
   "skipprevious", 
   "fullscreen", 
   "aspectratio", 
   "stepforward", 
   "stepback", 
   "bigstepforward", 
   "bigstepback", 
   "osd", 
   "showsubtitles", 
   "nextsubtitle", 
   "codecinfo", 
   "nextpicture", 
   "previouspicture", 
   "zoomout", 
   "zoomin", 
   "playlist", 
   "queue", 
   "zoomnormal", 
   "zoomlevel1", 
   "zoomlevel2", 
   "zoomlevel3", 
   "zoomlevel4", 
   "zoomlevel5", 
   "zoomlevel6", 
   "zoomlevel7", 
   "zoomlevel8", 
   "zoomlevel9", 
   "nextcalibration", 
   "resetcalibration", 
   "analogmove", 
   "rotate", 
   "rotateccw", 
   "close", 
   "subtitledelayminus", 
   "subtitledelay", 
   "subtitledelayplus", 
   "audiodelayminus", 
   "audiodelay", 
   "audiodelayplus", 
   "subtitleshiftup", 
   "subtitleshiftdown", 
   "subtitlealign", 
   "audionextlanguage", 
   "verticalshiftup", 
   "verticalshiftdown", 
   "nextresolution", 
   "audiotoggledigital", 
   "number0", 
   "number1", 
   "number2", 
   "number3", 
   "number4", 
   "number5", 
   "number6", 
   "number7", 
   "number8", 
   "number9", 
   "osdleft", 
   "osdright", 
   "osdup", 
   "osddown", 
   "osdselect", 
   "osdvalueplus", 
   "osdvalueminus", 
   "smallstepback", 
   "fastforward", 
   "rewind", 
   "play", 
   "playpause", 
   "delete", 
   "copy", 
   "move", 
   "mplayerosd", 
   "hidesubmenu", 
   "screenshot", 
   "rename", 
   "togglewatched", 
   "scanitem", 
   "reloadkeymaps", 
   "volumeup", 
   "volumedown", 
   "mute", 
   "backspace", 
   "scrollup", 
   "scrolldown", 
   "analogfastforward", 
   "analogrewind", 
   "moveitemup", 
   "moveitemdown", 
   "contextmenu", 
   "shift", 
   "symbols", 
   "cursorleft", 
   "cursorright", 
   "showtime", 
   "analogseekforward", 
   "analogseekback", 
   "showpreset", 
   "presetlist", 
   "nextpreset", 
   "previouspreset", 
   "lockpreset", 
   "randompreset", 
   "increasevisrating", 
   "decreasevisrating", 
   "showvideomenu", 
   "enter", 
   "increaserating", 
   "decreaserating", 
   "togglefullscreen", 
   "nextscene", 
   "previousscene", 
   "nextletter", 
   "prevletter", 
   "jumpsms2", 
   "jumpsms3", 
   "jumpsms4", 
   "jumpsms5", 
   "jumpsms6", 
   "jumpsms7", 
   "jumpsms8", 
   "jumpsms9", 
   "filter", 
   "filterclear", 
   "filtersms2", 
   "filtersms3", 
   "filtersms4", 
   "filtersms5", 
   "filtersms6", 
   "filtersms7", 
   "filtersms8", 
   "filtersms9", 
   "firstpage", 
   "lastpage", 
   "guiprofile", 
   "red", 
   "green", 
   "yellow", 
   "blue", 
   "increasepar", 
   "decreasepar", 
   "volampup", 
   "volampdown", 
   "channelup", 
   "channeldown", 
   "previouschannelgroup", 
   "nextchannelgroup", 
   "leftclick", 
   "rightclick", 
   "middleclick", 
   "doubleclick", 
   "wheelup", 
   "wheeldown", 
   "mousedrag", 
   "mousemove", 
   "noop"
```