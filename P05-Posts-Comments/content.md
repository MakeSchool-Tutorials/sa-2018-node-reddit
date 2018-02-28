---
title: "Posts, One-to-Many Relationships, Nested Routes"
slug: 05-posts
---

In this section we will give users the ability to create posts and comment on existing posts.  Along the way we'll also learn a little about nested routes–what they are, how they work, and why you would want them.

# One-to-Many Relationships

Objects in our database can be related to each other in different ways. For example, we might have a database of houses where each house has one owner and, for this system, each owner can have only one house–this is a *one-to-one* relationship.  Or, say you're building a music app where users can make playlists. Then, a playlist can have a bunch of different songs on it, and any of those songs could be on any number of other playlists–this is a *many-to-many* relationship.

The relationship we're concerned about here is the one between rooms and posts. Each room has a topic, and users are free to discuss that topic as much as they want, so one room needs to be able to contain multiple posts–this is a *many-to-many* relationship.

<!-- TODO: include schemas to demonstrate 3 options to set up in MongoDB, and introduce code for referencing other objects and finding by reference -->

# Post Model

First, let's define our Post model so that we can save it to the database.  Just like with Users and Rooms, we'll need to make a file for our model.  In that file, well define a `PostSchema` that has a subject and a body (both Strings), and references its room.

<!-- TODO: Hide solution behind fold -->

```Javascript
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const PostSchema = new Schema({
  subject: String,
  body: String,
  room: { type: Schema.Types.ObjectId, ref: 'Room' },
});

module.exports = mongoose.model('Post', PostSchema);
```

# Nested Routes

# Post Routes (__not__ POST routes...)

# Posts New and Create

# Add Posts to Rooms Show View

# Posts Delete

# Comments
