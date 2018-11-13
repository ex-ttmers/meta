# Feature Rollouts

## Global Features

### TL;DR HOW TO MAKE A FEATURE FLAG FOR ROLLING A FEATURE OUT

Add this to `lib/basic_features.rb`:

```ruby
class MyFancyFeature < GlobalFeature
  def description
    "Replace this with an explanation for your fellow developers of what this flag does and when this flag can be turned on and removed."
  end
end
```

Activate it like this (from the rails console):

```ruby
FeatureRollouts.activate :my_fancy_feature
```

Detect if it is being used like this:

```ruby
FeatureRollouts.active? :my_fancy_feature
```

### To see which flags are active or inactive in your system:
- `FeatureRollouts.active`
- `FeatureRollouts.inactive`
- Visit host/feature_rollouts as a superuser

### Current Features

Please see the flags defined in `lib/basic_features.rb`; each should have a description that explains what it's for. Please add these descriptions if you add a new flag!! If you `git grep` for the symbolized, underscore-separated version of the flag name, you will get all the uses of the flag.

### Update Details

An overhaul has been made on the feature flag code in Apangea.

#### A `FeatureRollouts` constant/class is introduced

This class has some convenient class methods for querying and modifying
feature flags.
See the tutorial section for details.

#### `FeatureFlags` renamed to `FeatureRollouts`

The problem with the former name is that feature "flag" tends to not
convey the impermanence of these pieces of code. A "flag" suggests
that a feature could be turned on or off for a given situation.

Really, we are using these to control the roll out of new features,
bug fixes, etc.


#### `FeaturesForUser` was removed

This seemed to be confusing and more trouble than it was worth.

As part of this change, `current_features`, that was available in application
controller, has been removed. It was never really used. All feature
checking should go through `FeatureRollouts`.

### Tutorial

#### Add a New Feature Flag

To add a new feature flag, you must:

1. add a new class inside `lib/basic_features.rb`

Lets go over these steps by implementing a new feature.

##### Creating a New Class

Each feature has a corresponding class which holds the logic
associated with that feature. A feature ought to inherit from the
`FeatureRollouts::Feature` class.

Lets pretend we are building Star Trek into our application. We'll
call the feature `StarTrek`.

Within `lib/basic_features.rb`, add a new class and a description:

```ruby
class StarTrek < GlobalFeature
  def self.description
    "Adding Star Trek functionality to the student math experience. Turn on before Star Date 41153.7 because we promised it to the Klingons by then."
  end
end
```

There are some methods that can be overridden in order to add more
behavior. For now, lets pretend this feature is controlled with an
environment variable:

```ruby
class StarTrek < GlobalFeature
  def active?
    ENV['PICARDERIZE'] == 'engage'
  end
end
```

Now, if `PICARDERIZE=engage` is set in the `.env` file before foreman is
started, the `StarTrek` feature will be enabled.

Of course, we don't *actually* use environment variables to control
feature enablement in the application because reasons. We use Redis
and Rollout. So, lets switch to using that:

```ruby
class StarTrek < GlobalRolloutFeature
```

Now we can enable/disable Star Trek through rollout. The key (`flag_name`) that
rollout uses will automatically be generated from the class name; in
this case, it would be `:star_trek`.

#### Querying Feature Flags

The simplest way to query the state of a feature flag is to do:

```ruby
FeatureRollouts.active?(:star_trek)
```

If your feature requires a user to determine if it is active, you
should do this:

```ruby
FeatureRollouts.active?(:star_trek, student)
```

##### In JavaScript

If you need a feature rollout in JS, use the `window.ttm.feature_rollouts_data` object.

#### Listing feature flags

A list of all feature flags that are active can be generated like
this:

```ruby
2.0.0-p481 :001 > FeatureRollouts.active
 => [:infinite_repeater, :mcr_updates, :create_evaluate, :create_evaluate_key_entry]
```

This is helpful for getting an idea of what features are active
currently.

#### Activating and Deactivating Flags

In order to activate a feature flag, use the activate method:

```ruby
FeatureRollouts.activate(:star_trek)
```

Conversely, to deactivate a feature:

```ruby
FeatureRollouts.deactivate(:star_trek)
```
The individual `Feature` classes will know how to activate this
feature. If they cannot be activated in this way, an exception will be
raised.

If a feature requires more data before a feature can be activated,
you can pass it as an options hash:

```ruby
FeatureRollouts.activate(:star_trek, {:student => student})
```

#### Grepping a Code Base to Remove a Feature

If we are consistent about how we check for features, removing a flag
should be much easier. For example, I can easily figure out where in
the app features are being tested for:

```sh
2  JoelMcCrackens-MacBook-Pro-3 in ~/vagrant-environment/apangea
±  |origin-jnm+sar-refactor_feature_flags ✗| → git grep FeatureRollouts.active?
app/controllers/classroom_students_controller.rb:      if FeatureRollouts.active? :mcr_updates
app/controllers/live_help_requests_controller.rb:    is_new_lesson_player = FeatureRollouts.active?
app/controllers/pathway_enrollments_controller.rb:    if FeatureRollouts.active? :mcr_updates
app/controllers/student_dashboards_controller.rb:    @is_new_avatar_widget_enabled = FeatureRollout
app/controllers/students_controller.rb:      FeatureRollouts.active?(:new_lesson_player, student)
app/decorators/item_decorators/create_evaluate_item_decorator.rb:    options.push "input" if Featur
app/decorators/student_list_decorator.rb:    @mcr_updates = FeatureRollouts.active? :mcr_updates
app/models/activity.rb:    if FeatureRollouts.active? :new_lesson_player, user
app/models/activity.rb:      (type == Activity::WARM_UP && activity_types.include?(Activity::WARM_U
app/models/enrollment.rb:      if !working_on_this_enrollment? && working_on_this_classroom? && !Fe
app/models/enrollment.rb:        unless FeatureRollouts.active? :mcr_updates
app/models/enrollment_activity.rb:    which_lesson_player = FeatureRollouts.active?(:new_lesson_pla
app/views/student_dashboards/_get_started.html.slim:- if FeatureRollouts.active? :mcr_updates
app/workers/pathway_lesson_change_worker.rb:    return unless FeatureRollouts.active? :reproject_en
```

Or, if I want to remove a specific flag:

```sh
JoelMcCrackens-MacBook-Pro-3 in ~/vagrant-environment/apangea
layerrigin-jnm+sar-refactor_feature_flags ✗| → git grep FeatureRollouts.active?\.:new_lesson_player
app/controllers/live_help_requests_controller.rb:    is_new_lesson_player = FeatureRollouts.active?
app/controllers/students_controller.rb:      FeatureRollouts.active?(:new_lesson_player, student)
app/models/activity.rb:    if FeatureRollouts.active? :new_lesson_player, user
app/models/activity.rb:      (type == Activity::WARM_UP && activity_types.include?(Activity::WARM_U
app/models/enrollment_activity.rb:    which_lesson_player = FeatureRollouts.active?(:new_lesson_pla
```

Etc. Since we want to make removing dead code as pain-free as
possible, it is a good idea to follow this pattern. Eventually we can even write a rake task to help run these commands.

## Customer Features

### Overview

These flags can be turned on and off for specific customers. The flags are a `key:value` hash
in Redis where the key is the flag name and the value is a list of customer ids that are in
said 'feature'.

### Creating a new customer feature

Add this to `lib/customer_features.rb#features`:

```ruby
def features
  {
    my_super_secret_flag: "shows all the secret flags"
  }
end
```


### Useful commands

To add a customer to a feature:

`CustomerFeatures.add_to(:feature, customer_id)`

To remove a customer from a feature:

`CustomerFeatures.remove_from(:feature, customer_id)`

To activate a feature:

`CustomerFeatures.activate(:feature)`

To deactivate a feature:

`CustomerFeatures.deactivate(:feature)`

To check if a customer is in a feature AND that feaure is active:

`CustomerFeatures.active_for_customer?(:feature, customer_id)`

To enable a customer feature flag globally

`CustomerFeatures.enable_globally(:feature)`

To disable a customer feature flag globally

`CustomerFeatures.disable_globally(:feature)`

### Javascript access

To get a list of active features for the currently logged in user in Javascript
use the global variable:

`window.ttm.customer_features_data`

### Removing flags

Our team's agreed upon process is to __always__ create a ticket to remove
the feature flag when a ticket to add it is created. These tickets should
both be assigned to the milestone associated with the project and the project
will not be considered "complete" until the feature flag has been removed.

Remove the flag from the `lib/customer_features#features` hash

Grep the code base for:

`CustomerFeatures.active_for_customer?(:feature_flag`

and

`window.ttm.customer_features_data.feature_flag`

To remove a feature flag from redis:

`CustomerFeatures.delete_feature!(:feature)`
