We have some unique configurations in production with respect to subdomains and even alternate domains.

## Texas-specific features
Since Texas is our biggest customer and the funding for our usage is part of a statewide initiative, we've made some specific accommodations for their usage. There is a secondary domain, http://lms.ttmtexas.com , that points to the load balancer for our application. This unique domain is used in the app a couple of ways:
* It determines whether we display the Texas SUCCESS logo on our login and other non-password-protected pages
* It determines whether we show a link for customers to self-register for the application (which is normally not allowed)
Note that due to a limitation in ELB where it can have only one SSL certificate, we redirect all traffic to the primary hostname lms.thinkthroughmath.com after login, even if it came from Texas.

## LACOE
We also have a custom setup for one of our customers, LA County Office of Education (LACOE). They have a very outdated firewall policy and can only filter traffic by IP address. They serve a student juvenile detention population with our application so they have more strict requirements than most. Amazon changes the IP addresses matched to our hostname in ELB frequently (sometimes twice a week), which causes LACOE to need to reconfigure their firewall to match the change.
Because of this and because they are a large customer, we have set up an alternate static location for LACOE using Scalr's nginx-based load balancer. This is mapped to the subdomain lacoe.thinkthroughmath.com.
Normally we use the environment variable BASE_URL to define the hostname under which the SSL version of the site should run. In the case of lacoe we don't want the application to redirect, so in Stage.rb we only check the BASE_URL code if the subdomain is lms. otherwise we use the current hostname.
