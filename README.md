# Meteor Router

Meteor Router builds on [page.js](https://github.com/visionmedia/page.js) to provide reactive, filterable routing for [Meteor](http://www.meteor.com/) applications.

## Installation

Meteor Router can be installed with [Meteorite](https://github.com/oortcloud/meteorite/). From inside a Meteorite-managed app:

``` sh
$ mrt add router
```

## API

### Basics

To get the current page:

``` javascript
Meteor.Router.page();
```

This is a reactive variable which will trigger invalidations as the app changes pages. Usually, you'll just want to render the template that corresponds to the current page:

``` handlebars
{{renderPage}}
```

To define a route, simply specify the URL it matches and the name of the template it should render. If you want to get fancy, you can specify a reactive function that returns a template name. It will get repeatedly executed as its reactive dependencies change.

``` javascript
Meteor.Router.add({
  '/news': 'news',

  '/about': function() {
    if (Session.get('aboutUs')) {
      return 'aboutUs';
    } else {
      return 'aboutThem';
    }
  },

  '*': 'not_found'
});
```

To navigate to such a URL from in the app, either create a link which links to the URL (the router will intercept clicks and trigger relevant state changes), or call directly:

``` javascript
Meteor.Router.to('/news');
```

Note that this doesn't reload the app, it instead uses HTML5 `pushState` to change the URL whilst remaining loaded.

### Route functions

When the route function is called, `this` corresponds to a page.js [`Context`](https://github.com/visionmedia/page.js#context) object, allowing you to do the following:

``` javascript
Meteor.Router.add({
  'posts/:id': function(id) {
    console.log('we are at ' + this.canonicalPath);
    console.log("our parameters: " + this.params);

    // access parameters in order a function args too
    Session.set('currentPostId', id);
    return 'showPost';
  }
});
```

### Filtering

The current system of filtering in this package is the equivalent of an `after\_filter` in Rails. To add a filter which will render the correct template for a page which requires login:

``` javascript
Meteor.Router.filters({
  'checkLoggedIn': function(page) {
    if (Meteor.loggingIn()) {
      return 'loading';
    } else if (Meteor.user()) {
      return page;
    } else {
      return 'signin';
    }
  }
});
```

To turn the filter on, use one of:

``` javascript
// applies to all pages
Meteor.Router.filter('checkLoggedIn');

// applies to specific pages
Meteor.Router.filter('checkLoggedIn', {only: 'profile'});
Meteor.Router.filter('checkLoggedIn', {except: 'home'});

// accepts an array of pages
Meteor.Router.filter('checkLoggedIn', {only: ['profile', 'notifications'] });
Meteor.Router.filter('checkLoggedIn', {except: ['home', 'browse'] });
```

Note that filters build on reactivity. So the URL will not change but the user will see different pages as the state of the `Meteor.user()` property changes.

### Server-side routing

The router also allows a very simple server side routing function with a similar API:

``` javascript
Meteor.Router.add('/posts/:id.xml', function(id) {
  return constructXMLForId(Posts.findOne(id));
});
```

Optionally you can also restrict the route by HTTP method:

``` javascript
Meteor.Router.add('/posts/:id.xml', 'GET', function(id) {
  return constructXMLForId(Posts.findOne(id));
});
Meteor.Router.add('/posts/:id.xml', 'DELETE', function(id) {
  Posts.remove(id);
  return [204, 'No Content'];
});
```

The arguments to the routing function are the parameters you've specified in your URL, and the `this` within the function is an object with three properties:

* `this.params` -- the list of parameters, page.js style
* `this.request` -- a [Connect](http://www.senchalabs.org/connect/) request
* `this.response` -- a Connect response (use this to e.g. set headers on your response).

Your routing function can return one of the following:

* a string, the body of the response
* a number, the http status code
* an array, in one of the following forms:
  * `[body]`
  * `[statusCode, body]`
  * `[statusCode, headers, body]`, where `headers` is an object mapping header names to values.

Alternatively, rather than a routing function, you can just provide a fixed response:

``` javascript
Meteor.Router.add('/404', [404, "There's nothing here!"]);
```

**NOTE**: Spark (meteor's template engine) does not currently run server side, so you are limited in what you can return here. Most likely you will want to return fairly simple things like JSON or XML documents, the construction of which is up to you.

## Examples

Check out `examples/simple-routed-app` for an extremely simple example of a filtered routed app. (To run, use meteorite: `cd examples/simple-routed-app; mrt run`).

Additionally, you might want to read [my blog post](http://bindle.me/blog/index.php/679/page-transitions-in-meteor-getleague-com) on page transitions in Meteor.

## Internet explorer 8+ support

The current version of the router doesn't support IE8 + 9. An [update is coming soon](https://github.com/tmeasday/meteor-router/issues/33). In the meantime, if you absolutely need it, you can use the `ie-support` branch:
```json
"packages": {
  "HTML5-History-API": {},
  "router": {
    "git": "https://github.com/tmeasday/meteor-router.git",
    "branch": "ie-support"
  }
}
```


## Contributing

To run the tests, ensure that the router is checked out to a folder called `router`, and then simply run:

``` sh
$ mrt
```
