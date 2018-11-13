# Infrastructure Documentation

## Environments

An **environment** is a copy of the application, with its
own servers, deployed copies of code, database snapshots, and anything
else that goes along with having a fully running system.

**Production** is the environment that our customers use. Don't mess
with it unless you are absolutely sure what's what.

Other environments are available for use in our day-to-day development
needs. These include **Lab**, **RC**, **Dev**, **QA**, and **Review**. **RC**
is reserved specifically for testing code that is soon to be deployed
to production. Jenkins will automatically deploy to **RC** when the build is
green.

All environments operate in production mode and run the same AMIs as production. They use
the same deploy scripts and procedures. They also have the same architecture as production. The
goal is to make them the same as production except for configuration variables and data.

### Environments for testing:

|Name|Warehouse|LT|Notes|
|----|-----|-----|------|
|RC  |Warehouse-Staging|live-teaching-rc|Reserved for testing soon-to-be-released code|
|Lab |Warehouse-Staging-Lab|live-teaching-lab|     |
|Dev |Warehouse-Staging-Dev|live-teaching-dev|     |to enable live-teaching go to https://live-teaching-dev.thinkthroughmath.com/ and accept use without certificate
|Review|Warehouse-Staging-Review|live-teaching-review||
|QA|Warehouse-Staging-QA|live-teaching-qa||

|Env Name|Redis|
|--------|-----|
|RC|@0f615.openredis.io:14187/0|
|lab|@0f615.openredis.io:14187/2|
|dev|@f9585.openredis.io:13590|
|Review|@0f615.openredis.io:14187/1|
|QA|@0f615.openredis.io:14187/8|
|Warehouse-Staging|@0f615.openredis.io:14187/4|N/A|
|Warehouse-Staging-Lab|@0f615.openredis.io:14187/5|N/A|
|Warehouse-Staging-Dev|@0f615.openredis.io:14187/7|N/A|
|Warehouse-Staging-Review|@0f615.openredis.io:14187/6|N/A|
|Warehouse-Staging-QA|@0f615.openredis.io:14187/8|N/A|
|live-teaching-dev|@f9585.openredis.io:13590/2|N/A|
|live-teaching-lab|@0f615.openredis.io:13590/9|N/A|
|live-teaching-rc|@0f615.openredis.io:13590/10|N/A|
|live-teaching-review|@0f615.openredis.io:13590/11|N/A|
|live-teaching-qa|@0f615.openredis.io:13590/12|N/A|

## Architecture

The app is hosted on [AWS](https://console.aws.amazon.com/console). It consists of Web Servers sitting behind an AWS ELB and Sidekiq servers split into three groups. Sidekiq uses Redis hosted by OpenRedis. The sidekiq servers are devided into the following Scalr Roles: Sidekiq, and Reports. Please review the Procfile.Production to see what queues are are each role.

Each Webserver runs Nginx which proxies requests to Unicorn. Unicorn must be proxied when speaking to slow clients, see the Unicorn documentation for more details.

The two main aps Apangea and Reporting are configured this way. In the Live Teaching app, ELB speaks directly to Node.

### Accessing AWS

The [login page for AWS](https://console.aws.amazon.com/console) allows access to all of our running infrastructure. We have two accounts with AWS:
1) `ttm-engineering` - is the one configured for Scalr use and hosts all of our prod infrastructure; EC2, ELB, RDS. Basically, anything that Scalr manages.
2) `apangea` - is a legacy from pre-Scalr, and contains our S3, Cloudfront, and DNS (Route53) configurations, as well as the prod and dev Flash Media Server virtual machines (and a Domain controller that we aren't sure is safe to remove)

Currently Jeff, CG, André and Jim have access - to log in you need an IAM account. We use 2FA for access so you'll need to set that up too - see André or Jim if you need an account.

### DNS

Our DNS registrar is [MyDomain](https://secure.mydomain.com/secure/login.bml). All of our domains are set to auto-renew so there shouldn't be anything you need to do here on a routine basis. The credentials for MyDomain are in LastPass; André, John D, and Jim have access if you need it. AWS Route 53 is the DNS server. To get in to Route53 you'll need to log in to the `apangea` account as referenced above and click on Route53. Each domain we have hosted there is listed as a 'Hosted Zone'. Click *Hosted Zones* then click the domain you want to manage. An entry here is called a record set; there are definitions of the different options [here](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/ResourceRecordTypes.html).   

### Logs

Logs are all routed to [Papertrail](https://papertrailapp.com) using the rails logger and syslog. There is a simple bash script called cmd_line_logger.sh in the
root of the repos that is used to catch stdout from process that dont write to syslog and redirect it to Papertrail.

### Monitoring

We use [Bugsnag](http://bugsnag.com), [Skylight](https://slylight.io), and [New Relic](https:/newrelic.com) for application monitoring.

