TTM includes functionality for educators to export real-time student-level usage data, at any pint. In some cases with our larger customers, for contractual reasons their single billing entity is split into serveral TTM customer records. Texas Education Agency is an example of this; Chicago Public Schools is also in this scenario, and some of our Charter School network customers are set up this way.

If these customers ask for student-level exports, it is impractical to log in to all of their accounts to export individual CSV files. There is a way to run them all at once:
* SSH to the debug server on the data warehouse: `ttmscalr ssh debug.1 -f dw-production`
* Set the `CUSTOMER_IDS` environment variable: `export CUSTOMER_IDS="[IDS YOU WANT TO EXPORT FROM"]`
* `export S3_BUCKET="ttm-student-export"`
* `cd /var/www`
* `rake export:students`

Once complete, you can use an s3 browser like Cyberduck on OS X to connect to that bucket to see the files.

NOTE: It could take a while for them to finish uploading to S3 since the export jobs are backgrounded. Also, the bucket `ttm-student-export` may contain other entries, so use the timestamps on the files to determine which ones you want.
