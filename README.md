# TagPro-EnhancedCommunicationPlatform
This script can be included (@require) in other userscripts to enable communication between balls with the script in realtime. It uses the [Realtime Framework](https://framework.realtime.co/messaging/)

## BIG FAT WARNING
Note that this communication is **NOT** encrypted in any way. **EVERYONE** could see **ALL** messages send by **ANYONE**.

## How to use this
First, you will need to include the script in your userscript. Do this by adding `// @require http://this/scripts/direct/link.js` to the metadata block.
You can than use all commands provided by the script.

## Commands
* <code>ecp.init( "<i>app</i>", <i>func</i> )</code> Initializing is required to use any of the ECP magic. *app* is an identifier for your userscript. Make sure to use a unique but clear name (in most cases just the name of your script). *func* is a function that is called every time a message is received. It should be defined before ecp.init() is called. For more info on the receive-funciton, read further.
* <code>ecp.send( "<i>to</i>", "<i>data</i>" )</code> This is the core function of ECP, which broadcasts a message. The *to* argument can be one of the **strings** in the table below, or a few of those separated by spaces. Include the quotes! The *data* obviously is the message that will be sent.

| to | description |
| --- | --- |
| `"game"` | Everyone in the current game (if you are in one) |
| <code>"game:<i>server</i>:<i>id</i>"</code> | Everyone in the game on a specific TP *server*, with this game *id* |
| `"team"` | Everyone on your team (if you are in one) |
| <code>"team:<i>name</i>"</code> | Everyone on the specified team (replace *color* by *red*, *blue*, *us* or *they*) |
| <code>"ball:<i>name</i>"</code> | The TagPro player with this *name* (See notes about names below) |
| `"server"` | Everyone on the same TP-server using ECP |
| <code>"server:<i>name</i>"</code> | Everyone on a specific TP server |
| `"app"`  | Everyone using your script (worldwide) |
| `"all"`  | Everyone using ECP (worldwide) |
| <code>"room:<i>name</i>"</code> | Everyone that has ecp.joined the room with this *name* |

* <code>ecp.join( "<i>room</i>" )</code> Join a room to opt in to messages sent to this room. You don't have to join a room to send a message in it.
* <code>ecp.leave( "<i>room</i>" )</code> Leave a room that you've previously ecp.joined
* <code>ecp.close( "<i>reasons</i>", <i>time</i> )</code> A formal way to disconnect from the platform. Other scripts can react to the special 'disconnect' message that they will receive if they join the special *"disconnects"* group. The *reason* could be one of the **strings** in the table below, or a few of those separated by spaces. It is also possible to make up your own. Leave a comment here if you think I should add one to this table. *time* is an optional integer estimate in minutes for how long you will be gone. When you do not send an ecp.close, but stop the userscript, an ecp.close will be sent with a blank reason after 30 seconds.

| reason | description |
| --- | --- |
| `"brb"` | When you go to close a window, drink a cup of coffee or whatever |
| `"mumble"` | When you leave, but will (still) be available on mumble  |
| `"stop"` | When you stop with the game, and don't plan on starting soon |
| <code>"server:<i>name</i>"</code>  | When you are going to another server with this *name* |
| <code>"game:<i>name</i>"</code> | When you will be playing another game with this *name* |
| `"sleep"`  | Good night |

* <code>ecp.allow( "<i>who/what</i>" )</code> Create a whitelist, and only allow these messages to be delivered to you. You can use the same **strings** as with the ecp.send command (excluding rooms). To combine them, seperate multiple strings by spaces. <code>ecp.allow( "game app" );</code> will only allow messages that originate from your app AND in your game. 
If you want to have more allowances, call ecp.allow() more than once. <code>ecp.allow( "ball:Ko" ); ecp.allow( "ball:LuckySpammer" );</code> will allow messages from both of us. However, <code>ecp.allow( "ball:Ko ball:LuckySpammer" );</code> would allow no messages, since a message always has just one origin. Rooms that you join to will always 

### The receive-function
The function you specified with the ecp.init command will receive one argument, a dictionary containing these properties.

| propertie | description |
| --- | --- |
| message | The message itself |
| app | The origin script name of this message |
| ball | The name of the user that sent it |
| server | The originating server |
| game | The originating game |
| team | The originating team |
| room | The room that this message was sent in (if any) |

### A note about ball names
When a name is verified, you can just use this verified name.
Unverified names are always followed by a colon and number. Example: When a ball with an unverified name *Ko* enters, his name will become *Ko:0*. The next unverified *Ko* that joins will be *Ko:1* etc. Intentional colons in names can be escaped with a colon. So a user called *CS:GO* will become *CS::GO* (or *CS::GO:0* when unverified). 

## Examples
* A simple honk script (The complete version will also be on this repo soon)
```javascript
// ==UserScript==
// @name      TagPro Honk ECP example
// @include   http://tagpro-*.koalabeast.com:*
// @require   http://this/scripts/direct/link.js    // This line is very important!
// ==/UserScript==

///// This script only needs to react to messages sent within the same game, and from this app
ecp.allow( "game app" );

///// We define the receive-function
function onReceive(data) {
    if ( data.message == "honk on" ) {
        honk(data.ball, true); }            // the definition of this function is excluded from this example
    elif ( data.message == "honk off" ) {
        honk(data.ball, false); }           // same
    }

///// Now let's set up listeners for the honk key (H, with char code 72)
///// These events trigger an ECP message being sent to everyone in the same game.
document.addEventListener("keydown", function keyDown(e) {
                                                if (e==72) { ecp.send( "game", "honk on" ) } } );
document.addEventListener("keyup", function keyUp(e) {
                                                if (e==72) { ecp.send( "game", "honk off" ) } } );

///// Almost done, we just need to initialize ECP.
ecp.init( "honkexample", onReceive );
