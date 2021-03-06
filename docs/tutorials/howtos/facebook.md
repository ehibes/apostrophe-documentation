---
title: Facebook open graph tags in Apostrophe
layout: tutorial
---

Facebook open graph tags can improve the way your pages appear when shared on Facebook, provided you use them correctly. The most frequently requested tag is `og:image`, followed by `og:title`, `og:description` and `og:type`. Often you can get away with just setting `og:image`. We'll look at how to do that.

However, any URLs you do set must be absolute URLs.

## Absolute URLs in Apostrophe

For reasons unknown to the rest of the Internet, Facebook has never implemented support for parsing relative URLs, even if they start with a `/`. They consider it an error. Why? Who knows.

In Apostrophe 2.3 or better you can get absolute URLs just by setting the `baseUrl` option:

```
var apos = require('apostrophe')({
  // Other options, then...
  // Note: NO TRAILING SLASH
  baseUrl: 'http://mysite.com'
  // Then other options...
});
```

Once you do this, all URLs generated by Apostrophe, including URLs for pages, permalink URLs for pieces and URLs for uploaded files (unless you have customized your uploadfs configuration) will be prefixed with `baseUrl`, making them absolute URLs.

Note that you SHOULD NOT add a trailing slash to this option.

## But what about URLs in development?

This is great, but you don't want your development URLs to point at your production server.

So use a `data/local.js` file on your production server. Everything here is *merged* with the object you pass to `apostrophe-site` in `app.js`. And `data/local.js` is never deployed. You set it up directly on the server. So you can set `absoluteUrls` differently in production. For instance, here's a typical production `data/local.js` file:

```
module.exports = {
  // Absolute URLs in production
  baseUrl: 'http://mysite.com',
  // minify css and js files in production
  modules: {
    'apostrophe-assets': {
      minify: true
    }
  }
}
```

You typically don't want to turn on `baseUrl` in your development environment, except for testing the feature.

> The object exported by `data/local.js` is merged with your Apostrophe configuration object. Since `data/local.js` is excluded from deployment, you can use it for server-specific settings.

## Great, I have absolute URLs. How do I generate my og:meta tags?

It's pretty easy, since `data.page._url` is now absolute, and so are your image URLs. Here's a classy example:

```
{# In lib/modules/apostrophe-templates/views/outerLayout.html #}
{% set context = data.piece or data.page %}
{% if context %}
  {% set firstImage = apos.images.first(data.context.thumbnail, 'thumbnail') or apos.images.first(data.context.body, 'body') %}
  {% set fbImage = apos.images.first(data.context, 'facebookImage') %}
  {% set description = apos.areas.plaintext(data.context.body) %}
  {% block extraHead %}
    {% if fbImage %}
      <meta property="og:image" content="{{ apos.attachments.url(fbImage, { size: 'full' }) }}" />
    {% elif firstImage %}
      <meta property="og:image" content="{{ apos.attachments.url(firstImage, { size: 'full' }) }}" />
    {% endif %}
    {% if description %}
      <meta name="description" property="og:description" content="{{ description | truncate(200) | e }}" />
    {% endif %}
    <meta property="og:url" content="{{ context._url }}">
    <meta property="og:title" content="{{ context.title }}">
  {% endblock %}
{% endif %}
```

### What's going on in this template?

* This is part of our `outerLayout.html` template, which all full-page templates extend when not doing an AJAX refresh. Some of those will just be "regular pages," others might be `show.html` templates for pieces. So we start by deciding if our context is a piece or a page. If it is neither, we don't try to set meta tags.

* If there is a singleton named `facebookImage` in our context (tip: add it to the schema), we use that for `og:url`. If not we use the first image we find in the `thumbnail` or `body` areas. This allows for custom Facebook images, if you want to include them in your page or piece schema, and also allows for Apostrophe to track something down to use instead.

* The body area is converted to plaintext and the first 200 characters are used to create the Facebook meta description. If you wanted, you could add a special Facebook description field to your schema, and output that instead.
