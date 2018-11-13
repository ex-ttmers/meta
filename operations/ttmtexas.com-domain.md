As part of our contract with the state of Texas they believe they are getting a 'Texas-specific instance' of TTM. There's been no discussion on what this would entail and it's not part of the contract, so we have tried to do as little as possible to meet the spirit of the law.

### Vanity domain
The domain http://lms.ttmtexas.com is an alias for http://lms.thinkthroughmath.com, assigned at the AWS Elastic Load Balancer. This only works in HTTP since ELB only allows one SSL certificate per ELB, and our setup with Scalr doesn't allow us to use two different ELBs. So when you hit the home page with HTTP it maintains the correct hostname, but under SSL we force everything to redirect to the lms.thinkthroughmath.com domain so that the certificate doesn't error. We do that by using [this method](https://github.com/thinkthroughmath/apangea/blob/rc/app/controllers/application_controller.rb#L69) to rewrite the url for the login form post based on the current environment. This reads the value of the ```BASE_URL``` environment variable to construct the correct URL.

### Customizations
When the domain is ttmtexas.com we customize the layouts a bit. In particular we add a ttmtexas logo on the [login page](https://github.com/thinkthroughmath/apangea/blob/rc/app/views/devise/sessions/new.html.erb#L10). There are also a couple of static routes to the [custom Texas-specific signup page](https://github.com/thinkthroughmath/apangea/blob/rc/config/routes.rb#L613).

### Custom signup flow
Speaking of custom signup... As part of the contract Texas stipulates some self-registration options. Specifically:
* Parents in Texas should be able to self-register as long as they know their school, and they can create students
* Teachers can self-register if they know their school's County-District-Campus number
* School and district-level administrators can request an account (but those must be independently verified manually)

These options are included in the [custom registration form](https://github.com/thinkthroughmath/apangea/blob/rc/app/views/customer_registrations/_texas_registration_form.html.slim) that's also keyed off of the [controller](https://github.com/thinkthroughmath/apangea/blob/rc/app/controllers/customer_registrations_controller.rb) based on hostname.
