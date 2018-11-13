We have scheduled DB snapshots that happen nightly and RDS also takes snapshots before and after some DB operations like point release upgrades. Ex. from Postgres 9.4.4 to 9.4.5

There are two ways to restore a DB snapshot
1. Command line
2. Using the Web Interface

#### Command Line

Use the command lines below to identfy and restore DB snapshots. Make sure to replace the snapshot id and db instance name with the correct ones.

1. `brew install awscli`
2. Get appropriate permissions then get an access key
3. Identify the snapshot you want to restore `aws rds describe-db-snapshots --db-instance-identifier ttm-lms-review`
4. OPTIONALLY DELETE the old DB instance `aws rds delete-db-instance --db-instance-identifier ttm-lms-review --skip-final-snapshot`
5. Restore the snapsot `aws rds restore-db-instance-from-db-snapshot --db-instance-identifier ttm-lms-review --db-snapshot-identifier rds:ttm-warehouse-rc-2015-11-16-05-02 --no-multi-az --db-instance-class db.r3.large --db-security-groups review-farm`
6. Check status with `aws rds describe-db-instances --db-instance-identifier ttm-lms-review`
7. Update parameters once the restore is finished `aws rds modify-db-instance --db-instance-identifier ttm-lms-review --db-parameter-group-name lms-production --engine-version 9.4.4 --apply-immediately`
8. Check status with `aws rds describe-db-instances --db-instance-identifier ttm-lms-review`

#### Web Interface
1. Login in to the AWS console. https://aws.amazon.com
2. Go to RDS
3. Slect snapshots on the left menu
4. Search for the db instance identifer to see all the snapshot associated with that instance
5. Select the appropriate snapshot and click restore
6. Fill in the following information in the Web Form and then click restore
    - Select the instance type: Dev is r3.large, Prod LMS is r3.4xl, Prod DW is r3.2xl change up or down or leave the same.
    - Multi AZ - PRoduction LMS: Yes, all other no
    - Storage Type: General Purpose SSD
    - Instance Identifier: ttm-lms-<FARM NAME> or ttm-warehouse-<FARM NAME> with an optional number after it if you need to have prod and prod-2 for example. Please note you will have to update ENV Variables with the new DB endpoint name
    - VPC: No
    - Availability Zone: No Preference
    - Port: 5432
    - Option Group: Default:Postrges-9-4
    - Auto Minor Version Upgrade: Yes
7. Once the inital restore is complete, the security group and other parameters must be updated so the farms can gain access.
8. Click Instanses on the left hand menu
9. Select the newly created instance
10. Select Modify from the Instance Actions menu on the top of the page
     - Set the security group to the correct Scalr Farm Security Group
     - Set the paramter group to production or warehouse
     - Set the engine version
     - Select Apply Immediatly
     - Click Continue

Thats it for the web interface. Once AWS says the modfications are complete, the DB is ready for use. Update the Apangea ENV vars and restart. 

#### Usernames and Passwords

Credentials are not altered by the restore process. If you are restoring a dev to dev then the credentials are the same. If you are restoring Prod to Dev, you will need to update credentials.
