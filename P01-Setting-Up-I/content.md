---
title: "Setting up our environment"
slug: setting-up
---
In this section we'll set up our environments. [...]
We're going to learn how to set up a web server using Express, generate HTML using Handlebars, and make it look good using Bootstrap.

Before we dive in, it's helpful to think about the big picture and understand how these tools work together.  We're going to build our app using an architecture called Model-View-Controller (MVC).

# MVC

Under the MVC architecture, we think of our app as having three main systems that work together:
  - The View is the part of our app the user sees and clicks on. [...]
  - The Model is the part where we store information [...]
  - The Controller is the part in between–the code that gets information from the Models and sends it to the Views, and takes information from the Views and stores it in the Models. [...]

(This section focuses mostly on Views–we'll get to Models in the next section, and controllers in part 4 [TODO: check if accurate])

The first section of the tutorial should summarize what the student will learn and what tools they'll use. It should also outline any prerequisites for understanding the material and link to resources for getting up to speed. It's usually a good idea to show a screenshot or video of the final product and link to the solution repository.



# Express

[Explain what a web server is] At its most basic, a web server is a piece of software that receives an HTTP request and returns an HTTP response.  What happens in between can be simple or insanely complicated–it depends on the situation. [Example, etc...]

Express is the most popular web server in the Node ecosystem. [... Express is unopinionated...]

## Express Generator

Express is unopinionated, which means we are free to organize the files in our code however we want.  Some structure can be helpful for getting started.  There are lots of apps and packages to help you start a new project [examples...]  For this tutorial, we're going to use the Express team's own generator, [Express Generator](https://expressjs.com/en/starter/generator.html)

We assume you have Node and NPM installed on your computer.  [If not, ...] Open your terminal and enter:
```
npm install express-generator -g
```
(The `-g` option tells NPM to install the package globally)

After it's installed, enter:
```
express --view=hbs --css=sass makereddit
```
(The `--view=hbs` means that we want to use a package called [Handlebars](http://handlebarsjs.com/) for our views and [Sass](https://sass-lang.com/) for our CSS.  We'll explain these in more detail below.  Express Generator has a ton of options–to see for yourself, enter `express -h`)

## Directory Structure

[Now we have a template for a basic, empty app.  Open that directory in your favorite text editor, and let's look at at the structue... [How much explanation does this need, will they all have the same setup, etc...]]

[image of directory structure here]

## Hello World

Let's open the `views/index.hbs` file.  Soon we'll learn more about what this file is and how it works, but for right now it's enough to understand that this is our "home page".  This is the file that renders when people first visit our website.

Let's change `Welcome to {{title}}` to `Hello, world`, so that `views/index.hbs` looks like this:
```HTML
<h1>{{title}}</h1>
<p>Hello, world</p>
```
[TODO: comment on why 'hello world']

Now, before we can run our server, we need to install our packages. [TODO explain better] In your terminal, enter:
```
npm install
```

And then wait while NPM installs all of the background software our app needs.  This can take anywhere from a few seconds to 10 minutes, so you may need to be patient!

Once it's finished–if there were no errors–start the server by entering into your terminal:
```
npm start
```

Then open your browser and go to `localhost:3000`.  You should see something like this:
[TODO: screenshot]

Congratulations! You just built a web app! We've still got a long way to go before it does anything useful, and we still need to learn about how it works, but we've taken our first big step.

## Nodemon

Let's add one more useful tool before we move on.  We just want our website to say "hello, world" so we know it's working, but the biggest letters on the page actually say "Express".  If we look back at our code in `views/index.hbs` it seems to be coming from the `<h1>{{title}}</h1>`, but what are those curly braces for, and how does it know `title` is supposed to be `Express`?

This feature comes from Handlebars (which is why these files end with '.hbs').  We'll learn about Handlebars in more detail further down this page, but for now let's just change that `title` to "Hello, world" instead of "Express".

First, open `/routes/index.js` and lines 5-7 should look like this:
```Javascript
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Express' });
});
```

We're going to learn all about routes starting in [part 4], so don't worry if this still seems mysterious–soon, it won't. This just tells our web server that when anybody visits the root path of our website, or the home page, it should render a file called 'index' (`/views/index.hbs`!), and then it assigns the value 'Express' to the key 'title'.

Let's change that 'title' to 'Hello, world', so that the entire `/routes/index.js` file looks like this:
```Javascript
var express = require('express');
var router = express.Router();

/* GET home page. */
router.get('/', function(req, res, next) {
  res.render('index', { title: 'Hello, world' });
});

module.exports = router;
```

Now–be sure to save your file!–then, let's go back to the browser and hit refresh.  And:
[TODO: screenshot]
:flushed:

Unfortunately, Express only reads these files when it's starting.  After that, it ignores any changes we make.  One quick fix is to simply restart the server:
- Go to the terminal where the server is running
- Hit `control` + `c` on your keyboard. (This is a very common command for stopping command line programs–get used to it because you'll see it again and again)
- enter `npm start` to restart the server

Now when we refresh the page:
[TODO: screenshot]
:relieved:

But it's going to be a huge pain if we have to stop and restart the server after every. single. little. change.  Luckily, there are lots of packages that will take care of restarting the server for us.  As you grow as a developer, you'll eventually want to learn about tools like [Webpack](https://webpack.js.org/) or [Yarn](https://yarnpkg.com/en/).  But for this project, we're going to use a super simple solution called [nodemon](https://nodemon.io/) (short for "Node Monitor").

First, open your terminal and enter:
```
npm install -g nodemon
```
(the `-g` means we're installing it globally, not just for this project)

Now, instead of typing `npm start`, we'll type `nodemon start`.  Go ahead:  use `control`+`c` to stop the running server, then enter:
```
nodemon start
```

Now in our `routes/index.js` file, let's set the title on our home page to be our real title, "MakeReddit":
```Javascript
router.get('/', function(req, res, next) {
  res.render('index', { title: 'MakeReddit' });
});
```

Now, **without** restarting the server, let's go to the browser and refresh the page:
[TODO: screenshot]
:satisfied:

# Bootstrap and Handlebars

[This tutorial is focused more on the back end than the front end–we're more concerned with how our app works than how it looks.  But thanks to pre-existing CSS libraries, it's really easy to make it look good enough [...]]

## Installing Bootstrap

We're going to use one of the most popular CSS frameworks available, [Bootstrap](https://getbootstrap.com/). Installing Bootstrap is really easy, and we'll follow [these instructions](https://getbootstrap.com/docs/4.0/getting-started/introduction/).   [...]

First, let's open the `views/layout.hbs` file.  At first, it should look like this:
```HTML
<!DOCTYPE html>
<html>
  <head>
    <title>{{title}}</title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
  </head>
  <body>
    {{{body}}}
  </body>
</html>
```

Remember what we learned about HTML [in previous tutorial], and then let's look back at `views/index` for a moment–what's missing?  This isn't a complete HTML document...
[...]

This layout file is special because it will load every page inside this html.  [... example with wireframes]

First, we'll include the CSS inside the `<head>` tag like this:
```HTML
<html>
  <head>
    <title>{{title}}</title>
    <link rel='stylesheet' href='/stylesheets/style.css' />
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
  </head>
  <body>
  ...
```

Next, Bootstrap includes some dynamic elements (for example, responsive elements rearrange themselves based on the size of the user's screen–if they're using a 4" smart phone or a 32" monitor).  These dynamic elements require some Javascript, and we'll load it at the end of the layout just before the closing `</body>` tag.  Copy/paste the three `<script>` tags into your `layout.hbs` file like this:
```HTML
...
  <body>
    {{{body}}}

    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
  </body>
</html>
```

[TODO: explain why css goes in the head and js goes in the body, review HTML structure]

## Handlebars

[TODO: no code writing here, just a little deeper overview of what Handlebars is, what the `{{...}}` tags are, etc...]

## Adding a Navbar

[TODO: explain what a navbar is, connect to explanation of Handlebars layout]

[TODO: explain Bootstrap components, and how to use Bootstrap documentation]

We'll install the Bootstrap Navbar together, step by step. But first, take a few minutes to [skim the instructions](https://getbootstrap.com/docs/4.0/components/navbar/).  

Bootstrap gives us a ton of options to customize our navbar.  For this app, we only want a simple navbar with a logo and a few links on the left and a login/logout link on the right, like this:

```
[TODO: include wireframe]
```

We'll include that navbar with the following HTML:
```HTML
<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
        <span class="sr-only">Toggle navigation</span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
        <span class="icon-bar"></span>
      </button>
      <a class="navbar-brand" href="/">MakeReddit</a>
    </div>

    <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
      <ul class="nav navbar-nav navbar-right">
          <li><a href="#">log in/log out</a></li>
      </ul>
    </div>
  </div>
</nav>
```
[TODO: (Stretch) explain this snippet]

Let's copy that snippet and paste it in our `views/layout.hbs` file.  Make it the first thing inside the `<body>` tag, like so:
```HTML
...
  <body>
    <nav class="navbar navbar-default">
      <div class="container-fluid">
        <div class="navbar-header">
          <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand" href="/">MakeReddit</a>
        </div>

        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
          <ul class="nav navbar-nav navbar-right">
              <li><a href="#">log in/log out</a></li>
          </ul>
        </div>
      </div>
    </nav>

    {{{body}}}

    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
  </body>
...
```

[TODO: give students .scss file to include w/ basic styling.  Set up further styling as a stretch challenge]


After the intro, the tutorial should continue with the core content. Longer tutorials should be broken up into multiple pages.

> [info]
Each `h1` header (headers created with a single `#`) will create a new tutorial step when it's accessed from the online academy. Make sure each step makes sense as a step. All the content in a step should be related but short enough to be digestible.

# The end

After the core content, summarize what student should know now. Also link to the solution repository.

Finish the tutorial by providing links and resources for the student to dive deeper into the subject.
