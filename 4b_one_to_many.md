# Creating A Relationship Between Two Models

## Resource Objectives

1. Learn about a strategy for storing model references in an array
2. Demonstrate creating, reading, updating, and deleting document references stored in an array

### One-to-Many - Review

In our lesson we used the example of a leaf to tree as an example of one to many data relations. Because each leaf "belongs to" the one tree it grew from, and each tree "has many" leaves.

![one to many erd example](https://cloud.githubusercontent.com/assets/3254910/18182445/e4bddb6c-7044-11e6-9099-314b773724f3.png)




---
# Show Articles on the Author Show Page

Wouldn't it be nice if we could visit the Author Show Page and view a list of all the articles the author has written?

## Add articles to the Author Schema

Our goal is to store article information in the author object. This way, when we request the data for a particular Author, we also have the data for the articles they have written as well.

We'll start by adding articles as a property on the Author schema.

```js
const authorSchema = new mongoose.Schema({
  authorName: { type: String, required: true },
  articles: [
    {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'Article'
    }
  ]
});
```


Here we add an articles property and define it as an array of **Object Ids**. In the author object we will store an array of Ids, each Id corresponding to an article belonging to the author. By storing an array of **Object Ids** we are storing an array of references to those articles.

An Author object will now look something like this.

```js
const author1 = {
  authorName: 'Frank Herbert',
  articles: [
    '6110940b6b12a8e5e6c714b7',
    '611093f66b12a8e5e6c714b3',
    '611091f213fabee565f1ef8a'
  ]
}
```

## Delete Existing Data

We are going to be changing the structure of our data agian. Run the `dropData.js` file to delete your existing data.

```bash
node dropData.js
```

## Add Article to Author When Creating an Article

Let's make our way to the articlesController, and then to the Article Create Route.

After creating the Article, we want to add the Article's id to the Author that created it.

```js
router.post('/', (req, res) => {
  // Create the new article with the data from the form
  db.Article.create(req.body, (err, newArticle) => {
    if (err) return console.log(err);
    
    // Find the author corresponding the new article, and add the article to their article array.
    db.Author.findByIdAndUpdate(
      newArticle.author,
      { $push: { articles: newArticle } },
      (err, updatedAuthor) => {
        if (err) return console.log(err);

        res.redirect('/articles');
      }
    );
  });
});
```

Here, we will use the `.findByIdAndUpdate()` method to find the correct author, and `push` the new article into the articles array for that author.

If Frank Herbert created the article we would push the article id to their articles array.

```js
const author1 = {
  authorName: 'Frank Herbert',
  articles: [ '6110940b6b12a8e5e6c714b7' ] // New Article Id pushed into this array.
}
```

Once complete we redirect back to the Articles Index Route.

Notice that we've placed the second query to update the author inside the callback function for the first query to create the article. This insures that the second query will only fire after the first one completes.

## Populate the Author Data with Articles

Let's make our way to the authorsController and then to the Authors Show Route.

Inside the callback to our `.findById()` query let's log out the foundAuthor with `console.log(foundAuthor)` and inspect what this author data looks like.

In controllers/authorsController.js
```js
router.get('/:id', (req, res) => {
  db.Author.findById(req.params.id, (err, foundAuthor) => {
    if (err) return console.log(err);

    console.log(foundAuthor);

    res.render('authors/authorsShow.ejs', {
      author: foundAuthor
    });
  });
});
```

Visit the Author Show Page in the browser and you should see your log message in the terminal.

![author obj](./images/author-obj.png)

An Author is an object with an `authorName` property, and `_id` property, and an `articles` property, set to an array of **Object Ids**.

Our goal is to populate this `articles` array with actual article data.

We'll update our Author Show Route from this
```js
router.get('/:id', (req, res) => {
  db.Author.findById(req.params.id, (err, foundAuthor) => {
    if (err) return console.log(err);

    console.log(foundAuthor)

    res.render('authors/authorsShow.ejs', {
      author: foundAuthor
    });
  });
});
```

to this
```js
router.get('/:id', (req, res) => {
  db.Author.findById(req.params.id)
    .populate('articles')
    .exec((err, foundAuthor) => {
      if (err) return console.log(err);

      console.log(foundAuthor);

      res.render('authors/authorsShow.ejs', {
        author: foundAuthor 
      });  
    });
});
```

Mongoose allows us to call the `.populate()` method and chain it after our initial query. Here we specify that we'd like to populate the `author` field on the Article data that comes back from our query.

After the `populate()` method we chain the `exec()` method, which will allow us to specify a callback function that we want to run once we have recieved the data.

Try visiting the Author Show Page again, and inspect the `foundAuthor` being logged out in the terminal. We should now see something much more interesting.

An Author is still an object with an `authorName`, `_id`, and an `articles` property. This time the `articles` property is set to an array of article objects. Each **Object Id** in the `articles` array was replaced with the actual article data for that article.

## Pass Article Data to the Author Show Template

Our goal is to see a list of an Author's articles on that Author's Show Page. Luckily we are alrady passing the author data to the template and the author data now has the article data imbedded within it.

Let's make our way to the `authorShow.ejs` template. We want to loop through the array of articles to display a list of articles.

Under where we display the Author's name, we'll include an `<ul>` and use a loop to display a `<li>` for each article.

In `./views/authors/authorsShow.ejs`
```html
...
<section>
  <ul>
    <li>Name: <%= author.authorName %></li>
  </ul>
</section>

<!-- Display a list of the author's articles -->
<ul>
  <% for (let i = 0; i < author.articles.length; i++) { %>
    <li><%= author.articles[i].name %></li>
  <% } %>
</ul>
...
```

Excellent! Let's test it out in the browser by visiting the Author Show Page for a particular Author.

...

We'll now update our code so that each article name links to the Show Page for that Article.

In `./views/authors/authorsShow.ejs`
```html
...
<!-- Display a list of the author's articles -->
<ul>
  <% for (let i = 0; i < author.articles.length; i++) { %>
    <li>
      <a href="/articles/<%= author.articles[i]._id %>">
        <%= author.articles[i].name %>
      </a>
    </li>
  <% } %>
</ul>
...
```

We'll now test that out in the browser. Visit the Author Show Page for an author and test your link. You should now be able to bounce back and forth between an Article Show Page and the Author Show Page! Amazing!

You made it! We've implemented the core features we set out to add!

---
# Hungry For More: Handling Delete

If you want more of a challenge check this out!

## Handling Delete of an Author

What happens when we delete an author? Wouldn't we want to delete all of the Articles that Author wrote? Let's see how we would do that.

In `controllers/authorsController.js`
```js
...
router.delete('/:id', (req, res) => {
  // Query DB to delete record by ID
  db.Author.findByIdAndDelete(req.params.id, (err, deletedAuthor) => {
    if (err) return console.log(err);

    db.Article.deleteMany(
      { _id: { $in: deletedAuthor.articles } },
      (err, result) => {
        if (err) return console.log(err);

        res.redirect('/authors');
      });
  });
});
...
```
This `.deleteMany()` query delete all Articles that have an `_id` that matches any found in the deleted Author.

## Handling Delete of an Article

When deleting an Article, we'll want to delete the **Object Id** reference of the Article for the corresponding Author.

In `controllers/articlesController.js`
```js
...
router.delete('/:id', (req, res) => {
  db.Article.findByIdAndDelete(req.params.id, (err, deletedArticle) => {
    if (err) return console.log(err);

    db.Author.findByIdAndUpdate(
      deletedArticle.author,
      { $pull: { articles: deletedArticle }},
      { new: true },
      (err, updatedAuthor) => {
        if (err) return console.log(err);

        res.redirect('/articles');
      }
    )
  });
});
...
```
This `.findByIdAndUpdate()` query will remove the **Object Id** reference of the deleted article from the Author.
