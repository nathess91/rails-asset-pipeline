## Precompiling Assets

> Caution: we won't normally pre-compile assets in the development environment.  You might encounter unexpected results in some cases.

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

```bash
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

```bash
➜  rake assets:clobber
```

Note you will typically only precompile your assets in development for debugging purposes. You'll set up your app to automatically precompile assets when deploying to production (we'll look at this when push to Heroku with Rails).

