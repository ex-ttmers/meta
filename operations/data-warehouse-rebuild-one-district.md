# Data Warehouse: How to rebuild a single district

*** WARNING: These steps should not be performed during the day for a very good reason because it involves shutting down ETL; doing that delays reporting updates for EVERYONE. Best to wait until evening. ***

* Log into the debug role of the warehouse farm where you'd like to perform the rebuild.
* Start a tmux session so that if you get disconnected you can get back to this session and see the progress.
* Export scalr credentials: SCALR_KEY_ID, SCALR_ACCESS_KEY
* Export the alias of the current farm: `export FARM=dw-lab`
* Stop ETL: `ttmscalr shutdown:sidekiq -f $FARM`
* Run a single etl loop: `rake etl:incremental`
* Run a single unresolved record loop: `rake etl:unresolved_records`
* Perform the actual rebuild:
  * By customer: `CUSTOMER_ID=XXXX rake etl:rebuild_partial` 
  * By district: `DISTRICT_ID=XXXX rake etl:rebuild_partial`
* ***WARNING: Definitely don't ever do this step in the middle of the day. If running the rebuild during heavy load, just forgo the aggregation step and let the nightly (~2 AM)  aggregate job take care of it.***
  * If it's not the middle of the day, recreate aggregates: `rake etl:aggregate`
* Restart ETL: `ttmscalr restart sidekiq -f $FARM`
