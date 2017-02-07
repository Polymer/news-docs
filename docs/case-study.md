---
title: "Case study: the News app"
---

<!-- toc -->

News is a full-featured Progressive Web App (PWA) demo built using the Toolbox. You can try it out here:

<a href="https://news.polymer-project.org/" class="docs-button">Launch News demo</a>

This case study shows how News uses the principles of Progressive Web App (PWA) design to deliver a great
user experience.

## App Structure

Like other PWAs, News is designed with an [application shell architecture](https://developers.google.com/web/fundamentals/architecture/app-shell). The app shell element, `<news-app>`, contains the main framework for News. Here's the basic structure:

<div class="image-container layout horizontal">
  <div class="image-wrapper">
    <img src="app structure.jpg" alt="app structure" width="600px">
  </div>
</div>

* `<news-app>` lays out the structure for the application.
* `<app-route>` contains routing logic that parses the user's selected URL.
* `<news-data>` fetches resources from the raw HTML files and images in the /data/ folder and its subfolders.
* `<iron-pages>` switches between the News application's two views - list view and article view.
* `<news-list>` is the application's *list view*.
* `<news-article>` is the application's *article view*.

<div class="image-container layout horizontal">
  <div class="image-wrapper">
    <img src="news-list.jpg" alt="screenshot of the list view" width="300px">
  </div>
  <div class="image-wrapper">
    <img src="news-article.jpg" alt="screenshot of the article view" width="300px">
  </div>
</div>

## Views and routing

Views in the News app use the same implementation as the Shop app. [See the Shop app case study for more information on views and routing](https://www.polymer-project.org/1.0/toolbox/case-study#views).

For more information on encapsulated routing, see [Encapsulated routing with elements](https://www.polymer-project.org/1.0/blog/routing).

For more information on the `<app-route>` Polymer element used in these implementations, see the [`<app-route>` API reference](https://elements.polymer-project.org/elements/app-route).

There are two main views in the News app. The `<news-list>` element displays a list of articles for the selected category - for example, Top Stories. The `<news-article>` element displays article content. When the user is offline and the category data is not cached, both views display a network error via the `<news-network-warning>` element:

`news-list.html`
```html
<news-network-warning
    hidden$="[[!failure]]"
    offline="[[offline]]"
    on-try-reconnect="_tryReconnect"></news-network-warning>
```

If the user is offline but category data is cached and ServiceWorker is installed,
the cached content is displayed.

-   [`<app-route>` API reference](https://elements.polymer-project.org/elements/app-route)

## Routing and Data Binding in the News App

Routing and data bindings work together in the News app to retrieve and display the data relevant to the URL that the user selects.

Polymer's data system allows for data to flow one-way (downward-only, from host element to target element) or two-way (from host to target and target to host). For more information on data binding in Polymer, see [fkjsdkfjl].

The News app uses both one-way and two-way binding to transfer data between elements.

The app shell element, `<news-app>`, acts as the host, while `<news-data>`, `<news-list>`, and `<news-article>` act as targets. `<news-app>` contains data properties for:

* The name of the current category (`categoryName`)
* The ID of the current article (`articleId`)
* The list of articles in the current category (`category`)
* The raw HTML for the current article (`article`)

### Part 1: Routing

`categoryName` and `articleId` are set by the routing logic in the view elements (`<news-list>` and `<news-article>`). The active view element processes the user's selected URL in order to set `categoryName` or `articleId`. The properties are two-way bound between the view elements and the app shell, so they are "pushed" up to the app shell.

For example, here's how the URL `/list/top_stories` would be processed.

The top-level `<app-route>` element in `<news-app>` consumes the first part of the URL, and `page` is set to `"list"`:

`news-app.html`
```html
<app-location route="{{route}}"></app-location>
<app-route
    route="{{route}}"
    pattern="/:page"
    data="{{routeData}}"
    tail="{{subroute}}"></app-route>
```

The `<iron-pages>` element in `<news-app>` displays the `<news-list>` element (because `page` == "list"):

`news-app.html`
```html
<iron-pages role="main" selected="[[page]]" attr-for-selected="name">
```

The `<app-route>` element in `<news-list>` consumes the next part of the URL (`/top-stories`), and sets `categoryName` to `top-stories`:

`news-list.html`
```html
<app-route
    route="[[route]]"
    pattern="/:category"
    data="{{_routeData}}"></app-route>
```

Two-way data binding in the host element ensures that the data is passed back to `<news-app>`:

`news-app.html`
```html
<news-list
    ...
    category-name="{{categoryName}}"
    ...
    ...></news-list>
```

### Part 2: Data retrieval

`categoryName` and `articleId` are one-way bound from `<news-app>` to `<news-data>`:

`news-app.html`
```html
<news-data
    ...
    category-name="[[categoryName]]"
    article-id="[[articleId]]"
    ...
    ...></news-data>
```

These properties tell `<news-data>` which resources to retrieve. `<news-data>` then creates an `XMLHttpRequest` for the relevant resources, and stores the resulting category or article information in its `category` and `article` properties.

For example, if the user has visited `/list/top_stories`, `categoryName` is set to `top_stories`. `<news-data>` queries `/data/top_stories.json` and retrieves a list of the articles in the `top_stories` category. This list is stored in `category`.

The `category` and `article` properties are two-way bound from `<news-app>` to `<news-data>`, and therefore propagate back to `<news-app>`:

`news-app.html`
```html
<news-data
    ...
    category="{{category}}"
    ...
    article="{{article}}"
    ...
    ...></news-data>
```

### Part 3: Data display

`category` and `article` are one-way bound from `<news-app>` to the elements that display this content (`<news-list>` and `<news-article>`. When these properties are updated in `<news-app>`, triggered by their change of state in `<news-data>`, they flow down to `<news-list>` or `<news-article>`, depending on routing.

For example, if the user has visited `/list/top_stories`, `<news-data>` has populated `<news-app>`'s `category` object with a list of articles in the `top_stories` category. The data now propagates to `<news-list>` through one-way data binding:

`news-app.html`
```html
<news-list
    ...
    category="[[category]]"
    ...
    ...></news-list>
```

These relationships are shown here:

<div class="image-container layout horizontal">
  <div class="image-wrapper">
    <img src="data binding.jpg" alt="data binding relationships" width="700px">
  </div>
</div>

This sequence diagram shows the interactions between the elements of the News app when the user visits `/list/top_stories`:

<div class="image-container layout horizontal">
  <div class="image-wrapper">
    <img src="sequence diagram.jpg" alt="sequence diagram of data flow" width="700px">
  </div>
</div>

## SEO and Social Sharing

A small piece of server code in the News app checks the requesting user agent whenever the user navigates to a URL in your app. If the user agent is a browser, the application returns index.html and the app's routing logic is executed to display the requested content.

However, some user agents (such as certain indexers and social sharing sites) don't run JavaScript. If the user agent is a well-known member of this category, the News app returns a static HTML page so that the page can be parsed and indexed or displayed correctly in a social site.

`main.py`
```python
class ArticlePage(webapp2.RequestHandler):
  def get(self, **kwargs):
    bots = '|'.join([
        'Googlebot',
        'bingbot',
        'msnbot',
        'facebookexternalhit',
        'Facebot',
        'Twitterbot',
        'Google-Structured-Data-Testing-Tool'
      ])
    if re.search(bots, self.request.headers.get('User-Agent'), re.I):
      self.response.out.write(open('data/articles/' + kwargs['article_id'] + '.html').read())
    else:
      self.response.out.write(open('index.html').read())
```

## Resource URLs

Resource URLs, like `/bower_components/`, `/data/` and `/src/` are served as static files. These static URLs are defined in `app.yaml`. These URLs are used internally by the app to load components and data.

## Displaying Ads

The News app attempts to display ads, without blocking a fast first paint.

The <news-iframe> element uses [`IntersectionObserver`](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API), a web platform API. `IntersectionObserver` is used to determine whether `<news-iframe>` element is visible on screen before creating an iframe that will display an ad.

`news-iframe.html`
```javascript
attached: function() {
  if ('IntersectionObserver' in window) {
    this._io = new IntersectionObserver(this._createIframe.bind(this));
    this._io.observe(this);
  } else {
    // Intersection Observer is not available, so just create the iframe
    // after a bit of a delay.
    window.setTimeout(this._createIframe.bind(this), 100);
  }
},
```

If `IntersectionObserver` isn't available, the application briefly delays displaying the ad to let the rest of the page load first.

## AMP version of the News app

We built a [version of the News app](https://github.com/PolymerLabs/news/tree/amp) for integration with [AMP (Accelerated Mobile Pages)](https://www.ampproject.org/) documents. In the AMP version of News, the HTML content files (for example, `/data/articles/it-takes-teacher.html`) are replaced by AMP documents which can be loaded as standalone pages.

The AMP project supports adding an AMP document to a shadow root on your page. Here’s an example: <code>[https://github.com/ampproject/amphtml/blob/master/examples/pwa/pwa.js](https://github.com/ampproject/amphtml/blob/master/examples/pwa/pwa.js)</code>

To support this in the AMP version of the News app, we created [the `<amp-viewer>` element](https://github.com/PolymerLabs/amp-viewer/). The `<amp-viewer>` element works similarly to an iframe. To load an AMP document inside the viewer, set its `src` attribute to the URL of the AMP document. Styles will be applied to the content, scoped by its shadow root.

The element is written as a native custom element, and can be used independently of the News app.

 Here's how the `<amp-viewer>` element is used in News:

`news-article.html`
```html
<div class="container" fade-in$="[[!loading]]" hidden$="[[failure]]">
  <article class="flex">
    <amp-viewer id="ampViewer" src="[[_getArticleSrc(article.id)]]"></amp-viewer>
  </article>
```

Note: the AMP version of the app does not support swiping between articles - doing so would require loading the title, image and author of the previous or next article. In the AMP version of News, that data is only loaded when the article content is loaded. It is not loaded with the list of articles.

## More Resources

If you want to look at the News app in more detail, you can find the full source
on GitHub:

[https://github.com/Polymer/news](https://github.com/Polymer/news)
