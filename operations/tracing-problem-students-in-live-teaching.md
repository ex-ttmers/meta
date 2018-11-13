* Go to http://papertrailapp.com and log in.
* Access the logs for TTM-Live-Teaching-Prod
* Search for the student username
* Find the value for studentToken in any chat message from the student
* Now clear that search and search for the student token value
* look for the log entry from heroku/router. The vallue for fwd= will be the IP address of the student. Note that this is most likely a building or other proxy server - you won't be able to find the student's actual IP.
* Go to http://ipaddress.com/reverse_ip/ and enter the IP. This should give you the city or location of the student. In some cases the IP may be registered to the district; in those cases you can work with Support to message an administrator or teacher there.
