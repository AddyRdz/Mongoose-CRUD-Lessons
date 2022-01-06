# Authentication

## Lesson Objectives

1. Explain what `bcrypt` does
1. Include `bcrypt` package
1. Hash a string using `bcrypt`
1. Compare a string to a hashed value to see if they are the same
1. Using `bcrypt` in login and register routes
1. Adding the users controller to server.js
1. Adding registration and login forms to the home page

## Explain what bcrypt does

`bcrypt` is a package that will encrypt passwords so that if your database gets hacked, people's passwords won't be exposed.

## Getting started

1. Navigate into `../bcrypt-sessions`.
2. `touch models/user.js controllers/users.js`
3. In `models/user.js`:

```javascript
const mongoose = require("mongoose");

const userSchema = mongoose.Schema({
  username: { type: String, required: true, unique: true },
  email: { type: String, required: true },
  password: { type: String, required: true },
});

const User = mongoose.model("User", userSchema);

module.exports = User;
```

4. In `controllers/users.js`:

```javascript
const express = require("express");
const router = express.Router();
const User = require("../models/user");

// Login Create route
router.post("/login", (req, res) => {
  // To be filled in later
});

// Register Create route
router.post("/registration", (req, res) => {
  // To be filled in later
});

module.exports = router;
```

## Include bcrypt package

Install `bcrypt`:

```
npm i bcrypt
```

Require the dependency in `controllers/users.js`:

```javascript
const bcrypt = require("bcrypt");
```

## Hash a string using bcrypt

`bcrypt` handles both hashing a string and salting a string. **Hashing** means, taking an input of indefinite length and converting it to random characters of a defined length. In other words, whether a user's password is "password123" or just "hi", it would convert both of these into, for example, a string of 60 characters.

**Salting** is adding extra characters to the password before it's hashed. `bcrypt` generates a random salt for each password, making it more difficult to know what that salt is. If this salt is not created for each password, the same string will get hashed to the same value each time. If there was one salt used for each password in the database, then a hacker could take a list of potential passwords, add that salt to each of the passwords in their list, and then compare the hashed/salted passwords to those in the hacked database until they found matches.

So, instead of using one predefined salt, we can use an algorithm from `bcrypt` to create that salt each time a user registers:

```javascript
const salt = bcrypt.genSaltSync(10);
const hashedPassword = bcrypt.hashSync("yourPassword", salt);
```

Now we can stored the `hashedPassword` in the database instead of an unhashed string.

## Compare a string to a hashed value to see if they are the same

Because the same string gets encrypted differently every time, we have no way of actually seeing what the value of the string is, which is a good thing. But, when that user comes back to our app and logs in, we can compare their login password to the hashed password in our database that was created when they registered. The salt is built into the hashed password, which means that the algorithm from `bcrypt` knows how to compare that hashed password to the login password, confirming whether or not they are mathematically equivalent:

```javascript
bcrypt.compareSync("yourGuessHere", hashedString);
// returns true or false
```

A good resource on `bcrypt` by Eric Lewis: [All about bcrypt](https://all-about-bcrypt.glitch.me/)

## Using bcrypt in login and register routes

Now we need to take what we know about hashing and salting and implement them into our user login and register routes.

### Register route

In `controllers/users.js`:

```javascript
router.post("/registration", (req, res) => {
  const passwordHash = bcrypt.hashSync(
    req.body.password,
    bcrypt.genSaltSync(10)
  );

  const userDbEntry = {
    username: req.body.username,
    password: passwordHash,
    email: req.body.email,
  };

  User.create(userDbEntry, (err, createdUser) => {
    if (err) {
      res.send(err);
    } else {
      req.session.currentUser = createdUser;
      res.redirect("/authors");
    }
  });
});
```

### Login route

Unlike our register route, we need to do comparison instead of hashing.

In `controllers/users.js`:

```javascript
router.post("/login", (req, res) => {
  // Because the username in our User model is unique, we can
  // use it to find the user

  User.findOne({ username: req.body.username }, (err, foundUser) => {
    if (err) {
      res.send(err);
    } else {
      if (!foundUser) {
        // If no user was found with that username, send them back to the
        // home page where they can try to log in with the correct username
        // or register in case that's the reason that that username was
        // not found in the database
        res.redirect("/");

      } else {
        // If a user was actually found, we can move on to the next step of
        // comparing passwords from the login form and from the foundUser in
        // the database

        if (bcrypt.compareSync(req.body.password, foundUser.password)) {
          // When we compare the password from the login form to the
          // password in the database, if it's true, redirect them to the authors index page
          console.log('logged in>>', foundUser._id)
          res.redirect("/authors");
        } else {
          // If the password from the login form and the password from
          // the database do not match, send them back to the home page
          // where they can attempt to log in again
          res.redirect("/")
        }
      }
    }
  });
});

```

## Adding the users controller to server.js

In `server.js` (below the other controllers):

```javascript
const usersController = require("./controllers/users");
app.use("/auth", usersController);
```

## Adding registration and login forms to the home page

Now we can create forms on the home index page to allow users to login or register.

In `views/index.ejs`:

```html
<h4>Login</h4>
<form action="/auth/login" method="POST">
  <input type="text" name="username" placeholder="username" /><br />
  <input type="password" name="password" placeholder="password" /><br />
  <button type="submit">Login</button>
</form>

<h4>Registration</h4>
<form action="/auth/registration" method="POST">
  <input type="text" name="username" placeholder="username" /><br />
  <input type="email" name="email" placeholder="email" /><br />
  <input type="password" name="password" placeholder="password" /><br />
  <button type="submit">Register</button>
</form>
```

## Lets Add Sessions to Our Blog App

- on same level as package.json
- `npm install express-session`
- configure express session

in `server.js`

in dependencies

```js
const session = require("express-session");
```

in middleware section

```js
app.use(
  session({
    secret: process.env.SECRET, //a random string do not copy this value or your stuff will get hacked
    resave: false, // default more info: https://www.npmjs.com/package/express-session#resave
    saveUninitialized: false, // default  more info: https://www.npmjs.com/package/express-session#resave
  })
);
```

in `.env` add

```yml
SECRET=FeedMeSeymour
```

- We only need the routes for our user
- the new form to log in
- the post route to create a new session
- the delete route to destroy a session

### Sessions Controller

in `server.js`

in `controllers/users.js`

```js
const bcrypt = require("bcrypt");
const express = require("express");
const sessions = express.Router();
const User = require("../models/users.js");

router.post("/login", (req, res) => {
  // Because the username in our User model is unique, we can
  // use it to find the user

  User.findOne({ username: req.body.username }, (err, foundUser) => {
    if (err) {
      res.send(err);
    } else {
      if (!foundUser) {
        // If no user was found with that username, send them back to the
        // home page where they can try to log in with the correct username
        // or register in case that's the reason that that username was
        // not found in the database

        res.redirect("/");
      } else {
        // If a user was actually found, we can move on to the next step of
        // comparing passwords from the login form and from the foundUser in
        // the database

        if (bcrypt.compareSync(req.body.password, foundUser.password)) {
          req.session.currentUser = foundUser;
          // When we compare the password from the login form to the
          // password in the database, if it's true, set the currentUser for the session
          // and redirect them to the authors index page
          res.redirect("/authors");
        } else {
          // If the password from the login form and the password from
          // the database do not match, send them back to the home page
          // where they can attempt to log in again
          res.redirect("/");
        }
      }
    }
  });
});

router.post("/registration", (req, res) => {
  const passwordHash = bcrypt.hashSync(
    req.body.password,
    bcrypt.genSaltSync(10)
  );

  const userDbEntry = {
    username: req.body.username,
    password: passwordHash,
    email: req.body.email,
  };
  User.create(userDbEntry, (err, createdUser) => {
    if (err) {
      res.send(err);
    } else {
      req.session.currentUser = createdUser;
      res.redirect("/authors");
    }
  });
});

router.delete("/logout", (req, res) => {
  req.session.destroy(() => {
    res.redirect("/");
  });
});

module.exports = sessions;
```

## Refactoring

- in every get route let's give access to the user

add the following to

```js
currentUser: req.session.currentUser;
```

Let's add a conditional navigation menu on the home page

```html
<ul>
  <li><a href="/article/new">Create a new article</a></li>
  <% if (currentUser) { %>
  <li>Welcome <%= currentUser.username %></li>
  <li>
    <form action="/logout?_method=DELETE" method="POST">
      <input type="submit" value="Log Out" />
    </form>
  </li>
  <% } else { %>
  <li><a href="/auth/registration">Sign Up</a></li>
  <li><a href="/auth/login">Log In</a></li>
  <% } %>
</ul>
```

# Using Middleware to Authenticate Routes

- Let's say we only want logged in users to be able to see the details of our Article.

We can write some good old JS logic. If you are not logged in, you'll be redirected to the log in page. Otherwise you can access the show page.

```js
router.get("/:id", (req, res) => {
  if (req.session.currentUser) {
    Article.findById(req.params.id, (error, foundArticle) => {
      res.render("article/show.ejs", {
        article: foundArticle,
        currentUser: req.session.currentUser,
      });
    });
  } else {
    res.redirect("/");
  }
});
```

It would be annoying to write this logic for every route. We can, write some custom middleware to handle this for us

```js
const isAuthenticated = (req, res, next) => {
  if (req.session.currentUser) {
    return next();
  } else {
    res.redirect("/");
  }
};
```

you can now prevent users who are not logged in from using the put and delete routes

```js
router.put('/:id', isAuthenticated, (req, res) =>
```

Bonus
You could also use `.use` to run this middleware above a series of routes. Use the documentation to figure out how!
