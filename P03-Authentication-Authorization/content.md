---
title: "Authentication and Authorization"
slug: 03-auth
---

<!-- TODO: [In this section we will learn how to require a username and password.  By the end we will have a Registration page where users can sign up, a log in page, and a button for logging out ...] -->

<!-- TODO: Authentication vs. Authorization -->
The word "auth" can refer to either authentication, authorization or both of them together. Authentication is determining who if someone truly is who they say they are–in a web application, we typically authenticate with a username and password. Authorization happens _after_ authentication, and has to do with permissions and access.  In a web app, requiring admin privileges to access a certain page is a form of authorization.

# Authentication: Passwords

User registration, authentication and authorization are pretty basic features needed by almost every web app, and there are lots of tools to help developers with the process. However, in this tutorial we're going to build our own _auth_ solution so that we can learn what goes on under-the-hood. Our solution is also going to be really basic–just username, password, log in, log out, that's it. If you want to add more advanced features, like email verification, password reset, or google/facebook/twitter login, you'll want to use a tool like [Passport](https://passportjs.com).

Even though we're building our system _mostly_ from scratch, we'll install a couple of tools to help us out.  [Bcrypt](https://www.npmjs.com/package/bcrypt) is a cryptography package used to *hash* users' passwords (we'll talk about _hashing_ below), and [express-session](https://github.com/expressjs/session), which will let us store encrypted data called *sessions* in a user's browser.

Let's install these packages by entering the following command into your terminal:

```
npm install bcrypt express-session --save
```

Now that the packages are all installed, let's set our app up to use them.  Let's start by giving our users passwords. Open the `models/user.js` file, and add `password: { type: String, required: true }` into the UserSchema. Your file should look like this:

```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const UserSchema = new Schema({
  username: { type: String, required: true },
  password: { type: String, required: true }
});

const User = mongoose.model('User', UserSchema);
module.exports = User;
```

(Don't forget to add the comma (`,`) at the end of the `username` line!)

Now that our users can have a password (actually, _must_ have a password since we said `required: true`...), let's give them a way to set it up.

Open the `views/users/new.hbs`.  Notice that it contains a form with only one input, for a username.  Try to add a 'Password' input field yourself.  When you're done, compare your file with the solution below.

<!-- TODO: fold behind solution -->

```HTML
<div>
  <form action="/users" method="post">
    <legend>New User</legend>

    <div class="form-group">
      <label for="user-username">Username</label>
      <input type="text" name="username" class="form-control" id="user-username" placeholder="Username">
    </div>

    <div class="form-group">
      <label for="user-password">Password</label>
      <input type="password" name="password" class="form-control" id="user-password" placeholder="Password">
    </div>

    <div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </div>
  </form>
</div>
```

Pay special attention to the input type of that password field, `<input type="password"`, not `text`. This will give our password field some useful properties such as hiding users' passwords when they type.

Let's try it out!  Go to `localhost:3000/users/new`, and now you should see a form with a username and a password field.

![new user](assets/new_user_1.png)

Enter any username and password you like, click 'Submit', and our new user should appear on our user index page.

![new user](assets/new_user_2.png)

That looks good.  Let's check out the new user on mLab to see what it looks like in the database. Go to (mlab.com)[https://mlab.com/home], click on your `makereddit` database, click on `Users` to see the documents, and find your new user in the list:

![mlab user](assets/mlab_user_1.png)

Great, our users have passwords now! If you look in the database you can see that my new password is 'password', and that's not so great.  Forget about the fact that 'password' is really, really not secure. We have a bigger problem–you know my password! If you only learn one rule about web security, learn this: NEVER STORE PLAIN TEXT PASSWORDS. It's a bad idea.  Databases are not magically secure, and storing passwords there makes them very easy to steal.

The trick to avoiding plain text passwords is _hashing_. That's why we installed *Bcrypt*. Bcrypt is like a black box where we can put in a regular password, and take out a _hash_, a version of our password that has been irreversibly scrambled. This way, we never know the user's password. Later, when we implement logging in, the user will enter their password into the log in form, we'll _hash_ it, and then compare the _hashes_. However, it would be almost impossible for us to guess what password produced the hash.

<!-- TODO: this could be explained so much better, and a diagram would be useful -->

Let's open our user model at `models/user.js`.  Require bcrypt at the top of the file (use `const bcrypt = require('bcrypt')`), then paste the following in, after you define the UserSchema:

```Javascript
UserSchema.pre('save', function(next) {
  let user = this;

  bcrypt.hash(user.password, 10, function (err, hash){
    if (err) return next(err);

    user.password = hash;
    next();
  })
});
```

This code comes from the [Bcrypt documentation page](https://github.com/kelektiv/node.bcrypt.js). First, we call the `.pre()` method on `UserSchema`, which takes a callback that fires _before_ (pre-) the action you pass as the first argument. So here we're saying, "before we 'save', call this function." If we wanted the function to happen _after_ we save, we could call `.post()`.

Before we save a user to the database, we call Bcrypt's `.hash()` method, which gives us the hash of the user's password. We then save the _hash_ to the database, _not the password_.

`next()` is a callback that will be supplied by the system later. It's a very common feature in Express apps and part of a feature called [middleware](https://expressjs.com/en/guide/using-middleware.html). You don't need to know all about it right now; you'll get used to seeing it.

<!-- TODO: info box on encryption (2-way) vs hashing (1-way) -->

When you're done, `models/user.js` should look exactly as it does below.

```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const bcrypt = require('bcrypt');

const UserSchema = new Schema({
  username: { type: String, required: true },
  password: { type: String, required: true }
});

UserSchema.pre('save', function(next) {
  let user = this;

  bcrypt.hash(user.password, 10, function (err, hash){
    if (err) return next(err);

    user.password = hash;
    next();
  })
});

const User = mongoose.model('User', UserSchema);
module.exports = User;
```

Let's make sure this works as expected. Go back to `localhost:3000/users/new` and make another new user.  Then, go over to mLab and check the new user in the database–pay special attention to the password, because it should look like a long string of random characters. This is our _hashed_ password.

![mlab user](assets/mlab_user_1.png)

Now that our users have passwords, and we're saving them so securely that even we can never know what they are, let's give them the ability to log in.

# Authentication: Log in

Let's start with the URL–where should our users go to log in? The usual way is to have our users go to `/login`. If we try that right now–in your browser, visit `localhost:3000/login`–what happens?

![404](assets/404.png)

[It doesn't work, that's what happens](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).  We never defined that endpoint–which is to say, we never told our app what to do if someone visits that address.

Let's open the `routes/index.js` file. Users get their own routes file in `routes/users.js`, and other resources will get their own files, too.  But login and logout are kind of special, along with the root (`/`), so we'll keep them right in `index.js`. Let's add a login route, like this:

```Javascript
// login
router.get('/login', (req, res, next) => {
  res.render('login');
});
```

Here we're telling our Controller (or _router_, as it's called in Express) what to do if we receive a GET request to `http://your-site.com/login`. When that happens, our app will fire the callback we provide, which takes three arguments: `req`, `res`, and `next`. We've seen `next` before–that's a callback function provided by Express at runtime, and we usually don't need to think about it too much. `req` and `res` represent the HTTP request and HTTP response–basically, `req` is whatever the browser sends to the server, and `res` is whatever the server sends back to the browser.

<!-- TODO: define runtime  -->

Now let's visit `localhost:3000/login` again.  It still doesn't work, but what's wrong now?

![missing file](assets/missing_file.png)

In our `/login` route, we told our app to render a file called `'login'`. So it looks in the `/views` directory expecting a file called "login", just like we told it to, but that file isn't there. Let's create it–`views/login.hbs`, and paste the following form inside:

```HTML
<form action="/login" method="post">
  <legend>Log in</legend>

  <div class="form-group">
    <label for="user-username">Username</label>
    <input type="text" name="username" class="form-control" id="user-username" placeholder="username">
  </div>

  <div class="form-group">
    <label for="user-password">Password</label>
    <input type="password" name="password" class="form-control" id="user-password" placeholder="password">
  </div>

  <div class='text-right'>
    <button type="submit" class="btn btn-primary">Submit</button>
  </div>
</form>
```
<!-- TODO: update new user form to have submit button in 'text-right' div -->

Now when we visit `localhost:3000/login`, we see our login form.

![login view](assets/login_view.png)

But if we try to log in now, what do you expect to happen?  It won't work yet.  When we submit a form, where does it send that data?  (Hint: check the form's `action`)
<!-- TODO: review HTTP verbs -->

Open `routes/index.js`.  We need to define a POST route to `/login`, so include the following snippet:

```Javascript
router.post('/login', (req, res, next) => {
  console.log('logging in!');
  console.log(req.body);

  res.redirect('/');
});
```

This tells our app what to do if ever it receives a POST request to `http://your-site.com/login`. It's the same URL as above, but our app will behave differently depending on whether it receives a GET or a POST request. We're not doing anything yet, except printing some information to the console. But with all of our groundwork laid, we're ready to authenticate users.

# Authentication

How does logging in even work?  It's gotta be crazy complicated, right?  Some parts are, actually.  We're going to use something called the Blowfish Cypher (hence: `bcrypt`).  You can read all about it in [this Wikipedia article](https://en.wikipedia.org/wiki/Blowfish_(cipher))–but, it may be best not to get too far down that rabbit hole.  The hard parts have been done for us and we have a convenient package to use.  And the basic principles of how secure logins work are pretty easy to understand.

When our users visit `/login`, we will present them with a login form.  In that form, they enter a username and password.  We take the username and use it to find them in the database.  If they exist, we check if they entered the correct password... but I think you can see the problem with that.

We don't know what the user's password is supposed to be (because we NEVER EVER STORE PLAIN TEXT PASSWORDS IN OUR DATABASE).  We have the function that hashed their password in the first place, so we can hash the password they just entered, and see if that matches the hashed password in our database.  In fact, that's what we'll do.

<!-- Wait.  If that's all there is to it, couldn't anybody with access to our database go read [this Wikipedia article about the Blowfish cypher](https://en.wikipedia.org/wiki/Blowfish_(cipher)), or just go download the [bcrypt NPM package](https://www.npmjs.com/package/bcrypt) and un-hash all the users' hashed passwords? Two things protect us from attacks this simple:
- First, _hashing_ is generally one-way, unlike _encryption_, which is usually designed to be _decrypted_.  So even if you have the hashed password _and_ the algorithm that hashed it, you still need the original password to get anywhere.
- Second, _salt_. In this case, _salt_ is a secret string that bcrypt adds to the formula for our app.  It's defined in .  [[TODO: Eep.  Am I salting? --yes, in ]] -->

<!-- TODO: set up sessions in app.js-->
<!-- var session = require('express-session');
app.use(session({ secret: 'jf0832po8', cookie: { maxAge: 3600000 }, resave: true, saveUninitialized: true })); -->

Let's open `models/user.js` to add an `.authenticate()` method to our users.
<!-- TODO: talk through method -->

let's add the `.authenticate()` function declaration to `models/user.js`, so that the entire file looks like this:
```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const bcrypt = require('bcrypt');

const UserSchema = new Schema({
  username: { type: String, required: true },
  password: { type: String, required: true }
});

UserSchema.pre('save', function(next) {
  let user = this;

  bcrypt.hash(user.password, 10, function (err, hash){
    if (err) return next(err);

    user.password = hash;
    next();
  })
});

UserSchema.statics.authenticate = function(username, password, next) {
  User.findOne({ username: username })
    .exec(function (err, user) {
      if (err) {
        return next(err)
      } else if (!user) {
        var err = new Error('User not found.');
        err.status = 401;
        return next(err);
      }
      bcrypt.compare(password, user.password, function (err, result) {
        if (result === true) {
          return next(null, user);
        } else {
          return next();
        }
      });
    });
}

const User = mongoose.model('User', UserSchema);
module.exports = User;
```

<!-- TODO: briefly explain `.statics.` -->

What's next? Somewhere we can call `User.authenticate('myusername', 'secretpassword')` to log our user in, but where do we do that?  We want to authenticate users when they send us their password, which happens when they submit the login form.  Where does that send the password?

<!-- TODO: fold behind solution -->

We wrote code for POST requests to `/login` in `routes/index.js`, so let's open that file.  Our call to `User.authenticate()` looks like this:

```Javascript
User.authenticate(req.body.username, req.body.password, (err, user) => {
    if (err || !user) {
      const next_error = new Error("Username or password incorrect");
      next_error.status = 401;

      return next(next_error);
    } else {
      req.session.userId = user._id;

      return res.redirect('/') ;
    }
  });
```

Let's take a minute to understand what's happening here.
<!-- TODO: talk through code -->

In the end, your complete `/routes/index.js` file should look like this:

```Javascript
const express = require('express');
const router = express.Router();
const User = require('../models/user');

router.get('/', (req, res, next) => {
  res.render('index', { title: 'MakeReddit' });
});

router.get('/login', (req, res, next) => {
  res.render('login');
});

router.post('/login', (req, res, next) => {
  User.authenticate(req.body.username, req.body.password, (err, user) => {
    if (err || !user) {
      const next_error = new Error("Username or password incorrect");
      next_error.status = 401;

      return next(next_error);
    } else {
      req.session.userId = user._id;

      return res.redirect('/') ;
    }
  });
});

module.exports = router;
```

# Toggle "Log in"/"Log out" Link

Go to `localhost:3000` in your browser and notice the header at the top, particularly the 'log in/log out' text on the right.  Obviously, this isn't how log in buttons are supposed to work–it isn't even a real link.  Let's add a bit of logic that checks if a user is logged in–if they're not logged in, we'll offer them a 'log in' link; if they are, the link will say 'log out'.

The header is defined in our layout file.  Open `views/layout.hbs`, and replace the `<nav>` section with the following code:

```HTML
<nav>
  <div class="logo">
    MakeReddit
  </div>

  {{# if current_user}}
    <a href="/logout">log out</a>
  {{else}}
    <a href="/login">log in</a>
  {{/if}}
</nav>
```

<!-- TODO: talk through code -->

However, this doesn't work yet.  Try it if you like–go to `localhost:3000/login`, enter your username and password... and still be offered a link to log _in_.  Take another look at that `{{if}}` statement.  Do you see the problem?  We never assign any value to `current_user`, so it's not possible to avoid the `{{else}}` branch; `current_user` will always be `undefined`.

We need to define the variables for this view in the controller (or in this case, our routes file at `routes/index.js`).   

Let's modify the GET request for our root route to check the session for the existence of a `userId` and pass that value, if it exists, to our view.  Replace the existing `router.get('/', ...)` function with the code below:

```Javascript
router.get('/', (req, res, next) => {
  const currentUser = req.session.userId;

  res.render('index', { title: 'MakeReddit', currentUser: currentUser });
});
```

And now let's try logging in.  Assuming you remembered your password correctly, you should be back at the home page and in the upper right-hand corner it _should_ say "Log out". Nice! But we're not quite finished yet...

Go back to `localhost:3000/login`, and what happens to our "Log in/Log out" link? It's back to "Log in", which means that either we're not logged in anymore (don't worry; we are), or that our layout forgot.  If you look back at `routes/index.js`, you might notice that when we call `res.render` in our `router.get('login', ...)` function, we don't pass any values for  `title` or `currentUser`.  This means that when we go to `/login`, the view (Handlebars) doesn't know what `currentUser` is, so it renders a "Log in" link.  (You might have noticed that the title is missing, too).

<!-- TODO: explain middleware, add the following to routes/index.js -->
```Javascript
// set layout variables
router.use(function(req, res, next) {
  res.locals.title = "MakeReddit";
  res.locals.currentUserId = req.session.userId;

  next();
})
```

<!-- TODO: remove passing locals to view in get('/'), verify all works as expected. -->

# Logout

<!-- TODO: forgot this section... just add post('/logout') function to routes/index.js -->

# Authorization

And now we come to _authorization_.

The key to our authorization strategy will be another piece of middleware that will allow users to access pages if they are logged in, and redirect them to the login page if they are not.  Remember that to the system, being "logged in" just means that your browser has an encrypted cookie with your user id in it.  To check if someone is logged in, we just need to check for the existence of that value.

Let's create a new file, and a new folder, called `routes/helpers/auth.js` and paste the following code inside:

```Javascript
exports.requireLogin = (req, res, next) => {
  if(req.session && req.session.userId) {
    return next();
  } else {
    let err = new Error('You must log in to view this page');
    err.status = 401;

    return res.redirect('/login');
  }
}
```

<!-- TODO: talk through code -->

Let's see an example of how to use this code by requiring authorization for users to view our users index–the list of all the users in the system.  To start, open the users routes file (`routes/users.js`) and include our authorization helpers at the top (`const auth = require('./helpers/auth')`). Now for any route where we want to require authorization, we can simply pass our `auth.requireLogin` middleware as the second argument when we declare the route.  So, `router.get('/', (req, res, next) => { ... })` becomes `router.get('/', auth.requireLogin, (req, res, next) => { ... })`.

At this point, our full `routes/index.js` should look like this:

```Javascript
const express = require('express');
const router = express.Router();
const User = require('../models/user');

// set layout variables
router.use(function(req, res, next) {
  res.locals.title = "MakeReddit";
  res.locals.currentUserId = req.session.userId;

  next();
})


/* GET home page. */
router.get('/', (req, res, next) => {
  // const currentUser = req.session.userId;
  console.log("locals:");
  console.log(res.locals);

  res.render('index');
});

router.get('/login', (req, res, next) => {
  res.render('login');
});

router.post('/login', (req, res, next) => {
  console.log('logging in!');
  console.log(req.body);

  User.authenticate(req.body.username, req.body.password, (err, user) => {
    console.log(err);
    console.log(user);
    if (err || !user) {
      const next_error = new Error("Username or password incorrect");
      next_error.status = 401;

      return next(next_error);
    } else {
      req.session.userId = user._id;
      console.log(req.session);

      // TODO: implement users/show
      // return res.redirect(`/users/${user._id}`);
      return res.redirect('/') ;
    }
  });
});

router.get('/logout', (req, res, next) => {
  if(req.session) {
    req.session.destroy((err) => {
      if(err) {
        return next(err);
      } else {
        return res.redirect('/login');
      }
    });
  }
});

module.exports = router;
```

Before we move on, let's make sure everything here works: be sure that you're logged out of the app (you should see a "Log in" link in the upper right) and try to visit `localhost:3000/users`.  If you're redirected to `localhost:3000/login`, we're ready to move on.
