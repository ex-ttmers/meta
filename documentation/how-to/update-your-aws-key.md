# Update your AWS key

If we need to rotate our AWS keys, you will need to update them locally.

You can get values for any farm's configuration variables, which includes AWS keys, by running:

    ttmscalr config:get -f <farmname>

The keys you re looking for are `AWS_ACCESS_KEY` and `AWS_ACCESS_KEY_ID`. Some scripts, like `db/archiving/school_restorer.rb`, require you to have your AWS keys set in your current environment. If you run:

    rake ttm:export_aws_keys

it will output a few lines that you can copy-paste and run to set these environment variables in your current terminal session.

Do not commit these keys to your dotfiles and definitely do not push these values to public github repos!

To make this work automatically:

    rake ttm:export_aws_keys | sed 's/export //' > ~/.ttm_aws_credentials
