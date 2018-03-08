---
title: "Setting up our environment part II"
slug: 02-setting-up-ii
---
In this section, we'll finish setting up our dev environment by installing and configuring our database.  We will also let users register accounts, and save their info in the database.

<!-- [TODO: Introduction to MongoDB] -->
<!-- [TODO: now adding users in this section] -->

# mLab

Installing and configuring MongoDB from scratch is a fairly complicated process, so instead of keeping our database on our local machines, we're going to use a DataBase-as-a-Service (DBaas) called [mLab](https://mlab.com/). Storing our database on mLab's servers offers us two advantages:
- First, it greatly simplifies the setup process.
- Second, it gives us some handy tools for looking around inside our database. Normally we would navigate and manage our MongoDB instance on the command line, but mLab has a Graphic Interface–much easier.

In your browser, go to (https://mlab.com/signup/) to create an account (unless you already have one–in which case, skip ahead).

![mLab Signup Page](assets/mlab-01-signup.png)

After your account is created, click on `+ Create New` to create a new MongoDB deployment.  Select 'Amazon Web Services' for your provider and 'Sandbox' for your plan type (as shown below) then click 'Continue'.

![mLab Deployment Creator](assets/mlab-02-setup.png)

Choose the AWS region closest to you, and click 'Continue'.

![mLab Deployment Creator](assets/mlab-03-setup.png)

Next, give your database a really cool name, like so:

![mLab Deployment Creator](assets/mlab-04-setup.png)

And finally, complete the process by clicking 'Submit Order'.

![mLab Deployment Creator](assets/mlab-05-setup.png)

We're almost finished setting up our database.  There's just one last step, which is to set up a username and password to keep nosy neighbors out.  First, let's open that new database by clicking on its name in your MongoDB Deployments list.

![mLab Deployment Creator](assets/mlab-06-deployments.png)

Inside the database, you should see something like the screenshot below.  First, there are instructions for connecting to a MongoDB driver and a `mongodb://` URI.  We're not quite ready for this yet, but I want to point it out because we'll be back for it soon.  Down below, let's click on the 'Users' tab.  The list will be empty because we don't have any database users yet, so let's click on '+ Add database user'.

![mLab Deployment Creator](assets/mlab-08-new-user.png)

Use whatever username and password you like (I used username: ms-user, password: makeschool), but be sure to remember it because we'll need it again soon.  Click on 'Create'.  You should now see one user in your user table.

![mLab Deployment Creator](assets/mlab-09-users-table.png)

# MongoDB Driver and Mongoose

Now our database is all set up and ready to go on mLab's servers–but now we need to make our app connect to it.  We need to install two packages to help us do that.  The first one–the MongoDB driver–lets us control a MongoDB database through a Node app. It will do its work mostly behind the scenes, though, because we're going to use [Mongoose](http://mongoosejs.com/) to work with our database.  Writing database code is not difficult (once you learn how), but it is complicated, error-prone and boring;  Mongoose makes things much simpler. [...]

## Installing the MongoDB Driver

<!-- [TODO: a step might be missing here.  QA on a computer that hasn't had mongodb installed before] -->

In your terminal, enter
```
npm install mongodb --save
```
(the `--save` tag adds the package to our `package.json` file so that will be included any time we run `npm install`)

## Installing Mongoose

In your terminal, enter
```
npm install mongoose --save
```

## Connecting to Our mLab Database

Open `app.js`, and paste the following code at the end of the document, just above the `module.exports = app;` line (probably line 52, but your file might be slightly different from mine):
```Javascript
// Database setup
const mongoose = require('mongoose');
const mongoURI = '(your mongodb URI)';

mongoose.connect(mongoURI)
mongoose.Promise = global.Promise;
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));
```

<!-- [TODO: explain this block of code, or link to Mongoose's Getting Started page at http://mongoosejs.com/docs/index.html, for reference] -->

Notice that mongoURI variable–let's go back to (mLab)[https://mlab.com/home] and click on your database. You'll see your MongoDB URI on that screen:

![mLab Deployment Creator](assets/mlab-06-deployments.png)

We want the address that starts with `mongodb://...`, and be sure to replace `<dbuser>` and `<dbpassword>` with the username and password for the database user we created above. In this example, my MongoDB URI is `mongodb://ms-user:makeschool@ds233228.mlab.com:33228/makereddit1`.

The last few lines of your `app.js` file should look like the following (except with *your* mongoURI):

```Javascript
// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  // render the error page
  res.status(err.status || 500);
  res.render('error');
});

// Database setup
const mongoose = require('mongoose');
const mongoURI = 'mongodb://ms-user:makeschool@ds233228.mlab.com:33228/makereddit1';

mongoose.connect(mongoURI)
mongoose.Promise = global.Promise;
var db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));

module.exports = app;
```

# Adding Users

Before we move on, let's make sure that our database actually works. We're going to add user accounts to our app [so that users can post under their own username, log in and out, have private information, etc...].  [set expectations: We're going to copy-paste a lot of code quickly/don't worry we'll explain it all later/...]

First, create a new folder in your root directory called 'models', and inside there create a new file called `user.js`.  Your file structure should look like this:

<!-- [TODO: add file structure tree] -->

Paste the following inside `models/user.js`:

```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const UserSchema = new Schema({
  username: { type: String, required: true }
});

const User = mongoose.model('User', UserSchema);
module.exports = User;
```

We'll learn about what's happening here in detail in later sections but, in short, this file defines what properties our users will have.  To begin with we will only store a username, but in the future we might also store a password, a name, an email address, etc...

Next, open the `routes/users.js` file and delete everything in it. Paste in the following:

```Javascript
const express = require('express');
const router = express.Router();
const User = require('../models/user');

router.get('/', (req, res, next) => {
  res.render('users/index');
});

router.get('/new', (req, res, next) => {
  res.render('users/new');
})

router.post('/', (req, res, next) => {
  const user = new User(req.body);

  user.save(function(err, user) {
    if(err) console.log(err);
    return res.redirect('/users');
  });
})

module.exports = router;
```

Again, we'll learn all about routes later on in this tutorial, so don't worry about understanding everything right now.  Basically, this file tells our app what to do when users request certain URLs.  We are inside the `/users` route, so if someone requests `ourwebsite.com/users/`, we will render something called `users/index`.  If someone requests `ourwebsite.com/users/new`, we will render something called `users/new`.  (Don't worry about the `post` right now–we'll get there.)

As for `users/index` and `users/new`, they don't exist yet.  Let's create them. Make a new folder inside the `views/` folder called `users`.  Inside `users`, make two new files called `index.hbs` and `new.hbs`.  Your file structure should look like this:

<!-- [TODO: add file structure tree] -->

In `views/users/index.hbs`, paste this:
```HTML
<div>
  Users Index
</div>


<ul>
  {{#each users as |user|}}
    <li>{{user.username}}</li>
  {{/each}}
</ul>
```

And paste this into `views/users/new.hbs`:
```HTML
<div>
  <form action="/users" method="post">
    <legend>New User</legend>

    <div class="form-group">
      <label for="user-username">Username</label>
      <input type="text" name="username" class="form-control" id="user-username" placeholder="Username">
    </div>

    <div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </div>
  </form>
</div>

```

With all of these pieces in place, let's create a user. In your terminal, if the server isn't already running, start it with:

```
nodemon start
```

Then, go to `localhost:3000/users/new` in your browser. There should be a new user form, with a single field for a username, like so:
<!-- [TODO: add screenshot] -->

Enter a username and click "Submit".  It should take you to a users index page, with a list of all the users in the database (at first there will be only one).  

<!-- [TODO: explain how this proves the db is working, show users on mLab website] -->

<!-- # Summary -->
<!-- TODO: Conclusion/segue to auth -->

# Links and Resources

- [mLab](https://mlab.com/)
