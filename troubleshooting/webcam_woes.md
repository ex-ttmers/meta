## Webcam woes on OS X

I'm trying to connect to a virtual meeting and my webcam won't turn on? but I'm otherwise connected

This is a well-known issue with OS X - sometimes applications that use the camera fail to release it. A full restart will always fix it, but at a terminal window you can also just enter `sudo killall VDCAssistant` and then reconnect to your meeting.
