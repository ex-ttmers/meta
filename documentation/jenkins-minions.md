# How to reserve and update jenkins minions

## Manually working with a minion
### Reserve a Minion
1. Go to the Jenkins web ui.
1. On the left sidebar, click the name of a minion that isn't doing anything (or is only doing stuff you don't care about killing, a.k.a. other people's tests).
1. click 'configure'. 
  1. Remove all the labels. 
  2. In `Usage` select "Only build jobs with label restrictions matching this node" under usage.
  3. Save.

The minion is now reserved for you.

### Using Minions
1. SSH into a reserved slave using `ttmscalr ssh slave.[num] -f jenkins`, where [num] matches the name of the slave you reserved.
1. Log in as the jenkins user by running:

        $ su - jenkins

### Freeing Minions

1. When done with a minion, click 'Delete' in the left hand menu of the minion's Jenkins UI.

Jenkins will shut down the minion.

## Updating Minions

1. Reserve a minion as described above.
1. Make the updates to the minion as you require.
1. Consider also running [this jenkins job](http://jenkins.thinkthroughmath.com:8080/view/Jenkins/job/update-packages) against this minion, as it will install all the latest ruby version and gems that our major repositories use. This makes setup for tests faster because less gems will need updated.
1. Login to my.scalr.com
1. click Farms
1. Find the Jenkins row in the table, then find the servers column in that row and click the View link in that cell.
1. click the number of the Jenkins slave that you are working on
1. click the action menu button (it's the downward pointing caret "v" in the "Actions" column) and choose Create Server Snapshot from the menu.
1. choose the option "Replace Image "JenkinsSlave-*some date* [ami-*some hash*]" on Role "JenkinsSlave" with the newly created Image"
2. submit
1. You are now on a page that should list just your bundle task and its status, which should say "pending" (this is not a ruby bundle)
1. Wait about 15 minutes for the bundle task to complete.
    * If you click on the row, a sidebar should pop up with more details and a button to view full log if you want.
    * It's not clear from the logs whether the bundle is done or not. Note that none of the logs that end in "complete!" indicate completion, including the one that says "Rebundle complete!" 
    * As of this writing, the last message in the logs is "Image replacement completed." 
    * Your bundle tasks' status in the table will say "Completed" in the Status column if it's done.
1. after it has finished, use http://jenkins.thinkthroughmath.com:8080/job/Node-delete/ without parameters (even though it says parameters are required, they're not). This will delete all the slave nodes, and they'll come back with the new image on their own.
1. Delete the minion that you updated as described above.

