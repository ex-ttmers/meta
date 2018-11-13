## How to create a Classroom Goal
As a teacher
* Go to /motivation/classroom_goals
* Click "Manage Classroom Goals"
* Find desired classroom and choose a goal from the dropdown then hit create

To see it appear on the all motivations goals tab you have to:
```
rake etl:aggregate
```
but i also 
```
rake etl:incremental 
rake reporting:clear_cache - in apangea 
```
Just to be safe

## How to create a Contest
1) If there are no charities yet created
As a Super user:
* Go to /charities
* Click add a charity
* When you fill it out make sure to add somthing to Donation action like: Donation_Butts

## How to create a Charity
As a Super User
* go to /contests
* Click add a contest
