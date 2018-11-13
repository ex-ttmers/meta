### The Deploy Script

Scalr supports scripts that can be executed on individual servers, roles or farms. ttmscalr triggers the `TTM-Launcher`
script (aka deploy script) to perform a deployment. The script performs the following tasks:

**What does it do?**

1. It writes a key to redis and waits for all roles in the farm to write their keys. This ensures that everyone deploys at the same time
2. It clones the appropriate git repo onto the server
3. Builds the app. Review the script to see how it builds ruby, node and single page apps like lesson player
4. Restarts unicorn and sidekiq depending on the server role
   * If the `--hard` flag was given to `ttmscalr` unicorn will be killed and restarted. Otherwise a graceful restart of unicorn will be performed. Sidekiq does not have a graceful restart option so it is always killed and restarted.

**Triggering a deploy**

Deploys are triggers on role startup, using the api via ttmscalr or manually using the scalr web interface

**Where are logs?**

Logs from script execution are stored on sclar. https://my.scalr.com/#/logs/scripting Filter by the target farm to see relevant logs.

### What happens when a server boots for the first time?

1. Initial config is performed with the post-boot-config script
2. Git deploy keys are installed via a script
3. The `TTM-Launcher` script is called.

See the orchestration configuration for each role to get the details.

### What happens when a server reboots?

The `TTM-Launcher` script is called with the launch arguments `-l -K` this tell the system to simply execute the appropriate lines in the procfile that correspond to the role name.
