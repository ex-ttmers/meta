# How to add soundfiles to mathbot play command

Record them however you see fit - a mobile phone mic is fine. You should get them into mp3 format (probably not required but easier to manage)

## Get the file on to the hp mini blue pc that we use for standups

http://jenkins.thinkthroughmath.com:8080/view/All/job/copy-file/

click `configure`

edit the file location so it matches the filename you are planning to upload

change the windows shell script (under 1Execute Windows batch command1) to rename the file - this is the script that will move it to the correct location when it's on the pc

click `save`

Now upload the file - go back to the copy file project:

http://jenkins.thinkthroughmath.com:8080/view/All/job/copy-file/

click `build with parameters`
find your local file with the chooser and click `build`

now edit the jenkins job that plays the file:

http://jenkins.thinkthroughmath.com:8080/view/mathbot/job/play/

click `configure`
add your phrase to the `Choice Parameter` list - this is what you want people to have to type to play your audio

scroll down to the Windows batch command

add a new `IF` branch at the top, and a new reference - use the existing options as a guide, and see Jim or Jeff if you need help.

Now you need to add a pr to mathbot to play the file.
