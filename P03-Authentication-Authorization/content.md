---
title: "Authentication and Authorization"
slug: auth
---

<!-- TODO: [In this section we will save users in the database, and learn how to require a username and password.  By the end we will have a Registration page where users can sign up, a log in page, and a button for logging out ...] -->

<!-- TODO: Authentication vs. Authorization -->
Authentication and authorization are closely related, but they're not actually the same thing, and it's important to understand the difference.  Authentication is determining who someone is, and if they truly are who they say–in a web application, we typically authenticate with a username and password.  Authorization happens *after* authentication, and has to do with permissions and access.  In a web app, requiring admin privileges to access a certain page is a form of authentication.

The word "auth" can refer to either authentication, authorization or both of them together.

# Authentication

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



# Authorization
