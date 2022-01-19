![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

# Data Associations with Mongoose (DRAFT)

### Why is this important?
<!-- framing the "why" in big-picture/real world examples -->
*This workshop is important because:*

- Real-world data usually consists of different types of things that are related to each other in some way. An invoicing app might need to track employees, customers, and accounts. A food ordering app needs to know about restaurants, menus, and its users!  

- We've seen that when data is very simple, we can combine it all into one model.  When data is more complex or more loosely related, we often create two or more related models.

- Understanding how to plan for, set up, and use related data will help us build more full-featured applications.

### What are the objectives?
<!-- specific/measurable goal for students to achieve -->
*After this workshop, developers will be able to:*

- Compare and contrast embedded & referenced data.
- Design nested server routes for associated resources.
- Build effective Mongoose queries for associated resources.

### Where should we be now?
<!-- call out the skills that are prerequisites -->
*Before this workshop, developers should already be able to:*

* Use Mongoose to code Schemas and Models for single resources.
* Create, Read, Update, and Delete data with Mongoose.


### Numerical Categories for Relationships

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

Come up with an example of related data.  Draw the ERD for your relationship, including a few attributes for each model.

### Association Categories for Mongoose

**Embedded Data** is directly nested *inside* of other data. Each record has a copy of the data.

It is often *efficient* to embed data because you don't have to make a separate request or a separate database query -- the first request or query gets you all the information you need.  


<img src="https://i.imgur.com/aMG36rT.png" width="60%">


**Referenced Data** is stored as an *id* inside other data. The id can be used to look up the information. All records that reference the same data look up the same copy.

It is usually easier to keep referenced records *consistent* because the data is only stored in one place and only needs to be updated in one place.  

![image](https://cloud.githubusercontent.com/assets/6520345/21190300/2c091f08-c1d6-11e6-89ed-0459874edf3a.png)

[Source: MongoDB docs](https://docs.mongodb.com/v3.2/tutorial/model-referenced-one-to-many-relationships-between-documents/)


While the question of one-to-one, one-to-many, or  many-to-many is often determined by real-world characteristics of a relationship, the decision to embed or reference data is a design decision.  

There are tradeoffs, such as between *efficiency* and *consistency*, depending on which one you choose.  

When using Mongo and Mongoose, though, many-to-many relationships often involve referenced associations, while one-to-many often involve embedding data.

#### Check for Understanding

How would you design the following? Draw an ERD for each set of related data? Can you draw an ERD for each?

* `User`s with many `Tweets`?
* `Food`s with many `Ingredients`?


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

Additional complexity is added when deleting an article from an author's articles array:
- 1. Query the Author and update the document - (and their associated articles array) using the MongoDB [pull method](https://mongoosejs.com/docs/api/array.html#mongoosearray_MongooseArray-pull)
OR
- 2. Pass the article ObjectId to the request - locate the Author by their id and remove (splice) the designated article from the articles array. Save the updated documented.  

This process is compounded for update routes as well. Each process might take multiple read / write operations.

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

## CodeAlong: Relationships By Reference in Mongoose
Right now, our Article model looks like this:

```javascript
const articleSchema = mongoose.Schema({
	title: String,
	body: String,
	author_id: String
});
```

This kind of relationship will work just fine. But, let's use the `ObjectId` so that we can reference one model (e.g., Author) inside another model (e.g., Article).

## Update Articles Model

Just like our example above, let's update our Article model in `models/article.js`:

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

Notice that the `type` is an `ObjectId` and that, with this `ObjectId`, we are referencing (`ref`) the `Author` model.

Also, note that we updated `author_id` to `author` since we are now referring to the full author object instead of just the author's id.

## Updating the Articles Show Route with a populate()

```javascript
router.get('/:id', async (req, res) => {
    Article.findById(req.params.id).populate('author').exec((err, foundArticle) => {
        res.render('articles/show.ejs', {
            article: foundArticle
        });
    })
});
```

## Update the Articles Show Page

Now that our author object is part of the article object, we need to reference the author differently in our show page:

```html
<!-- Inside the <head> tag -->
<title><%= article.title %> by <%= article.author.fullName %></title>
<!-- Further down -->
<h3>Author: <%= article.author.fullName %></h3>
```

## Update the Articles New and Edit Pages

Because we changed our property from `author_id` to `author`, we need to adjust our new and edit forms:

```html
<select name="author">
```

And, just in our edit form, we need to adjust the boolean expression to find the associated author:

```html
<% if(authors[i]._id.toString() === article.author._id.toString()) { %>
```

### Route Design

Remember RESTful routing? It's the most popular modern convention for designing resource paths for nested data. Here is an example of an application that has routes for `Articles` and `Notes` models:

### RESTful Routing

| | | | |
|---|---|---|---|
| **HTTP Verb** | **Path** | **Description** | **Key Mongoose Method(s)** |
| GET | /articles | Get all articles | <details><summary>click for ideas</summary>`.find`</details> |
| POST | /articles | Create a article | <details><summary>click for ideas</summary>`new`, `.save`</details> |
| GET | /articles/:id | Get a article | <details><summary>click for ideas</summary>`.findOne`</details> |
| DELETE | /articles/:id | Delete a article | <details><summary>click for ideas</summary>`.findOne`, `.remove`, `.findOneAndRemove`</details> |
| GET | /articles/:article_id/comments | Get all comments from a article | <details><summary>click for ideas</summary>`.findOne`, (`.populate` if referenced)</details> |
| POST | /articles/:article_id/comments | Create a comment for a article | <details><summary>click for ideas</summary>`.findOne`, `new`, `.save`</details> |
| GET | /articles/:article_id/comments/:comment_id | Get a comment from a article | <details><summary>click for ideas</summary>`.findOne`</details> |
| DELETE | /articles/:article_id/comments/:comment_id | Delete a comment from a article | <details><summary>click for ideas</summary>`.findOne`, `.remove`</details> |

*In routes, avoid nesting resources more than one level deep.*
