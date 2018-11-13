## List of Performance Concerns 

### Primary

1. Network latency 
    - Reduce HTTP requests
    - Cache like a bad ass
2. In-browser performance
    - When possible use CSS instead of JS (or, to a lesser extent, images)
    - Avoid repaints, problematic transforms and animation where possible
    - JS perf (? please expand ?)
3. Reduce file sizes
    - Supports the above two points
    - Automate: work with full size, separate assets, build into optomized and concatenated files for distrobution
4. Perceptual speed
    - Make sure the app appears fast, or at least working, moving, at all times
    - Conditionally / lazy load supporting assets after page blocking assets

### Secondary

1. Battery use optimization

## Best practices

From [Javascript performance on mobile devices](http://www.kendoui.com/blogs/teamblog/posts/11-10-07/javascript_performance_on_mobile_devices.aspx):
* Use CSS3 transitions instead of javascript for animations; they're optimized for the GPU of the device.
  * IE 8 and 9 don't support CSS3 transitions!!!!
      * But they're easy to make fallbacks for!
* And the end of this article reveals it's shilling for KendoUI.

From [25 Techniques for Javascript Performance](http://desalasworks.com/article/javascript-performance-techniques/), they fall into 5 categories:

1. Avoid interaction with host objects
2. Manage and Actively reduce your Dependencies
  * [Sizzle](http://sizzlejs.com/) is recommended as a lighter-weight jquery alternative for css selectors
  * Min.js, zepto, etc.
3. Be disciplined with event binding
4. Maximise the efficiency of your iterations
5. Become friends with the JavaScript lexicon

From [The Difference Between jquery's bind, live, and delegate](http://www.alfajango.com/blog/the-difference-between-jquerys-bind-live-and-delegate/), prefer `bind` or `delegate` over `on` (`live` has been deprecated)

[Yahoo - Best Practices for Speeding Up Your Web Site](http://developer.yahoo.com/performance/rules.html). 35 best practices in 7 categories, the mobile category says:

1. Keep Components Under 25 KB - in [2008](http://yuiblog.com/blog/2008/02/06/iphone-cacheability/), the iphone wouldn't cache any component larger than 25kb.
  * NOPE [this 2010 article](http://www.yuiblog.com/blog/2010/07/12/mobile-browser-cache-limits-revisited/) says it's more like 4MB for css/js. [More current browserstack results](http://www.browserscope.org/user/tests/table/agt1YS1wcm9maWxlcnINCxIEVGVzdBj_1OsBDA?v=3&layout=simple&f=Max%20js%20Cache%20Size%20%28kB%29) show 2MB for the stock android browser, 4MB for iphone/ipad.
2. Pack Components Into a Multipart Document

[If You're Programming a Cell Phone Like a Server You're Doing it Wrong](http://highscalability.com/blog/2013/9/18/if-youre-programming-a-cell-phone-like-a-server-youre-doing.html#) - Big cookie strategy-- batch data transfers and prefetch data for the next 2-5 mins (1-5mb) to minimize the amount of time the cell radio is active.

[Ebooks by Amy Hoy and Thomas Fuchs](http://javascriptrocks.com/performance/) that i'm probably going to buy and share

[Why mobile web apps are slow](http://sealedabstract.com/rants/why-mobile-web-apps-are-slow/) - 10k word, 100 citation article

[HTML5Rocks Performance Guide](http://www.html5rocks.com/en/features/performance)

### Things we already do

* Minify
* post-load dependency management with RequireJS

## Profiling

* Google - [Page Speed Insights](https://developers.google.com/speed/pagespeed/insights)
* [jsperf.com](http://jsperf.com) for testing specific pieces of code on different browsers kinda like browserscope but lighter weight
* [Mobile Perf Bookmarklet](http://stevesouders.com/mobileperf/mobileperfbkm.php) - a bookmarklet that links to all the other performance bookmarklets (firebug lite, dom monster) so that you only have to create one bookmarklet on your mobile device

* [Browserscope](http://www.browserscope.org/) profiles browsers in general

* "a Web site or app that loads faster is better than one that is slow" because it feels slower to the end user - [Measuring site performance with JavaScript on mobile](http://blog.trasatti.it/2012/11/measuring-site-performance-with-javascript-on-mobile.html)
