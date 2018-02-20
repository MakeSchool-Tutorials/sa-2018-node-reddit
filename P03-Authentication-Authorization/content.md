---
title: "Authentication and Authorization"
slug: auth
---

<!-- TODO: [In this section we will save users in the database, and learn how to require a username and password.  By the end we will have a Registration page where users can sign up, a log in page, and a button for logging out ...] -->

<!-- TODO: Authentication vs. Authorization -->
Authentication and authorization are closely related, but they're not actually the same thing, and it's important to understand the difference.  Authentication is determining who someone is, and if they truly are who they say–in a web application, we typically authenticate with a username and password.  Authorization happens *after* authentication, and has to do with permissions and access.  In a web app, requiring admin privileges to access a certain page is a form of authentication.

The word "auth" can refer to either authentication, authorization or both of them together.

# Installing Packages

User registration, authentication and authorization are pretty basic features needed by almost every web app, so there are lots of tools for us to choose from to help us get started.  The main package we'll use here is called [Passport](http://www.passportjs.org/).
<!-- TODO: a little more background on passport and alternatives, and passport-local -->

Passport requires us to install a couple more packages as dependencies.  [Bcrypt](https://www.npmjs.com/package/bcrypt) is a cryptography package used by Passport to *hash* users' passwords (we'll talk about that more below), and [express-session](https://github.com/expressjs/session), which will let us store encrypted cookies called *sessions* in a user's browser where we can store user data.

Let's install all of these packages by entering the following command into your terminal:
```
npm install passport passport-local bcrypt express-session --save
```

# Authentication



# Authorization
