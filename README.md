
## WebRTC Documentation 📃


## Setup for Backend
## 1 . Server and socket.io setup 

```javascript
const express = require("express")
const app = express()
const path = require("path")
const server = require("http").createServer(app)
const io = require("socket.io")(server)

app.set("view engine","ejs")
app.use(express.static(path.join(__dirname,"public")))


io.on("connection",function(socket){
    socket.on("signalingMessage",function(message){
        socket.broadcast.emit("signalingMessage",message)
    })
})

app.get("/",function(req,res){
    res.render("index")
})

server.listen(3000)
```

## Setup For FrontEnd(ejs)
## 2.Variable Declarations
```javascript
let localStream;          // To hold the local media stream (audio/video)
let remoteStream;         // To hold the remote media stream (audio/video)
let peerConnection;       // To manage the WebRTC peer-to-peer connection
let stunServer = {
    iceServers: [{ urls: "stun:stun.l.google.com:19302" }]
}
```

## 3. Initializing LocalStream
To capture the local audio and video streams using WebRTC, you can use the navigator.mediaDevices.getUserMedia API.

```javascript
 localStream = await navigator.mediaDevices.getUserMedia({
        audio: true,
        video: true
    })
```

## 4. Set Up the Peer Connection with a STUN Server
To connect to a STUN server and retrieve ICE candidates, including the encoded IP address.First, you'll need to set up the RTCPeerConnection to use a STUN server.

```javascript
  peerConnection = new RTCPeerConnection(stunServer)
```

ICE Candidate Generation: When WebRTC gathers connection candidates (including details like the encoded IP address, port, protocol, etc.), it triggers the onicecandidate event.

Emit to Signaling Server: The code listens for this event. When an ICE candidate is generated (event.candidate), it is immediately sent (or "emitted") to the signaling server using socket.emit.

```javascript
  peerConnection.onicecandidate = event => event.candidate && socket.emit("signalingMessage", { type: "candidate", candidate: event.candidate })

```
## 5. Adding Tracks to Peer Connection 
```javascript
  localStream.getTracks().forEach(function (track) {
        peerConnection.addTrack(track, localStream)
    })

```
## Displaying Local Stream
```javascript
   document.querySelector("#localVideo").srcObject = localStream

```
## 6. Create Blank video stream for Remote Stream 
```javascript
   remoteStream = new MediaStream()

```
## Displaying Remote Stream
```javascript
  document.querySelector("#remoteVideo").srcObject = remote

```

## 7. Handling Remote Media Tracks with peerConnection.ontrack

Handling Remote Media Tracks with peerConnection.ontrack
```javascript
  peerConnection.ontrack = event =>  event.streams[0].getTracks().forEach(track => remote.addTrack(track))

```

## 8.  Initiate Offer 
```javascript
 var offer = await peerConnection.createOffer()
```
## Set Local Description and emit Offer message
```javascript
 await peerConnection.setLocalDescription(offer)
    socket.emit("signalingMessage", { type: "offer", offer })
```

## 9. Handler for Signaling Message 
```javascript
socket.on("signalingMessage", async function (message) {
    let { type, candidate, offer, answer } = message

    if (type === "offer") {
        await peerConnection.setRemoteDescription(offer)
        let answer = await peerConnection.createAnswer()
        await peerConnection.setLocalDescription(answer)
        socket.emit("signalingMessage", { type: "answer", answer })
    }
    if (type === "answer") {
        if (!peerConnection.currentRemoteDescription) {
            peerConnection.setRemoteDescription(answer)
        }
    }

    if (type === "candidate" && peerConnection) {
        peerConnection.addIceCandidate(candidate)
    }
})
```



BY ❤️Ayush Ahirwar
