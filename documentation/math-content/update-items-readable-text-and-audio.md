# Update Items Readable Text and Audio

This script is a tool that allows for updating items' readable text and audio from our Content Team's item tracking documents.

### Column Headers the Script Uses:

  - Item Number
  - Stem - English
  - Stem - Spanish
  - Response [1..6] - English
  - Response [1..6] - Spanish
  - Feedback [1..6] Opening - English
  - Feedback [1..6] Opening - Spanish
  - Feedback [1..6] Body - English
  - Feedback [1..6] Body - Spanish
  - Feedback [1..6] Closing - English
  - Feedback [1..6] Closing - Spanish
  - Help 1 - English
  - Help 1 - Spanish
  - Help 2 - English
  - Help 2 - Spanish

_**Note:** When you see [1..6] listed above, this means that any integer between 1 and 6 needs to be used in its place._

### Downloading Item Tracking Document Steps

1. Navigate to the Content Team's current item tracking document
2. Locate and click the bottom tab with the requested batch identifier to be processed
3. Click File > Download as > Comma-seperated values (.csv, current sheet)

### Running the Script Steps

1. Upload the downloaded CSV to production
  - `ttmscalr scp debug.1 -f production --out --remote /var/www/items_info.csv --local ~/file_location/file_name.csv`
2. Connect to production via SSH
  - `ttmscalr ssh debug.1 -f production`
3. Open a tmux session (to prevent a broken pipe from stopping the script execution)
  - `tmux`
4. Navigate to the Rails directory
  - `cd /var/www`
5. Execute the script
  - `rails runner 'script/update_items_readable_text_and_audio.rb' 'items_info.csv'`
