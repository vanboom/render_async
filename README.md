[![Build Status](https://semaphoreci.com/api/v1/renderedtext/render_async/branches/master/shields_badge.svg)](https://semaphoreci.com/renderedtext/render_async)
[![All Contributors](https://img.shields.io/badge/all_contributors-21-orange.svg?style=flat-square)](#contributors)
[![Gem Version](https://badge.fury.io/rb/render_async.svg)](https://badge.fury.io/rb/render_async)
[![Donate](https://img.shields.io/badge/$-support-green.svg)](https://www.paypal.me/nikolalsvk/10)
[![Code Climate](https://codeclimate.com/github/renderedtext/render_async/badges/gpa.svg)](https://codeclimate.com/github/renderedtext/render_async)
[![Test Coverage](https://codeclimate.com/github/renderedtext/render_async/badges/coverage.svg)](https://codeclimate.com/github/renderedtext/render_async/coverage)
[![Help Contribute to Open Source](https://www.codetriage.com/renderedtext/render_async/badges/users.svg)](https://www.codetriage.com/renderedtext/render_async)

![render_async](http://s2blog.wpengine.com/wp-content/uploads/assets/images/2017-06-08/speed-up-rendering-rails-pages-with-render-async.png)

# render_async

Speed up rendering Rails pages with this gem.

`render_async` renders partials to your views **asynchronously**. This is done
through adding JavaScript code that does AJAX request to your controller which
then renders your partial into a Rails view.

Workflow:

1. user visits a Rails page
2. AJAX request on the controller action
3. controller renders a partial
4. partials renders in the place where you put `render_async` helper

JavaScript is injected into `<%= content_for :render_async %>` so you choose
where to put it.

## Installation
Add this line to your application's Gemfile:

```ruby
gem 'render_async'
```

And then execute:

    $ bundle install

## Usage

1. Include `render_async` view helper somewhere in your views (e.g. `app/views/comments/show.html.erb`):
    ```erb
    <%= render_async comment_stats_path %>
    ```

2. Then create a route that will `config/routes.rb`:
    ```ruby
    get :comment_stats, controller: :comments
    ```

3. Fill in the logic in your controller (e.g. `app/controllers/comments_controller.rb`):
    ```ruby
    def comment_stats
      @stats = Comment.get_stats

      render partial: "comment_stats"
    end
    ```

4. Create a partial that will render (e.g. `app/views/comments/_comment_stats.html.erb`):
    ```erb
    <div class="col-md-6">
      <%= @stats %>
    </div>
    ```

5. Add `content_for` in your base view file in the body part (e.g. `app/views/layouts/application.html.erb`):
    ```erb
    <%= content_for :render_async %>
    ```

## Advanced usage

Advanced usage includes information on different options, such as:

  - [Passing in a container ID](#passing-in-a-container-id)
  - [Passing in a container class name](#passing-in-a-container-class-name)
  - [Passing in HTML options](#passing-in-html-options)
  - [Passing in an HTML element name](#passing-in-an-html-element-name)
  - [Passing in a placeholder](#passing-in-a-placeholder)
  - [Passing in an event name](#passing-in-an-event-name)
  - [Using default events](#using-default-events)
  - [Retry on failure](#retry-on-failure)
  - [Toggle event](#toggle-event)
  - [Polling](#polling)
  - [Handling errors](#handling-errors)
  - [Caching](#caching)
  - [Doing non-GET requests](#doing-non-get-requests)
  - [Using with Turbolinks](#using-with-turbolinks)
  - [Using with respond_to and JS format](#using-with-respond_to-and-js-format)
  - [Nested Async Renders](#nested-async-renders)
  - [Customizing the content_for name](#customizing-the-content_for-name)
  - [Configuration](#configuration)

### Passing in a container ID

`render_async` renders an element that gets replaced with the content
of your request response. In order to have more control over the element
that renders first (before the request), you can set the ID of that element.

To set ID of the container element, you can do the following:
```erb
<%= render_async users_path, container_id: 'users-container' %>
```

Rendered code in the view:
```html
<div id="users-container">
</div>

...
```

### Passing in a container class name

`render_async` renders an element that gets replaced with the content of your
request response. If you want to style that element, you can set the class name
on it.

```erb
<%= render_async users_path, container_class: 'users-container-class' %>
```

Rendered code in the view:
```html
<div id="render_async_18b8a6cd161499117471" class="users-container-class">
</div>

...
```

### Passing in HTML options

`render_async` can accept `html_options` as a hash.
`html_options` is an optional hash that gets passed to a Rails'
`javascript_tag`, to drop HTML tags into the `script` element.

Example of utilizing `html_options` with a [nonce](https://edgeguides.rubyonrails.org/security.html#content-security-policy):
```erb
<%= render_async users_path, html_options: { nonce: true } %>
```

Rendered code in the view:
```html
<script nonce="2x012CYGxKgM8qAApxRHxA==">
//<![CDATA[
  ...
//]]>
</script>

...

<div id="render_async_18b8a6cd161499117471" class="">
</div>
```

### Passing in an HTML element name

`render_async` can take in an HTML element name, allowing you to control
what type of container gets rendered. This can be useful when you're using
[`render_async` inside a table](https://github.com/renderedtext/render_async/issues/12)
and you need it to render a `tr` element before your request gets loaded, so
your content doesn't get pushed out of the table.

Example of using HTML element name:
```erb
<%= render_async users_path, html_element_name: 'tr' %>
```

Rendered code in the view:
```html
<tr id="render_async_04229e7abe1507987376">
</tr>
...
```

### Passing in a placeholder

`render_async` can be called with a block that will act as a placeholder before
your AJAX call finishes.

Example of passing in a block:

```erb
<%= render_async users_path do %>
  <h1>Users are loading...</h1>
<% end %>
```

Rendered code in the view:
```html
<div id="render_async_14d7ac165d1505993721">
  <h1>Users are loading...</h1>
</div>

<script>
//<![CDATA[
  ...
//]]>
</script>
```

After AJAX is finished, placeholder will be replaced with the request's
response.

### Passing in an event name

`render_async` can receive `:event_name` option which will emit JavaScript
event after it's done with fetching and rendering request content to HTML.

This can be useful to have if you want to add some JavaScript functionality
after your partial is loaded through `render_async`.

You can also access the associated container (DOM node) in the event object
that gets emitted.

Example of passing it to `render_async`:
```erb
<%= render_async users_path, event_name: "users-loaded" %>
```

Rendered code in view:
```html
<div id="render_async_04229e7abe1507987376">
</div>

<script>
//<![CDATA[
  ...
    document.dispatchEvent(new Event("users-loaded"));
  ...
//]]>
</script>
```

Then, in your JavaScript code, you could do something like this:
```javascript
document.addEventListener("users-loaded", function(event) {
  console.log("Users have loaded!", event.container); // Access the container which loaded the users
});
```

NOTE: Dispatching events is also supported for older browsers that don't
support Event constructor.

### Using default events

`render_async` will fire the event `render_async_load` when an async partial
has loaded and rendered on page.

In case there is an error, the event `render_async_error` will fire instead.

This event will fire for all `render_async` partials on the page. For every
event, the associated container (DOM node) will be passed along.

This can be useful to apply JavaScript to content loaded after the page is
ready.

Example of using events:

```js
// Vanilla javascript
document.addEventListener('render_async_load', function(event) {
  console.log('Async partial loaded in this container:', event.container);
});
document.addEventListener('render_async_error', function(event) {
  console.log('Async partial could not load in this container:', event.container);
});

// with jQuery
$(document).on('render_async_load', function(event) {
  console.log('Async partial loaded in this container:', event.container);
});
$(document).on('render_async_error', function(event) {
  console.log('Async partial could not load in this container:', event.container);
});
```

### Retry on failure

`render_async` can retry your requests if they fail for some reason.

If you want `render_async` to retry a request for number of times, you can do
this:
```erb
<%= render_async users_path, retry_count: 5, error_message: "Couldn't fetch it" %>
```

Now render_async will retry `users_path` for 5 times. If it succeeds in
between, it will stop with dispatching requests. If it fails after 5 times,
it will show an [error message](#handling-errors) which you need to specify.

This can show useful when you know your requests often fail, and you don't want
to refresh the whole page just to retry them.

### Toggle event

You can trigger `render_async` loading by clicking or doing another event to a
certain HTML element. You can do this by passing in a selector and an event
name which will trigger `render_async`. If you don't specify an event name, the
default event that will trigger `render_async` will be 'click' event. You can
do this by doing the following:

```erb
<a href='#' id='comments-button'>Load comments</a>
<%= render_async comments_path, toggle: { selector: '#comments-button', event: :click } %>
```

This will trigger `render_async` to load the `comments_path` when you click the `#comments-button` element.
If you want to remove event once it's triggered, you can pass `once: true` in the toggle options.
The `once` option is false by default.

You can also pass in a placeholder before the `render_async` is triggered. That
way, the element that started `render_async` logic will be removed after the
request has been completed. You can achieve this behaviour with something like this:

```erb
<%= render_async comments_path, toggle: { selector: '#comments-button', event: :click } do %>
  <a href='#' id='comments-button'>Load comments</a>
<% end %>
```
Also, you can mix interval and toggle features. This way, you can turn polling
on, and off by clicking the "Load comments" button. In order to do this, you need to
pass `toggle` and `interval` arguments to `render_async` call like this:

```erb
<a href='#' id='comments-button'>Load comments</a>
<%= render_async comments_path, toggle: { selector: '#comments-button', event: :click }, interval: 2000 %>
```

### Polling

You can call `render_async` with interval argument. This will make render_async
call specified path at specified interval.

By doing this:
```erb
<%= render_async comments_path, interval: 5000 %>
```
You are telling `render_async` to fetch comments_path every 5 seconds.

This can be handy if you want to enable polling for a specific URL.

Polling may be stopped by sending the `async-stop` event to the container element. 
Polling may be started by sending the `async-start` event to the container element.

NOTE: By passing interval to `render_async`, initial container element
will remain in HTML tree, it will not be replaced with request response.
You can handle how that container element is rendered and its style by
[passing in an HTML element name](#passing-in-an-html-element-name) and
[HTML element class](#passing-in-a-container-class-name).

### Handling errors

`render_async` let's you handle errors by allowing you to pass in `error_message`
and `error_event_name`.

- `error_message`

  passing an `error_message` will render a message if the AJAX requests fails for
  some reason
  ```erb
  <%= render_async users_path,
                   error_message: '<p>Sorry, users loading went wrong :(</p>' %>
  ```

- `error_event_name`

  calling `render_async` with `error_event_name` will dispatch event in the case
  of an error with your AJAX call.
  ```erb
  <%= render_asyc users_path, error_event_name: 'users-error-event' %>
  ```

  You can then catch the event in your code with:
  ```js
  document.addEventListener('users-error-event', function() {
    // I'm on it
  })
  ```

### Caching

`render_async` can utilize view fragment caching to avoid extra AJAX calls.

In your views (e.g. `app/views/comments/show.html.erb`):
```erb
# note 'render_async_cache' instead of standard 'render_async'
<%= render_async_cache comment_stats_path %>
```

Then, in the partial (e.g. `app/views/comments/_comment_stats.html.erb`):
```erb
<% cache render_async_cache_key(request.path), skip_digest: true do %>
  <div class="col-md-6">
    <%= @stats %>
  </div>
<% end %>
```

* The first time the page renders, it will make the AJAX call.
* Any other times (until the cache expires), it will render from cache
  instantly, without making the AJAX call.
* You can expire cache simply by passing `:expires_in` in your view where
  you cache the partial

### Doing non-GET requests

By default, `render_async` creates AJAX GET requests for the path you provide.
If you want to change this behaviour, you can pass in a `method` argument to
`render_async` view helper.

```erb
<%= render_async users_path, method: 'POST' %>
```

You can also set `body` and `headers` of the request if you need them.

```erb
<%= render_async users_path,
                 method: 'POST',
                 data: { fresh: 'AF' },
                 headers: { 'Content-Type': 'text' } %>
```

### Using with Turbolinks

On Turbolinks applications, you may experience caching issues when navigating
away from, and then back to, a page with a `render_async` call on it. This will
likely show up as an empty div.

If you're using Turbolinks 5 or higher, you can resolve this by setting Turbolinks
configuration of `render_async` to true:

```rb
RenderAsync.configure do |config|
  config.turbolinks = true # Enable this option if you are using Turbolinks 5+
end
```

This way, you're not breaking Turbolinks flow of loading or reloading a page.
It makes it more efficient that the next option that is suggested below.

Another option:
If you want, you can tell Turbolinks to reload your `render_async` call as follows:

```erb
<%= render_async events_path, 'data-turbolinks-track': 'reload' %>
```

This will reload the whole page with Turbolinks.

Make sure to put `<%= content_for :render_async %>` in your base view file in
the `<head>` and not the `<body>`.

### Using with respond_to and JS format

If you need to restrict the action to only respond to AJAX requests, you'll
likely wrap it inside `respond_to`/`format.js` blocks like this:

```ruby
def comment_stats
  respond_to do |format|
    format.js do
      @stats = Comment.get_stats

      render partial: "comment_stats"
    end
  end
end
```

When you do this, Rails will sometime set the response's `Content-Type` header
to `text/javascript`. This causes the partial not to be rendered in the HTML.
This usually happens when there's browser caching.

You can get around it by specifying the content type to `text/html` in the
render call:

```ruby
render partial: "comment_stats", content_type: 'text/html'
```

### Nested Async Renders

It is possible to nest async templates within other async templates. When doing
so, another `content_for` is required to ensure the JavaScript needed to load
nested templates is included.

For example:
```erb
<%# app/views/comments/show.html.erb %>

<%= render_async comment_stats_path %>
```

```erb
<%# app/views/comments/_comment_stats.html.erb %>

<div class="col-md-6">
  <%= @stats %>
</div>

<div class="col-md-6">
  <%= render_async comment_advanced_stats_path %>
</div>

<%= content_for :render_async %>
```

### Customizing the content_for name

The `content_for` name may be customized by passing the `content_for_name`
option to `render_async`. This option is especially useful when doing [nested async
renders](#nested-async-renders) to better control the location of the injected JavaScript.

For example:
```erb
<%= render_async comment_stats_path, content_for_name: :render_async_comment_stats %>

<%= content_for :render_async_comment_stats %>
```

### Configuration

`render_async` renders Vanilla JS (regular JavaScript, non-jQuery code)
**by default** in order to fetch the request from the server.

If you want `render_async` to use jQuery code, you need to configure it to do
so.

You can configure it by doing the following anywhere before you call
`render_async`:
```rb
RenderAsync.configure do |config|
  config.jquery = true # This will render jQuery code, and skip Vanilla JS code
  config.turbolinks = false # Enable this option if you are using Turbolinks 5+
end
```

Also, you can do it like this:
```rb
# This will render jQuery code, and skip Vanilla JS code
RenderAsync.configuration.jquery = true
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run
`rake spec` to run the tests. You can also run `bin/console` for an interactive
prompt that will allow you to experiment.

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/renderedtext/render_async.

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

## Contributors

Thanks goes to these wonderful people ([emoji key](https://github.com/kentcdodds/all-contributors#emoji-key)):

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->
<!-- prettier-ignore -->
| [<img src="https://avatars2.githubusercontent.com/u/3028124?v=4" width="100px;"/><br /><sub><b>Nikola Đuza</b></sub>](https://nikolalsvk.github.io)<br />[💬](#question-nikolalsvk "Answering Questions") [💻](https://github.com/renderedtext/render_async/commits?author=nikolalsvk "Code") [📖](https://github.com/renderedtext/render_async/commits?author=nikolalsvk "Documentation") [👀](#review-nikolalsvk "Reviewed Pull Requests") | [<img src="https://avatars0.githubusercontent.com/u/3866868?v=4" width="100px;"/><br /><sub><b>Colin</b></sub>](http://www.colinxfleming.com)<br />[💻](https://github.com/renderedtext/render_async/commits?author=colinxfleming "Code") [📖](https://github.com/renderedtext/render_async/commits?author=colinxfleming "Documentation") [💡](#example-colinxfleming "Examples") | [<img src="https://avatars2.githubusercontent.com/u/334273?v=4" width="100px;"/><br /><sub><b>Kasper Grubbe</b></sub>](http://kaspergrubbe.com)<br />[💻](https://github.com/renderedtext/render_async/commits?author=kaspergrubbe "Code") | [<img src="https://avatars2.githubusercontent.com/u/163584?v=4" width="100px;"/><br /><sub><b>Sai Ram Kunala</b></sub>](https://sairam.xyz/)<br />[📖](https://github.com/renderedtext/render_async/commits?author=sairam "Documentation") | [<img src="https://avatars2.githubusercontent.com/u/3065882?v=4" width="100px;"/><br /><sub><b>Josh Arnold</b></sub>](https://github.com/nightsurge)<br />[💻](https://github.com/renderedtext/render_async/commits?author=nightsurge "Code") [📖](https://github.com/renderedtext/render_async/commits?author=nightsurge "Documentation") | [<img src="https://avatars3.githubusercontent.com/u/107798?v=4" width="100px;"/><br /><sub><b>Elad Shahar</b></sub>](https://eladshahar.com)<br />[💻](https://github.com/renderedtext/render_async/commits?author=SaladFork "Code") [💡](#example-SaladFork "Examples") | [<img src="https://avatars3.githubusercontent.com/u/232392?v=4" width="100px;"/><br /><sub><b>Sasha</b></sub>](http://www.revzin.co.il)<br />[💻](https://github.com/renderedtext/render_async/commits?author=sasharevzin "Code") [📖](https://github.com/renderedtext/render_async/commits?author=sasharevzin "Documentation") |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| [<img src="https://avatars3.githubusercontent.com/u/50223?v=4" width="100px;"/><br /><sub><b>Ernest Surudo</b></sub>](http://elsurudo.com)<br />[💻](https://github.com/renderedtext/render_async/commits?author=elsurudo "Code") | [<img src="https://avatars1.githubusercontent.com/u/334809?v=4" width="100px;"/><br /><sub><b>Kurtis Rainbolt-Greene</b></sub>](https://kurtis.rainbolt-greene.online)<br />[💻](https://github.com/renderedtext/render_async/commits?author=krainboltgreene "Code") | [<img src="https://avatars2.githubusercontent.com/u/59744?v=4" width="100px;"/><br /><sub><b>Richard Schneeman</b></sub>](https://www.schneems.com)<br />[📖](https://github.com/renderedtext/render_async/commits?author=schneems "Documentation") | [<img src="https://avatars1.githubusercontent.com/u/75705?v=4" width="100px;"/><br /><sub><b>Richard Venneman</b></sub>](https://www.cityspotters.com)<br />[📖](https://github.com/renderedtext/render_async/commits?author=richardvenneman "Documentation") | [<img src="https://avatars3.githubusercontent.com/u/381395?v=4" width="100px;"/><br /><sub><b>Filipe W. Lima</b></sub>](https://github.com/filipewl)<br />[📖](https://github.com/renderedtext/render_async/commits?author=filipewl "Documentation") | [<img src="https://avatars0.githubusercontent.com/u/3135638?v=4" width="100px;"/><br /><sub><b>Jesús Eduardo Clemens Chong</b></sub>](https://github.com/eclemens)<br />[💻](https://github.com/renderedtext/render_async/commits?author=eclemens "Code") | [<img src="https://avatars3.githubusercontent.com/u/1935686?v=4" width="100px;"/><br /><sub><b>René Klačan</b></sub>](https://github.com/reneklacan)<br />[💻](https://github.com/renderedtext/render_async/commits?author=reneklacan "Code") |
| [<img src="https://avatars1.githubusercontent.com/u/1313442?v=4" width="100px;"/><br /><sub><b>Gil Gomes</b></sub>](http://gilgomes.com.br)<br />[📖](https://github.com/renderedtext/render_async/commits?author=gil27 "Documentation") | [<img src="https://avatars0.githubusercontent.com/u/6081795?v=4" width="100px;"/><br /><sub><b>Khoa Nguyen</b></sub>](https://github.com/ThanhKhoaIT)<br />[💻](https://github.com/renderedtext/render_async/commits?author=ThanhKhoaIT "Code") [📖](https://github.com/renderedtext/render_async/commits?author=ThanhKhoaIT "Documentation") | [<img src="https://avatars2.githubusercontent.com/u/8645918?v=4" width="100px;"/><br /><sub><b>Preet Sethi</b></sub>](https://www.linkedin.com/in/preetsethila/)<br />[💻](https://github.com/renderedtext/render_async/commits?author=preetsethi "Code") | [<img src="https://avatars3.githubusercontent.com/u/11586335?v=4" width="100px;"/><br /><sub><b>fangxing</b></sub>](https://github.com/fffx)<br />[💻](https://github.com/renderedtext/render_async/commits?author=fffx "Code") | [<img src="https://avatars3.githubusercontent.com/u/1191418?v=4" width="100px;"/><br /><sub><b>Emmanuel Pire</b></sub>](http://blog.lipsumarium.com)<br />[💻](https://github.com/renderedtext/render_async/commits?author=lipsumar "Code") [📖](https://github.com/renderedtext/render_async/commits?author=lipsumar "Documentation") | [<img src="https://avatars1.githubusercontent.com/u/615509?v=4" width="100px;"/><br /><sub><b>Maxim Geerinck</b></sub>](https://github.com/maximgeerinck)<br />[💻](https://github.com/renderedtext/render_async/commits?author=maximgeerinck "Code") | [<img src="https://avatars1.githubusercontent.com/u/251706?v=4" width="100px;"/><br /><sub><b>Don</b></sub>](https://github.com/vanboom)<br />[💻](https://github.com/renderedtext/render_async/commits?author=vanboom "Code") |
<!-- ALL-CONTRIBUTORS-LIST:END -->

This project follows the [all-contributors](https://github.com/kentcdodds/all-contributors) specification. Contributions of any kind welcome!
