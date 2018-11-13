CORS is crappy.

TTM's games were developed by a [third party](http://www.gamegurus.com/portfolio.html) and use the [impact.js](http://impactjs.com/) framework, which is based on HTML5 Canvas. The games are structured as engine + skin to add a level of variety to play. In the TTM framework we consider the games as 'content' vs. a part of the platform and each combination of engine+skin has its own [GitHub repo](https://github.com/thinkthroughmath/GG-AreaPerimeter). The static assets however are still versioned and loaded through S3/Cloudfront, which is a separate subdomain from the main games which are delivered from the lms.thinkthroughmath.com site. So at launch the workflow for when a game is loaded was:

1. The core game assets are loaded from lms
#. The game engine makes http requests to fetch images and sound from content.thinkthroughmath.com
#. The images and sound are added to the canvas created by the game

Different versions of IE have different requirements for CORS headers. After multiple attempts to satisfy them all we eventually just game up. So now we have implemented a passthrough proxy in the nginx config for our base web server image. All requests to /games are routed through the web server to the corresponding cloudfront directory. So requests for https://lms.thinkthroughmath.com/games/image01.png gets routed to https://content.thinkthroughmath.com/games/image01.png . This avoids the CORS loading issues with these assets since they come from the same host. It does mean that if you want to test the games in a production-level environment you need to ensure you have the ```TTM_BASE_URL``` set for the environment (which should be present, but something to check.
