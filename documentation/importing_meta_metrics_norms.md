#Updating MetaMetrics Norms Data for your Local Environment

Norms data gets added to benchmarks in https://github.com/thinkthroughmath/apangea/pull/2507,
but you'll need to import the data itself from the MM-provided excel files for your
tests to update with this information.

## Populating Norms

The source data has already been prepped and uploaded to the TTM-Dev-Content
bucket.  To create the necessary norms on your local (or on production)

1.  Open a rails console
2.  Instantiate an importer
    `importeur = MetaMetricsNormsImporter.new 'ttm-dev-content', 'meta_metrics_uploads/norms'`
3.  Call `importeur.import` to import or update all the data.

### Note for Posterity :: How to prep the xls docs into csvs QUICKLY

Meta metrics delivered the norms data in three excel files,
each with a sheet for the norms data for one grade.  It takes a
LONG TIME to export each sheet as a separate csv.  But google docs is scriptable
so to convert MM's many many xls docs into folders of csv's that you can download:
https://script.google.com/a/thinkthroughmath.com/macros/d/MskucNYCV1gBhaSvJl4yImZxeXXwTpFvz/edit?uiv=2&mid=ACjPJvGeiCYbneOyADQczxRh3vjYmibkWYLEPT0L2AKywtXHL9Emq938PGzv7MzKrg5PK2KTGIjpIVZd9FBqLN-0y7sWRZ6rPHb1gfrnxsAb1jjLDXeuHQJYFnSoog0jZ0-Lv6Zr7LxQUJU

Choose publish->test as add-on, then select the xls, then pick 'csv' from the top menu.

Download all the things to a local folder with the following structure:

```
root_folder
  fall
    exported csvs from fall xls
  winter
    exported csvs from winter xls
  spring
    exported csvs from spring xls
```

Then upload that to the bucket you want to use for the importer, and adjust the
call to the importer as needed.
