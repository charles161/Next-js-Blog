---
title: "Video Calling App in React Native using WebRCT and MQTT"
date: "2020-01-01"
---

# VideoCalling App

#blog

Initialize React Native App

`npx react-native init VideoCaller`

`npm i react-native-webrtc`

```xml
//AndroidManifest.xml

<manifest … >

…

<uses-permission android:name=“android.permission.CAMERA” />
<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus"/>

<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name=“android.permission.WRITE_EXTERNAL_STORAGE"/>

<application>
…
</application>

<…/mainfest>
```

`Build -> Rebuild Project`

`error: package com.android.annotations does not exist`

-   This is a problem with androidX.

-   To automatically fix all android to androidx issues for React Native,(Prerequisite npx)

```properties
//gradle.properties

…
android.useAndroidX=true
android.enableJetifier=true

```

```bash
npm install --save-dev jetifier
npx jetify
```

-   If the above doesn’t work, then head on to
    [react native - error: package com.android.annotations does not exist - Stack Overflow](https://stackoverflow.com/questions/40380519/error-package-com-android-annotations-does-not-exist)

`Build -> Rebuild Project`

```js
//App.js

import {
    MediaStream,
    MediaStreamTrack,
    RTCIceCandidate,
    RTCPeerConnection,
    RTCSessionDescription,
    RTCView,
    mediaDevices,
    registerGlobals
} from "react-native-webrtc";
import React, { Component } from "react";

import { View } from "react-native";

class App extends Component {
    state = {
        stream: null
    };

    componentDidMount() {
        const configuration = {
            iceServers: [{ url: "stun:stun.l.google.com:19302" }]
        };
        const pc = new RTCPeerConnection(configuration);

        let isFront = true;
        mediaDevices.enumerateDevices().then(sourceInfos => {
            let videoSourceId;
            for (let i = 0; i < sourceInfos.length; i++) {
                const sourceInfo = sourceInfos[i];
                if (
                    sourceInfo.kind == "videoinput" &&
                    sourceInfo.facing == (isFront ? "front" : "back")
                ) {
                    videoSourceId = sourceInfo.deviceId;
                }
            }
            mediaDevices
                .getUserMedia({
                    audio: true,
                    video: {
                        mandatory: {
                            minWidth: 500, // Provide your own width, height and frame rate here
                            minHeight: 300,
                            minFrameRate: 30
                        },
                        facingMode: isFront ? "user" : "environment",
                        optional: videoSourceId
                            ? [{ sourceId: videoSourceId }]
                            : []
                    }
                })
                .then(stream => {
                    this.setState({ stream });
                })
                .catch(error => {
                    // Log error
                });
        });

        pc.createOffer().then(desc => {
            pc.setLocalDescription(desc).then(() => {
                // Send pc.localDescription to peer
            });
        });

        pc.onicecandidate = function(event) {
            // send event.candidate to peer
        };
    }

    render() {
        return (
            <View style={{ flex: 1 }}>
                <RTCView
                    streamURL={
                        this.state.stream ? this.state.stream.toURL() : null
                    }
                    style={{ flex: 1 }}
                />
                <RTCView
                    streamURL={
                        this.state.stream ? this.state.stream.toURL() : null
                    }
                    style={{ flex: 1 }}
                />
            </View>
        );
    }
}

export default App;
```

-   `react-native run-android`

![](VideoCalling%20App/Screenshot_2020-03-03-07-01-17-435_com.videocaller.jpg)

Voila We got our app working. But the code seems to be complex. So let's break it down.

Let’s talk about WebRTC

-   What is WebRTC?
    _ With WebRTC, you can add real-time communication capabilities to your application that works on top of an open standard. It supports video, voice, and generic data to be sent between peers, allowing developers to build powerful voice- and video-communication solutions.
    _ WebRTC provides real-time capabilities to your application , where you can send video, voice and generic data between peers (connected device) \* `source: [WebRTC](https://webrtc.org/)`

Let’s walk through our code once.

Oops!!! Peer Connection, Media devices, Offer, Description, ICE Candidates. It sounds a lot. So what are these and how do they work together.

-   Lets start with some Knowledge Transfer
    _ Media Devices:
    _ Media devices are basically the camera and the microphone. WebRTC provides standard APIs to see all connected media devices, listen for connection changes and receive media stream from those devices . Read through this to get into detail:[Getting started with media devices  |  WebRTC](https://webrtc.org/getting-started/media-devices). Apart from getting streams from camera and mic, we can also get stream from our Smartphone or Computer screen. Learn more about it and streams from here: [Media capture and constraints  |  WebRTC](https://webrtc.org/getting-started/media-capture-and-constraints)
    _ Peer Connections:
    _ Deals with connecting two WebRTC application no different computers with a peer-to-peer protocol
    _ The communication between peers can be audio, video or binar data.
    _ In the process of connecting two peers, WebRTC requires each peer to provide an ICE(Internet Connectivity Establishment) Server Configuration: which is basically a STUN or TURN server which provide ICE candidates to each peer which is then transferred to the remote peer. This transferring of ICE candidates is commonly called Signalling \* So as might you have understood, we need to get the ICE candidates from the STUN/TURN server and pass it to the other Peer. This is needed in order for the Peers to know how they should connect

Webrtc sides: http://io13webrtc.appspot.com/#23
