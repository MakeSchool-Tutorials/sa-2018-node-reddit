---
title: "Authentication and Authorization"
slug: auth
---

<!-- TODO: [In this section we will save users in the database, and learn how to require a username and password.  By the end we will have a Registration page where users can sign up, a log in page, and a button for logging out ...] -->

<!-- TODO: Authentication vs. Authorization -->
Authentication and authorization are closely related, but they're not actually the same thing, and it's important to understand the difference.  Authentication is determining who someone is, and if they truly are who they say–in a web application, we typically authenticate with a username and password.  Authorization happens *after* authentication, and has to do with permissions and access.  In a web app, requiring admin privileges to access a certain page is a form of authentication.

The word "auth" can refer to either authentication, authorization or both of them together.

# Authentication: Passwords

User registration, authentication and authorization are pretty basic features needed by almost every web app, so there are lots of tools for us to choose from to help us get started.  The main package we'll use here is called [Passport](http://www.passportjs.org/).
<!-- TODO: a little more background on passport and alternatives, and passport-local -->

Passport requires us to install a couple more packages as dependencies.  [Bcrypt](https://www.npmjs.com/package/bcrypt) is a cryptography package used by Passport to *hash* users' passwords (we'll talk about that more below), and [express-session](https://github.com/expressjs/session), which will let us store encrypted cookies called *sessions* in a user's browser where we can store user data.

Let's install all of these packages by entering the following command into your terminal:

```
npm install passport passport-local bcrypt express-session --save
```

Now that the packages are all installed, let's configure our app to use them.  Start by opening the `models/user.js` file so that we can let our users have passwords. Add `password: { type: String, required: true }` into the UserSchema so that your file looks like this:
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

Now that our users can have a password (actually, _must_ have a password...), let's give them a place to set it up.

Open the `views/users/new.hbs` file.  Notice that it contains a form with only one input, for a username.  Try to add a 'Password' input field yourself.  When you're done, compare your file with the solution below.

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

(Pay special attention to the input type, `<input type="password"`: it's password, instead of text.  This will give our password field some extremely useful properties, such as hiding users' passwords when they type)

Let's try it out!  Go to `localhost:3000/users/new`, and now you should see a form with a username and a password field.

<!-- TODO: add screenshot -->

Enter any username and password you like, click 'Submit', and our new user should appear on our user index page.

<!-- TODO: add screenshot -->

That looks good.  Let's check out the new user on mLab to see what it looks like in the database:

<!-- TODO: add screenshot -->

Great, our users have passwords now! But if you look in the database you can see that the password I entered is 'password', and that's not so great.  Forget about the fact that 'password' is really, really not secure, we have a bigger issue.  If you only learn one rule about web security, learn this: NEVER STORE PLAIN TEXT PASSWORDS.  It's a bad idea.  Databases are not magically secure, and storing passwords there makes them very easy to steal.

That's why we installed Bcrypt.  Let's open our user model at `models/user.js`.  Require bcrypt at the top of the file (use `const bcrypt = require('bcrypt')`), then paste the following in after you define the UserSchema:

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
<!-- TODO: walk through this code -->

When you're done, `models/user.js` should look exactly as it does in the solution below.

<!-- TODO: fold behind solution -->

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

Let's make sure this works as expected. Go back to `localhost:3000/users/new` and make another new user.  Then, go over to mLab and check the new user in the database–pay special attention to the password, because it should look like a long string of random characters. This is a *hashed* password.

<!-- TODO: explain hashing (and encryption), and why we're using hashing here -->

Now that our users have passwords, and we're saving them so securely that even we can never know what they are, let's give our users the ability to log in.

# Authentication: Log in

Let's start with this idea: our users should log in by going to `/login`.  If we try that right now–in your browser, visit `localhost:3000/login`–what happens? [It doesn't work, that's what happens](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status).  We never defined that endpoint–which is to say, we never told our app what to do if someone visits that address.

Let's open the `routes/index.js` file. (Users get their own routes file in `routes/users.js`, and other resources will get their own files, too.  But login/logout is kind of special, so we'll keep them right in the index). Let's add a login route, like this:

```Javascript
router.get('/login', (req, res, next) => {
  res.render('login');
});
```

Now let's visit `localhost:3000/login` again.  It still doesn't work, but what's wrong now?

<!-- TODO: add 'Failed to lookup view "login" in views directory' screenshot -->

It's trying to load a view, so it looks in the `/views` directory for a file called "login", just like we told it to, but that file isn't there. Let's create the file `views/login.hbs`, and paste the following form inside:

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

Now when we visit `localhost:3000/login`, we see our login form.

<!-- TODO: add screenshot -->

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

Now, with all of our groundwork laid, we're finally ready to authenticate users.

# Authentication

How does logging in even work?  It's gotta be crazy complicated, right?  Some parts are, actually.  We're going to use something called the Blowfish Cypher (hence: `bcrypt`).  You can read all about it in [this Wikipedia article](https://en.wikipedia.org/wiki/Blowfish_(cipher))–but, it may be best not to get too far down that rabbit hole.  The hard parts have been done for us and we have a convenient package to use.  And the basic principles of how secure logins work are pretty easy to understand.

When our users visit `/login`, we will present them with a login form.  In that form, they enter a username and password.  We take the username and use it to find them in the database.  If they exist, we check if they entered the correct password... but I think you can see the problem with that.

We don't know what the user's password is supposed to be (because we NEVER EVER STORE PLAIN TEXT PASSWORDS IN OUR DATABASE).  We have the function that hashed their password in the first place, so we can hash the password they just entered, and see if that matches the hashed password in our database.  In fact, that's what we'll do.

<!-- Wait.  If that's all there is to it, couldn't anybody with access to our database go read [this Wikipedia article about the Blowfish cypher](https://en.wikipedia.org/wiki/Blowfish_(cipher)), or just go download the [bcrypt NPM package](https://www.npmjs.com/package/bcrypt) and un-hash all the users' hashed passwords? Two things protect us from attacks this simple:
- First, _hashing_ is generally one-way, unlike _encryption_, which is usually designed to be _decrypted_.  So even if you have the hashed password _and_ the algorithm that hashed it, you still need the original password to get anywhere.
- Second, _salt_. In this case, _salt_ is a secret string that bcrypt adds to the formula for our app.  It's defined in .  [[Eep.  Am I salting? TODO: salt ]] -->

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
      console.log(req.session);

      return res.redirect('/') ;
    }
  });
```

<!-- TODO: talk through code -->

In the end, your complete `/routes/index.js` file should look like this:

```Javascript
var express = require('express');
var router = express.Router();

router.get('/', (req, res, next) => {
  res.render('index', { title: 'MakeReddit' });
});

router.get('/login', (req, res, next) => {
  res.render('login');
});

router.post('/login', (req, res, next) => {
  console.log('logging in!');
  console.log(req.body);

  User.authenticate(req.body.username, req.body.password, (err, user) => {
    if (err || !user) {
      const next_error = new Error("Username or password incorrect");
      next_error.status = 401;

      return next(next_error);
    } else {
      req.session.userId = user._id;
      console.log(req.session);

      return res.redirect('/') ;
    }
  });
  res.redirect('/');
});

module.exports = router;
```

# Logging out

# Authorization












.
