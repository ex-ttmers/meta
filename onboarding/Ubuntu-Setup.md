## From bare metal to passing integration tests in one easy lesson ...

### Preliminaries

* Your userid must be a sudoer, since many of the Linux commands require sudo
* You must have a GitHub account (Jim W)
* You must have a Heroku account (Chris S)
* If you are going to use SSH authentication to GitHub as I did, see SSH setup at the end. Otherwise everything will still work, but GitHub will ask for a username/password every time.

### My Config

* Hostname: ci.thinkthroughmath.com
* IP Address:  
* Distributor ID:	Ubuntu
* Description:	Ubuntu 12.04.1 LTS
* Release:	12.04
* Codename:	precise

### Basics

    # You probably have some of these already but it won't hurt to install them again. apt-get will just inform you that the program already exists.
    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo apt-get install curl 
    $ sudo apt-get install git-core
    # this is how you will log into GitHub
    $ git config --global user.name "John Doe" 
    $ git config --global user.email johndoe@mail.com
    # X11 is used by some of the integration tests - not sure why
    $ sudo apt-get install xorg


### RVM and Ruby

    # Make sure RVM is not already installed
    $ env | grep rvm
    # if no output, skip to Install RVM. If there is output, remove rvm:
    $ sudo apt-get --purge remove ruby-rvm
    $ sudo rm -rf /usr/share/ruby-rvm /etc/rvmrc /etc/profile.d/rvm.sh
    # Install RVM
    $ curl -L get.rvm.io | bash -s stable --auto
    $ sudo apt-get install build-essential openssl libreadline6 libreadline6-dev curl git-core zlib1g zlib1g-dev libssl-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt-dev autoconf libc6-dev ncurses-dev automake libtool bison subversion pkg-config
    # Install Ruby
    $ rvm install 1.9.3


### Install Postgresql
```
$ sudo apt-get install postgresql postgresql-client postgresql-server-dev libpq-dev postgresql-contrib postgresql-common pgadmin3
$ sudo -u postgres createdb apangea_test
```

### Change trust settings for local clients

    $ sudo vi /etc/postgresql/9.2/main/pg_hba.conf
    ...make it look like the following:
    local   all             all                                     trust
    host    all             all             127.0.0.1/32            trust
    host    all             all             ::1/128                 trust
    $ sudo /etc/init.d/postgresql restart


### Install Capybara

    $ sudo apt-get install libqt4-dev
    $ sudo apt-get install libcurl3 libcurl3-gnutls libcurl4-openssl-dev
    $ cd <apangea-app-home>
    $ gem install capybara-webkit
    $ sudo apt-get install imagemagick

Also download and the Linux binary for
[http://phantomjs.org/download.html](PhantomJS).


### Configuration Specific to Integration Tests

#### Xvfb

This is necessary for integration tests. Capybara-webkit needs an
X server to be running. However, for CI, it does not need a
window manager. It will not hurt to run CI with X11 and a window
manger, or to run it on a full-blown Linux desktop environment
like Gnome or KDE. But the minimum requirement is a virtual frame
buffer like Xvfb. To run it do the following.

    $ Xvfb :1 &

What this command does is start Xvfb in the background on display
:1. Integration tests (or Jenkins) must set the environment
variable DISPLAY to the value :1. The Xvfb command should be
configured with a script in /etc/init.d so that it will start
each time the server is rebooted. The system administrator will
probably have a different way to set it up, the important point
being that it will be running when integration tests need to use
it. A more sophisticated approach, involving starting and
stopping xvfb before and after each CI build, can be found
[here](https://gist.github.com/974392).

#### Javascript runtime engine

    $ sudo apt-get install nodejs

