### Summary

TTM currently provides one way audio chat between teacher and student. The teacher is able to speak and the student types
their response. The chat system is currently implemented using Adobe Flash. The current implementation uses an outdated version
of flash and requires a plugin. 

TTM wishes to modernize this part of the application. There are two options for this. 

    1. Modernize the flash application
    2. Migrate to WebRTC
    
The requirements for an updated audio chat system are: 

    1. One way audio
    2. One to One connections. One teacher to one student
    3. Teacher should be able to switch audio between multiple students
    4. Recording of each Audio Session
    5. All infrastructure hosted under the thinkthroughmath.com domain

### Options

##### WebRTC

As of this date WebRTC is only avaliabe in Firefox and Chrome. iOS support is only via a native ObjectiC or Swift app. Even with WebRTC's
limited support, it is the future of realtime web communication, and provides a path to mobile.

    * Native to Chrome and Firefox Only
    * IE and Safari requires a plugin
    * Even with a plugin, IE support depends on how the frame work is engineered.
    * Mobile support for Android
    * Mobile support for iOS (app required)
    * No Windows 10 / Edge in current frameworks
    * Requires STUN/TURN Server to support users behind firewall

##### Flash

Adobe flash provides a platform for rich media on the web. Flash support is becoming an issue on the desktop due to security
problems and there is no flash support on mobile devices.

    * Requires Plugin for all browsers
    * Supports any browser that has a flash plugin. IE, Chrome and Firefox
    * No mobile support
    * Requires Flash Media Server
    * Will require some third party consulting to modernize the existing Flash app and infrastructure

### WebRTC Review

APIs from the following groups were evaluated

    * http://peerjs.com
    * http://easyrtc.com
    * http://tokbox.com
    * http://simplewebrtc.com

A simple prototype application was produced for all except simplewebrtc. All the frameworks are standalone except for tokbox
which requires the use of their service.

Each API included a signalling server implemenation. The signalling servers were unique to each API.

The APIs all allow call setup and tear down and they each provide a channel or room abstraction at the singalling level. Because they
were all pretty similar the sinalling server was the deciding factor.

Tokbox was ruled out becasue their service could not be "white labeled". Clients would have to open a range of ports to any IP.

#### API Selection

EasyRTC (https://github.com/priologic/easyrtc) was selected becasue their signalling server supports authentication and has pretty good documentation. The API is similar to
all the others. Configuration of the singalling server is fairly simple and there is documentation availabe on their GitHub page.

EasyRTC is the only framework with a mobile pathway. 

##### IE Support

Temasys provides a plugin for IE and Safari that implementes the WebRTC standard. If the framework uses adapter.js it should be
compatible with the temasys plugin.

The plugin comes in two versions, free and commercial. The free version is not signed and only has email support. If we want a
signed plugin with support, based on our usage we would need the platinum plan from temasys. 

    * $50k / year
    * 100,000 installations. Additional installations are $0.45 per install per year
    * Ability to sign the plugin with a TTM Code signing certificate
    * Negiotiated SLA for support

More details can be found in the following PDF https://github.com/thinkthroughmath/meta/blob/master/infrastructure/WebRTC/Temasys-Pricing.pdf

#### Session Recording

EasyRTC does not provide a way to record a session. However you are able to capture the media stream and record from it directly.
The implementaion of recording uses this hack. 

A second copy of the media stream is passed to RecordRTC.js (http://recordrtc.org/). This produces an in memory WAV file. When the 
session is over or the teacher switches to a new student the wave file is transmitted to S3 via a signed URL where a lambda job
picks it up and sends it to Elastic Transcoder for conversion to MP3. Please note that one session could be split across multiple files
becasue each time the teacher switches to a new student a new recording is started.

##### Session Recording Challenges

The WAV file lives in memory and uses ~450MB per hour. Amazon S3 has a max upload of 5GB for a GET request. 

#### Infrastructure Requirements

WebRTC requires the following pieces of infrastructure

1. Signaling Server
2. STUN (Session Traversal Utilities for NAT) Server
3. TURN (Traversal Using Relays around NAT) Server

###### Signaling Server

EasyRTC provides a signaling server. Its a node app. It seems fairly extensable via a built in event emitter and callback structure.

###### STUN/TURN Server

Restund (http://www.creytiv.com/) seems to be a reasonable choice for a STUN and TURN server. CPU utilization of the TURN server will
have to be monitored. Since the teacher and student will both be behind a firewall, the majority of sessions will be relayed through the
TURN server.
