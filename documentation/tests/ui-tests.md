## UI Tests

### What are User Interface tests?

UI tests render pages in different environments and verifies that they have not significantly changed.

### How do they work?

We use [BrowserStack](https://www.browserstack.com/) and [Compatriot](https://github.com/carols10cents/compatriot) for performing our UI Tests.

BrowserStack is used as our environment host and Compatriot is used for taking and comparing screentshots.

The entire suite of UI tests are triggered by running `rake ui_tests`.  This will run all our UI tests for each environment defined in [browsers.json](https://github.com/thinkthroughmath/apangea/blob/jk/compatriot/test/browsers.json).  We will pass the browser options off to browserstack and they will setup and link us to an environment to run tests against.  We configure selenium to handle driving that browserstack environment. so then we can write tests just like unit and integration tests.  The one thing that is different is that we have added a new assertion, `assert_no_ui_changes`.  We use this new assertion to compare the current page in the test, to a screenshot of the same page that we call a 'control image'.  As a result, we will get a percentage changed, and we compare that to a defined threshold to decide if it passes or fails tests.

All images are stored in `test/screenshots/<environments>/`.  We check the 'control' directory into source control as our base images to compare to.

After the tests run, we can see a `diffs` and `variable` directory.  Diffs are the result of comparing our control image with our variable image.

### How to run UI tests from jenkins

See the [ui_tests job](http://jenkins.thinkthroughmath.com:8080/job/ui_tests/)

### How to run UI tests locally

1. Download and run BrowserStackLocal

        wget https://www.browserstack.com/browserstack-local/BrowserStackLocal-darwin-x64.zip
        unzip BrowserStackLocal-darwin-x64.zip
        ./BrowserStackLocal -localIdentifier "${HOSTNAME}" -skipCheck \<token\> 127.0.0.1,4000

2. Set a local environment variable `BROWSER_STACK_AUTH`. If you have a browserstack account, you can get it from the Automate section on https://www.browserstack.com/accounts/settings. This variable value should be formatted as username:token. Setting it would look like: 

        export BROWSER_STACK_AUTH=jeffkoenig:abc12357401dnfsdd1.

3. run the rake task

        rake ui_tests
        
### Run individual tests locally

1. After going through steps 1-3 above
2. You can use `m` to run an individual test locally. It will only run it for the default environment defined in [test/ui_test.rb](https://github.com/thinkthroughmath/apangea/blob/rc/test/ui_test.rb)

  - You can override the default environment used, by setting the following environment variables in your shell, like so..

                export BS_AUTOMATE_OS='Windows'
                export BS_AUTOMATE_OS_VERSION='7'
                export SELENIUM_BROWSER='ie'
                export SELENIUM_BROWSER_VERSION='10'
                
                
  - Or you can modify [browsers.json](https://github.com/thinkthroughmath/apangea/blob/jk/compatriot/test/browsers.json), and use `rake ui_tests`
  
### Bypassing BrowserStack locally

* If you're creating or debugging a test and want the test to run a little quicker, you can eliminate browserstack from the equation by changing the driver in [test/ui_test.rb](https://github.com/thinkthroughmath/apangea/blob/rc/test/ui_test.rb)

                Capybara.default_driver = :webkit
  
  When changing the driver, it is no longer necessary to start BrowserStackLocal.

### Adding new Tests

1. Write your new tests in `test/ui/<yourtest>.rb`
2. Generate base line screenshots for your new tests
   
   1. Start BrowserStackLocal using the steps listed above
   2. Then, to run your test against each environment, run 
   
            rake ui_tests TEST=test/ui/<yourtest>.rb
            
   3. Commit the newly added screentshots
