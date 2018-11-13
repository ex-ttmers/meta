Since a math_content item has many reproducible/standard components, and since there are many basic steps to creating the new item, why not make a rails generator to do this for us?  It will speed up generation time and (given it is correct) minimize errors.

So let's do it!

````
$ rails generate math_content sexy_buckets
$ rake db:migrate
````

And in your console:
````
$rollout.activate_group(:sexy_buckets, :all)
````

Then restart your rails server.

#### Done!
#### You now have a new math_content item (without any item specific attributes) ready to go!

***

### So what files were created/modified?

* app/models/feature_flags/features_for_user.rb
 * $rollout flag added
* db/migrate/20140729192848_add_sexy_buckets_item_presenter.rb
 * This adds sexy_buckets into the item_presenters table
* app/models/item_presenter.rb
 * Adds sexy_buckets to the ITEM_TYPE array
 * creates an instance method ````self.sexy_buckets````
* app/controllers/item_presenters_controller.rb
 * Adds sexy_buckets onto the stack of item_types that can be created (behind a rollout flag)
* app/models/item_types.rb
 * Create the SexyBuckets ItemType that inherits from MathContent
* app/views/items/_sexy_buckets.html.slim
 * View for initial item creation (item_number, grade_level, etc....)
* app/views/math_content/sexy_buckets
 * Created for sexy_bucket unique views
* app/views/math_content/sexy_buckets/_form.html.slim
 * The initial cms view that holds the stem and helps
* lib/serialize/sexy_buckets_serializer.rb
 * Basic serializer (stem and helps)
* app/decorators/item_decorators/sexy_buckets_item_decorator.rb
 * Initial class for the decorator (nothing in it really)
* app/json_builders/json/math_content/sexy_buckets_item_builder.rb
 * Stubs the SexyBucketsItemBuilder

[See the Lesson Player steps on building a new item type here.](https://github.com/thinkthroughmath/lesson-player/wiki/Building-An-Item-Type)
