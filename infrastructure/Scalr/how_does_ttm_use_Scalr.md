# Scalr internals

Scalr is a cloud automation framework. They have a master key to the Apangea subaccount of the TTM AWS account. Vist http://scalr.com to see all the features. Some features used by TTM are

1. Farms
2. Roles
3. Images
4. Orchestration Scripts
5. Global Variables
6. Scheduled Jobs

## TTM Scalr Infrastructure

Servers are origanized into a group called a Farm. A Farm is built from Roles. A Role is a server specialization and in a farm there can be multiple Roles, with multiple instances of a role. For Example:

```
Farm ----> Dev
    Roles ----> RailsAppServer
        Instances ----> RailsAppServer-1
                            .
                            .
                            .
                        RailsAppServer-N
                Sidekiq
        Instances ----> Sdiekiq-1
                            .
                            .
                            .
                        Sidekiq-N

                Reports
        Instances ----> Reports-1
                            .
                            .
                            .
                        Reports-N
```

Each role has an AMI that backs it. The image is built by Sclar using Chef.

Farm scoped global variables are used to store configuration data. These global variables are available via standard OS interfaces.

Orchestration scripts are scripts stored in Scalr. These scripts are triggered on specific events within Scalr, custom events, manually from the GUI or via the API. These scripts can be written in any language for which there is an interpreter on the instance and are executed on one or multiple instances based on the execution context.
