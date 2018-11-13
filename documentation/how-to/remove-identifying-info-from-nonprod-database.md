If you pull production data down to your local machine or to a farm, you need to remove all of the personally-identifying information. This is a practice that we have committed to do so it's not optional.

Here's how to do it.

1. From your application root directory of the environment you want to de-identify
2. `cd db/deidentify`
3. To see all options for the script, enter `ruby deidentify.rb`
4. Now run the command for your environment __(WHICH BETTER NOT BE PRODUCTION, BUB)__. The most common case is deidentifying local data, so you would run `ruby deidentify.rb local student` then `ruby deidentify.rb local user`
