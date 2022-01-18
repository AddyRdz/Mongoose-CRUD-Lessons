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
    The <a href="/authors/<%=authors[i]._id%>"><%=authors[i].name; %></a> is  <%=authors[i].color; %>.
    <% if(authors[i].isActive === true){ %>
        It is ready to eat
    <% } else { %>
        It is not ready to eat
    <% } %>
    <!--  ADD DELETE FORM HERE-->
    <form>
        <input type="submit" value="DELETE"/>
    </form>
</li>
```

## Create a Delete Route

```javascript
app.delete('/authors/:id', (req, res)=>{
    res.send('deleting...');
});
```

## Have the Delete Button send a DELETE request to the server

When we click "DELETE" on our index page (index.ejs), the form needs to make a DELETE request to our DELETE route.

The problem is that forms can't make DELETE requests.  Only POST and GET.  We can fake this, though.  First we need to install an npm package called `method-override`

```
npm install method-override
```

Now, in our server.js file, add:

```javascript
//include the method-override package
const methodOverride = require('method-override');
//...
//after app has been defined
//use methodOverride.  We'll be adding a query parameter to our delete form named _method
app.use(methodOverride('_method'));
```

Now go back and set up our delete form to send a DELETE request to the appropriate route

```html
<form action="/authors/<%=authors[i]._id%>?_method=DELETE" method="POST">
```

## Make the Delete Route Delete the Model from MongoDB

Also, have it redirect back to the authors index page when deletion is complete

```javascript
app.delete('/authors/:id', (req, res)=>{
    Fruit.findByIdAndRemove(req.params.id, (err, data)=>{
        res.redirect('/authors');//redirect back to authors index
    });
});
```

## Create a link to an edit route

In your `index.ejs` file:

```html
<a href="/authors/<%=authors[i]._id; %>/edit">Edit</a>
```

## Create an edit route/page

First the route:

```javascript
app.get('/authors/:id/edit', (req, res)=>{
    Fruit.findById(req.params.id, (err, foundFruit)=>{ //find the fruit
        res.render(
    		'edit.ejs',
    		{
    			fruit: foundFruit //pass in found fruit
    		}
    	);
    });
});
```

Now the EJS:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title></title>
    </head>
    <body>
        <h1>Edit Fruit Page</h1>
        <form>
    		<!--  NOTE: the form is pre-populated with values for the server-->
    		Name: <input type="text" name="name" value="<%=fruit.name%>"/><br/>
    		Color: <input type="text" name="color" value="<%=fruit.color%>"/><br/>
    		Is Ready To Eat:
            <input type="checkbox" name="isActive"
                <% if(fruit.isActive === true){ %>
                    checked
                <% } %>
            />
            <br/>
            <input type="submit" name="" value="Submit Changes"/>
        </form>
    </body>
</html>
```

## Create an PUT route

```javascript
app.put('/authors/:id', (req, res)=>{
    if(req.body.isActive === 'on'){
        req.body.isActive = true;
    } else {
        req.body.isActive = false;
    }
    res.send(req.body);
});
```

## Have the edit page send a PUT request

In the `edit.ejs`

```html
<form action="/authors/<%=fruit.id%>?_method=PUT" method="POST">
```

## Make the PUT Route Update the Model in MongoDB

```javascript
app.put('/authors/:id', (req, res)=>{
    if(req.body.isActive === 'on'){
        req.body.isActive = true;
    } else {
        req.body.isActive = false;
    }
    Fruit.findByIdAndUpdate(req.params.id, req.body, {new:true}, (err, updatedModel)=>{
        res.send(updatedModel);
    });
});
```

## Make the PUT Route Redirect Back to the Index Page

```javascript
Fruit.findByIdAndUpdate(req.params.id, req.body, (err, updatedModel)=>{
    res.redirect('/authors');
});
```
