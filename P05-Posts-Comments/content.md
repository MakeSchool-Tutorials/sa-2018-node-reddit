---
title: "Messages, Database Relationships, Nested Routes"
slug: 05-messages
---

In this section we will give users the ability to create posts and comment on existing posts.  Along the way we'll also learn a little about nested routes–what they are, how they work, and why you would want them.

# Database Relationships

Objects in our database can be related to each other in different ways. For example, we might have a database of houses where each house has one owner and (for this system) each owner can have only one house–this is a _one-to-one_ relationship. One owner, one house, that's all.

<!-- TODO: diagram & schema -->

Or, say you're building a music app where users can make playlists. Then, a playlist can have a bunch of different songs on it, and any of those songs could be on any number of other playlists–this is a _many-to-many_ relationship.

<!-- TODO: diagram & schema -->

The relationship we're concerned about here is the one between rooms and posts. Each room has a topic, and users are free to discuss that topic by posting as many messages as they like. So, one room needs to be able to contain multiple posts, and each post belongs to exactly one room–this is a _one-to-many_ relationship.

<!-- TODO: diagram & schema -->

# Message Model

First, let's define our Message model so that we can save it to the database.  Just like with Users and Rooms, we'll need to make a file for our model.  In that file, well define a `MessageSchema` that has a subject and a body (both Strings), and references its room.

Check out `models/room.js` and `models/user.js` to remind yourself of the syntax, then try it yourself. When you're done, check the solution below.

> [solution]
>
```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
//
const MessageSchema = new Schema({
  subject: String,
  body: String,
  room: { type: Schema.Types.ObjectId, ref: 'Room' },
});
//
module.exports = mongoose.model('Message', MessageSchema);
```

<!-- # Nested Routes -->

<!-- TODO: include adding code for nesting posts inside rooms -->

# Messages Routes

We won't create exactly the same set of REST actions for Messages that we did for Rooms. For one thing, we won't have an `index`–the Room's `show` action will basically serve that purpose by displaying all of the messages for that room.  And, although they could _possibly_ be useful, we won't create `show` or `edit` actions right now, either. We're starting with only _new_ and _create_.

Let's create a new file called `routes/messages.js` and paste in the following:

```Javascript
const express = require('express');
const router = express.Router({mergeParams: true});
const auth = require('./helpers/auth');
const Room = require('../models/room');

// Messages new
router.get('/new', auth.requireLogin, (req, res, next) => {
  // TODO
});

// Messages create
router.post('/', auth.requireLogin, (req, res, next) => {
  // TODO
})

module.exports = router;
```

Of course, we'll fill in these actions over the next few sections.

# Messages New and Create

Let's give our users a way to create new posts and save them in our database through these _new_ and _create_ actions. This is going to be very similar to the _new_ and _create_ actions we set up for rooms, but with one added twist–_nested routes_.

When we set up our Rooms _new_ view in the previous part (`views/rooms/new.hbs`), we didn't have to pass any values from the controller–unlike in our _edit_ view for example (`views/rooms/edit.hbs`), where we have to pass a `room` object from the _controller_ (`routes/rooms.js`). In nested routes, you'll _always_ have to pass a value. Because all of our messages will be associated with a room, we need to know _which_ room when we create a new message. Luckily, our route is `rooms/:roomId/messages/new` (see the section on REST in the previous part of this tutorial), which means we always have the `roomId`.

Your _new_ controller action can access the `roomId` value as `req.params.roomId`, and it will use this to find the correct room by calling `Room.findById()`. Review the Rooms _edit_ controller action (in `routes/rooms.js`) for an example of how to do it.

Try to implement the Messages _new_ and _create_ actions, then make sure your code matches the solutions below:

>![solution]
>
`views/posts/new.hbs`
>
```HTML
<div>
  <form action="/rooms/{{room.id}}/messages" method="post">
    <legend>New Message</legend>
>
    <div class="form-group">
      <label for="message-subject">Subject</label>
      <input type="text" name="subject" class="form-control" id="message-subject">
    </div>
>
    <div class="form-group">
      <label for="message-body">Body</label>
      <input type="text" name="body" class="form-control" id="message-body">
    </div>
>
    <div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </div>
  </form>
</div>
```
>
'routes/messages.js'
>
```Javascript
const express = require('express');
const router = express.Router({mergeParams: true});
const auth = require('./helpers/auth');
const Room = require('../models/room');
>
router.get('/new', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.roomId, function(err, room) {
    if(err) { console.error(err) };

    res.render('messages/new', { room: room });
  });
});
>
router.post('/', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.roomId, function(err, room) {
    if(err) { console.error(err) };
>
    let message = new Message(req.body);
    message.room = room;
>
    message.save(function(err, post) {
      if(err) { console.error(err) };
>
      return res.redirect(`/rooms/${room._id}`);
    });
  });
})
>
module.exports = router;
```

<!-- TODO: 'messages' or 'posts'? -->

# Add Posts to Rooms Show View

When we go into a room, we expect to see all of the posts on that topic. Let's update the Rooms show view so that when we go to a room, we find and render all of its related posts on the page.

>[action]
>
Let's start with the view itself.  Open `views/rooms/show.hbs` and replace the contents with the following:
>
```HTML
<div>
  <h1>{{room.topic}}</h1>
</div>
>
<div>
  {{#each posts as |post|}}
    <div class="post-div">
      <h3>{{post.subject}}</h3>
      <p>{{post.body}}</p>
    </div>
  {{/each}}
</div>
>
<div>
  <a href="/rooms/{{room.id}}/posts/new">New Post</a>
</div>
```

<!-- TODO: walk through code.  esp:, do I need to intro or review #each? New Post link is new, also-->

Let's define `posts` in `routes/rooms.js`.  Update the `show` action to the following:

```Javascript
// Rooms show
router.get('/:id', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.id, function(err, room) {
    if(err) { console.error(err) };

    Post.find({ room: room }, function(err, posts) {
      if(err) { console.error(err) };

      res.render('rooms/show', { room: room, posts: posts });
    });
  });
});
```

And let's require our Post model at the top of the file, so that Javascript knows what a `Post` is:

```Javascript
const Post = require('../models/post');
```

<!-- TODO: walk through code -->

Now, let's browse to `localhost:3000/rooms`.  Open any of the rooms you've created, then click on 'New Post'. Add your new post to the database, and you should be redirected to `room/:id`–which should display your post.

<!-- # Partials -->
<!-- TODO: stretch/optional -->

<!-- # Posts Delete -->
<!-- TODO/stretch -->

<!-- # Comments -->
<!-- TODO/student assigned -->
