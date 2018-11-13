# Setting Up Git

Git and GitHub drive pretty much everything in engineering at TTM.  As such it's important to have your git information up to date and set to some safe defaults.

## Who Are You?

Git wants you to identify who you are by name and email so that you may be linked with your commits in a way we can all recognize.  We encourage the engineering team to use their `@thinkthroughmath.com` email address on their work machines so that you can separate out your work commits from personal projects.  You can [add your TTM email address to GitHub](https://github.com/settings/emails) so that GitHub knows how to identify you and can send notifications on TTM repos if you set it up in GitHub's [notification custom routing](https://github.com/settings/notifications).  This will help you keep your personal email from being flooded with TTM notifications.  The process is laid out in the section below.  If you do indeed want all of your GitHub emails all going to your personal email you can disregard this information.

### Git Config

Here's how to setup git on your machine to use your name and email in a terminal window

1. `git config user.email "you@thinkthroughmath.com"`
1. `git config user.name "Your Name"`
1. Verify your configs have been set correctly by running `git config user.email` - you should see the email you just entered

#### Register Email with GitHub

As an enumerated list of the information in the paragraph above, to register your email with GitHub

1. Go to the [Email Settings Page](https://github.com/settings/emails)
1. Enter your TTM email address (it does *not* need to be set as primary)
1. Go to the [Notifications Setting Page](https://github.com/settings/notifications)
  1. Setup the custom routing you'd like.
  1. If you want to route all TTM notifications to your TTM email, set it as such under `Custom Routing`


## Safety First!

All devs are encouraged to use Git 2 (available through homebrew).  If you are still on Git 1.x.x you will need to set your default push behavior to be "simple" with the command `git config --global push.default simple`. This is much safer than the default git behavior in 1.x and allows you to push your current branch with only `git push` in a safe manner as long as you have the upstream set.

## Git commands

Add your favorite git commands with what they do!

### Show unpushed changes

`git log origin/master..`

This will show you all the commits that you have locally that are not on the master branch on github, assuming that you have your github remote named `origin`. This is shorthand for `git log origin/master..current-branch`. You can switch the order to see what is on origin/master that you don't have locally, for example, if you haven't merged `origin/master` in yet.

### Clean up unpushed commits

`git rebase -i origin/master`

This starts an interactive rebase of all your local work you've done that isn't yet on origin/master. You will be placed into an editor that has instructions in comments at the bottom. This lets you reorder commits, edit them, squash together multiple commits into one, get rid of some commits completely, etc. This is incredibly useful for cleaning up your history into nice commits with one logical change per commit before you push to github.

### Make a commit of only some of your changes

`git add -up`

This is adding in patch mode (the -p part) of only files that git is tracking already (the -u part). Git will give you one "hunk" of changes at a time that you can choose to add or not (among other things). This lets you separate your changes into logical commits if you've been changing a few things at once. [This site has two awesome screencasts about adding in patch mode](http://johnkary.net/git-add-p-the-most-powerful-git-feature-youre-not-using-yet/).

### Undo your last commit but keep the changes

`git reset --soft HEAD~1`

I am still new to using `reset`, but I have needed it a few times. Specifically, when I accidentally committed db/structure.sql and needed to "redo" my last commit. I have referenced this SO post twice now and it also provides a link to the Git manual page.
[http://stackoverflow.com/questions/927358/undo-last-git-commit](http://stackoverflow.com/questions/927358/undo-last-git-commit)

### Make git yell at you if you don't put a ticket id in your commit message

You can add a git commit hook that will yell at you and not let you commit unless you have a ticket ID in the message (or explicitly say you're not going to add one).

The commit hook and instructions on installing it are [in this gist](https://gist.github.com/4073120) -- feel free to customize it too!

### View the files you have changed on a topic branch

`git diff --name-only review`

So assume you are working on a branch called "myTopic" and you want to see what files you have changed relative to the review branch. While you are on the "myTopic" branch running `git diff --name-only review` will show you the names of the files that have been changed.

### Delete local branches that have been merged

    git branch --merged | grep -v "\*" | xargs -n 1 git branch -d
    git remote prune origin

## Other useful git resources

* [Must Have Git Aliases](http://durdn.com/blog/2012/11/22/must-have-git-aliases-advanced-examples/)
* [Git: Twelve Curated Tips And Workflows From The Trenches](http://durdn.com/blog/2012/12/05/git-12-curated-git-tips-and-workflows/)
* [reflog](http://gitready.com/intermediate/2009/02/09/reflog-your-safety-net.html) - for when stuff goes REALLY wrong
* [Git workflow and resolving conflicts video](http://vimeo.com/33165748)
* [Git workflow part 2 video](http://vimeo.com/33166064)
* [Gaggle of git tips](http://viget.com/extend/a-gaggle-of-git-tips)
* [Git grep tips including a nice alias](http://travisjeffery.com/b/2012/02/search-a-git-repo-like-a-ninja/)
* [Add the current branch to the command prompt](http://www.developerzen.com/2011/01/10/show-the-current-git-branch-in-your-command-prompt/)
* [Git tips from the pros](http://net.tutsplus.com/tutorials/tools-and-tips/git-tips-from-the-pros/)
* [merge vs rebase](http://mislav.uniqpath.com/2013/02/merge-vs-rebase/)
