#Upload MP4s
This page describes the workflow and steps to complete uploading mp4 movies.  Long story short - when a swf is associated with any non-json'ed item type (multiple choice, fitb, etc....) the art team provides an associated mp4, which this script below will split appropriately to match the swf hot spots.  It only needs to be done once per item.

For json items (matching basic, create & evaluate, etc....) there will NEVER be any NEW swfs added to these items.  Instead - it will mooch off of existing swfs/mp4s in the non-json'ed item types.  So when a content person uploads a swf to, for example, a matching basic help, we search the db for the existing swf (based on the uploaded filename) asociated to a non-json'ed item and split it's mp4 into the new item's s3 area.  Ugly, but effective and only short term.  Here is a mini workflow of that process:

##### Workflow for adding a swf to a matching basic item:

* There MUST be an existing published item with a SWF associated to it.  i.e. A1L250.swf
* User creates a new MB item and uploads a swf to a help.  The swf MUST be named A1L250.swf - so that it can be associated with the existing swf in the system.
* Do not have the MD5 included on the filename - just A1L250.swf - just like they uploaded it originally.
* Save.  (This finds the A1L250.swf and re-splits it to the new MB item s3 bucket AND RENAMES it match the new item)
* Publish the item.


##Workflow For Uploading MP4s

Download mp4's from dropbox.  All mp4 movies are uploaded to Drop box (see Adam or Keith for access)

Upload these mp4s to aws: `ttm-content/movies/`

Open a rails console on prod and add the following def to the console:


<pre>
def import(file_name, hit_points, dest_item_number, dest_filename)

  src_movie = "#{ENV['FOG_HOST']}/movies/#{file_name}"
  movie_data = {"hit_points"=>hit_points.map {|x| x.to_s }}

  if dest_item_number.present?

    puts "Associating #{dest_filename} to Item Number #{dest_item_number}"

    si = ShardedImage.from_filename(file_name.gsub('.mp4', '')).first
    SplitMovieWorker.perform_at(5.seconds, si.id, dest_item_number, dest_filename)

  else

    ShardedImage.from_filename(file_name.gsub('.mp4', '')).each do |si|

      success = true

      puts "Attaching #{src_movie} to Sharded Image #{si.id}"
      si.remote_movie_url = src_movie
      si.movie_data = movie_data.to_json

      unless si.save!
        success = false
        puts "Failed to upload #{url} with error #{si.errors.full_messages}"
      end

      if success
        puts "#{file_name} - Success!"
        if si.published?
          SplitMovieWorker.perform_at(5.seconds, si.id)
        end
      end

    end

  end

  puts "if splits occurred, ssh using sidekiq.1 to REMOVE /tmp/splits/*"

end
</pre>

Open the following google doc. It is used to run the scripts to upload the mp4s from the movies/ folder:

[https://docs.google.com/a/thinkthroughmath.com/spreadsheet/ccc?key=0Ar_MmnJr1ZMVdGN0QTB3YXJrR0d4ajNmQ2ZPeHRkdUE&usp=sharing#gid=0](https://docs.google.com/a/thinkthroughmath.com/spreadsheet/ccc?key=0Ar_MmnJr1ZMVdGN0QTB3YXJrR0d4ajNmQ2ZPeHRkdUE&usp=sharing#gid=0)

Copy the "Used for Keith's script" column and paste them in the console.  This will attach the movies using the above def.

like so:

`import('A1FL1GPQ1FA.mp4',[2000,3333,5555], '', '')`

or

`import('A1FL1GPQ1FA.mp4',[2000,3333,5555], '99042', 'A1FL1GPQ1FA')`

You can check the results with this:
`ShardedImage.from_filename('A1FL1GPQ1FA')`

remove split its if they were created:

`ttmscalr ssh debug.1 -f production`

`rm -r /var/www/public/tmp/splits/*`

delete the files from `ttm-content/movies`
