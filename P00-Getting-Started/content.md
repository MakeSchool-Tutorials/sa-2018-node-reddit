---
title: "Getting Started"
slug: 00-get-started
---

<!-- TODO: explain difference between web page and web app -->


1. Introduction

2. MVC; set up Node/Express, mLab & handlebars; 'Hello World' page
  • start with a few words about MVC and the background of Node, Express, MongoDB and Handlebars–particularly on how they all fit together
    - also point out that this section is all 'VC'; let students know we'll talk about models more in the next section
  • create project using Express Generator
    - explain how to read `express -h`, maybe touch on `man` files generally
  • add Bootstrap & style main layout
    - mention other style libraries, mdl, skeleton, semantic, foundation, etc...
    - explain how Handlebars works, including .hbs files, templating, and layouts
    - set up main layout:
      - navbar
      - styling to show how to modify/override bootstrap, (and add light MS branding)

  • misc:
    - use nodemon
    - also add user routes and controller in this section (as part of 'hello world')

3. MongoDB, models, architecture
  - (will use ERDs, but not actively describe them as such)
  • (re-)Explain Models, MongoDB, Mongoose; introduce DBaaS and mLab (mention others, e.g. firebase, atlas)
  • set up mLab (http://docs.mlab.com/)
    - account sign up
    - create db
      - sandbox, aws (though probably doesn't matter)
      - add user (u: ms-user, p: makeschool)
    - connect db
      - install mongoDB
      - install mongo driver (npm install mongodb --save)
      - (https://mongodb.github.io/node-mongodb-native/)
      - install mongoose
  • create User models here

4. Authentication/Authorization; Users
  • see: https://medium.com/of-all-things-tech-progress/starting-with-authentication-a-tutorial-with-node-js-and-mongodb-25d524ca0359
  • add `User` objects to demonstrate saving to DB
    - (give them the html for users/index and users/new to copy/paste, don't explain)
    - point out that further MongoDB actions (CRUD) will be explained in the 'posts' section, but that this section should serve as a preview
    - verify users are showing on mLab
  • install express-session (and explain sessions), bcrypt, passport, and passport-local
  • set up app to use passport

  Authentication:
  • explain authentication/authorization
  • explain hashing (compared to cryptography) & why it's important
  • set up user schema to hash password
  • update user model & views to take username/password
  • update user routes and controller to handle login
  • add authenticate method to User model
  • verify that you can log in
  • (stretch)(TODO)encrypt sessions
  • Add logout button/route
    - show (reinforce) how we know if a user is logged in (being logged in means having a session cookie with a user id)
    - add login/logout button
    - implement logging out (maybe do with GET request for simplicity)

  Authorization:
  • explain cookies & sessions (introduced in previous section, but more detail here)
  • (define middleware, like the medium article does)
  • write requireLogin function
  • add middleware to all routes we want to protect

5. HTTP (& REST?); rooms (subreddits)
  • Explain or review HTTP/REST as necessary (pending dev of earlier tutorials)
  • create `room` model
  • set up `rooms` index action
    - create rooms routes file
    - add .get('/') with simple res.send('kjkjl'), and update app.js
    - show that route works
    - create rooms controller file, define index function
    - create rooms index view file, point to it from controller
      - make index page with fake topics, and explain that we will pull real data from the db in the next section
  • set up `rooms` new action
    - same as index action, except for the form.  So, structure with prompts for students to implement themselves, hide instructions behind 'solution' fold
      - (this can also serve as a preview for how the later sections are all structured)
    - set up form
      - explain web forms, at least any interesting parts of this web form (such as the html5 tags, how labels work with inputs, etc...)
  • set up `rooms` (subreddits? Topics? discussions?) create action
    - add .post('/'), explain how POST is different from GET (has a body, used for forms, etc...)
    - add .create function to rooms controller, including redirect to /rooms, but...
    - don't implement writing to db in this section.  Just print to console or something and segue to next section

6. CRUD (& REST?); Rooms
  • discuss CRUD
    - define
    - preview mongoose methods that let us read/write db (.find, .save, )
  • Save rooms to db after POST (pick up from last section)
    - add save/redirect to roomsController.create
    - (include error handling/messaging here?  Break out into Bonus module?)
  • List rooms on the rooms/index
    - (explicitly review how the route and controller work together, and how they work with the model and view)
    - update .index on the controller
    - explain finding items with mongoose/mongodb
    - update rooms/index view
  • Implement Room show action
    - "now we want to turn this list into links..." (etc...)
    - update rooms/index.hbs, demonstrate that links don't work
    - update rooms routes and controller
      - explain Route Parameters
      - point out that in Express routes, /new must come before /:id
    - implement show view
  • Implement Rooms edit/update actions
    - discuss HTML forms and HTTP verbs
    - talk about possible solutions
      - use an npm module, e.g., https://github.com/expressjs/method-override
      - use a front end framework
      - use AJAX
      - (possibly add one of these solutions as a stretch module)
    - our app will 'cheat' and just use POST requests

    - add edit action to rooms routes and controllers
    - add edit view
    - add update action to routes and controllers

  • (stretch) Implement Rooms delete action
  • (stretch) Record created_at/updated_at

7. CRUD (& REST?) Practice; Posts and Comments
  • (This section should start to ask more of students (e.g., "do this, then compare your answer", or give them failing tests))
  • Structure of this section:
    - Give students a task and all the info they need to complete the task (document schemas, reminding them when they did a similar action)
    - Offer a hint behind a fold
    - Behind the solution fold, include completed code with any further explanations(if needed) below

  • Discuss how to store Posts in Rooms with MongoDB
    - (how to associate documents in MongoDB generally)
    - ~~see http://mongoosejs.com/docs/populate.html~~
  • Give students a schema and ask them to create a Post model
    - Put solution behind a fold, offer a hint
  • Ask students to implement the new and create actions for Posts
    - Introduce idea of nested routes, and give students the code to nest posts inside rooms
      - (avoid spending too much time explaining implementation details specific to Express, but be sure to explain _why_ nested routes are useful)
    - Put solution behind a fold, offer a hint
    * create Posts routes and controller files (only new action)
      * (at first, before view is made, res.send a response)
    * create posts#new view
      * pass room as variable to view
        * (for action ("/rooms/{{room.id}}/posts"))
    * create posts#create action (on routes/controller)
      * don't forget to assign the room to the new post
      * use mLab to verify posts are being created
  • Update the rooms index view to list all of the Posts
    - first, show posts as a ul <<<
    - explain concept of partials
    - create partials directory, create post partial
    - have room#show load all posts as partials
  • (Stretch) Go through all the same steps for comments
  • (Stretch) Ask students to implement the edit and update actions for Posts
    - Put solution behind a fold, offer a hint
  • (Stretch) Ask students to implement the delete action for Posts
    - Put solution behind a fold, offer a hint
  • (Stretch) Give a schema, and ask students to implement all CRUD actions for Comments

8. Voting on posts (review key concepts)
  • Same structure as Section 6
