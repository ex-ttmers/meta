The [CanCan Gem](https://github.com/ryanb/cancan) is used to provide a restrictive access model to Controller Actions

CanCan relies on a ability.rb configuration file located under app/models/ability.rb

CanCan expects that your controller actions each contain an instance variable of the model class that the controller resources. If you do not have an instance of the model class in your action CanCan's behavior can be erratic and it is good to consider excluding those methods from auto authentication.

There are many ways to implement CanCan described in their documentation. The most comprehensive method is to employ the load_and_authorize_resource filter method. This will call the authorize! method during each action call. The authorize! method assues that CanCan has a "can" definition to go along with the action associated with the users "current_user" session object maintained by [Devise](https://github.com/plataformatec/devise).

[Authorizing Actions](https://github.com/ryanb/cancan/wiki/authorizing-controller-actions)

Here is a nice concrete example

> class ActivitiesController < ApplicationController
>>   before_filter :authenticate_user!
>>
>>   layout "customer_management"
>>
>>   # use cancan to fillin default methods and set user permissions
>>   load_and_authorize_resource
>>   skip_authorize_resource only: [:add_item, :remove_item]

* We run authenticate_user! to make sure the current_user is in scope for CanCan
* load_and_authorize_resource will assure that each action must be authorized
* Finally the skip_authorize_resource lets us bail out on some of the authorizations for things that are just general access

Since load_and_authorize_resource is to act like a Rails before_filter it can also accept the only: or except: parameters in the same pattern. These are convienant if your Controller includes methods that do not implement and instance variable of the model that the controller resources.

So the tricky part is making sure your ability.rb is configured correctly. Remember this pattern is restrictive in the sense that if the controller is set to load_and_authorize_resource a "can" statement will be required for the associated user class in the ability.rb file to assure that the action can be completed.
