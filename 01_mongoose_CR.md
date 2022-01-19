# CRUD App with Mongoose - Create and Read

## Lesson Objectives

1. Initialize an application
1. Start express
1. Create New Route
1. Create Create Route
1. Connect Express to Mongo
1. Create Authors Model
1. Have Create Route Create Data in MongoDB
1. Create Index Route
1. Have Index Route Render All Authors
1. Have Create Route redirect to Index After Author Creation
1. Create Show Route
1. Have Index Page Link to Show Route
1. Create show.ejs

## Initialize a directory

1. Create a directory for the app in `unit2/lessons` and `cd` into it
1. `touch server.js`
1. `npm init`
1. `npm install express`
1. Confirm package.json points to entrypoint file `"main": "server.js",`

## Start express

```javascript
const express = require("express")
const app = express()

app.listen(8000, () => {
  console.log("listening on port 8000")
})
```

## Create New Route

```javascript
app.get("/authors/new", (req, res) => {
  res.send("new")
})
```

1. `mkdir views`
1. `npm install ejs`
1. `touch views/new.ejs`
1. Create the view

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
  <body>
    <h1>Author: New Page </h1>
    <form action="/authors" method="POST">
        <label for="fullName">
            Name (full name):  <input name="fullName" type="text"> <br/>
        </label>
        <label for="origin">
            Country of Origin: <input name="origin" type="text"> 
        </label>
        <label for="isActive">
            Currently Active: <input type="checkbox" name="isActive">
        </label>
       
        <label for="isActive">
            Bio: <br> 
            <textarea name="bio">Add bio</textarea>
        </label>
        <input type="submit" value="Create Author">
    </form>
  </body>
</html>
```

Render the view

```javascript
app.get("/authors/new", (req, res) => {
  res.render("new.ejs")
})
```

## Create Create Route

```javascript
app.post("/authors/", (req, res) => {
  res.send("received")
})
```

1. Use express.urlencoded in server.js:

```javascript
app.use(express.urlencoded({ extended: true }))
```

Check to see if req.body is defined and data fields are populated:

```javascript
app.post("/authors/", (req, res) => {
  res.send(req.body)
})
```

## Connect Express to Mongo

1. `npm install mongoose`
1. Inside server.js:

```javascript
const mongoose = require("mongoose")

//... and then farther down the file
mongoose.connect("mongodb://127.0.0.1:27017/authors")

// Note: configuring mongoose connections in version 6+ no longer requires 'useNewUrlParser' option

mongoose.connection.once("open", () => {
  console.log("connected to mongo")
})

// ... additional db events can custom logging messages

mongoose.connection.on("connected", () => console.log("connection made"))
mongoose.connection.on("disconnected", () => console.log("connetion lost"))
```

## Create authors Model

1. `mkdir models`
1. `touch models/Author.js`
1. Create the Author schema

```javascript
const mongoose = require("mongoose")

const authorschema = new mongoose.Schema(
  {
    fullName: { type: String, required: true, default: "Anonymous" },
    origin: {
      type: String,
      required: true,
      default: "United States of America",
    },
    bio: { type: String },
    isActive: { type: Boolean, default: true },
  },
  { timestamps: true }
  //
)

const Author = mongoose.model("Author", authorschema)

module.exports = Author
```

## Have Create Route Create data in MongoDB

Inside server.js:

```javascript
const Author = require("./models/Author.js")
//... and then farther down the file
app.post("/authors/", (req, res) => {
  if (req.body.isActive === "on") {
    //if checked, req.body.isActive is set to 'on'
    req.body.isActive = true
  } else {
    //if not checked, req.body.isActive is undefined
    req.body.isActive = false
  }

  Author.create(req.body, (err, createdAuthor) => {
    if(err){
      res.send(err)
    }else {
      res.send(createdAuthor)
    }
  })

  /*
    An alternative syntax using .then() promises
    ============================================
    Author.create(req.body)
    .then(createdAuthor=>{
        console.log(createdAuthor)
        res.redirect('/authors')
    }).catch(err=>{
        res.send(err)
    })
  */

  /*
    An alternative syntax using async/await - note you have to set your callback to be 'async'
    ============================================

    app.post('/authors', async (req,res)=>{
      try {

      // parsing form data 
        
      if (req.body.isActive === "on") {
        //if checked, req.body.isActive is set to 'on'
        req.body.isActive = true
      } else {
        //if not checked, req.body.isActive is undefined
        req.body.isActive = false
      }
      // await tells JS that Author.create returns a promise when the promise is resolved, its value will be stored in the variable. 

      const createdAuthor = await Author.create(req.body)
      console.log(createdAuthor)

      res.redirect('/authors')

      }catch(err){
        // if an error is detected, the 'catch' will provide an error param with the associated express error
        res.send(err)
      }

    })
    
  */
})
```

## Create Index Route

```javascript
app.get("/authors", (req, res) => {
  res.send("index")
})
```

`touch views/index.ejs`

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
  <body>
    <h1>Authors index page</h1>
  </body>
</html>
```

Render the ejs file

```javascript
app.get("/authors", (req, res) => {
  res.render("index.ejs")
})
```

## Have Index Route Render All authors

```javascript
app.get("/authors", (req, res) => {
  Author.find({}, (err, allAuthors) => {
    if(err){
      res.send(err)
    }else {
      res.render("index.ejs", {
      authors: allAuthors,
      })
    }
  })
})
```

Update the ejs file:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
  <body>
    <h1>authors index page</h1>
    <ul>
      <% for(let i = 0 i < authors.length i++){ %>
      <li><%=authors[i].fullName %> - <%=authors[i].origin %></li>
      <% if(authors[i].isActive === true){ %> They are currently active <% }
      else { %> They are not currently active <% } %> <% } %>
    </ul>
  </body>
</html>
```

Add a link to the create page:

```html
<nav>
  <a href="/authors/new">Add a new author</a>
</nav>
```

## Have Create Route redirect to Index After Author Creation

Inside the create route

```javascript
Author.create(req.body, (error, createdAuthor) => {
  // ...
  res.redirect("/authors")
  // ...
})
```

## Have Index Page Link to Show Route

```html
<li>
  <a href="/authors/<%=authors[i]._id%>"><%=authors[i].fullName %></a> -
  <%=authors[i].origin %>
</li>
```

## Create Show Route

```javascript
app.get("/authors/:id", (req, res) => {
  Author.findById(req.params.id, (err, foundAuthor) => {
    if(err){
      res.send(err)
    }else {
      res.send(foundAuthor)
    }
  })
})
```

## Create show.ejs

1. `touch views/show.ejs`
1. Add HTML

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
  <body>
    <h1>authors show page</h1>
    <h2>
      <%=author.fullName %> |
      <span>
        <% if(author.isActive === true){ %>
        <em>Active</em>
        <% } else { %>
        <em>Inactive </em>
        <% } %>
      </span>
    </h2>
    <p>Country of Origin: <%=author.origin %></p>
    <p><%=author.bio %></p>

    <nav>
      <a href="/authors">Back to Authors Index</a>
    </nav>
  </body>
</html>
```

Render the ejs

```javascript
app.get("/authors/:id", (req, res) => {
  Author.findById(req.params.id, (err, foundAuthor) => {
    if(err){
      res.send(err)
    }else {
      res.render("show.ejs", {
      Author: foundAuthor
    })
    }
  })
})
```
