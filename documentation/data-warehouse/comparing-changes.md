### Purpose

Sometimes when we make changes to the reporting application, we want to know how it will effect the data in each report.

### Solution!

So we wrote a simple tool,[endpoint-comparer](https://github.com/thinkthroughmath/endpoint-comparer). The basic idea is that you give it a control and experiment servers a bunch of endpoints.  It will then hit both servers for every endpoint and report only the differences.

### How to setup an Experiment server?

1. reserve a farm with mathbot.  `mathbot reserve review`
2. shutdown ETL on the warehouse for that farm.

    1. login to scalr, and set the min and max servers for the sidekiq role to 0's

3. disable mathbot / jenkins from being able to deploy to this farm

    1. goto http://jenkins.thinkthroughmath.com:8080/job/deploy-reporting/
    2. click 'configure', if you don't see this link, you'll have to ask for access
    3. remove all references to the farm you are using from the 'FARM' property. Remember these changes because you will want to set them back when you are done
    4. save the job

### How to use it?

1. Clone the repo

        git clone git@github.com:thinkthroughmath/endpoint-comparer.git

2. Copy down the [endpoint data list](../data/endpoints.txt) to the endpoint-comparer directory. This might need updated when you use it
3. Copy the test.yaml.template to test.yaml

        cp test.yaml.template test.yaml

4. Enter the URL and reporting token for both servers in `test.yaml`
5. run the diff!

        bundle
        ruby run.rb

6. View the results at `results/results.diff`.  We only capture results if there are differences, so empty results could mean no differences.
