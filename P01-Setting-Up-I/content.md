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

(This section focuses mostly on Views and Controllers–we'll get to Models in the next section)

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
[TODO: establish setup works with hello world page]
[Nodemon here]

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

## Handlebars

[TODO: no code writing here, just a little deeper overview of what Handlebars is, what the `{{...}}` tags are, etc...]

## Adding a Navbar

[TODO: explain what a navbar is, connect to explanation of Handlebars layout]




After the intro, the tutorial should continue with the core content. Longer tutorials should be broken up into multiple pages.

> [info]
Each `h1` header (headers created with a single `#`) will create a new tutorial step when it's accessed from the online academy. Make sure each step makes sense as a step. All the content in a step should be related but short enough to be digestible.

# The end

After the core content, summarize what student should know now. Also link to the solution repository.

Finish the tutorial by providing links and resources for the student to dive deeper into the subject.
