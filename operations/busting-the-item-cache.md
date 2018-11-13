TTM has hooks built in to delete and re-create (bust) the cache for items when they are edited. If this process doesn't work for some reason it's possible to bust the cache manually for a specific item:

*Note* This used to include Lessons but it proved to be problematic so now we are back to caching only individual items and item steps.

* Use `ttmscalr` to get a rails console on the affected farm (`prod`, `rc`, etc.)
* At the console enter `Json::ItemCache.new.bust <item_id>`
* The console will return `1` when it is finished successfully. Any other result is probably an error.
