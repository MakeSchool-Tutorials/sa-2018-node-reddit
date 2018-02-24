---
title: "HTTP, REST and Discussion Rooms"
slug: 04-http-rooms
---

In this section we will create "rooms" for our users to have discussions in.  Along the way, we'll learn more about how web apps rely on HTTP to communicate over the internet and how to follow REST conventions so that other developers will be able to predictably work with our app.

<!-- TODO: Also introduce REST -->

TODO: In the next section we'll implement all of these actions–after we introduce CRUD concepts–but in this section we'll just

# HTTP and REST

<!-- TODO -->

# Room Models

TODO: Does this belong in the next section?

Let's start by defining what a "room" is supposed to look like when we store it in our database–that is, what attributes does a room have and what information to we keep there?

Let's start by creating file for our room model called `models/room.js` and paste the following code into it:

```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const RoomSchema = new Schema({
  topic: { type: String, required: true },
});

module.exports = mongoose.model('Room', RoomSchema);
```

<!-- TODO: talk through code -->

Before we can check whether this works the way we expect, we need a way to add rooms to our database.

# Room Routes

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

Notice that we are requiring login for every route except the index. Any user may see what topics are being discussed, but they'll need to register and log in to read the conversations.

<!-- TODO: explain the routes (note that there's no delete action) -->

# Rooms Index
