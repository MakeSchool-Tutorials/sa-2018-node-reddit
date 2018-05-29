---
title: "Setting up our environment part II"
slug: 02-setting-up-ii
---

In this section, we will:

- Create a Mongo DB database with mLab
- Connect our app to our database using Mongoose
- Save user account information in the database

# mLab

Installing and configuring MongoDB from scratch can be a fairly complicated process, and our time for this tutorial is limited. So instead of keeping our database on our local machines, we're going to use a DataBase-as-a-Service (DBaas) called [mLab](https://mlab.com/). Storing our database on mLab's servers offers us two advantages:
- First, it greatly simplifies the setup process.
- Second, it gives us some handy tools for looking around inside our database. Normally we would navigate and manage MongoDB by typing commands in the terminal, but mLab has a Graphic Interface–much easier.

>[action]
>
In your browser, go to (https://mlab.com/signup/) to create an account (unless you already have one–in which case, skip ahead).
>
![mLab Signup Page](assets/mlab-01-signup.png)

>[action]
>
After your account is created, click on `+ Create New` to create a new MongoDB deployment.  Select 'Amazon Web Services' for your provider and 'Sandbox' for your plan type (as shown below) then click 'Continue'.
>
![mLab Deployment Creator](assets/mlab-02-setup.png)

>[action]
>
Choose the AWS region closest to you, and click 'Continue'.
>
![mLab Deployment Creator](assets/mlab-03-setup.png)

>[action]
>
Next, give your database a really cool name, like so:
>
![mLab Deployment Creator](assets/mlab-04-setup.png)

And finally, complete the process by clicking 'Submit Order'.

![mLab Deployment Creator](assets/mlab-05-setup.png)

>[action]
>
We're almost finished setting up our database.  There's just one last step, which is to set up a username and password to keep out nosy neighbors. First, let's open that new database by clicking on its name in your MongoDB Deployments list.
>
![mLab Deployment Creator](assets/mlab-06-deployments.png)

>[action]
>
Inside the database, you should see something like the screenshot below.  First, there are instructions for connecting to a MongoDB driver and a `mongodb://` URI.  We're not quite ready for this yet, but notice it now because we'll be back for it soon.  Down below, let's click on the 'Users' tab.  The list will be empty because we don't have any database users yet, so let's click on '+ Add database user'.
>
![mLab Deployment Creator](assets/mlab-08-new-user.png)

>[action]
>
Use whatever username and password you like (I used username: ms-user, password: makeschool), but be sure to remember it because we'll need it again.  Click on 'Create'.  You should now see one user in your user table.
>
![mLab Deployment Creator](assets/mlab-09-users-table.png)

# Mongoose

Now our database is all set up and ready to go on mLab's servers, but we still need to connect it to our app. We'll do this with a tool called [Mongoose](http://mongoosejs.com/), which is a kind of _Object Database Manager_ (ODM). Writing database code is not too difficult, but it is tedious, repetitive, error-prone, and boring. An ODM helps with writing database code by providing simple methods for common database tasks, such as searching and saving. For example, when we want to save an object, we don't have to go through many steps (connect to the database, start a transaction, locate a document, etc...), we can just call `Object.save()`, and Mongoose will do all the heavy lifting.

## Installing Mongoose

>[action]
>
In your terminal, enter:
>
```
npm install mongodb mongoose --save
```
>
The `--save` tag adds the package to our `package.json` file so that it will be included any time we run `npm install`–you'll need this if you want to work with other developers. Open `package.json` and check for yourself that it has been added under `"dependencies":`.

## Connecting to Our mLab Database

Setting up a database in Express is really easy–that's one of its big advantages.

>[action]
>
Open `app.js` (this file is for configuring Express) and paste the following code near the end of the document, just above the `module.exports = app;` line (which should be on or near line 52):
>
```Javascript
// Database setup
const mongoose = require('mongoose');
const mongoURI = '(your mongodb URI)';

mongoose.connect(mongoURI)
mongoose.Promise = global.Promise;
let db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));
```
>
This is [boilerplate code](https://en.wikipedia.org/wiki/Boilerplate_code) copy-pasted from [Mongoose's Getting Started guide](http://mongoosejs.com/docs/index.html). But, notice that `mongoURI` variable–we need to update that with the information from your mLab database.

>[action]
>
Let's go back to [mLab](https://mlab.com/home) and click on your database. You'll see your MongoDB URI on that screen:
>
![mLab Deployment Creator](assets/mlab-06-deployments.png)
>
![mLab Deployment Creator](assets/mlab-07-database.png)
>
We want the address that starts with `mongodb://...`. Also, be sure to replace `<dbuser>` and `<dbpassword>` with the username and password we set up with the database user we created above. In this example, my MongoDB URI is `mongodb://ms-user:makeschool@ds233228.mlab.com:33228/makereddit1`.
>
The last few lines of your `app.js` file should look like the following (except with **your** mongoURI):
>
```Javascript
// error handler
app.use(function(err, req, res, next) {
  // set locals, only providing error in development
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};
>
  // render the error page
  res.status(err.status || 500);
  res.render('error');
});
>
// Database setup
const mongoose = require('mongoose');
const mongoURI = 'mongodb://ms-user:makeschool@ds233228.mlab.com:33228/makereddit1';
>
mongoose.connect(mongoURI)
mongoose.Promise = global.Promise;
let db = mongoose.connection;
db.on('error', console.error.bind(console, 'MongoDB connection error:'));
>
module.exports = app;
```

# Adding Users

Before we move on, we want to make sure that our database actually, you know, _works_. We're going to add user accounts to our app so that users can post under their own username, log in and out, etc...  Our main purpose is to verify that our database is set up correctly, so we're going to go through a lot of code pretty quickly. Don't worry about understanding everything yet–just follow along, and consider it a preview of what we'll learn over the next few pages.

>[action]
>
First, create a new folder in your root directory called 'models'–obviously, this is for our MVC Models. Inside there, create a new file called `user.js`.  Your file structure should look like this:
>
![file tree](assets/file_tree.png)
>
Paste the following inside `models/user.js`:
>
```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
>
const UserSchema = new Schema({
  username: { type: String, required: true }
});
>
const User = mongoose.model('User', UserSchema);
module.exports = User;
```
>
We'll learn more later but, in short, this file defines what properties our users will have.  To begin with, we will only store a username, but in the future we might also store a password, a name, an email address, etc...

>[action]
>
Next, open the `routes/users.js` file and delete everything in it. Paste in the following:
>
```Javascript
const express = require('express');
const router = express.Router();
const User = require('../models/user');
>
//Users index
router.get('/', (req, res, next) => {
  User.find({}, 'username', function(err, users) {
    if(err) {
      console.error(err);
    } else {
      res.render('users/index', { users: users });
    }
  });
});
>
// Users new
router.get('/new', (req, res, next) => {
  res.render('users/new');
})
>
// Users create
router.post('/', (req, res, next) => {
  const user = new User(req.body);
>
  user.save(function(err, user) {
    if(err) console.log(err);
    return res.redirect('/users');
  });
})
>
module.exports = router;
```
>
This file tells our app what to do when users request certain URLs.  We are inside the `/users` route, so if someone requests `ourwebsite.com/users/`, we will render something called `users/index`.  If someone requests `ourwebsite.com/users/new`, we will render a file called `users/new`. (This system for organizing our resources is called REST, and we'll learn about it in part 4.)
>
As for the files`users/index` and `users/new`, they don't exist yet.  Let's create them.

>[action]
>
Make a new folder inside the `views/` folder called `users`.  Inside `users`, make two new files called `index.hbs` and `new.hbs`.  Your file structure should look like this:
>
![file tree 2](assets/file_tree_2.png)
>
Paste this in `views/users/index.hbs`:
>
```HTML
<div>
  Users Index
</div>
>
<ul>
  {{#each users as |user|}}
    <li>{{user.username}}</li>
  {{/each}}
</ul>
```
>
And paste this into `views/users/new.hbs`:
>
```HTML
<div>
  <form action="/users" method="post">
    <legend>New User</legend>
>
    <div class="form-group">
      <label for="user-username">Username</label>
      <input type="text" name="username" class="form-control" id="user-username" placeholder="Username">
    </div>
>
    <div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </div>
  </form>
</div>
```

>[action]
>
And finally, let's add a _little_ extra CSS to clean things up. Open `public/stylesheets/style.css` and add this:
>
```CSS
main {
  padding: 1em 2em;
}
>
form {
  max-width: 20em;
}
```

>[action]
>
With all of these pieces in place, let's create a user. If the server isn't already running, start it with:
>
```
nodemon start
```
>
Then, go to `localhost:3000/users/new` in your browser. There should be a new user form, with a single field for a username
>
![new user 1](assets/new_user_1.png)
>
Enter a username and click "Submit".  It should take you to a users index page, with a list of all the users in the database (at first there will be only one).
>
![new user 2](assets/new_user_2.png)

<!-- [TODO: explain how this proves the db is working, show users on mLab website] -->

# Summary

In this section, we set up a MongoDB instance on mLab and used Mongoose to connect our application to it. To prove that it works, we created User objects from our app and stored them in the database.

In the next section, we'll give our users the ability to sign in and out.
