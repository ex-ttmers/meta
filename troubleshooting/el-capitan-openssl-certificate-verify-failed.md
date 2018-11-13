### El Capitan OpenSSL Error: certificate verify failed

When upgrading to Mac OS X El Capitan and trying to run `rake ttm:rebuild_with_production_content` on the Apangea Repo you receive the error below during the _Seeding users_ task.

```bash
SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed
```

Upon further investigation this error can be be reproduced by running the command below in the Rails Console, because this is the command it is failing on during the task.

```ruby
Kernel.open("https://content.thinkthroughmath.com/uploads/image/image/20148/gamer-large.png")
```

To fix this issue you will simply need to re-install Ruby and pass the OpenSSL directory that `brew` creates during its installation. Please see the commands listed below on how to accomplish this fix.

#### Note
It may seem out of order to install `therubyracer` first, but it is needed.  Uninstall libv8 first if you are having issues, then reinstall after with system-v8.

```bash
brew uninstall openssl
brew update
brew install openssl

rvm install 2.1.7 --with-openssl-dir=/usr/local/Cellar/openssl/1.0.2d_1  # OpenSSL Directory that brew installed into from the previous step

# In the Apangea Repo directory
gem install bundler
gem install therubyracer -v '0.12.1'
gem install libv8 -v '3.16.14.7' -- --with-system-v8

bundle install
```
