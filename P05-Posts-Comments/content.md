---
title: "Posts, Database Relationships, Nested Routes"
slug: 05-posts
---

In this section we will:

- Give users the ability to create new Posts
- Let users comment on existing Posts
- Learn about nested routes

# Database Relationships

Objects in our database can be related to each other in different ways. For example, we might have a database of houses where each house has one owner and (for this system) each owner can have only one house–this is a _one-to-one_ relationship. One owner, one house, that's all.

<!-- TODO: [stretch] diagram & schema -->

Or, suppose you're building a music app where users can make playlists. In this case, a playlist can have a bunch of different songs on it, and any of those songs could be on multiple playlists–this is a _many-to-many_ relationship.

<!-- TODO: [stretch] diagram & schema -->

The relationship we're concerned about here is the one between rooms and posts. Each room has a topic, and users are free to discuss that topic by making as many posts as they like. So, one room needs to be able to have many posts, and each post belongs to exactly one room–this is a _one-to-many_ relationship.

<!-- TODO: [stretch] diagram & schema -->

# Post Model

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

Let's create a new file called `routes/posts.js` and add in the following:

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
`routes/posts.js`:
>
```Javascript
const express = require('express');
const router = express.Router({mergeParams: true});
const auth = require('./helpers/auth');
const Room = require('../models/room');
const Post = require('../models/post');
>
router.get('/new', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.roomId, function(err, room) {
    if(err) { console.error(err) };
>
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

<!-- TODO: Preview all the steps -->
<!-- create a Comment model and relate it to Posts
update the room `show` view to display a post's comments and a new comment button
create a Comments controller and define a `/new` action (and nest it inside posts)(you'll also add a `create` action in a later step)
create the view for making a comment
add the `create action` to the controller  -->

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
Look back at `models/post.js` to check the syntax for creating a new Mongoose Schema. (And note that the above example is **not** valid syntax – it's just to show what attributes the Schema needs).
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
  body: { type: String, required: true }
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

## Update the Room `show` View

We want to put a 'New Comment' button directly under each Post, and eventually we'll want to display all of its Comments there as well. Our result should look something like this:

<!-- TODO screenshot -->

The code to create those 'New Comment' links looks like this:

```HTML
<ul>
  <li><a href="/rooms/{{../room.id}}/posts/{{post.id}}/comments/new">New comment</a></li>
</ul>
```

There are a few interesting things about this code snippet: first, it's a list even though there's only one item (`ul` stands for __u__ nordered __l__ ist; your other option is `ol` if you want numbers instead of bullet points [•]). Later you'll include the post's comments as the other items in the list. The other thing to notice is that the comments are *deeply nested* – comments are nested inside posts, which are in turn nested inside rooms. We'll start setting that up in the next section.

>[action]
>
Add the 'New Comment' links to the Rooms `show` view in `views/rooms/show.hbs`. Where can you put the code snippet above so that it appears below each Post? Make sure your answer is similar to the solution below:

<!--  -->

>[solution]
>
`views/rooms/show.hbs`:
>
```HTML
<div class="row justify-content-center">
  <h1>{{room.topic}}</h1>
</div>
>
<div class="row justify-content-center">
  <div class="col-8">
    {{#each posts as |post|}}
      <div class="card mb-3">
        <div class="card-body">
          <h3 class="card-title text-center">{{post.subject}}</h3>
          <p>{{post.body}}</p>
        </div>
>
        <div class="card-footer text-muted text-right">
          <span class="points-span">{{post.points}} points </span>
>
          <div class="btn-group btn-group-sm">
            <button type="button" class="btn btn-success">+</button>
            <button type="button" class="btn btn-danger">-</button>
          </div>
        </div>
      </div>
>
      <!-- Add everything between the <ul>...</ul> tags -->
      <ul>
        {{#each post.comments as |comment|}}
          <li>{{comment.body}}</li>
        {{/each}}
        <li><a href="/rooms/{{../room.id}}/posts/{{post.id}}/comments/new">New comment</a></li>
      </ul>
>
    {{/each}}
>
  </div>
</div>
>
<div class="row justify-content-center">
  <a href="/rooms/{{room.id}}/posts/new">New Post</a>
</div>
```

## Comments Controller (`new` action)

>[action]
>
Next, you'll create a Controller to handle all of your incoming requests at `routes/comments.js` and implement a `new` action. Remember that a `new` action gives the user a form, while the `create` action saves it to the database. You'll add the `create` action a little later.
>
Look back at the `new` action in `routes/posts.js`. This will be similar, but with an extra level of complexity – the Post routes are nested inside the Room routes, and your comment routes will be nested one level deeper. In the Posts `new` action, we had to find the correct Room using the `RoomId`; in the comments `new` action, you'll need to find the correct Room **and** the correct Post. And just like in Posts `new`, you'll want to require authorization.
>
Don't forget to tell the Posts controller about the new nested routes. In our Rooms controller, we used the line `router.use('/:roomId/posts', posts)` to nest posts inside rooms – try to do the same thing with posts and comments. Also, when you rendered `'posts/new'` you only needed to pass the `room` value to Handlebars, but when you render `'comments/new'` you'll need to pass a `room` value and a `post` value.
>
<!-- TODO provide screenshot of error -->
In the end, you should be able to go to a Room with Posts, click on the 'New Comment' link, and receive an error (because the view doesn't exist yet – that's the next step!) Give it a try, and be sure your solution is similar to the one below:

<!--  -->

>[solution]
>
`routes/comments.js`
>
```Javascript
const express = require('express');
const router = express.Router({mergeParams: true});
const auth = require('./helpers/auth');
const Room = require('../models/room');
const Post = require('../models/post');
const Comment = require('../models/comment');
>
// Comments new
router.get('/new', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.roomId, function(err, room) {
    if(err) { console.error(err) };
>
    Post.findById(req.params.postId, function(err, post) {
      if(err) { console.error(err) };
>
      res.render('comments/new', { post: post, room: room });
    });
  })
});
>
module.exports = router;
```
>
`routes/posts.js`
>
```Javascript
// { ...Existing Code... }
  });
});
>
// Add this line:
router.use('/:postId/comments', commentsRouter);
>
module.exports = router;
```

## Comments `new` View

>[action]
>
Now we'll create the view so that the user can enter the information we'll need to create the comment. You'll need to make a new file, `views/comments/new.hbs`, which will be similar to `views/posts/new.hbs`.
>
Of course, it won't be *exactly* the same. You'll only need one text input field, for the Comment's body. For the form's `action` URL, you'll need to nest the comment inside the post and inside the room, like this: `/rooms/{{room.id}}/posts/{{post.id}}/comments`.
>
But, submitting that form won't work yet. Click 'Submit', and you should get a 404 error–implementing that route will be next step. Your code should look like the solution below...

<!--  -->

>[solution]
>
`views/comments/new.hbs`:
>
```HTML
<div class="row justify-content-md-center">
  <div class="col-8">
    <form action="/rooms/{{room.id}}/posts/{{post.id}}/comments" method="post" id="new-comment-form">
      <legend>New Comment</legend>
>
      <div class="form-group">
        <textarea type="text" name="body" class="form-control" id="comment-body" placeholder="Body" form="new-comment-form"></textarea>
      </div>
>
      <div>
        <button type="submit" class="btn btn-primary">Submit</button>
      </div>
    </form>
  </div>
</div>
```

## Comments `create` Action

>[action]
>
Now when a user submits a New Comment form, the data is sent as a POST request to `/rooms/:roomId/posts/:postId/comments` – but we receive an error because we haven't defined any action for that URL. We'll add a `create` action to the Comments controller at `routes/comments.js`.
>
This is going to be somewhat similar to the Posts `create` action in `routes/posts.js`, so look there to remind yourself how it works. However, we're storing references to the comments on the post object (rather than storing a reference to the Post on each Comment, like how we stored a reference to a Room on each Post), things get just a little more complicated. We'll help you out with some *pseudocode* so that you can see the exact steps you'll need to implement:
>
```Javascript
router.post('/', auth.requireLogin, (req, res, next) => {
  Find the correct room with :roomId {
    print the error, if there is one.
>
    Find the correct Post using :postId {
      print the error, if there is one.
>
      Get the  Comment data from req.body
      Use this line to store the comments on the Post: `post.comments.unshift(comment);`
>
      save the post {
        print the error, if there is one.
>
        save the comment {
          print the error, if there is one.
>
          Redirect the user to the Rooms show page.
        }
      }
    }
  }
}
```
>
Give it a shot, then check the solution below:

<!--  -->
<!-- TODO: • comment on unshift() -->

>[solution]
>
```Javascript
// { ...Existing Code... }
>
router.get('/new', auth.requireLogin, (req, res, next) => {
  // { ...Existing Code... }
});
>
// Add this method:
router.post('/', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.roomId, function(err, room) {
    if(err) { console.error(err) };
>
    Post.findById(req.params.postId, function(err, post) {
      if(err) { console.error(err) };
>
      let comment = new Comment(req.body);
      post.comments.unshift(comment);
>
      post.save(function(err, post) {
        if(err) { console.error(err) };
>
        comment.save(function(err, comment) {
          if(err) { console.error(err) };
>
          return res.redirect(`/rooms/${room.id}`);
        });
      });
    });
  });
});
>
module.exports = router;
```
>


## Update Rooms `show` (again)

Earlier we added a 'New Comment' link to each Post in the Rooms show action (`views/rooms/show.hbs`) with the following bit of code:

```HTML
<ul>
  <li><a href="/rooms/{{../room.id}}/posts/{{post.id}}/comments/new">New comment</a></li>
</ul>
```

The reason we put the link inside a list is so that we can easily list all of the comments for each room. To do that, you'll need to use the `{{#each}}` Handlebars helper, and wrap the body of each comment in `<li>...</li>` tags. We used a similar technique to list the rooms in `views/rooms/index.hbs`, so review that code if you need help getting started.

When you're finished, be sure your code matches the solution below:

>[solution]
>
```HTML
<!-- {{ ...Existing code... }} -->
>
{{#each posts as |post|}}
  <div class="card mb-3">
    <div class="card-body">
      <h3 class="card-title text-center">{{post.subject}}</h3>
      <p>{{post.body}}</p>
      <p>{{post}}</p>
    </div>
>
    <div class="card-footer text-muted text-right">
      <span class="points-span">{{post.points}} points </span>
>
      <div class="btn-group btn-group-sm">
        <button type="button" class="btn btn-success">+</button>
        <button type="button" class="btn btn-danger">-</button>
      </div>
    </div>
  </div>
>
  <ul>
    <!-- Add this section: -->
    {{#each post.comments as |comment|}}
      <li>{{comment.body}}</li>
    {{/each}}
    <li><a href="/rooms/{{../room.id}}/posts/{{post.id}}/comments/new">New comment</a></li>
  </ul>
>
{{/each}}
>
<!-- {{ ...Existing code... }} -->
```

Now before we move on, there is one last thing we need to do. We updated our handlebars code to show our comments when we view a room. Before those comments actually show though, we need to go and update one more piece of code. In our rooms router `routes/rooms.js` we are currently fetching each post. When we set up our `Comment` model we added our references to the `Post` model (refer to earlier in this section if you don't follow).

When we are fetching each `Post` we are currently ignoring the comments. In order to show the comments we need to grab our related items (the comments) by telling Mongoose to populate the query. It's okay if you don't quite understand what all of this means. To try and sum it up, we are telling Mongoose to grab the post, and all related comments to this post.

>[action]
Locate the following line of code in `rooms.js`:
>
```Javascript
/* Rooms Show */
router.get('/:id', auth.requireLogin, (req, res, next) => {
  // more code here
});
```
>
Now update your code that fetches the `Post` to look like this:
>
```Javascript
//                      V The new stuff starts here
Post.find({ room: room}).populate('comments').exec(function(err, posts) {
  // this internal code does not change
});
```

# Summary

In this section, we gave users the ability to create new posts in a room. We also let them comment on posts to keep the conversation going.

In the next section, we'll let users vote on comments, so that the top comments are always at the top of the list.
