Our current deploment process is based around git and bash. To summarize we have a custom bash script that runs on each server 
in the farm. It checks out the code from git and runs commands to restart unicorn, node or sidekiq.

Its a very static system and does not easily support new technolgies. To that rememdy that we have started working on a new system that
will provide some more flexibility.

Our plan for the revamped deployment process is as follows.
- Jenkins will build a .deb or .tgz package using https://github.com/crohr/pkgr
- This will them be uploaded to an S3 bucket
- The deploy script will download this package and install it
- Unicorn, node, or sidekiq will be appropriatly restarted.

Pkgr

Pkgr https://github.com/crohr/pkgr is a tool that uses Heroku build packs to make a self contained package of an application. In theory
any thing thats supported by a heroku build pack should work. So far we have tested node and ruby. The package includes application code
and runtime. So whatever version of ruby the app uses including its gems will all be packaged up into the .deb, similar for node, go 
etc...

Ruby

Becasue pkgr uses heroku build packs, they assume heroku dependencies. Heroku only supports Ruby 2.2.4 on its cedar-14 stack which is
based on Ubuntu 14.04 LTS. Luckily for us the ruby build pack is configurable. So we have to tell where to get ruby dependencies and give
it an optional stack parameter that is uses in the following way. https://s3.amazonaws.com/ttm-pkgr/rubies/STACK/

We are still on Ubuntu 12.04 so we have to provide ruby, bundler and libyaml for the build pack.

Packaging dependencies

Ruby

- Download a precompiled ruby from http://rubies.travis-ci.org/
- Convert to gz format.
   - Extract to a folder
   - cd into the folder then tar -cvzf ../ruby-2.2.4.tgz .
   - note the ../ruby-x.x.x 
   - upload to S3 bucket

Bundler

- Download the bundler gem from http://rubygems.org
- convert to gz format
   - gem install -i some_temp_folder bundler-x.x.x.gem
   - cd some_temp_folder that tar -cvzf ../bundler-x.x.x.tgz .
   - upload to S3 bucket

libyaml

- Libyaml will have to be download and compiled.
- Download from http://pyyaml.org/download/libyaml/
    - Build on either a 12.04 Virtual box or debug role on one of the farms
    - extract and run `configure --prefix some_tmp_folder`
    - make ; make install
    - cd some_tmp_folder ; tar -cvzf ../libyam-x.x.x.tgz .
    - upload to S3 bucket

Then to use them to make a package execute pkgr as follows

`````
pkgr package \
  --env "RAILS_SECRET_TOKEN=" \
  "AWS_ASSETS_BUCKET=" "AWS_ACCESS_KEY=" \
  "AWS_ACCESS_KEY_ID=" \
  "DATABASE_URL=" \
  BUILDPACK_VENDOR_URL="https://s3.amazonaws.com/ttm-pkgr/rubies" "STACK=\"\"" \
  "ROLE_ID=Debug.1" "BUNDLE_WITHOUT=\"development test\"" \
  --name=apangea --user=www-data --group=www-data \
  --buildpack="https://github.com/thinkthroughmath/heroku-buildpack-ruby.git" \
  --verbose /var/www
`````

Becasue the buildpack builds and includes assets it executes `rake assets:precompile`. This task boots rails so you will have to set
all the environment variables in the above command so that rails is able to boot without errors.
