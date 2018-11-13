# Scalr

[Scalr](https://my.scalr.com/) is a layer on top of AWS that allows us to manage our servers more easily in a heroku-like way.

## Glossary

**Farm** ~ A cluster of servers that work together to host a single application. Example: aws-production, aws-review, aws-rc

**Instance** ~ A single server with a set role in a farm. Instances should not be expected to be perminante, they can be cycled at will. This is similar to a Heroku Dyno

**Role** ~ A defined server template configured in scalr across all farms for a specfic job. Example: RailsAppServer, Sidkiq, Reports, DevDebug ...

**Application** ~ A code repository. Example: www.github.com/thinkthroughmath.com/aws-production, www.github.com/thinkthroughmath.com/aws-review

## Autoscaler

The RailsAppServer role on the production farm dynamically scales based on date and time. During the regular school year we use the following parameters:
* Min Servers 5
* Max Server 20
* With date and time based autoscaling with the following schedule
    * 20 Servers between 7:30 AM and  5:00 PM M,T,W,Th,F
    * 10 Servers between 5:00 PM and 11:00 PM M,T,W,Th,F
    * Outside of these hours Scalr falls back to the min servers setting. https://scalr-wiki.atlassian.net/wiki/display/docs/Autoscaling+Algorithms

During summer school:
* Min Servers 5
* Max Server 20
* With date and time based autoscaling with the following schedule
    * 10 Servers between 7:30 AM and 5:00 PM M,T,W,Th,F
    * Outside of these hours Scalr falls back to the min servers setting. https://scalr-wiki.atlassian.net/wiki/display/docs/Autoscaling+Algorithms

Things to note about the autoscaler:
* Manual scaling will not work. If you try to manually upscale the autoscaler will kick in and bring the instance count into compliance with the rules.
* Rules have to be updated for EST/DST because they are entered in UTC
* When the autoscaler downscales it starts with the oldest servers first

## Creating a new farm

### Creating the Farm

* Click on Farms in the main menu.
* Click the green + sign to add a new one
* Give the farm a descriptive name and set the timezone to UTC (Ask Carol for help with timezones)

This is the basic setup of the Farm. At the point the farm has no roles that can be used to launch servers.

### Adding Roles

We have some predefined roles in Scalr.
* RailsAppServer - Rails app server
* Sidekiq - Sidekiq
* Reports - Sidekiq
* SystemWatcher - Runs cron jobs and other misc tasks
* DevDebug - Debug for rails console and other usage
* PGSQL-9-2 - PostgreSQL 9.2

To add a role to the farm click add role on the left side of the screen in the farm builder. Important things to select are the Availability Zone and Instance type.

If you need DB services make sure the DB server is in the same zone as the other servers.

* Click "Add to Farm"
* Click the newly added role and configure it
* Set Min / Max Instances
* Review the default scripts added - Should not need to be changed unless you are doing something custom
* Configure any EC2 Perks - If this is a DB server you may want EBS optimized storage
* **If the role is a DB Server be sure to set your volume size**
* Create a new ELB if you need it. Only needed for web or API servers.

Repeat for every role that is needed.

Once the farm has been build Environment variables must be set. These tell the farm what to use for logging, Redis, S3 Credentials, etc ...

* Start by dumping the env from an existing farm and save it to a file

        $ ttmscalr config:get -f review > file.txt

* Create new accounts, DBs, endpoints etc.. if need on
  * Papertrail
  * Open Redis

* Enter a unique NewRelic application name
* Edit the file and set / change any variables

        $ ttmscalr config:set file.txt -f farm_name

## Internals

### The Deploy Script

Scalr supports scripts that can be executed on individual servers, roles or farms. ttmscalr triggers the `V2-TTMAppConfigAndLaunch` script (aka deploy script) to perform a deployment. The script performs the following tasks:

**What does it do?**

1. It writes a key to redis and waits for all roles in the farm to write their keys. This ensures that everyone deploys at the same time
2. It clones the appropriate git repo onto the server
3. Executes bundle `bundle install --without development test`
4. Executes the asset pipeline `rake assets:precompile`
5. Restarts unicorn and sidekiq depending on the server role
   * If the `--hard` flag was given to `ttmscalr` unicorn will be killed and restarted. Otherwise a graceful restart of unicorn will be performed. Sidekiq does not have a graceful restart option so it is always killed and restarted.

**Where are logs?**

Logs from script execution are stored on sclar. https://my.scalr.com/#/logs/scripting Filter by the target farm to see relevant logs.

### What happens when a server boots for the first time?

There is a script called `TTMBuildServerForRole` that configures the image after AWS provisions the server. In normal operation this script just confirms that the server has all its components since each server is based on a snapshot taken after this script has been run. The script configures each server based on the Scalr Role Name.

Once the build script is finished the deploy script runs. The deploy script is passed the `HostInit` option and looks at the `TTM_DEPLOY_ON_INIT` env variable to see what repos should be deployed on boot. A sample from the dev farm is `TTM_DEPLOY_ON_INIT=dev,lesson-player`. The deploy script prefixes `aws` to the repo names.

### What happens when a server reboots?

The deploy script is passed the `Reboot` option. This tell is to only start unicorn and sidekiq, no other actions are performed.

### Adding a scheduled task

For apangea, the script is TTMScheduledJobsMaster. The warehouse and live teaching have similarly named scripts.

* Go to Scripts and edit TTMScheduledJobsMaster (this will prompt you to switch to the Application Scope or something, do it)
* Add a key to the list and define your script to the right of it, follow existing patterns
* Save the script
* Switch out of the Application Scope by going to the like, wreath icon in the upper right and choosing aws
* Go to the scalr menu in the upper left and choose Task Scheduler
* Click the New Task button
* Name your task "Farmname-Description", like the others
* Task type is "Execute script"
* You may leave the description blank
* Pick your schedule - set the timezone to UTC and set the times in terms of that
* Under Target, select the farm you want to run the script on. Next to the farm, a new dropdown will appear with the farm roles-- choose SystemWatcher. You can leave the next dropdown as "On all instances" since we only ever have one SystemWatcher instance.
* Under Execution Options, choose the TTMScheduledJobsMaster script
* Under My Task, specify the key of the task you want to run that you added to the script
