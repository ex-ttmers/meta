### Setup Production Data Locally

#### From your local apangea directory:

- Get your AWS environment variables up to snuff (if you have these in your .env, you can bypass this and prefix `foreman run` to the `ruby` calls in future steps):

  ```
  $ $(rake ttm:export_aws_keys)
  ```

- Set your PGHOST environment variable to "localhost" by running this (if you have these in your .env, you can bypass this step and prefix `foreman run` to the `ruby` calls in future steps):

  ```
  $ export PGHOST=localhost
  ```

- Change directory to where the archiver is:

  ```
  $ cd db/archiving
  ```

---

##### For one or many schools in the same customer:

- Extract one school, or many schools in the same customer, using the school extractor and the school's ID (note - you may need to precede the following command with `bundle exec`):

  ```
  $ ruby extractor.rb --env ttm-production --school 123
  ```

  or

  ```
  $ ruby extractor.rb --env ttm-production --school 123,456,789
  ```

- Restore the school, or schools, you extracted (note - you may need to precede the following command with `bundle exec`):

  ```
  $ ruby school_restorer.rb --skip-schema --restore-core --school 123
  ```

  or

  ```
  $ ruby school_restorer.rb --skip-schema --restore-core --school 123,456,789
  ```

##### For a whole customer:

- Extract and restore a whole customer using the customer restorer and the customer ID (note - you may need to precede the following command with `bundle exec`)

  ```
  $ ruby customer_restorer.rb --skip-schema --from_env ttm-production --customer 123
  ```

  or

  ```
  $ ruby customer_restorer.rb --skip-schema --from_env ttm-production --customer 123,456
  ```
  
  or provide a 'to_env' (production is not allowed)
  
  ```
  $ ruby customer_restorer.rb --skip-schema --from_env ttm-production --to_env lab --customer 123
  ```
  
  

---

- In your reporting repo, reload your warehouse databases ([you can also see the long version in the rc readme](https://github.com/thinkthroughmath/reporting/blob/rc/README.md))

  ```
  $ PGHOST=localhost etl/reload_reporting
  ```

- Refer to [the apangea db/archiving readme](https://github.com/thinkthroughmath/apangea/tree/rc/db/archiving) for more info on how the Customer/School extract and restore scripts work.

### Troubleshooting

If something isn't right with your AWS keys/credentials, you'll see an error something like this:

    .../gems/aws-sdk-1.9.5/lib/aws/core/credential_providers.rb:125:in `credentials':  (AWS::Errors::MissingCredentialsError)
    Missing Credentials.

    Unable to find AWS credentials.  You can configure your AWS credentials
    a few different ways:
    ...

If the school extractor is working properly, you'll see something like this:

    Extracted: Aliquippa Elementary School (1234) (Shard: enrollments_shard_one) (Schema: 20150421204641)

If the school restorer is working properly, you'll see something like this:

    OK: [Action: restore_local] - school 1234
    -- COMPLETE --

If the school restorer fails, it is most likely due to conflicts between IDs on data that already exists in your local, and IDs of the data being restored.
