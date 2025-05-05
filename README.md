# spaceswarm-proxy
Proxy p2p connections using a duplex stream and Spaceswarm

## How it works

- A server initializes an instance of [spaceswarm](https://github.com/samooth/spaceswarm)
- A client opens a connection to the server (use any duplex stream)
- Client instance behaves the same as spaceswarm
- The client asks to join "topics"
- The server starts searching for peers and sends peer information to the client
- The client chooses which peers to connect to (all of them by default)
- The server establishes a connection to the peer and proxies it over to the client
- The client can relay whatever data they want to the peer.

## Server

```js
const SpaceswarmProxyServer = require('spaceswarm-proxy/server')

const server = new SpaceswarmProxyServer({
  // The bootstrap nodes to pass to spaceswarm (optional)
  bootstrap,

  // Whether your server will be sort lived (ephemeral: true)
  ephemeral,

  // Pass in an existing spaceswarm instance instead of creating a new one (optional)
  network,

  // Function that takes incoming connections to decide what to do with them
  // Can be used to notify clients about peers
  // Disconnect incoming by default
  handleIncoming: (socket) => socket.end(),
})

const somestream = getStreamForClient()

// Get the server to handle streams from clients
server.handleStream(somestream)

// Tell all clients about a peer for a particular topic
// This is useful in conjunction with `handleIncoming`
// Use handleIncoming to figure out which topic the connection is from
// Then tell clients to connect to them using `socket.address()`
server.connectClientsTo(SOME_TOPIC, port, hostname)

// Close connections to clients and any connected peers
server.destroy(() => {
  // Done!
})
```

## Client

```js
const SpaceswarmProxyClient = require('spaceswarm-proxy/client')

const somestream = getStreamForServer()

const swarm = new SpaceswarmProxyClient({
  // Pass in the stream which connects to the server
  // If this isn't passed in, invoke `.reconnect(somestream)` to start connecting
  connection: somestream,

  // Whether you should autoconnect to peers
  autoconnect: true,

  // The max number of peers to connect to before stopping to autoconnect
  maxPeers: 24,
})

// When we disconnect, reconnect to the server
swarm.on('disconnected', () => {
  swarm.reconnect(getStreamForServer())
})

// Same as spaceswarm
swarm.on('connection', (connection, info) => {

  const stream = getSomeStream(info.topic)
  // Pipe the data from the connection into something to process it
  // E.G. `spacedrive.replicate()`
  connection.pipe(stream).pipe(connection)
})

swarm.join(topic)

swarm.leave(topic)

// Listen for peers manually
swarm.on('peer', (peer) => {
  // If autoconnect is `false` you can connect manually here
  swarm.connect(peer, (err, connection) => {
    // Wow
  })
})

// Destroy the swarm
swarm.destroy()
```
