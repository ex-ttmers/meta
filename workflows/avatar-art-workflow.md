# Avatar art workflow

Hi! This file documents the process to create Avatar artwork for handoff to the development team. Following these guidelines will keep our artwork cohesive and allow us to automate many of the steps that turn an AI into web-ready SVGs.

### Tools

Adobe Illustrator. That's about it.

### Art guidelines

Refer to the [Avatar Art Guidelines](avatar-art-guidelines.md) for piece categories, category order in the Avatar Builder, and content guidelines.

### Creating a piece

By following the guidelines below we'll be able to automate much of our SVG creation. Pay special attention to naming of the file and layers.

##### Start with the template

Each Avatar piece should be delivered in a duplicate of the Avatar `template.ai` file, [available here](../assets/template.ai). The template contains:

- A 300x300 pixel artboard.
- A `NEW-PIECE-wearing` layer.
- A `NEW-PIECE-preview` layer.
- A locked and partially transparent Avatar `base-template` layer.

To begin a new piece:

- Give your piece a unique name. Use hyphenated lower case, no spaces or underscores.  
    (e.g., `smile`, `sassy-eyes`, `metal-wings`.)
- Copy the template and rename it to `{your-piece-name}.ai`.  
    (e.g., `smile.ai`, `sassy-eyes.ai`, `metal-wings.ai`.)
- Open the file. Replace `NEW-PIECE` in the `NEW-PIECE-wearing` and `NEW-PIECE-preview` layers with the name of your piece.

##### Design on the Wearing layer

Work on the art over the base model in the `wearing` layer first. When it's complete...

##### Specify custom color areas

First you'll want to import our [custom color palette](../assets/avatar-palette.ase). Using these colors *for the colorizable area* will make processing easier once art is delivered. You can use any colors you choose for the rest of the art.

Each Avatar piece should have one student-colorizable color. Join all shapes and paths that should be colorizable into one shape, and rename it `colorizable`. Fill it with a default color from the Avatar palette.

If an Avatar piece shows “skin” (hands, body, nose, ears, bare feet, etc.), name the skin color shape `skin`. This will allow us to inherit the color the student has chosen for the Avatar body.

##### Copy to the preview layer

Once the art and layer names are finalized, make a copy in the preview layer. Resize the artwork in the `preview` layer to take up most of the artboard. This is the layer students will see when browsing the Avatar builder store and their already purchased items.

##### Deliver the piece

That's it! Thanks!
