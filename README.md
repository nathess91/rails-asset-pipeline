# <img src="https://cloud.githubusercontent.com/assets/7833470/10899314/63829980-8188-11e5-8cdd-4ded5bcb6e36.png" height="60"> Rails Asset Pipeline

| Objectives |
| :--- |
| Describe how the Rails Asset Pipeline works and why we use it. |
| Require custom and third-party assets in Rails. |
| Precompile assets in development to debug asset issues in production. |

## Asset Pipeline Overview

**The goal of the asset pipeline is to:**

* Compress assets (CSS and JavaScript files) so they are as small as possible, since smaller files load faster.
* Reduce the number of files transferred to the client, by:
  * Combining files before sending them to the browser.
    * All CSS files are combined into a single `application.css` file.
    * All JavaScript files are combined into a single `application.js` file.
  * Caching files in the browser.  
* Facilitate use of languages like SASS and CoffeeScript, which compile into CSS or JavaScript.

## How the Rails Asset Pipeline Works

#### Remember this?

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Load all the things!</title>
</head>
<body>
  <script type="text/javascript" src="vendor/jquery-1.11.3.min.js"></script>
  <script type="text/javascript" src="vendor/handlebars-v4.0.5.js"></script>
  <script type="text/javascript" src="application.js"></script>
</body>
</html>
```

That's three different requests to the server, for three different files. And that's just for JavaScript! Ouch! There's got to be a better way!

How Rails requires files (using the asset pipeline):

```html
<!DOCTYPE html>
<html>
<head>
  <!-- sign the page for security -->
  <%= csrf_meta_tags %>
  <!-- application stylesheet -->
  <%= stylesheet_link_tag :application, media: :all %>
  <title>Rails Asset Pipeline!</title>
</head>
<body>
  <!-- application javascript file -->
  <%= javascript_include_tag :application %>
</body>
</html>
```

That's just two files! (Note that by default, Rails puts `<%= javascript_include_tag :application %>` in the HTML `<head></head>` tag, but you can move it to the bottom of the `<body></body>` to make sure it doesn't block the loading of your HTML).

### The Manifest

With the Asset Pipeline, instead of manually adding `<script>` or CSS `<link>` tags, you're going to take advantage of `app/assets/javascripts/application.js`. Inside this file, there's a weird looking series of comments called a "manifest":

``` javascript
//= require jquery
//= require jquery_ujs
//= require turbolinks
//= require_tree .
```

Actually, **these aren't comments!** This manifest file lists a series of instructions saying *which files* need to be loaded in your HTML, and *in what order*.  With these instructions, sprockets loads the files specified, processes them if necessary, concatenates them into one single file, and then compresses them (if `Rails.application.config.assets.compress` is `true`).

It will look for the name of the file (e.g. `jquery`) in the following directories:

1. `app/assets/` - application-specific code
2. `lib/assets/` - custom libraries not specific to this app
3. `vendor/assets/` - third-party libraries

### Precompiling Assets

#### Rails Environments

You may have noticed that when you create a Rails application it has three databases, `development`, `test`, and `production`. You may have also noticed the Gemfile allows `development`, `test`, and `production` as "groups" your gems can belong to.  Rails sets up three different environments for you with options to configure them differently. So far we've been working on our applications in *development*.

**The asset pipeline is designed for *production* applications.** That's when we care about *speed*!

#### How to Precompile

To turn your *many* JavaScript and CSS files into *one* JavaScript file and *one* CSS file in development, you can "precompile" your assets.  We will almost never need to precompile assets during development, but try it today to see what it does.

To set up your app to allow precompiling assets in development, the following three lines need to be inside `config/environments/development.rb`:

```ruby
config.assets.debug = false
config.assets.compile = false
config.assets.digest = true
```

Precompiled assets live in `public/assets/`. In a new Rails app, that's just an empty folder!

You can run the following command to precompile your assets in development:

```zsh
➜  rake assets:precompile
```

Now look inside `public/assets/`, and you'll see *compressed* and *fingerprinted* versions of your assets, compiled into a single file. Restart your server to start using your precompiled assets. Note that when you're precompiling in development, you must `rake assets:precompile` and restart your server anytime you make a change in your JavaScript or CSS files.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <link href="/assets/application-4dd5b109ee3439da54f5bdfd78a80473.css" rel="stylesheet" />
  <title>Precompiled Assets!</title>
</head>
<body>
  <script src="/assets/application-908e25f4bf641868d8683022a5b62f54.js"></script>
</body>
</html>
```

To destroy your precompiled assets in development, simply run:

```zsh
➜  rake assets:clobber
```

Note you will typically only precompile your assets in development for debugging purposes. You'll set up your app to automatically precompile assets when deploying to production (we'll look at this when push to Heroku with Rails).

### Caching

A web application can be configured to "cache" common JavaScript and CSS files in the client's browser.

Cached files don't need to be requested *on every page load*. They're already there, ready to go, meaning **less wait time** and **faster page loads**.

But how does the browser know when to update a cached file? What if you changed the file and the browser is using an older version?

We need a way to "bust" the cache!

#### Cache-Busting with Fingerprints

By default, Rails enables caching. In Rails, assets are given a "fingerprint" that changes every time the file is updated. The technique the asset pipeline uses is to insert a hash of the content into the name, usually at the end.

For example, the `application.js` file with a fingerprint looks like this: `application-908e25f4bf641868d8683022a5b62f54.js`.

**Cache-busting works like this:**

* If the fingerprint is the same, the browser simply uses its cached copy.
* If the fingerprint has changed, the browser requests the new version of the file (and then caches it!).

## Compression / Minification / Uglification

What's the difference between <a href="https://code.jquery.com/jquery-1.11.3.js">jquery-1.11.3.js</a> and <a href="https://code.jquery.com/jquery-1.11.3.min.js">jquery-1.11.3.min.js</a>?

Let's do a quick comparison on the command line:

```zsh
# uncompressed jQuery file
➜  curl https://code.jquery.com/jquery-1.11.3.js | wc
#   lines   words   bytes
#   10351   43235   284394

# compressed (minified) jQuery file
➜  curl https://code.jquery.com/jquery-1.11.3.min.js | wc
#   lines   words   bytes
#   5       1413    95957
```

## What about CDNs?

A CDN is a "content delivery network" and a handy way to deliver common "vendor" or "third-party" libraries to your application. It's common to use a CDN for jQuery, Bootstrap, Handlebars, etc.

But is it fast?

If a common file, like jQuery is delivered via CDN it is likely:

  1. Cached in your browser.
  2. Cached by your ISP (Internet Service Provider).
  3. Dispatched from a nearby server.

But is that faster than just sending one JavaScript file from your own server?

It could be! It's more likely for very widespread libraries that your user's browser might have cached already. You'll have to make a decision about whether you want to host third-party libraries (like jQuery) on your server, or if you want to use a public CDN.

## Challenges

* <a href="https://github.com/sf-wdi-30/asset_pipeline_poem">Asset Pipeline Poem</a>

## Resources

* [Asset Pipeline Additional Reading](./additional-reading.md)
* <a href="http://guides.rubyonrails.org/asset_pipeline.html">Rails Guides: Asset Pipeline</a>
* <a href="https://github.com/rails/sprockets#sprockets-directives">Sprockets Directives</a>
* List of [view helpers for asset tags](http://api.rubyonrails.org/classes/ActionView/Helpers/AssetTagHelper.html).
