## Preparation

Building a new Apanga environment requires a few things before you can start.

1. A reporting instance. The reporting instance also needs
    * Redis, either a new instance or a new DB on an existing instance
    * Papertrail port
    * RDS Database
    * GitHub Repo
2. A Live teaching instance. The live teaching instance
    * Redis, either a new instance or a new DB on an existing instance
    * Papertrail port
    * SSL certificates
    * GitHub Repo
3. Redis. Either a new instance or a new DB on an existing instance
4. A papertrail port
5. SSL Certificate
6. GitHub repos

### Getting Started
* Create Github repos
* Create RDS Instances
* Create Warehouse, Live Teaching and Apangea Farms
* Modify RDS instances with appopriate security groups
* Deploy software

#### 1. Create Github repos for each Farm
* Repo is name aws-<farm-name>
* Repo has a deploy key configured. Generate with `ssh-keygen -t rsa -f <some-name>`
* Put the public key in GitHub
* Put the private key in the scalr scritpt called TTMGitDeployKeys. This script only runs once when AWS provides the an instance

The deploy script will automatically look for this repo (aws-farm-name) when ttmscalr is used to deploy code to the farm. 
If the repo is not setup correctly deploys to the Farm will fail.

#### 2. Create the RDS Instances for the Warehouse and Apangea
The quickest way to is start with a snapshot of an existing RDS instance. Select a warehouse or apangea instance. 
Find the latest nightly snapshot and restore it to a new instance. Once the instance has booted:

* Configure the paramter group. (lms-production, lms-warehouse)

RDS Credentails are stored in LastPass

#### 3. Build Scalr Farms

1. Log into to scalr and click the `New Farm` button.
2. Add the required roles. SystemWatcher, DevDebug, RailsAppServer, Sidekiq, Reports
    * Pay attention to the instance types
    * Change availabilty zones if necessary
    * Change min and max instances if necessary
3. For the RailsAppServer / NodeAppServer roles we use an ELB to gie them a static endpoit for DNS (Route 53)
    * Under the Network tab select use ELB.
    * When you create a DNS entry for the new farm, make it a CNAME pointing to the ELB address
    * Later on modify the ELB and add the SSL certificates. `Scalr Menu -> AWS -> EC2 ELB`
3. Configure Security groups for each role in the farm
    * There are predfined groups starting with TTM for common things like SSH, HTTP etc ...
    * Create a security group for the farm and add it to each role in the farm. This will be needed to configure RDS
4. Under advnaced setting set the host name to `{SCALR_FARM_NAME}-{SCALR_FARM_ROLE_ALIAS}-{SCALR-INSTANCE-INDEX}`
5. Save the Farm
6. Use ttmscalr to export the env from a working farm
    * Change all necessary setting to match the new farm
    * Use ttmscalr to bulk load the prepared env file into the new farm

Repeat the above steps for to create the Warehouse and Live Teaching instance.

    * Warehouse roles are: SystemWatcher, DevDebug, RailsAppServer, Sidekiq, Reports
    * Live Teaching roles are: NodeAppServer

#### 4. Configure RDS Security
You must configure RDS to allow access from the newly created Farms. To do this

* Create an RDS security group for the farm.
* Add the warehouse security group, and the apangea security group
* Add Jenkins security group
* Add the office IP
* Modify the RDS instance and update the security group from default to the newly created one.

#### 5. Deploy
* Configure .ttm_scalr_aliases.json with the id and name of the new farms
* Use `ttmscalr deploy --hard <FARM NAME>` to deploy code
* Monitor papertrail for errors

