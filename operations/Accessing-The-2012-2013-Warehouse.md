### Accessing the 2012-2013 Data Warehouse

The 2012-2013 warehouse is no longer accessable. 
In order to fully use the app the data muse be restored and a local copy of the app updated to point to the restored database.

#### Location of the Databases
The databases are located on S3 in the ttm-production-archive bucket in the folder 2012-2013. Note, you only need the ETL database if you are going to test or otherwise use ETL functionality. If you just need to access data, no need for ETL

1. Warehouse-2012-2013-ETL.dump
2. Warehouse-2012-2013-Warehouse.dump

#### Restoring Databases
1. Spin up a new RDS instance, Probably best to use the web UI for this since its a one time task.
  * The Database engine is Postgres
  * At least 300GB of storage. The raw data size on Heroku before the backup was 256GB
  * Create a security group giving 0.0.0.0/0 access to the Postgres port 5432
  * dm.m3.large || xlarge should be an appropriate instnace type, but feel free to vary based on the workload
2. Get the endpoint from the aws console
3. Recommend doing this process from an EC2 instance in one of the farms vs locally, due to the time involved in data transfer
4. Test using psql ENDPOINT_URL. This should result in a postgres prompt. If not check your security groups. By default the RDS process will add a security group for access from the office only, so you may need to update it.
5. Once connected with psql create two databases, etl and warehouse. These will be the targe DBs for the restore operation
6. Use PG restore to restore the databases. 
   * `pg_restore --clean --no-acl --no-owner -h RDS_ENDPOINT -U RDS_DB_USER -d etl Warehouse-2012-2013-ETL.dump`
   * `pg_restore --clean --no-acl --no-owner -h RDS_ENDPOINT -U RDS_DB_USER -d warehouse Warehouse-2012-2013-Warehouse.dump`

#### Get your local environment set up to the last good commit for the Heroku version of the warehouse
In your local environment: `git reset --hard e1ac97c`

Update the Ruby version in the Gemfile to 2.1.3 or later (otherwise you'll get a billion OpenSSL errors) and `bundle install`

In your .env, set the following env vars using the endpoint from above
   * WAREHOUSE_DATABASE_URL = ENDPOINT_URL/warehouse
   * ETL_DATABASE_URL = ENDPOINT_URL/etl
   * AWS_ACCESS_KEY_ID = Our AWS Key (if you are running adhoc reports the app will need to be able to store them)
   * AWS_SECRET_ACCESS_KEY - Our AWS Secret (same thing)
   

`foreman start` and your app should boot. You can then navigate to `http://localhost:3000/adhoc/new` (log in with `admin`/`admin%123`) and run your report. When finished DESTROY THE RDS INSTANCE so we don't get charged for it.
