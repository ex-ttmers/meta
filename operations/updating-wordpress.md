## The Custom Geoswitch functionality
The custom Geoswitch plugin consists of a server side component [here](https://github.com/thinkthroughmath/ttm-wp-geoswitch) and a [small flux app](https://github.com/thinkthroughmath/ttm_maxmind_geoip_flux) that allows the end user to set and change their geolocation. 

Steps to recreate the custom geoswitch if there's a problem or the theme wipes out the changes.

Make sure that an update hasn't removed the settings for the GeoIP Plugin. Visit:
`[WORDPRESS_HOST]/wp-admin/options-general.php?page=geoswitch%2Fclass.geoswitch_admin.php`

Credentials should be:
* User ID: `96337`
* License Key: `GgXXyLYD1WSv`

Make sure it's set to `Local Database`, the database name is `GeoLite2-City.mmdb`, and the distance is in miles.

The customization for the geoswitch is made up of several js and css files, and a block of html for the switcher itself.
These can all be added to the Custom Actions found in the Theme Panel menu: 
`[WORDPRESS_HOST]/wp-admin/admin.php?page=wpex-panel-user-actions`

The CSS links go in the `wp_head` custom action
```
<link rel="stylesheet" type="text/css" href="/wp-content/themes/Total/css/chosen.min.css" media="screen">
<link rel="stylesheet" type="text/css" href="/wp-content/themes/Total/css/state-chooser.css" media="screen">
```
The JS links go in the `wp_footer` custom action
```
<script type="text/javascript" src="/wp-content/themes/Total/js/plugins/jquery.cookie-1.4.1.min.js"></script>
<script type="text/javascript" src="/wp-content/themes/Total/js/plugins/chosen.jquery.min.js"></script>
<script type="text/javascript" src="/wp-content/themes/Total/js/geoip2.min.js"></script>
<script type="text/javascript" src="/wp-content/themes/Total/js/geolocale.min.js"></script>
```
The HTML to build the state selector panel goes in `wpex_hook_wrap_top`
That HTML is
```
<div class="geoswitch--wrapper">
    <div class="geoswitch--center"> 
      <div class="geoswitch--heading">Welcome</div>
      <p class="geoswitch--colortext">In order to ensure that you view the most applicable content, please select a default location. </p>
      <select class="geoswitch--select" data-placeholder="Choose a State...">
        <option value="NA:Not Listed">Not Listed</option>
        <option value="AL:Alabama">Alabama</option>
        <option value="AK:Alaska">Alaska</option>
        <option value="AZ:Arizona">Arizona</option>
        <option value="AR:Arkansas">Arkansas</option>
        <option value="CA:California">California</option>
        <option value="CO:Colorado">Colorado</option>
        <option value="CT:Connecticut">Connecticut</option>
        <option value="DE:Delaware">Delaware</option>
        <option value="DC:District Of Columbia">District Of Columbia</option>
        <option value="FL:Florida">Florida</option>
        <option value="GA:Georgia">Georgia</option>
        <option value="HI:Hawaii">Hawaii</option>
        <option value="ID:Idaho">Idaho</option>
        <option value="IL:Illinois">Illinois</option>
        <option value="IN:Indiana">Indiana</option>
        <option value="IA:Iowa">Iowa</option>
        <option value="KS:Kansas">Kansas</option>
        <option value="KY:Kentucky">Kentucky</option>
        <option value="LA:Louisiana">Louisiana</option>
        <option value="ME:Maine">Maine</option>
        <option value="MD:Maryland">Maryland</option>
        <option value="MA:Massachusetts">Massachusetts</option>
        <option value="MI:Michigan">Michigan</option>
        <option value="MN:Minnesota">Minnesota</option>
        <option value="MS:Mississippi">Mississippi</option>
        <option value="MO:Missouri">Missouri</option>
        <option value="MT:Montana">Montana</option>
        <option value="NE:Nebraska">Nebraska</option>
        <option value="NV:Nevada">Nevada</option>
        <option value="NH:New Hampshire">New Hampshire</option>
        <option value="NJ:New Jersey">New Jersey</option>
        <option value="NM:New Mexico">New Mexico</option>
        <option value="NY:New York">New York</option>
        <option value="NC:North Carolina">North Carolina</option>
        <option value="ND:North Dakota">North Dakota</option>
        <option value="OH:Ohio">Ohio</option>
        <option value="OK:Oklahoma">Oklahoma</option>
        <option value="OR:Oregon">Oregon</option>
        <option value="PA:Pennsylvania">Pennsylvania</option>
        <option value="RI:Rhode Island">Rhode Island</option>
        <option value="SC:South Carolina">South Carolina</option>
        <option value="SD:South Dakota">South Dakota</option>
        <option value="TN:Tennessee">Tennessee</option>
        <option value="TX:Texas">Texas</option>
        <option value="UT:Utah">Utah</option>
        <option value="VT:Vermont">Vermont</option>
        <option value="VA:Virginia">Virginia</option>
        <option value="WA:Washington">Washington</option>
        <option value="WV:West">West Virginia</option>
        <option value="WI:Wisconsin">Wisconsin</option>
        <option value="WY:Wyoming">Wyoming</option>
      </select>
      <div class="geoswitch--regulartext">If you need to change your location, just click the "Change State" link above</div>
      <div class="geoswitch--button">
        <a class="geoswitch--accept"> 
          Accept
        </a>
      </div>
    </div>
  </div>
```

Problems with the home page not setting state properly? Pagely has created a custom page cache bypass for the home page since caching was a problem. If it becomes a problem, go to the edit screen for the site home page and make sure the template is set to `Home Page`. That should fix it.

