This should work for both RubyMine and IDEA + Ruby plugin. YMMV.

Since we use Foreman to launch processes, including Rails, you can't follow the normal procedure out of the box for running the app in the IDE and to enable debugging hooks. So the trick is to create a separate `Procfile` for running everything _except_ Rails, then run Rails inside the IDE.

1. Copy `Procfile` to `Procfile.debug`

2. Remove the line in `Procfile.debug` that starts with 'web:...'

3. Start Foreman with a separate file:

    $ bundle exec foreman start -f Procfile.debug

4. Setup the IDE for debugging

If you setup your Rails project properly IDEA will give you a couple of run configurations by default, like this:

![Run configurations](idea_run_configurations.png)

Since our setup is tied to port 5000 (particularly wrt assets), I
had to change that before debugging. Click 'Edit
Configurations...' in the menu above, then choose the appropriate
one -- in this case, 'Development'. Also choose the 'default'
server rather than 'Unicorn':

![Edit run configuration](idea_edit_run_configuration.png)

Once that's done, click the Debug button (the green arrow with a
bug on it, ho ho) or hit Shift-F9. The first time you do so IDEA
will complain that it needs to download some additional gems
related to debugging. And when it says it will 'take a while' to
build, it's not kidding. But be patient. For soon you'll be
rewarded with a happy debugger. Set breakpoints, evaluate
conditions, set watches, etc.

This worked fine using the 'default' webserver; using 'unicorn'
gave me some odd issues and I didn't see any point in continuing
with it.
