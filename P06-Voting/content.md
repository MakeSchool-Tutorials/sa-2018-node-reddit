---
title: "Voting"
slug: 06-voting
---

The final step–and the main feature that separates Reddit from any other message board–is up-voting and down-voting posts.

# Add Voting to Posts View

Let's start by adding up-vote and down-vote buttons to our posts. Our posts are rendered on our Rooms `show` view, so let's open `views/rooms/show.hbs`.

```HTML
<div>
  <h1>{{room.topic}}</h1>
</div>

<div>
  {{#each posts as |post|}}
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

<div>
  <a href="/rooms/{{room.id}}/posts/new">New Post</a>
</div>
```

<!-- TODO: talk through code, esp. why these are in forms, classes to make forms like buttons (or even links), end on discussing post.points for segue -->


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

# Make Buttons That POST

# Sort Posts by Vote
