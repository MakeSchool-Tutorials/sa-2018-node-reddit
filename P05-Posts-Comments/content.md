---
title: "Posts, Database Relationships, Nested Routes"
slug: 05-posts
---

In this section we will give users the ability to create posts and comment on existing posts.  Along the way we'll also learn a little about nested routes–what they are, how they work, and why you would want them.

# Database Relationships

Objects in our database can be related to each other in different ways. For example, we might have a database of houses where each house has one owner and (for this system) each owner can have only one house–this is a _one-to-one_ relationship. One owner, one house, that's all.

<!-- TODO: [stretch] diagram & schema -->

Or, say you're building a music app where users can make playlists. In this case, a playlist can have a bunch of different songs on it, and any of those songs could be on multiple playlists–this is a _many-to-many_ relationship.

<!-- TODO: [stretch] diagram & schema -->

The relationship we're concerned about here is the one between rooms and posts. Each room has a topic, and users are free to discuss that topic by making as many posts as they like. So, one room needs to be able to have many posts, and each post belongs to exactly one room–this is a _one-to-many_ relationship.

<!-- TODO: [stretch] diagram & schema -->

# Post Model

<!-- TODO: add schema for all Models -->

First, let's define our Post model so that we can save it to the database.  Just like with Users and Rooms, we'll need to make a file for our model.  In that file, well define a `PostSchema` that has a subject and a body (both Strings), and references its room.

Check out `models/room.js` and `models/user.js` to remind yourself of the syntax, then try it yourself. When you're done, check the solution below.

> [solution]
>
`models/posts.js`:
>
```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
>
const PostSchema = new Schema({
  subject: String,
  body: String,
  room: { type: Schema.Types.ObjectId, ref: 'Room' },
});
>
module.exports = mongoose.model('Post', PostSchema);
```

# Post Routes

We won't create exactly the same set of REST actions for Posts that we did for Rooms. For one thing, we won't have an `index`–later the Room's `show` action will display all of its related posts, and that will serve the purpose of an `index`.  And, although they could _possibly_ be useful, we won't create `show` or `edit` actions right now, either. We're starting with only _new_ and _create_.

Let's create a new file called `routes/posts.js` and paste in the following:

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

Of course, we'll fill in these actions over the next few sections.

## Nested Routes

![Matryoshka Dolls](assets/matryoshka-dolls.jpg)

The REST routes we've seen so far have all been really simple–`/rooms` will give you all of the rooms, and `/rooms/:id` will give you a room with a specific ID. Now we want to do something a little more complicated–because every post belongs to a room (and exactly one room–never 0, never 2 or more), we can _nest the routes_. Nested routes look like `/rooms/:roomId/posts` or `/rooms/:roomId/posts/new`. We still have all seven REST actions, but now all of the routes will be appended to `rooms/:id`

>[action]
>
In order to nest the routes, open the Rooms controller at `routes/rooms.js`, and add the following two lines of code:
>
```Javascript
const express = require('express');
const router = express.Router();
>
// Add this line to require the Posts controller
const posts = require('./posts');
>
// { Existing code here... }
>
// Add this line to nest the routes
router.use('/:roomId/posts', posts)
>
module.exports = router;
```

# Posts New and Create

Let's give our users a way to create new posts and save them in our database through these `new` and `create` actions. This is going to be very similar to the _new_ and _create_ actions we set up for rooms, but with one added twist–our views must reflect our newly nested routes.

Compare the URL we will use for a new Room (`rooms/new`–) to the URL we will use for a new Post(`rooms/:id/posts/new`). We need an extra piece of information for the Post–the ID of the room. This also means that when we render any forms, we have to know the Room's ID to correctly set the form's `action` attribute. At first, this might seem like kind of a bad idea–why restrict ourselves this way? One reason is that we want to design our app so that a Post _must_ belong to a Room, and nested routes are a straightforward way to enforce the requirement.

When we set up our Rooms _new_ view in the previous part (`views/rooms/new.hbs`), we didn't have to pass any values from the controller–and remember that this was different in our `edit` view for example (`views/rooms/edit.hbs`), where we have to pass a `room` object from the _controller_ (`routes/rooms.js`). With nested routes, you'll _always_ have to pass some value into your views. Because all of our posts will be associated with a room, we need to know _which_ Room to use when we create a new Post. Luckily, our route is `rooms/:roomId/posts/new` (see the section on REST in the previous part of this tutorial), which means we always have access to the `roomId`.

In your controller, you can access the `roomId` value as `req.params.roomId`, and it will use this to find the correct room by calling `Room.findById()`. Review the Rooms `edit` controller action (in `routes/rooms.js`) for an example of how to do it.

Try to implement the Posts `new` and `create` actions, then make sure your code matches the solutions below:

>[solution]
>
`views/posts/new.hbs`:
>
```HTML
<div>
  <form action="/rooms/{{room.id}}/posts" method="post">
    <legend>New Post</legend>
>
    <div class="form-group">
      <label for="post-subject">Subject</label>
      <input type="text" name="subject" class="form-control" id="post-subject">
    </div>
>
    <div class="form-group">
      <label for="post-body">Body</label>
      <input type="text" name="body" class="form-control" id="post-body">
    </div>
>
    <div>
      <button type="submit" class="btn btn-primary">Submit</button>
    </div>
  </form>
</div>
```
>
'routes/posts.js':
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

    res.render('posts/new', { room: room });
  });
});
>
router.post('/', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.roomId, function(err, room) {
    if(err) { console.error(err) };
>
    let post = new Post(req.body);
    post.room = room;
>
    post.save(function(err, post) {
      if(err) { console.error(err) };
>
      return res.redirect(`/rooms/${room._id}`);
    });
  });
})
>
module.exports = router;
```

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
>
Here we see another [Handlebars Helper](http://handlebarsjs.com/block_helpers.html) in the form of `#each`. If we have a collection of objects, such as all of the posts that belong in a room, we can create a block of HTML for _each_ one of them. Also notice the `<a>` tag towards the bottom–it's another example of using nested routes.

Before we can use the `posts` collection in that view, we need to define it in our controller (`routes/rooms.js`).  

>[action]
>
Require the Post model at the top of the file, so that Javascript knows what a `Post` is, and update the `show` action to the following:
>
```Javascript
// { existing code... }
>
// Add this line:
const Post = require('../models/post');
>
// { existing code... }
>
// Add the following:
// Rooms show
router.get('/:id', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.id, function(err, room) {
    if(err) { console.error(err) };
>
    Post.find({ room: room }, function(err, posts) {
      if(err) { console.error(err) };
>
      res.render('rooms/show', { room: room, posts: posts });
    });
  });
});
```

Let's check that it works. Browse to `localhost:3000/rooms`.  Open any of the rooms you've created, then click on 'New Post'. Add your new post to the database, and you should be redirected to `room/:id`–which should display your post.

<!-- TODO: add a screenshot -->

<!-- # Partials -->
<!-- TODO: stretch/optional/probably won't do -->

<!-- # Posts Delete -->
<!-- TODO/stretch/probably won't do -->

# Comments

So far our users can create rooms to discuss topics, and post in those rooms–this is starting to look like a real discussion app! But what we have so far still doesn't feel interactive. We can make posts, but the posts don't feel related to each other. On Reddit, users can reply to posts, and other users can reply to those posts, and conversations can become deeply nested. We won't go quite that far in this tutorial (although I encourage you to try it on your own!), but we will let users reply to posts by making comments.

The process of implementing comments will be almost the same as implementing posts (with a *little* extra complexity), so for the next few sections I'll give you hints and let you try to implement each section on your own. Of course, I'll provide my solutions in case you get stuck.

## Comments Model

>[action]
>
First, you'll define your `Comment` model in a `models/comment.js` file. The schema will be really simple, containing only one string attribute called `body`. Like this:
>
```
Comment: {
  body: String
}
```
>
Look back at `models/post.js` to check the syntax for creating a new Mongoose Schema. (And note that the above example is **not** valid syntax – it's just to show what attributes the Schema needs.)
>
You might notice an important difference between our `Post` model and our `Comment` model – `Post` has an attribute to record the room it belongs to. We *could* do that here, but this is a great opportunity to explore the other way to relate Schemas with MongoDB. Rather than each `Comment` recording its `Post`, we'll update our `Post`s  in `models/post.js` to hold an Array of all their comments:
>
```
Post: {
  subject: String,
  body: String,
  room: Room,
  points: Number,
  comments: [Comment]
}
```  

When you're done, you should have something similar to the solutions below:

>[solution]
>
`models/comment.js`:
>
```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
>
const CommentSchema = new Schema({
  body: String,
});
>
module.exports = mongoose.model('Comment', CommentSchema);
```
>
`models/post.js`:
>
```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
>
const PostSchema = new Schema({
  subject: String,
  body: String,
  room: { type: Schema.Types.ObjectId, ref: 'Room' },
  points: { type: Number, default: 0 },
  comments: [{ type: Schema.Types.ObjectId, ref: 'Comment' }],
});
>
module.exports = mongoose.model('Post', PostSchema);
```

## Comments Controller
