TTM has a custom mustaching server. Because reasons. It's hooked in to mathbot so that you can type `mathbot mustache me [search terms]` and it will use the image search plugin to find an image, parse it through a mustaching service, and return the result.

The mustaching service is a heroku app named `ttm-mustachify`, running under our organizational account. It uses a [local fork](https://github.com/thinkthroughmath/mustachio) of a [branch of the mustachio sinatra app](https://github.com/wendelscardua/mustachio) that's been modded to use the [Sky Biometry](https://skybiometry.com) face detection algorithm. The API key and secret are stored as environment vars in the `ttm-mustachify` app, if you want to run them locally. We have a limit of 5000 requests per month before we have to pay.

If for some reason you need to access our Sky Biometry account, the credentials are in Lastpass.
