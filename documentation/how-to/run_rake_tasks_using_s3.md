These are the steps needed to run rake tasks that use s3
(like the `rake export:students` in the warehouse)

1. `brew install s3cmd`
2. create a file in your home directory `~/.s3cfg`
3. in this file, add the lines:

    ```
    [default]
    access_key=#{s3 access key from your .env file}
    secret_key=#{s3 secret key from your .env file}
    ```
4. back in your shell, you need to export some variables:

    ```
    export AWS_ACCESS_KEY_ID=#{s3 access key from your .env file}
    export AWS_SECRET_ACCESS_KEY=#{s3 secret key from your .env file}
    ```
5. run the rake task!
