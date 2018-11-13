# Games

## Deploying Updates to Existing Games

1. Content/Product Teams will advise Tim once they have tested and approved updates on Game Gurus' server.
2. Tim will engage Engineering to sync the code with Game Gurus and deploy to RC for testing by Content/Product.
  * Sync the game by typing `mathbot sync games` in HipChat.
  * Deploy the game to RC by typing `mathbot deploy games rc rc` in HipChat.
3. Content/Product Teams will test the game update. Once they have tested the and approved the updates on RC, Content/Product (Christy) will advise Tim.
4. Tim will engage engineering to deploy the game to production.
  * Deploy the game to prod by typing `mathbot deploy games **TBD**` in HipChat.

Tim will file bugs or escalate any issues discovered in the deployment process.

## Adding New Games to the CMS

### The Ruby Part

The CMS depends on the game naming schema to match that of https://github.com/thinkthroughmath/apangea/blob/rc/app/models/game_config.rb#L4-L14 by default the naming is upper CamelCase and will expect that the game being deployed will have a folder on the s3 content bucket matching the constant in `apangea`

The deploy script will assure that these folders are in place. The folder is named by the package.json file located in the TRUNK directory of each game engine like: https://github.com/thinkthroughmath/GG-Recipe/blob/rc/TRUNK/package.json

Deployment is reliant on this repo for orchestration: https://github.com/thinkthroughmath/GG-Server

To enforce separation of ownership for now Jenkins relies on the `bower` configuration from `GG-Server` to identify safe versions for games. When :shipit: finally happens these `bower` dependencies should be all set to `rc` for their respective repos. For the moment during development that is not the case.

Jenkins on `sync` will update `rc` of the sync'd repo and also create a tag revision at that point. This can be used in `GG-Server` to fix a version to a previous revision easily. The tags are based off the HEAD svn rev number at time of sync.

Finally, there is currently a game config decorator: https://github.com/thinkthroughmath/apangea/blob/rc/app/decorators/game_config_decorator.rb used to allow for feature flagging of specific engines. You should add your feature flag here and update the [[Game-Engines]] to reflect the change

### The JS Part

For each engine most of its configuration is provided when the CMS loads its metadata. But when we allow the CMS user to change engines a new empty object must be created in `javascripts/game_cms/game_data_switcher.js`.

Here is what this looks like for a common engine like Recipe
```javascript
var gameDataPresets = {
  'Recipe':{}
};
```
`gameDataPresets` holds a key for each engine in the system and an empty object which represents its required interface for an empty CMS form.

A further example of a default that requires nesting is like
```javascript
var gameDataPresets = {
  'Tile':{'grid':{}}
};
```

If you observe the CMS when loading a tile game you will notice a top level common set of configurations and then a nested body.

Why doesn't recipe require this same design since it has nested `bar` configurations well that is because the `bar` configuration is only required for saving not for displaying.

You will need to extract the empty interface from the engine you are going to add to the system. For this you can clone and build the individual engine repo or load them through `GG-Server` either way you need to look for a block like this in the engines `src/data/game-data.js`

```javascript
gg.gameDataPresets = {
  'Empty': {},
  'Preset': {
    skin: 1,
    coinsSmall: 10,
    coinsLarge: 0,
    ...
    }
};
```

We need to extract what is in `'Empty'` into `game_data_switcher.js`

A long explanation for something pretty quick but easy to forget.

### Upgrading the CMS

The game CMS is basically a single page JS app backed by jQueryUI and Handlebars

Repo: (https://github.com/thinkthroughmath/GG-CMS)

Builds with Grunt and is part of GG-Server for quick testing

### Integration Steps

#### Prep

* clone the repo
* run `grunt ready-deploy`

The following processes are listed by importance in most cases only scripts need to be copied. Styles only if prompted by GG to do so.

#### 1 Scripts

* Copy `cms.js` JS to `app/assets/javascripts/game-cms/`
* Copy `validations.js` to `app/assets/javascripts/game-cms/`
* Copy `common.js` to `app/assets/javascripts/game-cms/`
* Copy `templates.js` to `app/assets/javascripts/game-cms/`
* Copy `helpers.js` to `app/assets/javascripts/game-cms/`
* Leave existing overrides (`*_ours.js`) in place as the sprockets manifest will load them after their parent

I have had problems after converting their files to coffee so you can always try and include them as I am building them with `read-deploy` but I wouldn't waste your time. The scripts have some issues with variable scoping and are unfortunately order dependent (something to fix later).

#### 2 Styles

* Convert the css at `build/web/css` to SCSS using your preferred method. I use http://css2sass.heroku.com/
* Make sure you wrap the output thing in a `.gg-editor` selector because this defines body and will blow everything up
* Copy cms SCSS to `app/assets/stylesheets/gg-cms/game_cms.scss`
* Adjust overrides as appropriate in `game_cms_ours.scss` (This is optional and if GG hasn't changed anything serious you can skip it)

Target of this strategy is to keep their CSS pure and copyable until the contract is over. At which point these can be combined into what we will call our CSS file.

#### 3 Templates

This is probably the easiest part I convert the extension of the repos Handlebars templates to `.hbs` as this is automaticalluy picked up by the Handlebars Runtime.

* Move all `.hbs` files create by `ready-deploy` to `app/assets/javascripts/templates/game-cms/`
* You will notice 2 copies of `data_model.hbs` one is picked up as a template and one as a partial
* Because our environment is a little larger than GG's test environment make some manual changes to templates that nest `data_model`. Change `data_model` to `game_cms/_data-model` so it loads the partial
  * object-array-tab-content.hbs
  * object-control.hbs
  * object-array-control.hbs


#### 4 Extras

* Don't bother with externals unless GG alerts us
* Don't worry about media I have done my best to remove all use of external media for the system

## Game Engines

### Recipe

[Repo](https://github.com/thinkthroughmath/GG-Recipe)

Flag: :game_recipe

The most mature and test case for new framework builds.

If you are testing a new build from GG start here.

### AreaPerimeter

[Repo](https://github.com/thinkthroughmath/GG-AreaPerimeter)

Flag: :game_area_perimeter

### Chain

[Repo](https://github.com/thinkthroughmath/GG-Chain)

Flag: :game_chain

### CMS

[Repo](https://github.com/thinkthroughmath/GG-CMS)

### Customer

[Repo](https://github.com/thinkthroughmath/GG-Customer)

Flag: :game_customer

### Estimation

[Repo](https://github.com/thinkthroughmath/GG-Estimation)

Flag: :game_estimation

### OrderOfOperations

[Repo](https://github.com/thinkthroughmath/GG-OrderOfOperations)

Flag: :game_order_of_operations

### Tile

[Repo](https://github.com/thinkthroughmath/GG-Tile)

Flag: :game_tile

### Target

[Repo](https://github.com/thinkthroughmath/GG-Target)

Flag: :game_target

### Sorting

[Repo](https://github.com/thinkthroughmath/GG-Sorting)

Flag: :game_sorting

### FactorPairs

[Repo](https://github.com/thinkthroughmath/GG-FactorPairs)

Flag: :game_factor_pairs

### Graphing

[Repo](https://github.com/thinkthroughmath/GG-Graphing)

Flag: :game_graphing

### Probability

[Repo](https://github.com/thinkthroughmath/GG-Probability)

Flag: :game_probability

### Stacks

[Repo](https://github.com/thinkthroughmath/GG-Stacks)

Flag: :game_stacks

### Connect

[Repo](https://github.com/thinkthroughmath/GG-Connect)

Flag: :game_connect

### Match

[Repo](https://github.com/thinkthroughmath/GG-Match)

Flag: :game_match
