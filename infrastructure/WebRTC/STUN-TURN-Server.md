### Purpose of the STUN / TURN Server

If one or both clients are behind a NAT device WebRTC uses a STUN server to determine the public IP of the clients.
If WebRTC is unable to make a direct peer to peer connection the media data is relayed through the TURN server.

### Restund STUN/TURN Server

We selected the restund STUN/TURN server. http://www.creytiv.com/restund.html 

#### Building restund

The build process involves downloading the source distributions, and building .deb packages.

1. Log into a debug role for a farm
2. Install the build tools
      - apt-get install devscripts debhelper
3. Download restund - `restund-x.x.x.tar.gz`
4. Download re - `re-x.x.x.tar.gz` from the same location
5. Extract re `tar -xvzf re-x.x.x.tar.gz`
6. Extract restund `tar -xvzf restund-x.x.x.tar.gz`
7. Build .deb packages starting with re
      - `cd re-x.x.x`
      - `debuild -us -uc`
      - Built packages are placed in the parent dir so `cd ..`
      - Install the packages so we can build restund. `dpkg -i libre_x.x.x_amd64.deb libre-dev_x.x.x_amd64.deb`
      - `cd restund-x.x.x`
      - `debuild -us -uc`
      - Built packages are placed in the parent dir so `cd ..`
      - Install the packages so we can test. `dpkg -i restund_x.x.x_amd64.deb`
8. Copy packages to your local machine for upload to chef server. See below. 

#### Configuring for AWS deployment

We need to generate .deb packages for libre, libre-dev and restund. Luckily for us they come configured to generate .deb packages.

1. If needed generate new .deb packages for libre, libre-dev and restund
2. Download the chef-repo project from the TTM Github
3. Edit the recipe TTM-Role-Restund
     - Under files/default update or replace the config files or .deb packages
     - Under recipes/default make any changes to the recipe
     - checkin to github
     - upload to chef
4. Test. The template farm `Rebundle-Staging-2.0` has the template role for the STUN/TURN Server called `TurnServer`
     - Start or Terminate to have the role bootstrap using the updated Chef recipe and packages
     - Go to roles in Scalr and find the latest version of the `TurnServer` role
     - Snapshot the role in the `Rebundle-Staging-2.0`
         - Check `Also Create New Role`
         - Name it `TurnServer-2-x` replace x with the next version number
         - When the snapshot is finished edit the `STUN-TURN-Dev` farm and replace the existing role with the newly created one using the replace role feature. Click on the role name, and then in the window to the right under `role alias` the are two arrows pointing in opposite directions. This is the replace role feature.
         - Test
5. If it works in dev, its time to migrate to production
     - Click Roles
     - Find the `TurnServer-2-x` role. Don't forget to replace x with the version number
     - Click it. In the window on the right click the image name. Copy the AMI ID `ami-xxxxxxx`
     - Go back to Roles. Find the `TurnServer` role. Note this is the base role without a version number
     - Edit the role. Click `Images` on the left. Select the image thats there
     - Click the replace image icon, two sideways facing arrows
     - search for the AMI from above
     - Select it and click `Replace`
     - Terminate the production server so it respawns with the new image.
     - Test.

#### Testing Live Teaching using the TURN Server

In order to test the TURN server, we must make sure that the two client (teacher and student) computers can not connect to each other.

To do this from a Mac.

1. goto System Preferences -> Security and privacy -> Firewall
2. Click "Firewall Options"
3. Check "Block all incoming connections"
 You should now get routed through the turn server if both teacher and student connections are coming from this computer

#### Testing with Vagrant

If needed you can test the Chef recipes and updated packages with Vagrant. Use the following config to spin up a box. Paste it into a Vagrant file and do vagrant up Restund. 

`````
# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # Every Vagrant virtual environment requires a box to build off of.
  # config.vm.box = "hashicorp/precise64"
  config.vm.box = "ubuntu/trusty64"

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  # config.vm.box_url = "http://files.vagrantup.com/precise64.box"

  config.vm.define "Restund" do |restund|
    restund.vm.hostname = "Restund"
    restund.vm.provider "virtualbox" do |v|
      v.memory = 1024
    end

    restund.vm.provision :chef_client do |chef|
      chef.node_name = "andre-restund"
      chef.chef_server_url = "https://api.opscode.com/organizations/ttm-lms"
      chef.validation_key_path = "/Users/andre/src/chef-repo/.chef/ttm-lms-validator.pem"
      chef.validation_client_name = "ttm-lms-validator"
      chef.environment = "Dev"
      chef.add_role "Restund"
      chef.delete_node = true
      chef.delete_client = true
      chef.json = { 'vagrant_host' => true, 'platform_family' => 'debian' }
    end


  end
end
`````

#### Booting on AWS

If you review the config file for restund you will notice that it has IP addresses hard coded in the file. Restund will not listen on `*` or `0.0.0.0` so on boot there is a script in scalr called `TTM-Build-Restund-Conf`. This script replaces instances of `127.0.0.1` in the config file with the actual IP of the AWS instance.

#### Firewalls

We have opted to run STUN and TURN on port 80 and 443. This is to give them a better chance of making it through firewalls

#### TLS/DTLS

TBD
