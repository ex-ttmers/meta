# Math Content Batch Updater

A tool that allows updating a batch of Structured Math Content items all at once from a formatted csv.

- This is only for Structured Math Content items (currently only multiple choice)
- It can only update In Process items (no published data)

There are glorious plans to make this tool be general enough that we would not need to modify the tool each time a new requirement comes in, but for now the specific columns that can be updated are white-listed.

Currently available updatable columns:

- Position (order)
- Stem's Display and Readable Text for English and Spanish
- Response's Display and Readable Text for English and Spanish
- Meta Metrics specific data (form name, qsc_id, etc....)

##### Known Caveat: Please make sure your csv files are UTF-8.
i.e. do not save them from excel.  If you do - control chars leak their way in.
Open in TextEdit and save as UTF-8

##### Another known caveat: Published Items aren't batch updateable!
Make sure that the ids in your csv file are the ids of the working version of the items; you cannot batch update already-published items. If you have a file with the published ids instead of the working ones, you can generate a new file by making sure that your local db has recent production content and doing something like the following in your apangea database (you will need to tweak filenames and columns to suit your needs):

```psql
CREATE TABLE tmp_meta_metrics (
  id int,
  display_text text,
  readable_text text);
  
copy tmp_meta_metrics (id, display_text, readable_text) from '/Users/mandybrown/ThinkThroughMath/apangea/B9-10_Stem_Eng_Readable_10192015_UTF-8.csv' with csv;

copy (
  select wv.id as id,
    tmp_meta_metrics.display_text,
    tmp_meta_metrics.readable_text
  from tmp_meta_metrics
  inner join items on tmp_meta_metrics.id = items.id
  inner join items wv on wv.version_of_item_id = items.id
)
to '/tmp/B9-10_Stem_Eng_Readable_Working_Version_Ids.txt' with csv header;
```

### Current Workflow
(again, plans to change - proposed workflow is below)

1. We fill in a formatted csv file with the Content department's data
2. We move the csv to the production farm
3. We run the `ItemsUpdateCsvParser` to update the data
4. We run any manual process afterwords they would need (i.e. generate audio, publish the items)

#
## How to move the data to the farm
To update the data on production, ftp the csv to the farm then run rails c on the farm like so:

````
ttmscalr sftp debug.1 -f production
cd /var/www
put yourfile.csv
````
Then run it in the console like so:

````
ttmscalr ssh debug.1 -f production
cd /var/www
rails c
````
And don't forget to remove the csv when you are done!

## ItemsUpdateCsvParser

#### Position

Position is the `Item.item_steps[0].position` field.

An example of a CSV for position is as follows:

````
id,value
10528,5
9949,8
9787,3
````

To update the **Position** from a csv:

````
params = {
  import_file: File.open('position.csv'),
  import_type: :position
}
ItemsUpdateCsvParser.new(params).update!
````

#### Stem Text

These are the `Item.math_content.stem.text` fields.

An example of a CSV: (used for english or spanish)

````
id,display_text,readable_text
10528,this is a display text for stem 10528,readable text for 10528
9949,display for 9949,readable for 9949
9787,display for 9787,readable for 9787
````

To update the **English Stem Text**:

````
params = {
  import_file: File.open('stem_english_text.csv'),
  import_type: :stem_english_text
}
ItemsUpdateCsvParser.new(params).update!
````

To update the **Spanish Stem Text**:

````
params = {
  import_file: File.open('stem_spanish_text.csv'),
  import_type: :stem_spanish_text
}
ItemsUpdateCsvParser.new(params).update!
````

#### Responses Text

These are the `Item.math_content.responses[].text` fields.

An example of a CSV: (used for english or spanish)

````
id,index,display_text,readable_text
10528,0,Display for response,Readable Response 0 for 10528
10528,1,Display for response,Readable Response 1 for 10528
10528,2,Display for response,Readable Response 2 for 10528
10528,3,Display for response,Readable Response 3 for 10528
9787,0,Display for response,Readable Response 0 for 9787
9787,1,Display for response,Readable Response 1 for 9787
9787,2,Display for response,Readable Response 2 for 9787
9787,3,Display for response,Readable Response 3 for 9787
````

To update the **English Responses Text**:

````
params = {
  import_file: File.open('response_english_text.csv'),
  import_type: :response_english_text
}
ItemsUpdateCsvParser.new(params).update!
````

To update the **Spanish Responses Text**:

````
params = {
  import_file: File.open('response_spanish_text.csv'),
  import_type: :response_spanish_text
}
ItemsUpdateCsvParser.new(params).update!
````

#### Meta Metrics Specific Data

This is used to update these fields:

 - `Item.test_name`
 - `Item.form_name`
 - `Item.strand`
 - `Item.qsc_id`
 - `Item.dok_level`
 - `Item.mm_cms_id`
 - `Item.qsc_measure`
 - `Item.player_settings['allow_protractor']`
 - `Item.player_settings['allow_calculator']`
 - `Item.player_settings['allow_ruler']`
 - `Item.player_settings['allow_formulas']`

An example of a CSV:

````
id,test_name,form_name,strand,qsc_id,dok_level,mm_cms_id,qsc_measure,allow_protractor,allow_calculator,allow_ruler,allow_formulas
22977,test,form,1,2,3,Q123,4,TRUE,TRUE,TRUE,TRUE
22975,test,form,,5,,,,,,,
````

Note: For the 1st row - all the columns will be updated.  For the 2nd row - only the test_name, form_name, and qsc_id will be.  The other columns will keep their current values (not overwriting with nil)

To update the **Meta Metrics Specific Data**:

````
params = {
  import_file: File.open('meta_metrics_specific.csv'),
  import_type: :meta_metrics_specific
}
ItemsUpdateCsvParser.new(params).update!
````

## Proposed Workflow

1. Content goes to a view in CMS
2. They choose from a list of available columns to be updated
3. They choose from a list of Structured Math Content items they want to update
4. They download a formatted CSV (that has the items and the columns to update)
5. They fill in the CSV with the data they want
6. They upload the CSV back into CMS
7. They choose optional things to do after the import (i.e. update it's audio, publish the items)
8. It updates the columns and runs any optional things they wanted


### Batch audio creation and item publishing

1. Gather your item ids into an array named `ids`
2. decide what audio, and language for each item to generate, and update the following line, and paste it into rails console

    ```
    params = {
    stem: true,
    responses: true,
    feedbacks: true,
    helps: true,
    languages: [:en, :es] }
    ```

3. Then run this loop

    ```
    ids.each do |id|
      Item.find(id).regenerate_audio(params)
    end
    ```

4. Wait until the sidekiq queue for generating audio emptys
5. Run this loop to publish the items

    ```
    ids.each do |id|
      Item.find(id).publish!
    end
    ```
