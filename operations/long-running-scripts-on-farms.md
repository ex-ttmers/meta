# Considerations for Long-Running Scripts

When you run a long-running script on a farm, there are a few gotchas to keep in mind.

1. Run inside tmux/screen. These applications make it so that if your terminal session ends, the script will continue to run. Later, you can reattach

Here's are some cheat sheets for working with tmux:

- http://alvinalexander.com/linux-unix/tmux-cheat-sheet-commands-pdf
- https://gist.github.com/MohamedAlaa/2961058

It is a good idea to run scripts inside these programs, but this is almost essential for scripts that run for more than a few minutes.


1. What will running this impact, and how does that need to be handled? Examples:
  - Do you need to disable sidekiq? Will your script interfere with any sidekiq jobs that may run?
  - Do you need to disable scalr scheduled jobs? Will your script interfere with any of our scheduled jobs?

2. Does your script require scalr keys to be set in the environment?
  Anything that uses PsqlHelper will require these be set.

  Setting these looks like:
  ```
  export SCALR_KEY_ID=<your KEY_ID>
  export SCALR_ACCESS_KEY=<your ACCESS_KEY>
  ```

3. Will your job need more disk space than is available by default on the farm? The `/mnt/`
   on farms has plenty of space, but you may need to create symlinks into it. An example,
   taken from the DW readme:

   ```
   mkdir /mnt/initial_load_data
   mkdir /mnt/initial_load_data/archive
   mkdir /mnt/initial_load_data/dump
   mkdir /mnt/initial_load_data/process
   ln -s /mnt/initial_load_data/archive /var/www/etl/initial_load/archive
   ln -s /mnt/initial_load_data/dump /var/www/etl/initial_load/dump
   ln -s /mnt/initial_load_data/process /var/www/etl/initial_load/process
   ```
4. Do you have all the Scalr credentials you'll need to complete the task? Its a good idea to double-check, so that you don't get so far in to the task, only to realize that you can't continue.


# Resources
## 2015 Archiving proceedure
This is a helpful document we created as part of the 2015 archiving process. It might have information that is useful to you:
https://github.com/thinkthroughmath/storyboard/issues/1727
