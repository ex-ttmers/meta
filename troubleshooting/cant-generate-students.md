# I can't generate students!!!!1

Don't panic.  If the error is that it's grabbing the id of nil instead of being able to get the DefaultPathway for students, I have a brute-force fix, but it will blow away your old DB.

Ensure that all processes contacting the postgres database are dead, with a command like so (depending on what port it's listening on):

```bash
kill -9 `lsof -i tcp:6380 | awk -e '{ print $2}'`
```

Re-build with production content.

```
rake ttm:rebuild_with_production_content
```

According to Jeff, you may be able to just do a:

```bash
ruby db/copy_sharded_content.rb --from production --to local
```

but this has not been tried yet.
