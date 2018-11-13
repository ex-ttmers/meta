# Math Content JSON Structures
This is (will be) a comprehensive collection of all the structures we use to store our math content items in our database.  Right now the structure is saved on the Item's first item step's object_model field.  In due time it will be directly on the Item, but until then....
We categorize structures in 3 parts, Media Structures, Core Structures, and Items.
* _**Media Structures**_ are the very low level components we use throughout our items.  They all relate to media - an image, a text message, and audio snippet etc....
* _**Core Structures**_ are mid level structures that use the media structures and have extra attributes associated with them.  For example, a Response.  A Response can have an image associated with it, but also specific response attributes like correctness, notes, etc....
* _**Items**_ are comprised of Core Structures to make up, well, an item.  For example a Multiple Choice Item, or a Create Evaluate Item.

## Media Structures
Media structures are low level components that are used in our math items.
We have 3 types of media in our system right now.  Text, Image, Movie.  Each of these items have a key of "type" that differentiates it and a key of "media" with it's specific attributes.  This format makes it easy to add new media types (svg maybe?) and easy to pass along the data to the Lesson Player.

For example:
````json
{
  "type":  "text|image|movie",
  "media": Text|Image|Movie
}
````

#### Text
Text is one English (en) and one Spanish (es) Message.
````json
{
  "en": Message,
  "es": Message
}
````

More languages can be added easily in the future if needed.  We're going global baby.

A Message sub component is displayable, readable, and has an associated audio file (audioable :smile:).

##### Message
````json
{
  "display_text":  "This is the ENGLISH display text of the message.",
  "readable_text": "And this is it's readable text used for making the audio.",
  "audio":         "This is the url to it's mp3 which is stored on s3."
}
````

Here is an example of a full **Text** Media:
````json
{
  "type":  "text",
  "media": {
    "en": {
      "display_text":  "What is 4 + 5?",
      "readable_text": "What is four plus five?",
      "audio":         "http://amazons3.com/whatever/file.mp3"
    },
    "es": {
      "display_text":  "¿Qué es 4 más 5 ?",
      "readable_text": "¿Qué es cuatro más cinco ?",
      "audio":         "http://amazons3.com/whatever/file.mp3"
    }
  }
}
````

#### Image
Images are of type jpg, png, gif.
````json
{
  "src": "url to the image on s3."
}
````

Here is an example of a full **Image** Media:
````json
{
  "type":  "image",
  "media": {
    "src": "http://amazons3.com/whatever/file.jpg"
  }
}
````

#### Movie
Movies are a combination of a flash file (swf) and it's converted mp4 file.
Hit Points are timeframes that the movie is 'split' on.
````json
{
  "flash_src":  "http://amazons3.com/whatever/file.swf",
  "mp4_src":    "http://amazons3.com/whatever/file.mp4",
  "hit_points": [4000, 6500]
}
````

Here is an example of a full **Movie** Media:
````json
{
  "type":  "movie",
  "media": {
    "flash_src":  "url to swf file",
    "mp4_src":    "url to parent mp4 file",
    "hit_points": [4000, 6500]
  }
}
````
## Core Structures
Core Structures are mid level structures that use the media structures and also have extra attributes associated with them.  The ones we have defined right now are:
* Stem
* Help
* Feedback
* Response
* Category
* Categorizable

#### Stem
A Stem is the primary question the student is trying to answer.  It is simply a special case of Media.  It must have 1 Text and an optional Image structure.  Later on, if we ever allow a movie in the stem - no code problem - just change the Image to a Media.
````json
{
  "medias": [ Text, Image ]
}
````

#### Help
A help is general helpful information to help the student.  Since all it is is types of media to display - it has a simple structure.
````json
{
  "medias": [ Media ]
}
````
ez pz. You will often see many structures having an array of Helps:

````json
{ "helps": [ Help ] }
````

#### Feedback
Feedback is specific helpful information based on a user's interaction with part of an item.  Feedback has a Text header and footer, and an array of Medias main block.
````json
{
  "opening": Text,
  "block":   [ Media ],
  "closing": Text
}
````
Are you starting to see how simple these things are?  Big fan.  Let's get a little more complicated now (just a little):

#### Response
Response is our way to store an answer that a kid can choose.  Maybe we'll rename it here - Choice?  Anywho - it will be Response for now.
Note - A Response can have an Image and audio behind it - so "medias" is an array of Media structure(s).
````json
{
  "correct":   true,
  "rationale": "A content specific reason for whatever",
  "position":  2, # Used to order the responses
  "medias":    [ Media ],
  "feedback":  Feedback
}
````

#### Category
TBD

#### Categorizable
TBD
