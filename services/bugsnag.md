# Bugsnag

Bugsnag is our error capturing/reporting service. It currently tracks ruby errors from apangea and javascript errors from lesson player.

TTM shares a single Bugsnag account.

These credentials are also in Lastpass

[Documentation for Ruby Runtimes](https://bugsnag.com/docs/notifiers/ruby)

## Github integration

Bugsnag is configured so that you can click on a button in bugsnag to create an issue for it in the thinkthroughmath/storyboard repo. Bugsnag will also reopen issues if closed ones created from bugsnag
reoccur-- sometimes this is annoying, say, if the fix has been made but not deployed yet. You can
tell bugsnag to unlink from the issue in that case.

The integration is done by giving bugsnag a github access token that was created by the ttm-jenkins github account.
Jeff manages the github 2FA for ttm-jenkins, so ask him if you need help logging in.

## Sending Error Report Emails While in Development

You can send emails from development by updating the [config values](https://bugsnag.com/docs/notifiers/ruby#configuration) to include development:

<pre>
Bugsnag.configure do |config|
  config.notify_release_stages = ["development"]
end
</pre>

So the above tells bugsnag to send emails ONLY in development. This can be useful to see what info will be contained in your email. This only lasts for the length of the session/runtime.
