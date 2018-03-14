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

Try it yourself, then check the solution below.

<!-- TODO: Hide solution behind fold -->

> [solution]
> This content is hidden until the user hovers over the box. Check it out with ms-markdown-preview!
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

<!-- TODO: include code for nesting posts inside rooms -->

# Post Routes (__not__ POST routes...)

We won't create exactly the same set of options for Posts that we did for Rooms. For one thing, we won't have an `index`, because the Room's `show` action will basically serve that purpose by displaying all of the posts for that room.  And, although they could be useful, we won't create `show` or `edit` actions right now, either. So we're starting with only

For now, let's create a new file called `routes/posts.js` and paste in the following:

```Javascript
const express = require('express');
const router = express.Router({mergeParams: true});
const auth = require('./helpers/auth');
const Room = require('../models/room');

// Posts new
router.get('/new', auth.requireLogin, (req, res, next) => {
  // TODO
});

// Posts create
router.post('/', auth.requireLogin, (req, res, next) => {
  // TODO
})

module.exports = router;
```

# Posts New and Create

First, let's give our users a way to create new posts and save them to our database. This is going to be very similar to the `new` and `create` actions we set up for rooms, but...

<!-- TODO: set up and give code snippets, then ask students to implement -->

<!-- TODO: hide solutions behind fold -->

`views/posts/new.hbs`
```HTML
<div>
  <form action="/rooms/{{room.id}}/posts" method="post">
    <legend>New Post</legend>

    <div class="form-group">
      <label for="post-title">Subject</label>
      <input type="text" name="subject" class="form-control" id="post-subject" placeholder="Subject">
    </div>

    <div class="form-group">
      <label for="post-title">Body</label>
      <input type="text" name="body" class="form-control" id="post-body" placeholder="Body">
    </div>

    <div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </div>
  </form>
</div>
```

'routes/posts.js'
```Javascript
const express = require('express');
const router = express.Router({mergeParams: true});
const auth = require('./helpers/auth');
const Room = require('../models/room');

router.get('/new', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.roomId, function(err, room) {
    if(err) { console.error(err) };

    res.render('posts/new', { room: room });
  });
});

router.post('/', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.roomId, function(err, room) {
    if(err) { console.error(err) };

    let post = new Post(req.body);
    post.room = room;

    post.save(function(err, post) {
      if(err) { console.error(err) };

      return res.redirect(`/rooms/${room._id}`);
    });
  });
})

module.exports = router;
```

# Add Posts to Rooms Show View

When we go into a room, we expect to see all of the posts on that topic.  Let's update the Rooms show view so that when we go to a room, we find all of the related posts and render them on the page.

Let's start with the view itself.  Open `views/rooms/show.hbs` and replace the contents with the following:

```HTML
<div>
  <h1>{{room.topic}}</h1>
</div>

<div>
  {{#each posts as |post|}}
    <div class="post-div">
      <h3>{{post.subject}}</h3>
      <p>{{post.body}}</p>
    </div>
  {{/each}}
</div>

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
<!-- TODO -->

<!-- # Comments -->
<!-- TODO -->
