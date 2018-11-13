# Hound

[Hound](https://houndci.com) is a service run by Thoughtbot that will run various code style tools
on each pull request and comment on code that changes in that pull request that doesn't follow
the styles that we've configured it to check for. [Hound is actually open source](https://github.com/thoughtbot/hound) and free for public repos, but we pay them to check our
private repos for us.

The ttm-jenkins github user is the manager for the hound integration, and it is configured to email
the receipt to expenses@thinkthroughmath.com. Jeff manages github 2FA for ttm-jenkins, so ask him for
help signing in if you need it.

If hound comments on your pull request, we treat that similarly to a code review comment and it's
something you should fix.

If hound comments a LOT on your pull request, the hound service probably just updated the tools it
uses. They probably added new rules that we haven't had a chance to disable and we don't abide by--
so to fix it we update the configuration files that hound uses.

If you would like to enable a rule that we don't already use, create a pull request that turns it
on and fixes all currently-existing violations, then draw the team's attention to the PR to get
consensus about whether this is something we want to enforce as a group or not. If it's a change
that requires a lot of work to get our codebase to conform to, you should probably look for
consensus with the group *before* doing that work.

## Config files

In apangea, the hound config files are in `config` and all start with a `.`. In lesson player, the
hound config files are in the root directory and all start with a `.`. In the rest of this page,
this directory will be referred to by `$CONFIGDIR`.

If these files are moved around, the values in the `.hound.yml` will need to be updated (and that's
a good place to look if this wiki page appears to be incorrect).

## Ruby

The ruby style checking is done by [rubocop](https://github.com/bbatsov/rubocop), which is in
apangea's Gemfile so you have it if you've `bundle`d. If you'd like to run rubocop locally on all
our ruby code with the config hound uses:

    $ rubocop -c $CONFIGDIR/.rubocop.yml

Many cops support autocorrect, which is awesome since it's less likely to introduce human errors
(if you trust rubocop's code ;) You can see which cops support autocorrect by running `rubocop --show-cops`, and then use autocorrect by doing `rubocop -c config/.rubocop.yml --auto-correct`.

## Coffeescript

The coffeescript style checking is done by [coffeelint](http://www.coffeelint.org/). Install it
with `npm install -g coffeelint`, and then you can run it on all our coffeescript files in apangea
by doing:

    $ git ls-files *.coffee | xargs -I {} coffeelint -f $CONFIGDIR/.coffeelint.json {}

or this is what appeared to work better in lesson-player:

    $ git ls-files | grep coffee | xargs -I {} coffeelint -f $CONFIGDIR/.coffeelint.json {}

There's probably a better way to do this, suggestions welcome!

## Javascript

The javascript style checking is done by [jshint](https://github.com/jshint/jshint/). Install it with `npm install jshint -g`, and then you can run it on all our js files in apangea by doing:

    $ jshint --verbose -c $CONFIGDIR/.jshint.json --exclude-path $CONFIGDIR/.javascript_ignore .

JS from third parties that we do not maintain should be ignored in the
`$CONFIGDIR/.javascript_ignore` file.

Some of the warnings are a bit... obfuscated and can only be configured with W or E and a number
which only appears when running with `--verbose`. [jshint would love help documenting these](https://github.com/jshint/jshint/issues/1734).

Useful docs:
* [Hound's default config that our config gets MERGED with](https://raw.githubusercontent.com/thoughtbot/hound/master/config/style_guides/javascript.json)
* [Explanation of all of jshint's options](http://jshint.com/docs/options/)

## SCSS

The SCSS style checking is done by [scss-lint](https://github.com/causes/scss-lint). Install it with `gem install scss-lint`, and then you can run it on all our SCSS files in apangea by doing:

    $ scss-lint --config $CONFIGDIR/.scss.yml app/assets/stylesheets/**/*.scss

## Haml

The Haml style checking is done by [haml-lint](https://github.com/brigade/haml-lint). Install it with `gem install haml_lint`, and then you can run it on all our Haml files in apangea by doing:

    $ haml-lint --config $CONFIGDIR/.haml.yml app/**/*.haml
