### How to delete dormant users

In our system, a `dormant user` is a user who was created more than 370 
days ago and either never logged in or hasn't logged in for 180 days.

The class that houses the logic is `DormantUserDeleter.rb`. This
takes an options has that has three possible keys:

```
days_old: Delete users who were created before this date. (default: 370 days)
days_unused: Delete users who have not logged in since this date. (default: 180 days)
log: boolean to turn logging on or off. (default: true)
```

There is also a rake task to run this file:

`rake ttm:delete_dormant_users`
