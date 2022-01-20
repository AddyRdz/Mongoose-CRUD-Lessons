![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

# Data Associations with Mongoose

## Overview: 
<!-- framing the "why" in big-picture/real world examples -->

- Real-world data usually consists of different types of things that are related to each other in some way. An invoicing app might need to track employees, customers, and accounts. A food ordering app needs to know about restaurants, menus, and its users!  

- We've seen that when data is very simple, we can combine it all into one model.  When data is more complex or more loosely related, we often create two or more related models.

- Understanding how to plan for, set up, and use related data will help us build more full-featured applications.

## Objectives:
<!-- specific/measurable goal for students to achieve -->
*After this lesson you will be able to:*

- Compare and contrast embedded & referenced data.
- Design nested server routes for associated resources.
- Build effective Mongoose queries for associated resources.

### Where should we be now?
<!-- call out the skills that are prerequisites -->
*Prerequisites:*

* Use Mongoose to code Schemas and Models for single resources.
* Create, Read, Update, and Delete data with Mongoose.


## Numerical Categories for Data Relationships

### One-to-One

Each person has one brain, and each (living human) brain belongs to one person.

![one to one erd example](https://cloud.githubusercontent.com/assets/3254910/18140904/4d85c04e-6f6c-11e6-8301-c06bacff3dd3.png)

One-to-one relationships can sometimes just be modeled with simple attributes. A person and a brain are both complex enough that we might want to have their data in different models, with lots of different attributes on each.


### One-to-Many

Each leaf "belongs to" the one tree it grew from, and each tree "has many" leaves.

![one to many erd example](https://cloud.githubusercontent.com/assets/3254910/18182445/e4bddb6c-7044-11e6-9099-314b773724f3.png)


### Many-to-Many

Each student "has many" classes they attend, and each class "has many" students.

![many to many erd example](https://cloud.githubusercontent.com/assets/3254910/18140903/4c56c3ee-6f6c-11e6-9b6d-4c6ffae81323.png)


#### Entity Relationship Diagrams

Entity relationship diagrams (ERDs) represent information about the numerical relationships between data, or entities.

![entity relationship diagram example](https://cloud.githubusercontent.com/assets/3254910/18141666/439d9392-6f6f-11e6-953f-c91415b85f3f.png)


Note: In the example above, all of the Item1, Item2, Item3 under each heading are standing in for attributes.

[More guidelines for ERDs](http://docs.oracle.com/cd/A87860_01/doc/java.817/a81358/05_dev1.htm)

#### Check for Understanding

How would you design the following? What properties would each model contain? Bonus: Draw an ERD for each set of related data? (see the above examples)

* `User`s with many `Tweets`?
* `Food`s with many `Ingredients`?


## Association Categories for Mongoose

There are two common methods for structuring data in mongoose: 

**Embedded Data** is directly nested *inside* of other data. Each record has a copy of the data.

_Pros_: Efficiency

_Usecase_: When you embed data you won't have to make multiple requests (database queries) to get all of the information you need. Everything you need is contained within the document.

<img src="https://i.imgur.com/aMG36rT.png" width="60%">

**Referenced Data** 
A document stores as an *id* that 'reference' other documents. The id can be used to look up the information. 

_Pros_: Consistency 

It is usually easier to keep referenced records *consistent* because the data is only stored in one place and only needs to be updated in one place.  

![image](https://cloud.githubusercontent.com/assets/6520345/21190300/2c091f08-c1d6-11e6-89ed-0459874edf3a.png)

[Source: MongoDB docs](https://docs.mongodb.com/v3.2/tutorial/model-referenced-one-to-many-relationships-between-documents/)

While the question of one-to-one, one-to-many, or  many-to-many is often determined by real-world characteristics of a relationship, the decision to embed or reference data is a design decision.  

There are tradeoffs, such as between *efficiency* and *consistency*, depending on which one you choose.  

When using Mongo and Mongoose, though, many-to-many relationships often involve referenced associations, while one-to-many often involve embedding data.


## Where to Define the Relationship

In the case of our authors and articles, we can define our relationship one of three ways:


### 0. Embedded Data - (Sub-documents ie nested articles or array of articles) :

This approach for creating unique authors that 'contains' or embeds articles sub-documents. This methodology might be useful for query efficiency - one query collects all of the data. However, updating and managing data consistency becomes more challenging. The process for working with subdocuments is outlined [here](/Sub-Docs.md) but is not covered in this workshop. 

### 1. An array of articles in our `Author` model:

```javascript
// in /models/Article.js

const articleSchema = mongoose.Schema({
    title: String,
    body: String,
    author_id: String,
}
// in /models/Author.js
const authorSchema = mongoose.Schema({
    fullName: String,
    origin: String,
    isActive: Boolean,
    bio: String, 
    articles: [{
        type: mongoose.Schema.Types.ObjectId,
		ref: 'Article'
    }]
});
```

So, an author's document in the db might look like this:

```javascript
{ 
    "_id" : ObjectId("5df7022a25072e3380c97249"),
    "name" : "James Patterson",
    "origin": "France",
    "isActive" true,
    "bio": "Lorem Ipsum Dolor",
    "articles": [ObjectId("5df7022a25072e3380c9723e"), ObjectId("5df7022a25072e3380c977d8"), ObjectId("5df7022a25072e3380c9709c")]
}
```

Which, when the `ObjectId`s are expanded using .populate() (more on this later), our query would look like this:

```javascript
{ 
    "_id" : ObjectId("5df7022a25072e3380c97249"),
    "name" : "James Patterson",
    "origin": "France",
    "isActive" true,
    "bio": "Lorem Ipsum Dolor",
    "articles": [
        {
            "_id" : ObjectId("5df7022a25072e3380c9723e"),
            "title" : "First article",
            "body": "Lorem ipsum"
        },
        {
            "_id" : ObjectId("5df7022a25072e3380c977d8"),
            "title" : "Second article",
            "body": "Lorem ipsum"
        },
        {
            "_id" : ObjectId("5df7022a25072e3380c9709c"),
            "title" : "Third article",
            "body": "Lorem ipsum"
        }
    ]
}
```

In this approach, we are saying that we want to store article ids in an array inside the `Author` model so that, whenever we query an author or all the authors, we can get all their articles along with it!

While this is beneficial in a Read query, the complexity is increased when we want to create an article (this assumes the author's id was part of the form data):

```html
<!--  Inside the article's new form:  -->
        <label>
            <select name="author_id">
                <%for(let i=0;i<authors.length;i++){ %>
                    <option value="<%=authors[i]._id%>"><%=authors[i].fullName%></option>
                <%}%>
            </select>
        </label>
<!-- When creating a new article, this selector sends the ObjectId of an existing author with the article's form data -->
```


```javascript
Author.findById(req.body.author_id, (err, foundAuthor) => {
    Article.create(req.body, (err, createdArticle) => {
        foundAuthor.articles.push(createdArticle);
        foundAuthor.save();
    })
    res.redirect('/articles');
})
```

Like our classes from Unit 1, `foundAuthor` is an instance of the `Author` class. So, we'd need to take the newly-created article and `.push` it into the `foundAuthor`'s `articles` property. Then, once that's done, we can then `.save` the `foundAuthor`, which effectively updates that author by mutating its `articles` array.

Additional complexity is added when deleting an article from an author's articles array. For more on this process, review the materials found in the [4b_one_to_many.md documentation](./4b_one_to_many.md)

### 2. An author id in each `Article` model.

```javascript
const articleSchema = mongoose.Schema({
	title: String,
	body: String,
	author: {
		type: mongoose.Schema.Types.ObjectId,
		ref: 'Author'
	}
});
```

Notice that the `author` property is an object, not an array. In this approach, each article would just have the author's id attached to it. So, an article document might look like this:

```javascript
{
    "_id" : ObjectId("5df7022a25072e3380c9723e"),
    "title" : "First article",
    "body": "Lorem ipsum",
    "author": ObjectId("5df7022a25072e3380c97249")
}
```

Again, when the `ObjectId` is expanded, the object would look like this:

```javascript
{
    "_id" : ObjectId("5df7022a25072e3380c9723e"),
    "title" : "First article",
    "body": "Lorem ipsum",
    "author": { 
        "_id" : ObjectId("5df7022a25072e3380c97249"),
        "name" : "James Patterson",
        "origin": "France",
        "isActive" true,
        "bio": "Lorem Ipsum Dolor"
    }
}
```

Now, when we create a new article, the query might look like this (again, assuming that the author's id is part of the form data):

```javascript
Article.create(req.body, (err, createdArticle) => {
    res.redirect('/articles');
})
```

So, the tradeoff is:

1. Store all the article ids in the `Author` model, which means that all the article documents would be a little lighter because they wouldn't have an `author/author_id`, or
2. Store the `author/author_id` in the `Article` model, which keeps the indefinitely growing `articles` array out of the `Author` model.

Choosing between these two usually comes down to this: In a one-to-many relationship, would the number of the **many** objects be limited? If so, put the array in the **one** model. In other words, option #1.

If, however, the number of **many** objects is potentially indefinite (your app logic might requiere numerous updates/deletions), then store the id of the **one** in each of the **many** models. In other words, option #2.

---
## Guide for associations using option [2]: - Many to One

This guide demonstrates the process for making associations between related documents. Rather than store an array of sub documents refs inside our Author (one-to-many), we will demonstrate how to work with associations with articles (many) storing a reference to the author(one). 

We'll start by updating the Article Schema, to accomodate storing an Author information.

In `./models/Article.js`
```js
const articleSchema = new mongoose.Schema({
    name: { type: String, required: true },
    content: String,
    author: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'Author'
    }
});
```

Here, we add an author property to the Article Schema, and define it as an **Object Id**. In Mongo DB, every record has a unique **Object Id** that identifies it in the collection. By storing the **Object Id** of the author we are storing a reference to the author.

Now when creating an article, it will look something like this.

```js
{
  name: 'Top 10 Best Recipes of 2021',
  content: 'If you are looking for tasty recipe you\'ve come to the right place...',
  author: ObjectId('611088372d0f1edf1f6fce78')
}
```

## Add Author Selection on Article New Page

Now when visiting our form to add a new Article, we want to be able to select which Author wrote that article.

In `./views/articles/new.ejs` we'll add an HTML select box, with an option for each author

First check out the documentation for the [HTML Select Box](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select) to get a sense for how it works.

```html
<form action="/articles" method="POST">
  
  <label for="author">Author: </label>

  <select name="author" id="author">
    <% for (let i = 0; i < allAuthors.length; i++) { %>
        <option value="<%= allAuthors[i]._id %>">
            <%= allAuthors[i].authorName %>
        </option>
    <% } %>
  </select>

  <div>
    <label for="name">Name of Article:</label>
    <input type="text" name="name">
  </div>

  <div>
    <label for="content">Article Content:</label>
    <textarea type="text" name="content"></textarea>
  </div>

  <button type="submit">Add Article</button>
</form>
```

Notice the name atribute of the select tag is set to author. This is the name of the property we will send back in the request body. The value of each option tag is set to the author id. This is the value we will send back for author.

Right now, this will break because we aren't passing in `allAuthors` to the `articlesNew.ejs` template. Let's pass the authors data to the `articlesNew.ejs` template.

## Pass Author Data to the Article New Template

In the articlesController, find the Article New route and add a query to get back all authors.

```js
router.get("/new", (req, res) => {
  db.Author.find({}, (err, allAuthors) => {
    // render the template here
  });
});
```

Now let's render the `articlesNew.ejs` template and pass it the `allAuthors` data.

```js
router.get("/new", (req, res) => {
  db.Author.find({}, (err, allAuthors) => {
    res.render("articles/new.ejs", { allAuthors: allAuthors });
  });
});
```

Now, when visiting the form to add a new Author, we should see a select box dropdown to pick from the list of our authors! If we don't have any authors, the dropdown will be empty. Let's add some authors, and then visit the Article New page to test our select box dropdown.

## Delete Our Old Data

We are going to be changing the structure of our data. The new articles that we add will now have an `author` field. For this reason it will be helpful to delete our existing data and start clean.

Create a file in the root of your project called `dropData.js`.

```bash
touch dropData.js
```

We will add two queries, one to delete all authors and one to delete all articles.

```js
const db = require('./models/index.js');

db.Author.deleteMany({}, (err) => {
  if (err) return res.send(err);

  db.Article.deleteMany({}, (err) => {
    if (err) return res.send(err);

    console.log('Deleted all authors and articles');

    process.exit();
  });
});
```

Notice the second query is found inside the callback to the first query. This is because we'll run the second query only after the first one has completed. Afterwards we log out a message to ourselves indicated the queries have completed and then we exit the node process with `process.exit()`.

We'll now run our delete queries.

```bash
node dropData.js
```

This process can also be run directly in mongosh:

```bash
> use authors
> db.dropDatabase()
> use articles
> db.dropDatabase()

```

## Test the Form and Add a New Article

We'll find the Article Create route and add a `console.log(createdArticle)` to inspect the new Article that gets created.

In `controllers/articlesController.js`
```js
router.post("/", (req, res) => {
  // console.log(req.body);
  Article.create(req.body, (err, createdArticle) => {
    if (err) return console.log(err);
    console.log(createdArticle);
    res.redirect('/articles');
  });
});
```

Let's return to the browser and visit our form to add a new article at `/articles/new`. Fill out the form, selecting an author, and submit. In the terminal we should now see the article object with the author property. Notice the author property is set to the Id of the author we selected.

## Display the Author on the Article Show Page

Now we are storing the author information in an article. When querying the database for a particular article, we will now get the author id as well. We'll be able use this Id to display the author's name on the Article Show Page for that article.

In the `articleShow.ejs` template, let's display the author information. Our article object now has an author property. Let's render it into our template under the article name.

In `./views/articles/show.ejs`
```html
<body>
  <h1>View One Article</h1>

  <h3><%= oneArticle.name %></h3>

  <p>Author: <%= oneArticle.author %></p>

  <p><%= oneArticle.content %></p>
</body>
```

Now when we visit the Article Show Page, we should see the a value inputed for author. However, at the moment we're just inserting the Author's **Object Id** to the page. Our eventual goal is to have the Author's name display.

## Populate the Article Data with Author Data

Currently we just have the author's Id being stored in the article. We are going to use [Mongoose's populate method](https://mongoosejs.com/docs/populate.html), to populate the `author` property with actual Author data.

In our articleController let's update our query from this
```js
Article.findById(req.params.id, (err, foundArticle) => {
    if (err) return res.send(err);

    console.log(foundArticle);

    res.render('articles/articlesShow.ejs', { oneArticle: foundArticle });
  });
```

to this

```js
Article.findById(req.params.id)
  .populate('author')
  .exec((err, foundArticle) => {
    if (err) return res.render(err);

    console.log(foundArticle);

    res.render('articles/articlesShow.ejs', { oneArticle: foundArticle });
  });
```

Mongoose allows us to call the `.populate()` method and chain it after our initial query. Here we specify that we'd like to populate the `author` field on the Article data that comes back from our query.

After the `populate()` method we chain the `exec()` method, which will allow us to specify a callback function that we want to run once we have recieved the data.

## Display the Author's Name on the Article Show Page

Now let's return to our browser, and make our way to the Article Show Page for a particular article. What do we see now?

![author show](./images/article-show.png)

We're inserting the entire author object onto the page. We're definitely making progress!

Our goal, however, is to display the author's name on the page. Let's update our template to pull the author's name from the author object.

In `./views/articles/articlesShow.ejs`
```html
<body>
  <h1>View One Article</h1>

  <h3><%= oneArticle.name %></h3>

  <p>Author: <%= oneArticle.author.authorName %></p>

  <p><%= oneArticle.content %></p>
</body>
```

Awesome! Now let's make the author's name a link that directs to the show page for that author.

```html
<body>
  <h1>View One Article</h1>

  <h3><%= oneArticle.name %></h3>

  <a href="/authors/<%= oneArticle.author._id %>">
    <p>Author: <%= oneArticle.author.authorName %></p>
  </a>

  <p><%= oneArticle.content %></p>
</body>
```

This anchor tag will direct to `/authors/:id` taking us to the show page for that author. Now visit the Article show page and test it out.

<br>

## Deleting an Author resource 

```javascript
Article.deleteMany({author:req.params.id}, (err,deletedArticles)=>{
    // Step 1: Delete all articles with an association
    // console.log(deletedArticles) - note: deleteMany returns a delete status message, not documents
    Author.findByIdAndRemove(req.params.id, (err, deletedAuthor) => {
      res.redirect("/authors");
    });
  })

```

## Alternative routing + creation of articles 

```js
// new authors-articles view
router.get("/:authorId/articles/new", (req, res) => {
  // render a template that implicitly passes the reference with the form (rather than with a dropdown)
  Author.findById(req.params.authorId, (err, foundAuthor)=>{
    if(err){
      res.send(err)
    }else {
      res.render('authors/newArticle.ejs', {author: foundAuthor})
    }
  })
});

```


```html
<!-- views/authors/newArticle.ejs -->

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>New Author Page </title>
</head>
<body>
    <h1>Articles: New Article for <%=author.fullName%> </h1>
    <form action="/authors/<%=author._id%>/articles" method="POST">
        <label for="title">
            Title:  <input name="title" type="text"> <br/>
        </label>
        <label for="body">
            Body: <br> 
            <textarea name="body">Add body</textarea>
            <input type="hidden" name="author" value="<%=author._id%>">
            <!-- the author is passed directly as a hidden field-->
            <!-- the formfield will not appear on the page (and wouldn't require a dropdown) -->

        <input type="submit" value="Create Article">
    </form>
</body>
</html>
```
### Route Design

Remember RESTful routing? It's the most popular modern convention for designing resource paths for nested data. Here is an example of an application that has routes for `Articles` and `Notes` models:

### RESTful Routing

| | | | |
|---|---|---|---|
| **HTTP Verb** | **Path** | **Description** | **Key Mongoose Method(s)** |
| GET | /articles | Get all articles | <details><summary>click for ideas</summary>`.find`</details> |
| POST | /articles | Create a article | <details><summary>click for ideas</summary>`.create`</details> |
| GET | /articles/:id | Get a article | <details><summary>click for ideas</summary>`findOne`,`.findById`</details> |
| DELETE | /articles/:id | Delete a article | <details><summary>click for ideas</summary>`.findOne`, `.remove`, `.findByIdAndRemove`</details> |
| GET | /articles/:article_id/comments | Get all comments from a article | <details><summary>click for ideas</summary>`.findOne`, (`.populate` if referenced)</details> |
| POST | /articles/:article_id/comments | Create a comment for a article | <details><summary>click for ideas</summary>`.findOne`, `new`, `.save`</details> |
| GET | /articles/:article_id/comments/:comment_id | Get a comment from a article | <details><summary>click for ideas</summary>`.findOne`</details> |
| DELETE | /articles/:article_id/comments/:comment_id | Delete a comment from a article | <details><summary>click for ideas</summary>`.findOne`, `.deleteMany`</details> |

*In routes, avoid nesting resources more than one level deep.*

