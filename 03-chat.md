# Create chat app

## Server side

1.  `mkdir chatApp && cd chatApp`
1.  `npm init -y`
1.  `npm i express ejs socket.io`
1.  In index.js

    ```js
    const express = require('express');
    const app = express();

    /* set the template engine to ejs */
    app.set('view engine', 'ejs');

    /* middleware */
    app.use(express.static('public'));

    /* routes */
    app.get('/', (req, res) => {
      res.send('hello world');
    });

    /* listen on port 3000 */
    const server = app.listen(3000);
    ```

1.  if you run `nodemon` it you should see hello world
1.  in index.js continued:

    ```js
    /* socket.io instantiation; pass in our server variable */
    const io = require('socket.io')(server);

    /* listen on every connection */
    io.on('connection', socket => {
      console.log('New user connected');
    });
    ```

- Here the io object will give us access to socket.io library
- The io object is now listening to each connection in our app
- Each time a new user connects it will printout 'New user connected'

Try reloading browser, nothing happened yet.
Q. Why?
A. Because our client side is not yet ready. For the moment socket.io is only installed on server.
We have to code client side.

1.  Change hello world to a form to send messages:

    ```js
    app.get('/', (req, res) => {
      res.render('index');
    });
    ```

## Client side

1.  In views/index.ejs (or see blackboard under week 9 for html code below)

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="//fonts.googleapis.com/css?family=Roboto:300,300italic,700,700italic">
    <link rel="stylesheet" href="//cdn.rawgit.com/necolas/normalize.css/master/normalize.css">
    <link rel="stylesheet" href="//cdn.rawgit.com/milligram/milligram/master/dist/milligram.min.css">
    </head>
    <body>
        <h1>My chatApp</h1>
        <div class="container ">
            <div class="row ">
                <div class="column column-50 column-offset-25 ">

                    <fieldset>
                        <label for="username">username</label>
                        <input type="text" name="username" id="username">
                        <button id="send_username">Change username</button>

                        <hr>

                        <label for="chatroom">chatroom</label>
                        <main id="chatroom" style="height: 40vh; border: 1px solid #ccc;"></main>

                        <label for="message">message</label>
                        <textarea name="message" id="message" style="height: 10vh"></textarea>

                        <button class="column" id="send_message">Send</button>
                        <hr>
                    </fieldset>

                </div>
            </div>
        </div>
    </body>
    </html>
    ```

- now that we have our basic form, we have to install socket.io on each client
- which in turn will attempt to connect to our server
- To do that we have to import socket.io library on client side (google: 'cdn socket.io')

1.  in index.ejs

    ```html
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.1.1/socket.io.js"></script>
    ...
    <script src="chat.js"></script>
    ```

1.  Create /public/chat.js
    ```js
    /* make connection */
    const socket = io.connect('http://localhost:3000');
    ```

So when the client loads the page, it will automatically connect and so create a new socket
and in terminal you should see 'New user connected' -- Try it out!

## Username

- When a user connects to our app, we'll assign 'anonymous' if no username given
- To do that, we have to go on the server and add some key/property to the socket

1.  Change index.js on server side to:

    ```js
    /* listen on every connection */
    io.on('connection', socket => {
      console.log('New user connected');

      /* default username */
      socket.username = 'Anonymous';

      /* We also will listen to any call made in « change_username. 
      If a message is sent to this event, the username will be changed. */
      socket.on('change_username', data => {
        socket.username = data.username;
      });

      /* listen on new message */
      socket.on('new_message', data => {
        console.log('I heard we got a new message');
        console.log(data);

        /* broadcast the new message */
        io.sockets.emit('new_message', {
          message: data.message,
          username: socket.username
        });
      });
    });
    ```

- On the client side, the goal is to do the opposite
- Each time the button change username is clicked, the client will send an event with the new value

1.  In index.ejs in script part:

    ```js
    /* make connection */
    const socket = io.connect('http://localhost:3000');

    /* buttons and inputs */
    const message = document.getElementById('message');
    const username = document.getElementById('username');
    const sendMessage = document.getElementById('send_message');
    const sendUsername = document.getElementById('send_username');
    const chatroom = document.getElementById('chatroom');

    /* Emit message */
    sendMessage.addEventListener('click', () => {
      socket.emit('new_message', { message: message.value });
    });

    /* listen on new_message */
    socket.on('new_message', data => {
      console.log(data);
      chatroom.insertAdjacentHTML(
        'beforeend',
        `<p>${data.username}: ${data.message}</p>`
      );
    });

    /* Emit a username */
    sendUsername.addEventListener('click', () => {
      console.log(username.textContent);
      socket.emit('change_username', { username: username.value });
    });
    ```
