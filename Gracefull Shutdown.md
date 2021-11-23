# Graceful Shutdown Nodejs server.

Looking to gracefully shutdown your Nodejs HTTP server? Well, it's actually a little more difficult than you think. Without handling process signals, like "ctrl-c", your application terminates immediately. Which means, there may be requests still processing that you terminated before completion. In a live environment, this is a horrible user experience. Unfortunately, even if you did catch the process' signals, Node doesn't actually have a built in mechanism for gracefully shutting down a running HTTP server.

### Defining the problem
Graceful shutdown is the process by which a running HTTP server:

Stops accepting new connections
Stops fulfilling new requests
Waits for in-flight requests to complete
Once these three prerequisites are fufilled the HTTP server is "shutdown". All the while, it has not interrupted any concurrent request while transitioning to this state.

### Node's naive solution
Let's take the classic HTTP server written with Node's native HTTP library:

```
var http = require('http');

var server = http.createServer(function(req, res) {
    res.end('Goodbye!');
});

server.listen(3000);
```

```
server.close(function() {
    console.log('We closed!');
    process.exit();
});
```
Unfortunately, not really, there's a problem with this. Reading the documentation might make it a little more clear:
> Stops the server from accepting new connections and keeps existing connections. This function is asynchronous, the server is finally closed when all connections are ended and the server emits a 'close' event. - From Nodejs Docs

The important part of the description to remember is "keeps existing connections". While close does stop the listening socket from accepting new ones, sockets that are already connected may continue to operate - which is fine if they're mid-request - but this also includes sockets that are connected with a 'keep-alive' connection type.

>HTTP persistent connection, also called HTTP keep-alive, or HTTP connection reuse, is the idea of using a single TCP connection to send and receive multiple HTTP requests/responses, as opposed to opening a new connection for every single request/response pair - From Wikipedia

This means that sockets that are kept alive will remain alive and still capable of making additional HTTP requests. This is obviously not what we want. While it fulfils points one and three of our definition of graceful shutdown, it does not fulfill number two and is thus, not a complete solution. A graceful shutdown should programatically:
1. Close the listening socket to prevent new connections
2. Close all idle keep-alive sockets to prevent new requests during shutdown
3. Wait for all in-flight requests to finish before closing their sockets.

### The solution
http-shutdown is a NPM package that provides graceful shutdown functionality. It does this by leveraging several mechanisms for keeping track of active vs idle sockets.

### Links:
* [http://dillonbuchanan.com/programming/gracefully-shutting-down-a-nodejs-http-server/]
* [https://www.npmjs.com/package/http-shutdown]
