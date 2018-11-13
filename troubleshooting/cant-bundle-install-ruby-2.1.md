### Can't Bundle Install With Ruby 2.1 - capybara-webkit errors

Does capybara-webkit refuse to build when you `bundle install` with Ruby 2.1?  You may get errors like the following:

```
ERROR:  Error installing capybara-webkit:
ERROR: Failed to build gem native extension.

...

fatal error: too many errors emitted, stopping now [-ferror-limit=]
20 errors generated.
make[1]: *** [build/webkit_server.gch/objective-c++] Error 1
make: *** [sub-src-webkit_server-pro-make_default-ordered] Error 2
```

If so, you need to install the latest qt5.  From the command line, run:

```
brew install qt5
```

After that, set the QMAKE environment variable:

```
set QMAKE=/usr/local/Cellar/qt5/5.4.1/bin/qmake; export QMAKE
```

You may want to add that last line to your .bash_profile / .bashrc / whatever file your shell reads at startup.

Finally, `bundle install` again and everything should work.

### If you'd like more information

If you want the details of what is going on, please see: https://github.com/thoughtbot/capybara-webkit/wiki/Installing-Qt-and-compiling-capybara-webkit#video-playback-mp4-on-osx-requires-qt-5
