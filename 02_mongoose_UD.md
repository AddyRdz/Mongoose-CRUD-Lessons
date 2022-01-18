# CRUD App with Mongoose - Delete and Update

## Lesson Objectives

Delete (Destroy):

1. Create a Delete Button
1. Create a DELETE Route
1. Have the Delete Button send a DELETE request to the server
1. Make the DELETE Route Delete the Model from MongoDB

Edit (Update):

1. Create a link to the edit route
1. Create an edit route
1. Create an PUT route
1. Have the edit page send a PUT request
1. Make the PUT Route Update the Model in MongoDB
1. Make the PUT Route Redirect Back to the Index Page

## Create a Delete Button

In your index.ejs file

```html
<li>
  The <a href="/authors/<%=authors[i]._id%>"><%=authors[i].name %></a> is
  <%=authors[i].color %>. <% if(authors[i].isActive === true){ %> It is ready
  to eat <% } else { %> It is not ready to eat <% } %>
  <!--  ADD DELETE FORM HERE-->
  <form>
    <input type="submit" value="DELETE" />
  </form>
</li>
```

## Create a Delete Route

```javascript
app.delete("/authors/:id", (req, res) => {
  res.send("deleting...")
})
```

## Have the Delete Button send a DELETE request to the server

When we click "DELETE" on our index page (index.ejs), the form needs to make a DELETE request to our DELETE route.

The problem is that forms can't make DELETE requests. Only POST and GET. We can fake this, though. First we need to install an npm package called `method-override`

```
npm install method-override
```

Now, in our server.js file, add:

```javascript
//include the method-override package
const methodOverride = require("method-override")
//...
//after app has been defined
//use methodOverride.  We'll be adding a query parameter to our delete form named _method
app.use(methodOverride("_method"))
```

Now go back and set up our delete form to send a DELETE request to the appropriate route

```html
<form action="/authors/<%=authors[i]._id%>?_method=DELETE" method="POST"></form>
```

## Make the Delete Route Delete the Model from MongoDB

Also, have it redirect back to the authors index page when deletion is complete

```javascript
app.delete("/authors/:id", (req, res) => {
  Author.findByIdAndRemove(req.params.id, (err, data) => {
    res.redirect("/authors") //redirect back to authors index
  })
})
```

## Create a link to an edit route
In your `index.ejs` file:

```html
<a href="/authors/<%=authors[i]._id%>/edit">Edit</a>
```

## Create an edit route/page

First the route:

```javascript
app.get("/authors/:id/edit", (req, res) => {
  Author.findById(req.params.id, (err, foundAuthor) => {
    //find the author
    if (err) {
      res.send(err)
    } else {
      res.render("edit.ejs", {
        author: foundAuthor, //pass found author to template
      })
    }
  })
})
```

Now the EJS:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title></title>
  </head>
  <body>
    <h1>Author: Edit Page</h1>
    <form action="/authors" method="POST">
      <label for="fullName">
        Name (full name):
        <input name="fullName" type="text" value="<%=author.fullName%>" />
        <br />
      </label>
      <label for="origin">
        Country of Origin:
        <input name="origin" type="text" value="<%=author.origin%>" />
      </label>
      <label for="isActive">
        Currently Active:
        <input type="checkbox" name="isActive" <% if(author.isActive) {% />
        checked <%}%> >
      </label>
      <label for="bio">
        Bio: <br />
        <textarea name="bio"><%=author.origin%></textarea>
      </label>
      <input type="submit" value="Edit Author" />
    </form>
  </body>
</html>
```

## Create an PUT route

```javascript
app.put("/authors/:id", (req, res) => {
  if (req.body.isActive === "on") {
    req.body.isActive = true
  } else {
    req.body.isActive = false
  }
  res.send(req.body)
})
```

## Have the edit page send a PUT request

In the `edit.ejs`

```html
<form action="/authors/<%=author.id%>?_method=PUT" method="POST"></form>
```

## Make the PUT Route Update the Model in MongoDB

```javascript
app.put("/authors/:id", (req, res) => {
  if (req.body.isActive === "on") {
    req.body.isActive = true
  } else {
    req.body.isActive = false
  }
  author.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true },
    (err, updatedModel) => {
        if(err){
            res.send(err)
        }else {
            res.send(updatedModel)
        }
      
    }
  )
})
```

## Make the PUT Route Redirect Back to the Index Page

```javascript
author.findByIdAndUpdate(req.params.id, req.body, (err, updatedModel) => {
  res.redirect("/authors")
})
```
