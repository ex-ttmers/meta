EC2 VMs are created from Amazon Machine Images (AMIs). Scalr handles much of the work involved with creating a machine image,
however the image must still be customized for our use and added to a Scalr Role which can then be added to a Farm.

Roles are a way of specifying a configuration for similar servers. Ex the Role RalsAppServer is used for any server that runs
the web line in the Procfile.Production. Role names are also used by infrastructure scripts to identify the function of each instance in the Farm so names matter.

If you want something only on the web server then it goes on the RailsAppServer role. Generally speaking role names correspond
to lines in the Procfile.Production. The one execption is web, which is mapped to RailsAppServer in the TTM-Launcher script.

### Images

Each image is customized with Chef, the general process is as follows. We have a staging farm (Rebundle-Staging), each Role in
this Farm is configured to bootstrap with chef. Each time servers on the farm boot, chef is used to configure them. We then take a snapshot of each role on the staging farm, and then use that AMI in production.

We only do this on the staging farm becasue we dont want to wait for Chef in Production or take a chace that the chef run might fail
during scaling. The chef run is initiated each time the server spawns. To initiate a chef run terminate the server and let it reprovision.

Once the server has been provisioned we create a snapshot and give it a version number. The production roles are then updated to use
the AMI created in the snapshot.

### Staging Farm

The staging Farm is called Rebundle-Staging. There may be a version number after it. As of 4/2016 we are on Ubutu 12.04 as configured on the Rebundle-Staging farm. Live teaching is on 14.04 configured on Rebundle-Staging-2.0 

There is a base role called TTM-Base this role is added to the farm multiple times, once for each role thats needed. The current roles are:

    * RailsAppServer
    * NodeAppServer
    * Sidekiq
    * DevDebug
    * SystemWatcher

While the role is the same, each time its added to the farm its given a different alias corresponding to the the desired role name.

##### Chef

As you add each role to the farm click the `BOOTSTRAP WITH CHEF` tab and configure chef.

    * Select Chef Server
    * Chef environemnt: Dev
    * Configuration: Role, then select role from the drop down

Chef roles are mapped to TTM / Scalr as follows

    * RailsAppServer    -> RailsAppServer
    * NodeAppServer     -> NodeAppServer
    * Sidekiq           -> SidekiqServer
    * Reports           -> SidekiqServer
    * SystemWatcher     -> SystemWatcher
    * DevDebug          -> DevDebug

### Making updates and provisioning new images

Rember there is only one set of Roles for all Farms. So any config change or package update will apply to all servers in the fleet. So think about what you are doing.

1. Start by applying any OS updates to the `TTM-Base` image. This will make sure that all roles based on this one have the latest OS updates applied.
    * Use `apt-get update` , `apt-get upgrade` , and `apt-get install` to apply update
    * reboot the image after updates have been applied
    * Snapshot the image and check the "Replace Image" checkbox
2. Get a copy of the `chef-repo` github project
3. Update the cookbooks as needed and publish the updates to the chef server.
    * `knife cookbook upload <COOK BOOK NAME>`
4. Terminate each role in the `Rebundle-Staging` farm. 
    * This will pick up the updated base image from above
    * This will make the role respawn and reconfigure with any changes from chef.
5. After verifying that chef configured the role properly, snapshot the newly provisioned role, making sure to create a new role.
     * Give the snpshot a versioned name. Ex. snapshot `RailsAppServer-Base` and give it a new name `RailsAppServer-2-5` where 2-5 represents the version of the image.
     * Version numbers are of the form X-Y. X is major and Y is minor. I typically bump the major number for OS changes. Ubuntu 12.0.4 to 14.0.4 for example. We are currently on 1-x as of 4/2016
     * The easiest way to find the next version numbers is to use the Roles menu in Scalr GUI
     * The snapshot after configuration has a version attached (ex Sidekiq-1-6), and the roles used to launch farms (production roles) are name without any suffixes. (RailsAppServer, Sidekiq, Reports, DataLoad, SystemWatcher, DevDebug)
     * **Names Matter.** Please use the existing TTM Role names because all the infrastructure scripts know what to do based on the role names. If you make a new role, the scripts will have to be updated.
6. The Dev farm is configured as the test farm for new roles. Edit the Dev farm in the Farm Builder, for each role, click the swap role button (up and down arrows below the role name/alias) and swap in the current version.
    * Terminate the roles in the Dev farm to reboot with the new roles
    * Test with an Apangea Deploy and ssh in to verify any config changes
7. To use the new role after testing, edit the roles in the role builder to use the image from the versioned role. 
    * Edit the RailsAppServer role, click image and replace the image there with the image from the RailsAppServer-v-next image. The image name starts with `ami-`
    * Repeat for all other roles
8. Terminate the old instances on PRoduction and Deevelopment farms and wait for them to respawn.


