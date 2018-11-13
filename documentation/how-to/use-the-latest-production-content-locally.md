# Use the latest production content locally

To copy the most recent production content (lessons, items, pathways) to your local environment:

`ruby db/copy_sharded_content.rb --from production --to local`

To copy the most recent production game versions (this is necessary so the LP will know where the game resources are located):

`ruby db/copy_game_versions.rb --from ttm-production --to local`

This will only reset content.  If you want to rebuild your user database too, see "Rebuild with production content" in the [Readme](https://github.com/thinkthroughmath/apangea/).
