[koa]:https://github.com/koajs/koa/
[handlebars]:http://handlebarsjs.com
koa-hbs
=======

[Handlebars][handlebars] templates for [Koa][koa]

[![Build Status][travis-badge]][repo-url]

## Usage
koa-hbs is middleware. We stash an instance of koa-hbs for you in the library
so you don't have to manage it separately. Configure the default instance by
passing an [options](#options) hash to #middleware. To render a template then, just `yield this.render('templateName');`. Here's a basic app demonstrating all that:

```javascript
var koa = require('koa');
var hbs = require('koa-hbs');

var app = koa();

// koa-hbs is middleware. `use` it before you want to render a view
app.use(hbs.middleware({
  viewPath: __dirname + '/views'
}));

// Render is attached to the koa context. Call `this.render` in your middleware
// to attach rendered html to the koa response body.
app.use(function *() {
  yield this.render('main', {title: 'koa-hbs'});
})

app.listen(3000);
```

After a template has been rendered, the template function is cached. `#render`
accepts two arguements - the template to render, and an object containing local
variables to be inserted into the template. The result is assigned to Koa's
`this.response.body`.

### Registering Helpers
Helpers are registered using the #registerHelper method. Here is an example
using the default instance (helper stolen from official Handlebars
[docs](http://handlebarsjs.com):

```javascript
hbs = require('koa-hbs');

hbs.registerHelper('link', function(text, url) {
  text = hbs.Utils.escapeExpression(text);
  url  = hbs.Utils.escapeExpression(url);

  var result = '<a href="' + url + '">' + text + '</a>';

  return new hbs.SafeString(result);
});
```
Your helper is then accessible in all views by using, `{{link "Google" "http://google.com"}}`

The `registerHelper`, `Utils`, and `SafeString` methods all proxy to an
internal Handlebars instance. If passing an alternative instance of
Handlebars to the middleware configurator, make sure to do so before
registering helpers via the koa-hbs proxy of the above functions, or
just register your helpers directly via your Handlebars instance.

You can also access the current Koa context in your helper. If you want to have
a helper that outputs the current URL, you could write a helper like the following
and call it in any template as `{{requestURL}}`.

```
hbs.registerHelper('requestURL', function() {
  var url = hbs.templateOptions.data.koa.request.url;
  return url;
});
```

### Registering Partials
The simple way to register partials is to stick them all in a directory, and
pass the `partialsPath` option when generating the middleware. Say your views
are in `./views`, and your partials are in `./views/partials`. Configuring the
middleware via

```
app.use(hbs.middleware({
  viewPath: __dirname + '/views',
  partialsPath: __dirname + '/views/partials'
}));
```

will cause them to be automatically registered. Alternatively, you may register partials one at a time by calling `hbs.registerPartial` which proxies to the cached handlebars `#registerPartial` method.

### Layouts
Passing `defaultLayout` with the a layout name will cause all templates to be
inserted into the `{{{body}}}` expression of the layout. This might look like
the following.

```html
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  {{{body}}}
</body>
</html>
```

In addition to, or alternatively, you may specify a layout to render a template
into. Simply specify `{{!< layoutName }}` somewhere in your template. koa-hbs
will load your layout from `layoutsPath` if defined, or from `viewPath`
otherwise.

At this time, only a single content block (`{{{body}}}`) is supported.

### Block content
Reserve areas in a layout by using the `block` helper like so.

```html
{{#block "sidebar"}}
  <!-- default content for the sidebar block -->
{{/block}}
```

Then in a template, use the `contentFor` helper to render content into the
block.

```html
{{#contentFor "sidebar"}}
  <aside>
    <h2>{{sidebarTitleLocal}}</h2>
    <p>{{sidebarContentLocal}}</p>
  </aside>
{{/contentFor}}
```

### Options
The plan for koa-hbs is to offer identical functionality as express-hbs
(eventaully). These options are supported _now_.

- `viewPath`: [_required_] Full path from which to load templates
  (`Array|String`)
- `handlebars`: Pass your own instance of handlebars
- `templateOptions`: Hash of
  [handlebars options](http://handlebarsjs.com/execution.html#Options) to pass
  to `template()`
- `extname`: Alter the default template extension (default: `'.hbs'`)
- `partialsPath`: Full path to partials directory (`Array|String`)
- `defaultLayout`: Name of the default layout
- `layoutsPath`: Full path to layouts directory (`String`)
- `contentHelperName`: Alter `contentFor` helper name
- `blockHelperName`: Alter `block` helper name
- `disableCache`: Disable template caching

### Locals

Application local variables (```[this.state](https://github.com/koajs/koa/blob/master/docs/api/context.md#ctxstate)```) are provided to all templates rendered within the application.

```javascript
app.use(function *() {
  this.state.title = 'My App';
  this.state.email = 'me@myapp.com';
});
```

The state object is a JavaScript Object. The properties added to it will be exposed as local variables within your views.

```
<title>{{title}}</title>

<p>Contact : {{email}}</p>
```

## Example
You can run the included example via `npm install koa` and
`node --harmony app.js` from the example folder.

## Unsupported Features

Here's a few things _koa-hbs_ does not plan to support unless someone can provide really compelling justification.

### Async Helpers
_koa-hbs_ does not support asynchronous helpers. No, really - just load your data before rendering a view. This helps on performance and separation of concerns in your app.

### Disable template caching
Add a new option `disableCache` for support this feature, but for performance reasons remember turn off this option for production environment.

## Credits
Functionality and code were inspired/taken from
[express-hbs](https://github.com/barc/express-hbs/).

[travis-badge]: https://travis-ci.org/jwilm/koa-hbs.png?branch=master
[repo-url]: https://travis-ci.org/jwilm/koa-hbs
