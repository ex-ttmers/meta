# Game JSON Structure

To see any of the Media or Core Structures, see [[Math Content JSON Structures]]

## Game

A Game is the standard Stem structure plus the Game Config Structure which is specific for Games.

````json
{
  "stem": Stem,
  GameConfig
}
````

#### Game Config

The Game Config holds specific attributes like skin, engine, etc.... for a Game.  It also has an array of Bars which is essentially a number line.

````json
{
           "skin": 1,
        "engine": "Recipe",
          "type": "game",
     "timeLimit": 180,
  "maxSuccesses": 6,
   "maxAttempts": 0,
    "coinsSmall": 4,
    "coinsLarge": 4,
          "bars": [ Bar ]
}
````

#### Bar

````json
{
            "contentType": "integer",
            "contentTags": [ "Potato", "Spinach" ],
             "rangeLower": 0,
             "rangeUpper": 10,
         "referencePoint": 5,
               "stepSize": 1,
              "precision": "",
         "trailingZeroes": false,
    "rangeLowerNumerator": 0,
  "rangeLowerDenominator": 1,
    "rangeUpperNumerator": 1,
  "rangeUpperDenominator": 1,
      "midpointNumerator": 0,
    "midpointDenominator": 1,
           "numeratorSet": nil,
         "denominatorSet": nil
}
