This is the master language for the customer-facing KB article [here](https://thinkthroughlearning.zendesk.com/hc/en-us/articles/205929686).

## KB Text

TTM leverages the Amazon Web Service Cloud to help us quickly and easily scale to meet performance and capacity needs. We understand that schools need to ensure the privacy and security of their networks and their students, which is why we ensure that all DNS hostnames for Think Through Math services use the *.thinkthroughmath.com pattern.

Some firewalls and security devices cannot filter by domain name, only by IP. Since Amazon Web Services is a massive platform, the IP address ranges that TTM could be deployed on are very broad. Although we strongly recommend using DNS for traffic filtering, If you need to whitelist IP ranges for Think Through Math, these are the ranges of IP addresses TTM could use.

* [EC2/ELB servers](https://forums.aws.amazon.com/ann.jspa?annID=1701)
* [Cloudfront ranges](https://forums.aws.amazon.com/ann.jspa?annID=2051)
