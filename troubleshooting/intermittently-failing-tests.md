## Intro

A test suite that passes or fails tests reliably and repeatably is vital to TTM being able to make software improvements at a fast pace. When tests fail intermittently, the team loses confidence in the suite and must spend valuable time trying to discern the "real" failures from the intermittent ones. Not only does this interrupt a developer's flow, but it also might result in a "real" failure slipping into the product because it was mistaken for an intermittent one.

Apangea is a complex app and making its integration tests run reliably is **hard**. But, here are a handful of guidelines for writing integration tests that should make them run more reliably:

## If it looks like the database isn't getting cleaned between tests on Jenkins

There's probably a test class that does not inherit from `UnitTest`, `IntegrationTest`, or `ApiTest`.

Database Cleaner is managed by `before` blocks included in the base test classes, so if the test doesn't inherit from those, the database won't get cleaned before or after that test.

No bueno:

```
# in file test/unit/whatever_test.rb
describe 'whatever' do
  describe 'when stuff'...
    it ...
    end
  end
end
```

Muy bueno:

```
# in file test/unit/whatever_test.rb
class WhateverTest < UnitTest
  describe 'when stuff'...
    it ...
    end
  end
end
```


## Use Capybara's automatic waiting

Capybara was built with asynchronicity in mind. It provides a way to write integration tests in a synchronous style even though the page under test might issue AJAX requests that update elements on the page asynchronously. When a test makes an assertion like:

    page.must_have_content('Banana runts are gross.')

(#must_have_content actually comes from the [CapybaraMiniTestSpec gem](https://github.com/ordinaryzelig/capybara_minitest_spec))

or

    page.has_content?('Banana runts are gross.').must_equal true

or

    assert page.has_content?('Banana runts are delicious.')

Capybara will query the page repeatedly until either:

 * The condition is true, or
 * A configurable timeout period has elapsed (which will result in a failed test).

With this in mind, tests should **not** be written in a way that bypasses this very handy Capybara behavior, such as:

    page.text.must_include('Banana runts are gross.')

Because Capybara wasn't told what the test is looking for (since the test just asks for `page.text`, which is the text of the page on initial load), Capy's automatic waiting logic won't be invoked. Therefore, most assertions on DOM elements should be made through Capybara's custom methods and not through Minispec's built-in matchers.

This [section of the Capybara README](https://github.com/jnicklas/capybara#asynchronous-javascript-ajax-and-friends) has more information.

## Don't navigate to another page after issuing an AJAX request
If you navigate to a new page after issuing an AJAX request, you may load the new page before the AJAX request has completed.  You may be tempted to use wait_for_ajax, but this is actually unreliable for most purposes, as it just waits to ensure that there are no AJAX calls in flight. However, it does not take into account any callback JavaScript called after the AJAX call returns.  Thus, a better way to ensure that an AJAX call was actually completed is to assert that the change on the page which you expected has occurred before moving on.  Capybara will automatically poll this for you, so it acts as a better wait_for_ajax call, albeit one which you must specify the expected change.

Consider this snippet of code from [student_listing_integration_test.rb](https://github.com/thinkthroughmath/apangea/blob/rc/test/integration/student_integration_test.rb).  The concept of the test is to go to a page of inactive users, check the "Active" checkbox (thus making the user active), then reloading the page and asserting that the student who was made active no longer appears on the "Inactive" listing page.

      visit list_students_path(show_active: 'Inactive')
      page.check ("active_#{student.id}")
      visit list_students_path(show_active: 'Inactive')
      assert page.has_no_unchecked_field? "active_#{student.id}"

However, this test will non-deterministically fail depending on if the AJAX call was completed before the page was reloaded.  Note that since the tests on the server are run on more performant hardware, often times you will see failures of this kind more often on the server.  In many cases, you may never see the failure at all locally.

A better way is to insert a call checking that whatever the action of the AJAX call is has been propagated to the browser.  For example, to take our earlier example, after clicking the 'Active' button, a JQuery fadeout (which takes > 400 ms) and then a remove is called on the row which contains the student.  If we wait until content corresponding to the user no longer exists on the page before moving on, we can ensure that the back-end changes have been made and all relevant JavaScript has been completed before moving on.

      visit list_students_path(show_active: 'Inactive')
      page.check ("active_#{student.id}")
      # Now we can't move on until this field has disappeared, meaning
      # the AJAX call is entirely complete
      assert page.has_no_content?("#{student.last_name}")
      visit list_students_path(show_active: 'Inactive')
      assert page.has_no_unchecked_field? "active_#{student.id}"

Note that in this specific case, a wait_for_ajax call would actually work, since we are reloading the page and a 200 OK would indicate that all back-end changes have been made and persisted to the database.  However, this also checks that the callback is working correctly, and is the proper method for testing AJAX calls in general.

## Lack of explicit database ordering
TODO

## Don't make external web requests (especially via `script` tags)

Capybara will fail a test if downloading external content fails. This is true for both data retrieved via AJAX calls and scripts that are loaded from an external source. Making requests to the internet during a test not only slows it down, but also runs the risk of causing an intermittent failure if there is a hiccup in the network/internet while the test runs.

With this in mind, external web requests should be avoided in tests. For AJAX data that comes from external sources, we use [a gem called VCR](https://github.com/vcr/vcr) to record web requests and play them back during integration tests. This technique is currently in use to stub out data from [Clever](https://github.com/thinkthroughmath/apangea/blob/rc/test/integration/clever_importing_integration_test.rb) and from the [data warehouse](https://github.com/thinkthroughmath/apangea/blob/rc/test/integration/leaderboard_report_integration_test.rb).  

TODO: talk about stubbed out script tags for Bugsnag, Nitro, and MathJax
