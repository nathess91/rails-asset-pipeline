# <img src="https://cloud.githubusercontent.com/assets/7833470/10899314/63829980-8188-11e5-8cdd-4ded5bcb6e36.png" height="60"> Asset Pipeline - Additional Reading

##### Javascript

A new Rails 4 application includes a default `app/assets/javascripts/application.js` file containing the following lines:

```javascript
// app/assets/javascripts/application.js

// ...
//= require jquery
//= require jquery_ujs
//= require turbolinks
//= require_tree .
```

In JavaScript files, Sprockets directives begin with `//=`. In the above case, the file is using the `require` and the `require_tree` directives. The require directive is used to tell Sprockets the files you wish to require. Here, you are requiring the files `jquery.js`, `jquery_ujs.js`, and `turbolinks.js` that are available somewhere in the search path for Sprockets. You need not supply the extensions explicitly. Sprockets assumes you are requiring a `.js` file when done from within a `.js` file.

**Note about turbolinks:** Turbolinks is a gem that ships with Rails and that eliminates page refreshes when navigating around your app in the browser. This sounds pretty cool, but the downside is that it only loads your assets once, on the first page-load, then it never loads them again, so any jQuery events you have set to run on page-load won't work on subsequent pages. We suggest removing turbolinks for the time being.

**Removing turbolinks requires three steps:**

1. Remove `'data-turbolinks-track' => true` from `<%= stylesheet_link_tag 'application' %>` and `<%= javascript_include_tag 'application' %>` in `app/views/layouts.application.html.erb`.
2. Remove `//= require turbolinks` from `app/assets/javascripts/application.js`.
3. Comment-out (or delete) `gem 'turbolinks'` from your `Gemfile`.

The `require_tree` directive tells Sprockets to recursively include all JavaScript files in the specified directory into the output. These paths must be specified relative to the manifest file. You can also use the `require_directory` directive which includes all JavaScript files only in the directory specified, without recursion.

Requiring these JS files:

```
app/assets/javascripts/home.js
lib/assets/javascripts/moovinator.js
vendor/assets/javascripts/slider.js
vendor/assets/somepackage/phonebox.js
```

Will be as follows:

```javascript
// app/assets/javascripts/application.js

// ...
//= require slider
//= require phonebox
//= require moovinator
//= require home
```

##### CSS

The remarks made above about ordering in your app's JavaScript manifest apply to your CSS manifest comments (inside `app/assets/stylesheets/application.css`). In particular, you can specify individual files, and they are compiled in the order specified.

Requiring these CSS files:

```
app/assets/stylesheets/buttons.css
lib/assets/stylesheets/nav-bar.css
vendor/assets/stylesheets/bootstrap.css
```

Will be as follows:

```css
/* app/assets/javascripts/application.css */

/* ...
*= require bootstrap
*= require nav-bar
*= require buttons
*/
```

##### Images

In views, you can access images in the `public/assets/images` directory like this: `<%= image_tag "rails.png" %>`.

Images can also be organized into subdirectories, and then can be accessed by specifying the directory's name in the tag: `<%= image_tag "icons/rails.png" %>`.

##### ERB and Asset Path Helpers

The asset pipeline automatically evaluates ERB. This means if you add a `.erb` extension to a CSS asset (for example, `application.css.erb`), then helpers like [`asset_path`](http://api.rubyonrails.org/classes/ActionView/Helpers/AssetUrlHelper.html) are available in your CSS rules:

```css
.header {
  background-image: url(<%= asset_path 'image.png' %>)
}
```

This writes the path to the particular asset being referenced. In this example, it would make sense to have an image in one of the asset load paths, such as `app/assets/images/image.png`, which would be referenced here. If this image is already available in `public/assets` as a fingerprinted file, then that path is referenced.

Similarly, if you add a `.erb` extension to a JavaScript asset (like `application.js.erb`), you can then use the `asset_path` helper in your JavaScript code:

```js
$('#logo').attr({ src: "<%= asset_path('logo.png') %>" });
```
