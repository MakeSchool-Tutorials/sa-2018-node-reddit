---
title: "Voting"
slug: 06-voting
---

The final step–and the main feature that separates Reddit from any other message board–is up-voting and down-voting posts. We'll sort all of the posts by their number of votes so that (in theory) the most interesting content will always be at the top.

# Add Voting to Posts View

Let's start by adding up-vote and down-vote buttons to our posts.

>[action]
>
Our posts are rendered on our Rooms `show` view, so let's open `views/rooms/show.hbs`, and replace everything inside the `{{#each}}...{{/each}}` loop like so:
>
```HTML
<div>
  <h1>{{room.topic}}</h1>
</div>
>
<div>
  {{#each posts as |post|}}
    <!-- New code below: -->
    <div class="post-div">
      <h3>{{post.subject}}</h3>
      <p>{{post.body}}</p>
      <div class="vote-div">
        <span class="points-span">{{post.points}} points </span>
        <span class="downvote-span">
          <form action="/rooms/{{post.room}}/posts/{{post.id}}" method="post" class="inline-form">
            <input type="hidden" name="points" id="post-points" value="-1">
            <button type="submit" class="downvote-button">-</button>
          </form>
        </span> | <span class="upvote-span">
          <form action="/rooms/{{post.room}}/posts/{{post.id}}" method="post" class="inline-form">
            <input type="hidden" name="points" id="post-points" value="1">
            <button type="submit" class="upvote-button">+</button>
          </form>
        </span>
      </div>
    </div>
  {{/each}}
</div>
>
<div>
  <a href="/rooms/{{room.id}}/posts/new">New Post</a>
</div>
```

<!-- TODO: talk through code, esp. why these are in forms, classes to make forms like buttons (or even links), end on discussing post.points for segue -->

<!-- # Make Buttons That POST -->
<!-- TODO: replace forms in above HTML w/ +/- signs, put and discuss forms in this section. -->

# Add Points to Post Model

Next, let's add points to our Post model. With MongoDB, as opposed to traditional SQL databases, it isn't really _necessary_ to add attributes to our model. We can pass any attributes we like, any time we like and MongoDB will happily save them for us.  However, adding the attributes to our Mongoose schema (by adding them to the model) lets us use them to query and sort our objects. This app is pretty simple, but as you write bigger apps, with more objects and more complex schemas, these features become really handy.

Open the `models/post.js` file, and update it with the following:

```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const PostSchema = new Schema({
  subject: String,
  body: String,
  room: { type: Schema.Types.ObjectId, ref: 'Room' },
  points: { type: Number, default: 0 },
});

module.exports = mongoose.model('Post', PostSchema);
```

<!-- TODO: talk through code, only points line is new -->

# Posts Update Action

Now for the tricky part–what happens when a user clicks on our new up/down-vote buttons? Let's look ahead–in the next section, we're going to make our up/down-vote buttons send POST requests at the HTML:

```HTML
<form action="/rooms/{{post.room}}/posts/{{post.id}}" method="post" class="inline-form">
  <input type="hidden" name="points" id="post-points" value="-1">
  <button type="submit" class="downvote-button">-</button>
</form>

<form action="/rooms/{{post.room}}/posts/{{post.id}}" method="post" class="inline-form">
  <input type="hidden" name="points" id="post-points" value="1">
  <button type="submit" class="upvote-button">+</button>
</form>
```

Both of these forms POST to the same endpoint–`/rooms/{{post.room}}/posts/{{post.id}}`. That address will call our posts `update` action (where we alter and save an existing object). Let's open `routes/posts.js` and define an action for `.post('/rooms/{{post.room}}/posts/{{post.id}}', ...)`:

```Javascript
router.post('/:id', auth.requireLogin, (req, res, next) => {
  Post.findById(req.params.id, function(err, post) {
    post.points += parseInt(req.body.points);

    post.save(function(err, post) {
      if(err) { console.error(err) };

      return res.redirect(`/rooms/${post.room}`);
    });
  });
});
```
<!-- TODO: talk through code, esp. how we determine whether it's an up-vote or down-vote and point out parseInt -->

# Sort Posts by Vote

Finally, let's set it up so that we see our top-voted posts at the top of the page. Open `routes/rooms.js` and update the `show` action like so:

```Javascript
// Rooms show
router.get('/:id', auth.requireLogin, (req, res, next) => {
  Room.findById(req.params.id, function(err, room) {
    if(err) { console.error(err) };

    Post.find({ room: room })
      .sort({ points: -1 })
      .exec()
      .then(function(posts) {
        res.render('rooms/show', { room: room, posts: posts });
      });
  });
});
```

<!-- TODO: this code is a lot to unpack, but reiterate that this is why mongoose models are useful, and point the way toward learning about Promises in the future -->

# Conclusion
