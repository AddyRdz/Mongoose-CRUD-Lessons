# Express Router

## Lesson Objectives

1. Explain What Express.Router does for us
1. Create External Controller File for Routes
1. Move Server.js Routes to External Controller File
1. Require Mongoose in Controller File
1. Use Controller File in Server.js
1. Remove References to Base of Controller's URLs

## Explain What Express.Router does for us

- Our server.js file is getting rather bloated
- express.Router will let us put our routes in a separate file

## Create External Controller File for Routes

1. `mkdir controllers`
1. `touch controllers/authors.js`
1. Edit controllers/authors.js

```javascript
const express = require('express');
const router = express.Router();

module.exports = router;
```

## Move Server.js Routes to External Controller File

rename `app` to `router`

```javascript
const express = require("express");
const router = express.Router();

//NEW ROUTE
router.get("/authors/new", (req, res) => res.render("new"));

//INDEX ROUTE
router.get("/authors", (req, res) => {
  Author.find({}, (err, allAuthors) => {
    if (err) {
      res.send(err);
    } else {
      res.render("index", { authors: allAuthors });
    }
  });
});

//SHOW ROUTE
router.get("/authors/:id", (req, res) => {
  Author.findById(req.params.id, (err, foundAuthor) => {
    if (err) {
      res.send(err);
    } else {
      res.render("show", { author: foundAuthor });
    }
  });
});
//EDIT ROUTE
router.get("/authors/:id/edit", (req, res) => {
  Author.findById(req.params.id, (err, foundAuthor) => {
    if (err) {
      res.send(err);
    } else {
      res.render("edit", { author: foundAuthor });
    }
  });
});
//CREATE ROUTE
router.post("/authors", (req, res) => {
  Author.create(req.body, (err, createdAuthor)=>{
    if(err){
      res.send(err)
    }else {
      console.log(createdAuthor);
      res.redirect("/authors");
    }
  })
});

// UPDATE ROUTE
router.put("/authors/:id", (req, res) => {
  if (req.body.isActive == "on") {
    req.body.isActive = true;
  } else {
    req.body.isActive = false;
  }

  Author.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true },
    (err, updatedAuthor) => {
      if (err) {
        res.send(err);
      } else {
        console.log(updatedAuthor);
        res.redirect("/authors/" + updated._id);
      }
    }
  );
});

// DELETE ROUTE
router.delete("/authors/:id", (req, res) => {
  Author.findByIdAndRemove(req.params.id, (err, data) => {
    if(err){
      res.send(err)
    }else {
      res.redirect("/authors");
    }
  });
});

module.exports = router;

```

## Require Author Model in Controller File

```javascript
const express = require('express');
const router = express.Router();
const Author = require('../models/authors.js')
//...
```

The `Author` model is no longer needed in `server.js`.  Remove it:

```javascript
const express = require('express');
const app = express();
const mongoose = require('mongoose');
const methodOverride = require('method-override');
```

## Use Controller File in Server.js

```javascript
const authorsController = require('./controllers/authors.js');
app.use(authorsController);

```

## Remove References to Base of Controller's URLs

You can specify a route when a middleware runs

```javascript
const authorsController = require('./controllers/authors.js');
app.use('/authors', authorsController);
```

Since we've specified that the controller works with all urls starting with /authors, we can remove this from the controller file:

```javascript

const express = require("express");
const Author = require("../models/Author");
const router = express.Router();

//NEW ROUTE
router.get("/new", (req, res) => res.render("new"));

//INDEX ROUTE
router.get("/", (req, res) => {
  Author.find({}, (err, allAuthors) => {
    if (err) {
      res.send(err);
    } else {
      res.render("index", { authors: allAuthors });
    }
  });
});

//SHOW ROUTE
router.get("/:id", (req, res) => {
  Author.findById(req.params.id, (err, foundAuthor) => {
    if (err) {
      res.send(err);
    } else {
      res.render("show", { author: foundAuthor });
    }
  });
});
//EDIT ROUTE
router.get("/:id/edit", (req, res) => {
  Author.findById(req.params.id, (err, foundAuthor) => {
    if (err) {
      res.send(err);
    } else {
      res.render("edit", { author: foundAuthor });
    }
  });
});
//CREATE ROUTE
router.post("/", (req, res) => {
  Author.create(req.body, (err, createdAuthor)=>{
    if(err){
      res.send(err)
    }else {
      console.log(createdAuthor);
      res.redirect("/authors");
    }
  })
});

// UPDATE ROUTE
router.put("/:id", (req, res) => {
  if (req.body.isActive == "on") {
    req.body.isActive = true;
  } else {
    req.body.isActive = false;
  }

  Author.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true },
    (err, updatedAuthor) => {
      if (err) {
        res.send(err);
      } else {
        console.log(updatedAuthor);
        res.redirect("/authors/" + updated._id);
      }
    }
  );
});

// DELETE ROUTE
router.delete("/:id", (req, res) => {
  Author.findByIdAndRemove(req.params.id, (err, data) => {
    if(err){
      res.send(err)
    }else {
      res.redirect("/authors");
    }
  });
});

module.exports = router;
```
