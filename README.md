# ember-devise-simple-auth

A plugin that allows an Ember app to integrate with a (mostly stock) Devise setup.

## What it does

Provides the necessary Ember plumbing to integrate with an out-of-the-box Devise setup. This means it uses cookies for session storage, but does not perform any redirects.

### Provides

`app/routes/session.js` - The route that handles sign in. You will need to create a template named `sessions`.

`app/models/authenticator.js` - A plain object that provides the current signed-in state, along with methods to sign in/out and lookup the current user.

`config/initializers/authenticator.js` - Injects the authenticator into your routes & controllers so you have access to the signed-in state anywhere you are!

`config/initializers/csrf.js` - jQuery ajax response handler that reads new CSRF tokens handed in from devise (see [companion gem][companion_gem]).

`config/initializers/session-route.js` - Adds a route named "session" to your app's router. The path defaults to `/sign-in` but [is configurable][configurable].

`config/plugin.js` - The main file that loads the plugin. Extends the base `Ember.Route` with some auth-related action handlers.  Extends `Ember.Controller` with properties for signed-in state.

## Installing

Currently this plugin works best with apps built with `ember-appkit-rails`. We will be adding support for any `ember-rails` apps soon, followed by full `ember-app-kit`/`ember-cli` support in the future.

Install with bower:

`bower install ember-devise-simple-auth`

### Gem for Rails Support

To support some small customizations in `Devise::SessionsController` you need to install our gem and update your routes. Add the following to your `Gemfile`:

```
gem "ember_devise_simple_auth"
```

Then run:

```sh
bundle install
rails g ember_devise_simple_auth:install
```

### ember-rails apps

In `config/application.js` add the following:

```javascript
//... vendor requires
//= require ember-devise-simple-auth/globals
//... the rest of your requires
```

### ember-appkit-rails apps

In `config/application.js` add the following:

```javascript
//... vendor requires
//= require router
//= require ember-devise-simple-auth/appkit
//... the rest of your requires
//= require_self

require('ember-devise-simple-auth');
```

**NOTE:** Make sure you require the router before ember-devise-simple-auth

## Configuring

There are a few options you can specify now, and more to come in the future. If there's something you need to configure but can't figure out how, please [open an issue](issues/new) describing what you needa nd we'll see if we can provide it.

Configuration happens in `config/application.js` as part of the call to `create()`:

```javascript
window.App = require('app').default.create({
  deviseEmberAuth: {
    signInPath: "/sign-in", // the URL users will see in the browser for the sign in page
    deviseSignInPath: "/users/sign_in", // the URL to POST to for creating a session
    deviseSignOutPath: "/users/sign_out", // the URL to DELETE to for signing out
    currentSessionPath: "/sessions/current" // the URL for getting the current signed-in state; this is currently added by the gem
  }
});
```

## Usage

**NOTE:** This assumes you have configured Devise and followed the instructions above in [Installation][installation].

The only thing you need to do is provide a template named `session` (for eak-rails that would be `app/templates/session.hbs`). Then assign `{{action signIn}}` to a button or form and you should be good to go.

### Common Tasks

There are a few actions that you can choose to handle in your application's routes if you need to override the default behavior.

#### Redirect After Sign In

To transition to another route on successful sign in, you can handle the `validSignIn` action in your `application` route. For example:

```javascript
export default Ember.Route.extend({
  actions: {
    validSignIn: function() {
      this.transitionTo("dashboard");
    }
  }
});
```

#### Handle Failed Sign In

If a user enters invalid credentials, you can handle the `invalidSignIn` action. For example:

```javascript
export default Ember.Route.extend({
  actions: {
    invalidSignIn: function() {
      this.controllerFor("application").set("errorMessage", "Invalid credentials");
    }
  }
});
```

#### Customize Transition on Sign Out

On sign out, `ember-devise-simple-auth` automatically transitions back to sign in. If you prefer it goes somehwere different, you can handle the `didSignOut` action:

```javascript
export default Ember.Route.extend({
  actions: {
    didSignOut: function() {
      this.transitionTo("home");
    }
  }
});
```

#### Log Unauthorized Requests

Anytime an unauthorized request is made, `ember-devise-simple-auth` will send an `unauthorizedRequest` action. By default, this action transitions back to sign in, but you can override it to do something else first.

```javascript
export default Ember.Route.extend({
  actions: {
    unauthorizedRequest: function(original) {
      this.logAction("unauthorizedRequest");
      original();
    }
  }
});
```

[&copy;2014 D-I](http://www.d-i.co)
