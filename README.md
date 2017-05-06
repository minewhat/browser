# Browser
Remote management client for browsers supporting the [ Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/), built to be easy to use. And futher than that, be ready for production use. Currently being battle-tested over at [Passmarked](https://passmarked.com) before we launch.

We've been quite excited for the developments allowing us to better use Chrome, and move away from PhantomJS (which has a ever aging version of webkit). After following the announcements we wanted to get ready for the switch and started building a library we would able to use in our production setup.

We set out with a few goals:

* Learn from [PhantomJs](https://phantomjs.org) which we have been using for a year now and address issues we had.
* Layout the API in a way similiar to a simple HTTP client like [request](https://github.com/request/request). To reduce the barrier to entry for everyone.
* Built from the ground up to support parallel processing and a keeping a long running instance sane by cleaning up every chance it gets.
* Built in configurable queue-ing as to make it easy to build systems that won't fall over once traffic starts hitting
* Keeping it generic enough to use multiple browsers supporting the same protocol as well.
* Build generic convience methods to get information users are normally interested in. Such as building a HAR, taking screenshots, emulation, in page code execution (much like [PhantomJs](https://phantomjs.org)).

Join our [Slack](http://passmarked.com/chat) channel to join the conversation and help us build this out.

## Status

As you might know, tons of features like the `--headless` feature from Chrome is still only in Canary. With it being released to mainline stream in the next release. **With a growing list of features following.**

### Development Status

Currently building out and figuring out the API, will be ready for use in the next week. Currently release does work, API might just be tweaked a bit after usage.

TODO:

* [ ] Add samples
* [ ] Add proper logging with options
* [ ] Add testing suite
* [ ] Testing after integration with our live system

Current version - 0.0.1

### Production Status

The module is currently being tested for use in our website testing suite at [Passmarked](https://passmarked.com), which we hope to have out of beta in the next few weeks and running Chrome in headless mode. Expect tons of updates and performance updates.

Join our [Slack](http://passmarked.com/chat) channel to join the conversation and help us build this out.

## Installing

### Install the module

To install the module use either [NPM](https://npmjs.com) or [Yarn](https://yarnpkg.com/en/):

```bash
npm install [-g] [--save] @passmarked/browser
```

or

```bash
yarn install add [--save] @passmarked/browser
```

### Run a instance of browser

To use the library we need a browser instance we can connect to. As of writing this (6 May 2017) `--headless` is only available in Canary along with most of the required features to run the library. So first download and install [Chrome Canary](https://www.google.com/chrome/browser/desktop/).

Then run using:

```
google-chrome-canary [--headless] --disable-gpu --remote-debugging-port=9222
```

Using `--headless` will allow a server to run the instance of the browser, or simply on your desktop without showing a actual browser. No **xvfb** needed !

####Noteable mentions:

* It seems at the moment Chrome only allows connections from `localhost`, (being carefull off course) you can open up to a ip using a reverse proxy like NGINX. See [samples/nginx.conf](https://github.com/passmarked/browser/blob/master/samples/nginx.conf) for a sample NGINX config.
* For security reasons Chrome does not return the amount of memory being used precisely, to enable and use the `page.getMemory()` function add `--enable-precise-memory-info`.
* For now `--disable-gpu` is required in `--headless` mode as issues are being sorted out.

## Running

As example to get you up and running quick, here is a sample showing how to save a screenshot with the screenshot of [example.com](http://example.com):

```nodejs
// Gotta start somewhere !
const Browser = require('@passmarked/browser')

/** 
* create a client context that will be used to
* the remote browser instance. Supplying the host
* and port of the remote instance to connect to.
var client = Browser({

  host:    'localhost', // defaults host
  port:    9222, // default port

});

/**
* Load a page with the specified url and options.
* See #Options for all possible config options.
**/
client.load({

  url: 'http://example.com',
  background: '#fff // default background to white,
  timeout: 30 * 1000 // 30 second timeout

}, function(err, page) {

  // check for a error related to connecting
  // to the remote instance
  if(err) {
  
    // debugging message
    console.log('failed to connect');
    
    // output the error
    console.dir(err);
    
    // done
    return;
  
  }
  
  // get the result of the connection
  var result = page.getResult();
  
  // check if we were able to load
  if(result != 'success') {
  
    // output the result
    console.log('Problem loading page: ' + result);
    
    // close the page context to allow next page
    // to run in queue -- see #tabs under #Options
    page.close();
    
    // finish up
    return;
  
  }
  
  // render pdf
  page.render({
  
    format:      'jpeg', // pdf, png or jpeg
    width:       320,
    height:      480,
    mobile:      true,
    orientation: ''
  
  }, function(err, buf) {
  
    // write the buffer to file
    fs.writeFile('output.pdf', buf, function() {
    
      // close the context
      page.close();
    
      // done
      console.log('loaded, and writen to output.pdf')
    
    });
  
  });
  
});
*/

});
```

## Options

Various options are provided which control how the library interacts with the browser and what the tab itself will look like (IE emulation/width/height etc).

### Browser Options

These options apply only when creating a browser client:

* **host** - Host where the remote instance of Chrome is running - default is `localhost`
* **port** - Port where the remote instance is running - default is `9222`
* **tabs** - Maximum number of tabs to run in parallel - default is `1`

### Browser/Page Options

Options that can be applied to pages being loaded which control how it looks/feels. Options can be supplied to both the browser and the `.load` function, where `.load` will default to the browser options and override when passed anything in the `.load` method itself.

* **url** - URL to load
* **timeout** - Number of ms before stopping and returning a "timeout" error - default no limit
* **disableCache** - TRUE/FALSE to enable or disable cache - default FALSE
* **bypassServiceWorkers** - TRUE/FALSE to enable or disable server worker cache - default FALSE
* **ignoreSSL** - TRUE/FALSE to return of any SSL verification - default TRUE
* **userAgent** - String of user agent to use when sending request - default set by system/browser
* **cpu** - Trottle the cpu, number from 1 up. 1 = normal speed, 2=x2 slower. And so on.
* **downloadRate** - Trottle the speed of network in bytes
* **uploadRate** - Trottle the speed of network in bytes
* **latency** - Amount of latecy to add to each request
* **offline** - Emulate the network being offline
* **connectionType** - Emulate a network type (packet drops and lateny fluctations) = `2g, 3g, adsl`
* **width** - Width of tab
* **height** - Height of tab (viewport)
* **mobile** - TRUE/FALSE to emulate mobile behaviour
* **scale** - Adjust the scale factor for mobile, default=1
* **orientation** - Adjust the orientation of the tab, could be `portraitPrimary`, `portraitSecondary`, `landscapePrimary`, `landscapeSecondary`
* **media** - Emulate a media for `@media` queries - IE `'print'`
* **recordEvery** - Record a screenshot every supplied ms - IE `100` for every 100ms
* **headers** - Map of headers to send with request
* **cookies** - Cookies to send in request

## API

There are a few important calls that are needed to work with the library:

### Browser(options)

Creates a client that will allow you to load pages.

```
const Browser = require('@passmarked/browser');
var browser = Browser({ ... options ... });
```

Options provided for the browser will be the default for all pages loaded in that browser, unless override by the `.load` calls themselves.

> The browser client does not maintain a connection to Chrome instances, rather it simply saves the options and connects as needed. Each page context is a websocket connection being used to control the remote browser.

### browser.load(options, fn)

Creates a page context that can be interacted with. When created the context is assigned to a tab in the browser, and as per configurations (see #tabs under #Options) .load calls exceeding the currently executing tabs will **block** till a tab is available.

```
browser.load({ ... options ... }, function(err, page) {
	page <- the context for the page
});
```

The function will use any options defined when creating the browser connection, but any of those options can also be overrided here:

```
browser.load({ ... options ... }, function(err, page) {
	page <- the context for the page
});
```

### page.close()

Closes the context and adds the tab back to pool, which will cause the next url specified to load. This must be called to continue processing once you are done with a page. The function also handles cleaning up and closing the connections. 

Along with flushing stale tabs once reaching idle again.

```nodejs
page.close()
```

### page.getResult()

Returns the result of trying to connect to the specified URL which is either:

* **success** - all good was able to connect and find a document to respond. This could be a any status code just as long as we are able to load the page in Chrome without connection problems.
* **insecure** - There was a issue connecting due to HTTPS/SSL issues - see #ignoreSSL under #Options to disable checking
* **timeout** - The configured timeout was exceeded
* **failed** - We just not able to connect to the specified url

## Page Functions

The module also provides quite a few convience functions that can be accessed once a page context has be retrieved:

### page.getURL(fn)

Returns the final URL (supports Pushstate as well) of the page:

```nodejs
page.getURL(function(err, url) { 
  console.log(url); 
});
```

### page.getTrackedWebsockets(fn)

Returns a list of all websockets connections that took place:

```nodejs
page.getTrackedWebsockets(function(err, websockets) { 
  console.dir(websockets)
});
```

### page.getTrackedWebsocketFrames(fn)

Returns a list of all frames sent and received over websocket connections:

```nodejs
page.getTrackedWebsocketFrames(function(err, frames) { 
  console.dir(frames)
});
```

### page.getConsoleMessages(fn)

Returns a list of all console messages along with severity logged to the javascript console

```nodejs
page.getConsoleMessages(function(err, messages) { 
  console.dir(messages)
});
```

### page.getJavascriptDialogs(fn)

Returns a list of javascript dialogs encountered while loading the page along with text and type (prompt/alert)

```nodejs
page.getJavascriptDialogs(function(err, dialogs) { 
  console.dir(dialogs)
});
```

### page.getNavigationHistory(fn)

Returns the navigation history of documents that led to the final doc:

```nodejs
page.getNavigationHistory(function(err, documents) { 
  console.dir(documents)
});
```

### page.getDocument(fn)

Returns the raw request/response for the final page document:

```nodejs
page.getDocument(function(err, doc) { 
  console.dir(doc)
});
```

### page.getDocument(fn)

Returns the raw request/response for the final page document:

```nodejs
page.getDocument(function(err, doc) { 
  console.dir(doc)
});
```

### page.getMemoryUsage(fn)

Returns the memory usage of the Heap in bytes.

```nodejs
page.getMemoryUsage(function(err, memory) { 
  console.log((memory / 1024) + 'kb used'); 
});
```

### page.render(params, fn)

Renders out the page in various formats, with options to emulate various sizes and devices:


```nodejs
page.render({

	format: 'png', // REQUIRED - jpeg, png or pdf,
	quality: 30, // OPTIONAL - quality of jpeg produced, default 100
	
	// OPTIONAL Emulation Settings
	width: 320, // width of page
	height: 420, // height of page - null = till the bottom
	scale: 1, // scale factor of viewport
	orientation: 'portraitPrimary',
	media: 'print', // emulate a @media query media
	
}, {
  hello: 'world' // params to pass
}, function(err, value) {
  console.log(value); // http://example.com
});
```

### page.exec(functionToExec, params, fn)

Runs the specified javascript in the context of the page itself. 

Handles serializing the javascript, sending arguments and retrieving the response.

Supports returning values/objects or just nothing.

```nodejs
page.exec(function(params) { 
  return document.location.toString()
}, {
  hello: 'world' // params to pass
}, function(err, value) {
  console.log(value); // http://example.com
});
```

### page.getHAR(fn)

Returns the requests/responses formatted according to the [HAR](https://dvcs.w3.org/hg/webperf/raw-file/tip/specs/HAR/Overview.html) specification:

```nodejs
page.getHAR(function(err, har) { 
  console.dir(har); 
});
```

### page.getContent(fn)

Returns the content (after executing javascript) of the page:

```nodejs
page.getContent(function(err, content) { 
  console.log(content); 
});
```

### page.getSize(fn)

Returns the total size of the page in bytes:

```nodejs
page.getSize(function(err, size) { 
  console.dir(size); 
});
```

### page.getDuration(fn)

Returns the total time it took for the page to load in ms:

```nodejs
page.getDuration(function(err, duration) { 
  console.dir(duration); 
});
```

### 

## License

Copyright 2017 Passmarked Inc

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.