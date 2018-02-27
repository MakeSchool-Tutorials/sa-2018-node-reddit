---
title: "HTTP, REST and Discussion Rooms"
slug: 04-http-rooms
---

In this section we will create "rooms" for our users to have discussions in.  Along the way, we'll learn more about how web apps rely on HTTP to communicate over the internet and how to follow REST conventions so that other developers will be able to predictably work with our app.

TODO: In the next section we'll implement all of these actions–after we introduce CRUD concepts–but in this section we'll just

# HTTP and REST

<!-- TODO -->

# Room Models

TODO: Does this belong in the next section?

Let's start by defining what a "room" is supposed to look like when we store it in our database–that is, what attributes does a room have and what information should we keep there?

Create a file for our room model called `models/room.js` and paste the following code into it:

```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const RoomSchema = new Schema({
  topic: { type: String, required: true },
});

module.exports = mongoose.model('Room', RoomSchema);
```

<!-- TODO: talk through code -->

Before we can check whether this works the way we expect, we need a way to add rooms to our database.  Let's set up `new` and `create` actions for our Rooms to do just that.

# Room New and Create

Following the REST convention, we expect to go to `rooms/new` to find the form to create a new room, and then POST a form to `rooms` to add it to our database. Let's create a new file to hold our room routes called `routes/rooms.js`, and paste in the following:

```Javascript
const express = require('express');
const router = express.Router();

const auth = require('./helpers/auth');
const Room = require('../models/room');

// Rooms index
router.get('/', (req, res, next) => {
  // TODO
});

// Rooms new
router.get('/new', auth.requireLogin, (req, res, next) => {
  // TODO
});

// Rooms show
router.get('/:id', auth.requireLogin, (req, res, next) => {
  // TODO
});

// Rooms edit
router.get('/:id/edit', auth.requireLogin, (req, res, next) => {
  // TODO
});

// Rooms update
router.post('/:id', auth.requireLogin, (req, res, next) => {
  // TODO
});

// Rooms create
router.post('/', auth.requireLogin, (req, res, next) => {
  // TODO
});

module.exports = router;
```
<!-- TODO: explain the routes (note that there's no delete action) -->

Notice that we are requiring login for every route except the index. Any user may see what topics are being discussed, but they'll need to register and log in to read the conversations.

<!-- TODO: Also add these routes to routes/index.js -->
```Javascript
app.use('/rooms', rooms);
```

Recall that the sole purpose of our `new` action is to show users the form they need to make a new room, so we need to create a view with a form on it, then configure `get.('/new', ...)` in `routes/rooms.js` to return that view.

Create a file (in a new `views/rooms` folder) called `views/rooms/new.hbs` and paste in the following code, which will render a web form for creating a new room:
```HTML
<div>
  <form action="/rooms" method="post">
    <legend>New Room</legend>
    <div class="form-group">
      <label for="room-title">Topic</label>
      <input type="text" name="topic" class="form-control" id="room-topic" placeholder="Topic">
    </div>

    <div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </div>
  </form>
</div>
```

And in `routes/rooms.js`, replace `router.get('/new', (req, res, next) => { ... });` with:

```Javascript
// Rooms new
router.get('/new', auth.requireLogin, (req, res, next) => {
  res.render('rooms/new');
});
```
Visit `localhost:3000/rooms/new` (be sure you're logged in!), and you should see this:

<!-- TODO: add screenshot -->

Now, recall that our `create` action listens for a POST request to `/rooms`, then uses the data from that request to store a new Room object in the database. So let's look at `routes/rooms.js`, and replace `.post('/', ...)` with:

```Javascript
// Rooms create
router.post('/', auth.requireLogin, (req, res, next) => {
  let room = new Room(req.body);

  room.save(function(err, room) {
    if(err) { console.error(err) };

    return res.redirect('/rooms');
  });
});
```
<!-- TODO: talk through code -->

Now when you submit a new form on `rooms/new`, you won't get anything back because we haven't implemented our `index` action yet (that's next). If we check out our database on `mlab.com`, however, we can see that rooms are being saved in our database.

<!-- TODO: add mLab screenshot  -->

# Rooms Index

Our Rooms `index` action lists all of the rooms in our database.  Notice that our `create` action above redirects users to the rooms index (`/rooms`).

Let's set up the view at `views/rooms/index.hbs`.  Paste the following inside that file:

```HTML
<div>
  Rooms Index
</div>

<div>
  <ul>
    {{#each rooms as |room|}}
      <li>
        <a href="rooms/{{room.id}}">{{room.title}} | <a href="rooms/{{room.id}}/edit">edit</a>
      </li>
    {{/each}}
  </ul>
<div>
```

Now we need to set up our `index` controller action in `routes/rooms.js`.  (Remember that, by default, Express calls its controller files 'routes' files, and we'll sometimes switch between saying 'controller' and 'routes').  Replace the `.get('/', ...)` function with:
```Javascript
// Rooms index
router.get('/', (req, res, next) => {
  Room.find({}, 'title', function(err, rooms) {
    if(err) {
      console.error(err);
    } else {
      res.render('rooms/index', { rooms: rooms });
    }
  });
});
```

<!-- TODO: talk through code, draw attention to assigning rooms variable -->

In the end, we should be able to visit `localhost:3000/rooms/new`, save a new room, and then see something like this:

<!-- TODO: rooms index screenshot -->

# Rooms Edit and Update











.
