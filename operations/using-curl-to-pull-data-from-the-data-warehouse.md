You can use curl to pull data from the warehouse as json by hitting the endpoints and passing parameters. To do so you will need a few pieces of information.

### Which Server?
* production? `http://warehouse.thinkthroughmath.com`
* rc? `http://warehouse-staging.thinkthroughmath.com`
* lab? `http://warehouse-staging-lab.thinkthroughmath.com`
* dev? `http://warehouse-staging-dev.thinkthroughmath.com`
* qa? `http://warehouse-staging-qa.thinkthroughmath.com`
* 
### Which Report?
The defined endpoints are in the [routes file](https://github.com/thinkthroughmath/reporting/blob/rc/config/routes.rb).

### Which parameters?
This can be a little tricky and will need some experimentation depending on what you are looking for, but there's a list of supported parameters [here](https://github.com/thinkthroughmath/reporting/blob/rc/app/reports/param_parser.rb). Not all reports support all parameters though.

## Putting it together
you also need to add a secret token in the header, which you can get from the `ttmscalr config` list for the farm you are using. the value is `TTM_REPORTING_SECRET_TOKEN`

So the final command is `curl http://[SERVER]/[REPORT]?[FILTERS_AS_NAME=VALUE_PAIRS] -H 'Authorization: Token token="[REPORTING_SECRET_TOKEN_BY_ENVIRONMENT]"'`
