This presentation is available [here](https://presentations.generalassemb.ly/e9571e16e19a8b0e9504).

---

![](http://www.cnydev.org/wp-content/uploads/2015/03/socketio2.png)
# Realtime with Socket.io

---

## Learning Objectives
<br>

- <p>Add realtime communication between browser and server in a Node/Express application.</p>

---

## Roadmap
<br>

- Intro to _socket.io_

- Our Sample App: **`realtime-circles`**

- Config _socket.io_ on the Server & Client (browser)

- Display Circles in Realtime

- Clear the Display

- Deploy to Heroku

- Want More? Track Players!

---

### Intro to <span style="text-transform:lowercase">socket.io</span>
<br>

- The HTTP protocol does not enable bidirectional realtime communication.

- Fortunately, HTML5 included a new protocol that does - the __websocket__ protocol.

- **socket.io** is a JavaScript library that makes realtime bidirectional communication easier than working with websockets directly. 

---

### <span style="text-transform:lowercase">socket.io</span> - Basic Architecture
<br>

<img src="https://lh5.googleusercontent.com/unfpPe6OC4zzXxe89VXn0Sbmp5uQBifvTx6illIno-OofyFXm-PmMYXe5gGaokGLcu7VCJjB_koRspcneTHfjuMct9yhk_YiwX4XaLCY6O13vKzHGsQ0A8RkB_oYhzmrzFM" width="800">

- <p>Clients can send a messages to the server...</p>

- and the server can send messages to all connected clients.</p>

---

### What types of applications can you think of that can take advantage of realtime communications?

---

### Our Sample App: <span style="text-transform:lowercase">realtime-circles</span>


- Copy the `realtime-circles` starter code to your working directory.

- `? npm install`

- `? nodemon` and browse to `localhost:3000`

- Clicking creates a circle of random size and color.

- Our goal is to make this a realtime multi-player circle-fest!

- Let's review the starter code...

---

### Review the Code for <span style="text-transform:lowercase">realtime-circles</span>
<br>

- This is an Express app generated using `express-generator`.

- As a usual best practice in the land of the MEAN Stack,<br>`app.js` has been renamed to `server.js`.

- Examining `server.js` reveals that much of the default middleware has been removed. This is cool since we're not going to be using cookies, parsing the body for posted data, etc.

---

### Review the Code for <span style="text-transform:lowercase">realtime-circles</span>
<br>

- We only have one view - `index.ejs`. There will not be any full-page refreshes, therefore,<br>**this is considered to be what type of app?**

- Near the bottom of `index.ejs`, we are loading our app's JavaScript file - `app.js`...

---

### Review the Code for <span style="text-transform:lowercase">realtime-circles</span>
<br>

- Reviewing `app.js` reveals that we are using _native_ JavaScript for DOM manipulation. No _jQuery_ - we're going old school!

- `var circles` references a `<div>` that takes up most of the page.

- There's a click event listener on `circles`. This is where the action starts - let's take a look.

---

## Setting up <span style="text-transform:lowercase">socket.io</span>
<br>

#### Both the client and server<br>need to be configured with <span style="text-transform:lowercase">socket.io</span>

---

## Configuring the Server

---

#### Configure the Server
<br>

- To use _socket.io_, we first need to install its module:

	```sh
	? npm install --save socket.io
	```

- No, the dot in the name is not a typo, it's legit.

- There's going to be some server-side code pertaining to _socket.io_...

#### Should we put the _socket.io_ code<br>in our _server.js_ file?

---

#### Configure the Server (cont.)
<br>

- We don't want to unnecessarily clutter _server.js_, so we're going to put our _socket.io_ related code in a separate module file.

- Let's create a file named `io.js` in our project's root folder:

	```sh
	? touch io.js
	```

---

#### Configure the Server (cont.)
<br>

- _socket.io_, needs to "attach" to the _http server_, not the Express app.

- In an Express app scaffolded using `express-generator`, the _http server_ lives inside of the `/bin/www` file, so that is where we will require our new `io.js` module and attach to the _http server_:

	```js
	var server = http.createServer(app);
	
	// load and attach socket.io to http server
	var io = require('../io');
	io.attach(server);
	```

---

#### Configure the Server (cont.)
<br>

- Now we need to put some code in our `io.js` module. For now let's put some test code in it to make sure things are loading correctly:

	```js
	// io.js
	
	var io = require('socket.io')();
	
	io.on('connection', function (socket) {
	  console.log('Client connected to socket.io!');
  	});
	
	module.exports = io;
	```

---

#### Configure the Server (cont.)
<br>

- Check that `nodemon` is running our app without errors.

- No errors? Congrats the server is configured - time to configure the client!

---

## Configuring the Client<br><small>(Browser)</small>

---

#### Configure the Client
<br>

- It takes quite a bit of JavaScript in the browser to connect to _socket.io_ on the server and implement all of its goodness.

- Lucky for us, the _socket.io_ module on the server helps us out by creating a secret route that returns dynamically generated JavaScript for the client - hassle free!

---

#### Configure the Client (cont.)
<br>

- All we need to do is load this special client configuration script in our `index.ejs`:

	```html
		...
    	<script src="/socket.io/socket.io.js"></script>
    	<script src="/javascripts/app.js"></script>
    </body>
	```

- Be sure to load it before `app.js`.

- Refresh the browser and make sure there are no errors in the console.

---

#### Configure the Client (cont.)
<br>

- The `socket.io.js` client script exposes an `io` global function that we call to obtain our connection to the server.

- Let's call it and assign the returned connection object to a variable named `socket`.

	```js
	document.addEventListener("DOMContentLoaded", function() { 
  
  		// get our connection to the socket.io server
  		var socket = io();
  		console.log(socket);
  
  		...
	
	```     

---

#### Congrats, the client and server<br>have both been configured!
<br>

<p>But are we still error free?<br>Let's check...</p>

---

#### Test the Configuration
<br>

- Refresh the browser and verify that:<br><br>
  
  - The `socket` object logged in the browser's console has a<br>`connected: true` property.<br><br>
  
  - The server's terminal window logged out the message<br>"Client connected to socket.io!".

---

## Displaying Circles in Realtime

---

#### Our Realtime Requirements
<br>

- We are going to code along to transform the app into a realtime<br>multi-player circle-fest that:<br><br>

  -  Displays circles created by all players in realtime.<br><br>

  -  Clears all circles from all connected browsers when the `clear` button is clicked (a practice exercise).

---

#### Code Logic - Server
<br>

<p style="text-align:left">To accomplish our requirements, this is what we will need to do on the server:</p>

1. Listen for `add-circle` messages being sent from the clients.

2. When an `add-circle` message is received, emit (send) it (along with the data received with the message) to all connected clients (including the client that sent the message to begin with).

---

#### Code Logic - Client

<p style="text-align:left">To accomplish our requirements, this is what we will need to do on the client:</p>

1. Listen for `add-circle` messages from the server.<br>**Will the message have originated on the server?**

2. When the `add-circle` message is received, it will have an object with the properties necessary to pass to the existing `addCircle()` function that creates circles!

3. In the existing click handler, emit the `add-circle` message to the server, passing along an object containing `initials`, `x`, `y`, `dia` and `rgba` properties.

---

#### Messages 
<br>

- The `add-circle` message is a custom message that we "created" based upon what made sense for this application.

- **We can create as many different custom messages as we wish**.

- As already noted, each message can be emitted with data. The data can be any type except for a function. Objects and arrays come in handy for sending more than a single piece of primitive data.

---

### Displaying Circles - Server Code

- This code for `io.js` will accomplish the goal for our server's code logic:

	```js
	io.on('connection', function (socket) {
		//new code in here
   		socket.on('add-circle', function (data) {
      		io.emit('add-circle', data);
   		});
	});
	```

- When a client (`socket`) connects, we're setting up a listener for that client with the `on` method.

- When a client sends the server an `add-circle` message, the server (`io`) emits it to all clients using the `emit` method.

---

### Displaying Circles - Client Code
<br>

- Listen for an `add-circle` message from the server in `app.js`:

	```js
	var socket = io();	
  	// listen to the server for the `add-circle` event
  	socket.on('add-circle', function (data) {
   		console.log(data);
  	});
	```

- For now, we're simply going to log out the data received - baby steps :)

---

### Displaying Circles - Client Code (cont.)

- Now let's update the click event listener to emit an `add-circle` message with the data when user clicks:

	```js
  	circles.addEventListener('click', function(evt) {
  		// new code below
    	socket.emit('add-circle', {
      		initials: initials,
      		x: evt.clientX,
      		y: evt.clientY,
      		dia: randomBetween(10,100),
      		rgba: getRandomRGBA()
    	});
    	// new code above
  	});
	```

---

### Displaying Circles - Messaging Check
<br>

<p style="text-align:left">To recap, our code so far:</p>

1. Emit's `add-circle` messages and data to the server when a user clicks.

2. Receives those messages emitted from the server and console logs their data.

<p style="text-align:left">Let's open two browsers on `localhost:3000` and make sure our console shows the messages as we click!</p>

---

### Displaying Circles - Client Code (cont.)

<p style="text-align:left">Next, let's refactor <em>addCircle()</em> so that we can just pass in the data object received with the message:</p>

```js
// was -> function addCircle(x, y, dia, rgba) {
function addCircle(data) {
	var el = document.createElement('div');
	el.style.left = data.x - Math.floor(data.dia / 2 + 0.5) + 'px';
	el.style.top = data.y - Math.floor(data.dia / 2 + 0.5) + 'px';
	el.style.width = el.style.height = data.dia + 'px';
	el.style.backgroundColor = data.rgba;
	el.style.fontSize = Math.floor(data.dia / 3) + 'px';
	el.style.color = 'white';
	el.style.textAlign = 'center';
	el.style.lineHeight = data.dia + 'px';
	el.innerHTML = data.initials;
	circles.appendChild(el);
}
```

---

### Displaying Circles - Client Code (cont.)
<br>

- All that's left is to call the `addCircle()` function from our `socket.on` listener inside `app.js`:

	```js
  	// listen to the server for the `add-circle` event
  	socket.on('add-circle', function (data) {
   		// console.log(data);
   		addCircle(data);
  	});
	```

- Use two browsers with different initials and test drive that sucka!

---

#### Now that we have our circles displaying in realtime<br>let's turn our attention to the next item on the roadmap - clearing the display!

---

### Clear All Circles<br>Practice (10 - 15 mins)
<br>

- Partner up and make the `clear` button clear all connected user's displays instead of just yours.

- Hints: This will require another event in addition to the `add-circle` event.

---

### Who would like to come up and share their solution?

---

## Deploy to Heroku
<br>

- **Set aside your fears and:**<br><br>
  1. Create a local git repo: `? git init`
  2. Add and commit: `? git add -A` & `> git commit -m "Initial commit"`
  3. Make sure you are logged in to Heroku: `? heroku login`
  4. Create a Heroku deployment: `? heroku create`
  5. Deploy your repo to Heroku: `? git push heroku master`
  6. Ensure that at least one instance is running: `? heroku ps:scale web=1` 
  7. Once deployed, open the app: `? heroku open`

---

## Realtime Is Fun!

---

## Questions
<br>

- **What is the name of the method used to send methods from the server/client to the client/server?**

- **What method is used to set up a listener for a message?**

- **What are the names of the messages available to us?**

---

## Want More? Track Players!
<br>

- In the realm of realtime, tracking connected users or players is know as tracking **presence**.

- It would be nice to know who's connected to our `realtime-circles` app, so let's do this!

---

### Track Players - Server Code Logic
<br>

1. When a client connects, set up a listener for a `register-player` message from that client. The client will send their initials as data with the message.
2. When a client emits the `register-player` message, the server will:<br> (a) Add the player's initials as a key to a `players` object variable. We are going to use an object instead of an array because it will make it easier to both prevent duplicates and remove players when they disconnect.<br> (b) Then we will then emit an `update-player-list` message, along with the updated list of initials, as an array, to all clients.
3. When a client disconnects, we will remove the key from `players` and again, emit the `update-player-list` message.

---

### Track Players - Client Code Logic
<br>

1. After the player has entered their initials, emit the `register-player` message, sending the initials as data.

2. Listen for the `update-player-list` message and update the DOM by writing `<li>` tags (one for each player in the array) inside of the provided `<ul>`.

---

### Tracking Players - Server Code
<br>

- Define the `players` object to hold player's initials in `io.js`:

	```js
	var io = require('socket.io')();
	
	// object to hold player's initials as keys
	var players = {};
	```

---

### Tracking Players - Server Code (cont.)

<p style="text-align:left">Set up the listener for the `register-player` message in which we will take care of business:</p>

```js
io.on('connection', function (socket) {
	// new code below
	socket.on('register-player', function (data) {
  		// assigning true is arbitrary, we just need to create a key
  		players[data.initials] = true;
  		socket.initials = data.initials;
  		io.emit('update-player-list', Object.keys(players));
	});
	
	...
	
```

<p style="text-align:left">Notice we're taking advantage of the dynamic nature of JS by adding our own custom `initals` property onto the client `socket` object!</p>

---

### Tracking Players - Server Code (cont.)

- Set up the listener for when the player disconnects. Add this along with the other listeners:

	```js
	io.on('connection', function (socket) {
	
    	// when the player disconnects, remove key & notify clients
    	socket.on('disconnect', function (data) {
      		delete players[socket.initials];
      		io.emit('update-player-list', Object.keys(players));
    	});
    	
    	...
    	
	```

---

### Tracking Players - Client Code
<br>

<p style="text-align:left">After the player has entered their initials,<br>emit the <em>register-player</em> message, sending the initials<br>as data in <em>app.js</em>:</p>

```js
...
	
while (initials.length < 2 || initials.length > 3) {
	initials = prompt("Please enter your initials").toUpperCase();
	// new code below
	if (initials.length > 1 && initials.length < 4) {
		socket.emit('register-player', {initials: initials});
	}
}
	
...
	
```

---

### Tracking Players - Client Code (cont.)
<br>

<p style="text-align:left">Let's cache the players ul element into a var:</p>

```js
...
	
var circles = document.getElementById('circles');
  	
// players <ul> element in the footer
var players = document.getElementById('players');
	
...
```

---

### Tracking Players - Client Code

- Add the listener for the `update-player-list` event:

	```js
	...
	
  	// listen to the server for when the player list has changed
  	socket.on('update-player-list', function (data) {
   		var playerList = '<li>' + data.join('</li><li>') + '</li>';
    	players.innerHTML = playerList;
  	});
   	 	
    ...
    ```

- Using the `join()` method to create a string from an array is very efficient!

---

## Tracking Players - Run It!

---

### Tracking Players - Summary
<br>

- When a player visits the page and enters their initials, the app informs the server by emitting the `register-player` message.

- The server adds the player's initials to the `players` object as a key and notifies all connected clients by emitting the `update-player-list` message.

- Clients then receive the `update-player-list` message, generate a nice list of `<li>` tags in a string, and blast that baby in the `<ul>`'s `innerHTML`.

--- 

## Questions?

---

## References
<br>

- [Socket.IO](http://socket.io/)

- [WebSockets Protocol](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
