# What will require downtime

**UPDATE** As of rails 4, we no longer need to tell ActiveRecord to reject the column. See the "Unshackled" section at the bottom of this [How to Remove a Column with Zero Downtime in Ruby on Rails article](http://jakeyesbeck.com/2016/02/07/how-to-remove-a-column-with-zero-downtime-in-ruby-on-rails/).

Most code changes won't really require downtime, but it is not always obvious what changes will require downtime, and what changes can be broken into steps done across multiple deployments to avoid downtime.

This blog explains some of this very well and how to avoid downtime http://blog.codeship.com/rails-migrations-zero-downtime/

Examples of code changes that need to be broken into steps to avoid downtime...

## Table or column removal

1. Remove all code references to a column, ~~[tell active record to ignore a column from its cache](#tell-active-record-to-ignore-a-column-from-its-cache),~~ deploy
2. Add a migration to remove the column, deploy

## Table or column renames

1. Add the new column, migrate the data from the old column to the new one, deploy
2. Follow the steps for column removal above

## Column additions with a NOT NULL constraint that does not also have a DEFAULT

1. Add the column and populate it with one deployment
2. Then in another deployment, add the "NOT NULL" constraint

## Exceptions to the above

Some changes require a change to the production environment, before they can be deployed. In these cases we might opt to do a rolling deploying, where we terminate and replace small batches of servers at a time, until we have replaces all our production servers.

-  Currently this is common when adding a new environment variable

Some code changes could require downtime without some special care.  They may be handled by using different techniques, such as ordered deployments to production. 
See this blog post for additional details about situations such as column removals: 

## ~~Tell Active Record to ignore a column from its cache~~

[This is no longer necessary as of Rails 4!](http://jakeyesbeck.com/2016/02/07/how-to-remove-a-column-with-zero-downtime-in-ruby-on-rails/)

```ruby
class User
  def self.columns
    super.reject { |c| c.name == "notes" }
  end
end
```

### Why do I need to tell active record to ignore a column? 

[You don't as of Rails 4!](http://jakeyesbeck.com/2016/02/07/how-to-remove-a-column-with-zero-downtime-in-ruby-on-rails/) Let me show you! (there are other demonstrations in that article)

1. Create a migration to add a column:

        rails generate migration AddGarbageToClassroom garbage:string

2. Run the generated migration and restart your apangea app

        rake db:migrate

3. Create a migration to remove the column:

        rails generate migration RemoveGarbageFromClassroom garbage:string

4. Run the migration ( Do not restart your app )

        rake db:migrate

5. Try to create a new classroom in the app, you WON'T get an error because Rails 4 can handle it.

## Changes that do NOT require special care for deployments are...

- Table additions
- Column additions (as long as they don't fall into the NOT NULL w/o DEFAULT case above)
- rails route changes
