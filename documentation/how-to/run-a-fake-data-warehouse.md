# Fake Data Warehouse

While testing District Admin Reporting, I found that a useful utility would be a "fake" data warehouse.  I wrote a very simple Ruby / Sinatra script so that it will return arbitrary JSON as a response.  You can thus put your own, edited JSON to get whatever results you want.

Note that for this current version, ALL requests (no matter what the params) will always return the same JSON.  I may make some 10%-time improvements to this to allow more specialized returns if there is interest.

## Installation Instructions

First, install the sinatra and thin gems.

```
gem install sinatra
gem install thin
```

Save the following Ruby script to a local file, fake_reporting.rb.

```ruby
require 'sinatra'

set :port, 3000

get '*' do
  content_type :json
  response = ARGV[0]
  response ||= 'response.json'
  File.read(response)
end
```

## Execution Instructions

Save the JSON you want to return to a file in the same directory as fake_reporting.rb.  By default, the script will look for response.json, but you can define others.

NOTE: get this via curl or a similar tool; Chrome and Firefox will pretty it up and display it as "pretend JSON", which will *not* work.  Try something like:

```
curl http://localhost:3000//district_usage?district=500001&group=school > noogie.json
```

Now run the program, with a path to the JSON file (starting from current working directory) as the first argument.  If this argument is not included, it will use the default file "response.json".

```
ruby fake_reporting.rb noogie.json
```

You're now running a fake data warehouse on port 3000.  You can go to localhost:3000/whatever and see the JSON that is being sent back whenever a request is made.

## Running A Fake-Broken Data Warehouse

Another script that can be handy will run a web server which will always respond with a status code of your choice.  This is useful when seeing what will happen when you get 404s, 500 errors, etc.  Save this as fake_status.rb on your local machine.

```ruby
require 'sinatra'

set :port, 3000

status_code = ARGV[0].to_i

get '*' do
  error status_code, "RETURNED STATUS CODE #{status_code}"
end
```

Run by just passing the status you would like returned as an argument.
```bash
ruby fake_status.rb 500
```

You can test that it's working by using curl to ensure that the proper status code is returned.

```bash
(6619) $ curl -I localhost:3000
HTTP/1.1 500 Internal Server Error
```
