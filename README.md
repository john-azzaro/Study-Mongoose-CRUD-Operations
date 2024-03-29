# Mongoose CRUD Operatations Study
> *IMPORTANT: This study is a continuation of the Mongoose Configuration and Data Modeling Study. If you want to find out more about configuration of a Mongoose server, build schemas, models, virtuals, and instance methods, click [here](https://github.com/john-azzaro/Study-Mongoose-Configuration-and-Data-Modeling "Link to Mongoose Study").*

<br>

## What is the Mongoose CRUD Operations Study
The Mongoose CRUD Operations study is an examination of built-in methods that Create, Read, Update, and Delete information from the MongoDB database. The following outline follows the book and bookSchema project from the previous study, but also take a look at the restaurant prototype project in the repo.

Here's some questions covered in the study:

* [What are CRUD operations for Mongoose?](#What-are-CRUD-operations-for-Mongoose)
* [How do you CREATE a document using Mongoose?](#How-do-you-CREATE-a-document-using-Mongoose)
* [How do you FIND documents using Mongoose?](#How-do-you-FIND-documents-using-Mongoose)
* [How do you FIND BY ID using Mongoose?](#How-do-you-FIND-BY-ID-using-Mongoose)
* [How do you FIND BY VALUE using Mongoose?](#How-do-you-FIND-BY-VALUE-using-Mongoose)
* [How do you UPDATE a document using Mongoose?](#How-do-you-UPDATE-a-document-using-Mongoose)
* [How do you DELETE a document using Mongoose?](#How-do-you-DELETE-a-document-using-Mongoose)
* [How do you handle non-existent endpoints?](#How-do-you-handle-non-existent-endpoints)


<br>

## What are CRUD operations for Mongoose?
<dl>
<dd>

CRUD operations are common HTTP methods for communicating with a given server. CRUD operations allow you to make queries in the server's JavaScript code using Mongooseand can be peformed anywhere in your code. For example, the *read* CRUD operation refers to when the client request a document and send a *GET* request. 

In the case of this study, Mongoose has a few ways to *get* the document (stored on MongoDB). This applies to *create* (i.e. POST), *update* (i.e. PUT), and *delete* (i.e. DELETE) as well.

</dd>
</dl>

<br>






## How do you CREATE a document using Mongoose?
To CREATE a new book, you need to use the ```.create()``` method.

<dl>
<dd>

<br>

### STEP 1: Create a POST route:
-----
First, we need to send a GET request to the /books endpoint.
```JavaScript
    app.post("/books", (req, res) => {                                  // POST request to /books endpoint.
        ...
        ...
    });
```

<br>

### STEP 2: Check if required fields are supplied
When you create an instance of a document, you want to make sure that at least the required fields for creating an instance are supplied by the client. The required fields are 
stipulated in the ```requiredFields``` variable. If any of the required fields are missing, we send a message that the field is missing. Note that the required fields outlined here
are less than the fields allowed in the bookSchema.

```JavaScript
app.post("/books", (req, res) => {
  const requiredFields = ["name", "borough", "cuisine"];
  for (let i = 0; i < requiredFields.length; i++) {
    const field = requiredFields[i];
    if (!(field in req.body)) {
      const message = `Missing \`${field}\` in request body`;
      console.error(message);
      return res.status(400).send(message);
    }
  }

  ...
  ...

```

<br>

### STEP 3: Create new instance using the the bookSchema model:
Following the previous step, which checked to see f the required fields were in the request body, you can bow create an instance of a book. To do this, you first call ```Book.create``` with the permissible fields.

```JavaScript
app.post("/books", (req, res) => {
  const requiredFields = ["name", "borough", "cuisine"];
  for (let i = 0; i < requiredFields.length; i++) {
    const field = requiredFields[i];
    if (!(field in req.body)) {
      const message = `Missing \`${field}\` in request body`;
      console.error(message);
      return res.status(400).send(message);
    }
  }

  Book.create({
    name: req.body.name,
    borough: req.body.borough,
    cuisine: req.body.cuisine,
    grades: req.body.grades,
    address: req.body.address
  })
    ...
    ...
});

```

<br>

### STEP 4: Callback success!
If the creation of a book succeeeds, the return response to the client will be the book object created by the ```book.serialize``` instance method.

```JavaScript
app.post("/books", (req, res) => {
  const requiredFields = ["name", "borough", "cuisine"];
  for (let i = 0; i < requiredFields.length; i++) {
    const field = requiredFields[i];
    if (!(field in req.body)) {
      const message = `Missing \`${field}\` in request body`;
      console.error(message);
      return res.status(400).send(message);
    }
  }

  Book.create({
    name: req.body.name,
    borough: req.body.borough,
    cuisine: req.body.cuisine,
    grades: req.body.grades,
    address: req.body.address
  })
    .then(book => res.status(201).json(book.serialize()))
    ...
    ...
});
```

<br>

### STEP 5: IF error, send error:
And if there is an error, send a 500 status code (server error) and an message to the client.

```JavaScript
app.post("/books", (req, res) => {
  const requiredFields = ["name", "borough", "cuisine"];
  for (let i = 0; i < requiredFields.length; i++) {
    const field = requiredFields[i];
    if (!(field in req.body)) {
      const message = `Missing \`${field}\` in request body`;
      console.error(message);
      return res.status(400).send(message);
    }
  }

  Book.create({
    name: req.body.name,
    borough: req.body.borough,
    cuisine: req.body.cuisine,
    grades: req.body.grades,
    address: req.body.address
  })
    .then(book => res.status(201).json(book.serialize()))
    .catch(err => {
      console.error(err);
      res.status(500).json({ message: "Internal server error" });
    });
});
```

</dd>
</dl>


<br>

## How do you FIND documents using Mongoose?
When you *find* a document, you are essentially reading the documents from the database. In the example below, the client is sending a GET request to the /books endpoint to return 10 books. The way we do this is as follows:

1. **Send a GET request** to the endpoint ```/books```.
2. **Find ALL the documents** at the ```/books``` endpoint.
3. **Only get the first 10 documents** from the ```/books``` endpoint.
4. **If successful, send a response with an object** whose value is an array of ```books``` and for each book in the, apply the 
   ```serialize``` instance method so that only the information you want is sent back as a response.
5. **If unsuccessful, send error message** with server error.

So to put this into practice, start by creating a GET route:

<dl>
<dd>

<br>

### STEP 1: Create a GET route:
-----
First, we need to send a GET request to the /books endpoint.
```JavaScript
    app.get("/books", (req, res) => {                                  // GET request to /books endpoint.
        ...
        ...
    });
```

<br>

### STEP 2: Call the model with the .find method:
-----
When you call ```Books.find()```, by default will retrieve all the documents in the collection.
```JavaScript
    app.get("/books", (req, res) => {
       Books.find()                                          // Find all documents in the Book collection.
       ...
       ...
    });
```

<br>

### STEP 3: Add any additional methods before success callback:
-----
Sometimes, you have to narrow down the scope of the documents you find in the database. For example, suppose you had a database with hundreds, if not *thousands* of documents. In cases like those, you want to limit the return to only a few documents with the ```.limit``` method. The ```.limit``` method takes one parameter, which is a number defining how many documents to return (i.e. 10).
```JavaScript
    app.get("/books", (req, res) => {
        Books.find()
            .limit(10)                                                 // Limit return to 10 documents.
            ...
            ...
    });
```

<br>

### STEP 4: Successful callback!
-----
If the request is successful, we use the ```.then``` method to *return a promise* (from models.js). This will send an object with the property ```books``` whose value is an array of ```books``` objects. Then, for each```books``` we get back (i.e. ```books.map()```) from the collection query, call the ```.serialize``` instance method (see the serialize instance method from models.js) so that only certain info will be exposed when the API returns the data (i.e. does not return sensitive information).

```JavaScript
    app.get("/books", (req, res) => {
    Books.find()
        .limit(10)
        .then(books => {                                                      // successful callback
        res.json({
            books: books.map(books => books.serialize())
        });
        })
        ...
        ...
    });
```

<br>

### STEP 5: And if there is an error, catch as an error:
-----
If there is an error with the GET request, send a response with a 500 status code (i.e. server error) and a message to the client saying that there was in fact a server error.
```JavaScript
    app.get("/books", (req, res) => {
    Books.find()
        .limit(10)
        .then(books => {
        res.json({
            books: books.map(books => books.serialize())
        });
        })
        .catch(err => {                                                 // catch error (if any).
        console.error(err);
            res.status(500).json({ message: "Internal server error" });
        });
    });
```

</dd>
</dl>

<br>

## How do you FIND BY ID using Mongoose?
With **find by ID**, your objective is to find an *exact match* for your query. Then the server will send back a single object representing the requested book. In this example, we're finding a book by id. This is how we do it.

1. **Send a GET request** to the endpoint ```/books:id```.
2. **Call the model with the ```.findById```**.
3. **If the callback is successful**, return a json string with an object of the book.
4. **if unsuccessful**, send an error message.

<dl>
<dd>


<br>

### STEP 1: Create a GET route:
-----
Almost identical to the standard ```.find``` route, only this time we are adding an ```/:id``` to the endpoint. 
```JavaScript
    app.get("/books/:id", (req, res) => {             // get request to the /books/:id endpoint.
        ...
        ...
    });
```

<br>

### STEP 2: Call the model with the ".findById" method:
-----
Again, almost identical to th the ```.find``` route, except we want to find the document by id and pass in the user requested endpoint.
```JavaScript
    app.get("/books/:id", (req, res) => {
        Books                                              // from imported model (models.js)...
            .findById(req.params.id)                       // ... find by id parameter provided by user.
            ...
            ...
    });
```

<br>

### STEP 3: Successful callback! 
-----
If the promise is successful (i.e. if the get request to the endpoint is valid and the id is found), *then* the reponse will be a json object of the book
which has been processed through the ```.serialize``` instance method
```JavaScript
    app.get("/books/:id", (req, res) => {
        Books
            .findById(req.params.id)
            .then(books => res.json(books.serialize()))    // return a json string representing the object
            ...                                            // produced by the Books serialization method.
            ...
    });
```

<br>

### STEP 4: If unsuccessful, send back error:
-----
And of course, if there is an error, send back a 500 internal server error asd a response with a message.

```JavaScript
    app.get("/books/:id", (req, res) => {
        Books
            .findById(req.params.id)
            .then(books => res.json(books.serialize()))
            .catch(err => {                                                       // send error if needed.
            console.error(err); 
            res.status(500).json({ message: "Internal server error" });
            });
    });
```


</dd>
</dl>

<br>

## How do you FIND BY VALUE using Mongoose?
When you find by value, that is, find by optional requests such as author and book or book and isPublished, you need to use```findByValue```. There is a little more to ```findByValue``` than the previous READ operations you need to do:

1. **Send a GET request** to the endpoint ```/books```.
2. **Modify the GET endpoint** to support optional search parameters.
3. **Call the model with the ```.find``` method**.
4. **If successful, send the results**.
5. **If unsuccessful**, send an error message.


<dl>
<dd>

<br>

### STEP 1: Create a GET route:
-----
First, we need to send a GET request to the /books endpoint.
```JavaScript
    app.get("/books", (req, res) => {                                  // GET request to /books endpoint.
        ...
        ...
    });
```

<br>

### STEP 2: Limit GET requests to certain properties:
-----
If the client includes either "author" or "title" as URL parameters, it will only return those that match the requested values. This *query criteria* (i.e. ```filter```) will be passed to ```find``` in the next step (i.e. Books.find(filters)).

```JavaScript
    app.get("/books", (req, res) => {       
        const filter = {};                                       // variable to store query criteria
        const queryableFields = ['author', 'title'];             // query fields
        queryableFields.forEach(field => {                       // for each of the query fields:
            if (req.query[field]) {                              // if the field matches return.... 
                Books.[field] = req.query[field];                // return the requested values.
            }
        })
        ...
        ...
    });
```

<br>


### STEP 4: Use the .find method and pass in your filters:
Remember, if you just use ```.find()``` you will get back ALL the documents in the collection. When you pass in the filters, you are passing in a query criteria object which narrows the search down to those fields you had previously stipulated.

```JavaScript
    app.get("/books", (req, res) => {       
        const filter = {};                   
        const queryableFields = ['author', 'title'];     
        queryableFields.forEach(field => {           
            if (req.query[field]) {              
                Books.[field] = req.query[field];     
            }
        })

        Books
            .find(filters)
            ...
            ...
    });
```

<br>

### STEP 5: Successful callback:
-----
If the request is successful, we use the ```.then``` method to *return a promise* (from models.js). This will send an object with the property ```books``` whose value is an array of ```books``` objects. Then, for each```books``` we get back (i.e. ```books.map()```) from the collection query, call the ```.serialize``` instance method (see the serialize instance method from models.js) so that only certain info will be exposed when the API returns the data (i.e. does not return sensitive information).
```JavaScript
    app.get("/books", (req, res) => {       
        const filter = {};                   
        const queryableFields = ['author', 'title'];     
        queryableFields.forEach(field => {           
            if (req.query[field]) {              
                Books.[field] = req.query[field];     
            }
        })

        Books
            .find(filters)
            .then(Books => res.json(
                Books.map(books => books.serialize());     // return a json string representing the object
            ))                                             // produced by the Books serialization method.
            ...
            ...
    });
```

<br>

### STEP 4: If unsuccessful, send back error:
-----
And of course, if there is an error, send back a 500 internal server error asd a response with a message.
```JavaScript
    app.get("/books", (req, res) => {       
        const filter = {};                   
        const queryableFields = ['author', 'title'];     
        queryableFields.forEach(field => {           
            if (req.query[field]) {              
                Books.[field] = req.query[field];     
            }
        })

        Books
            .find(filters)
            .then(Books => res.json(
                Books.map(books => books.serialize());    
            ))                                              
            .catch(err => {                                                     // send error if needed.
            console.error(err); 
            res.status(500).json({ message: "Internal server error" });
            });
    });
```

</dd>
</dl>

<br>

## How do you UPDATE a document using Mongoose?
When you UPDATE items in your database, you query the database and send a change to be made using the method ```.findByItAndUpdate```. To do this, we need to:

1. **Send a GET request** to the endpoint ```/books/:id```.
2. **Check to see whether or not the id in the request path AND the request body match**.
3. **Create an object of fields to be updated**.
4. **Use the ```findByIdAndUpdate```**.
5. **If successful, send the results**.
6. **If unsuccessful**, send an error message.


<br>

### STEP 1: Create a GET route:
-----
First, we need to send a GET request to the /books/:id endpoint.
```JavaScript
    app.get("/books/:id", (req, res) => {                          // GET request to /books/:id endpoint.
        ...
        ...
    });
```

<br>

### STEP 2: Make sure the id in the request path match the body:
-----
Basically, check to see if the id matches whats in the database and, if not, send an error message.
```JavaScript
    app.put("/books/:id", (req, res) => {
        if (!(req.params.id && req.body.id && req.params.id === req.body.id)) {
            const message =
            `Request path id (${req.params.id}) and request body id ` +
            `(${req.body.id}) must match`;
            console.error(message);
            return res.status(400).json({ message: message });
        }
        ...
        ...
    });
```

<br>

### STEP 3: Create an object of fields to be updated:
--------
This object is basically going to hold the updatable fields that we permit to be updated (see ```updateableFields```). Then, for each field that can be updateable, if the field 
is updated, update.

```JavaScript
    app.put("/books/:id", (req, res) => {
        if (!(req.params.id && req.body.id && req.params.id === req.body.id)) {
            const message =
            `Request path id (${req.params.id}) and request body id ` +
            `(${req.body.id}) must match`;
            console.error(message);
            return res.status(400).json({ message: message });
        }

        const toUpdate = {};
        const updateableFields = ["name", "borough", "cuisine", "address"];
        
        updateableFields.forEach(field => {
            if (field in req.body) {
            toUpdate[field] = req.body[field];
        }
        ...
        ...
```

<br>

### STEP 4: Update what needs to be updated:
--------
Use ```.findByIdAndUpdate``` and for those documents that you have identified by ID, all key/values that have been stored to ```.toUpdate``` will be update (see ```$set```). 
Also note that ```findByIdAndUpdate``` takes two arguments, the first being an id of the doucment you want to update and the second is an object describing how the document 
should be updated.

```JavaScript
    app.put("/books/:id", (req, res) => {
        if (!(req.params.id && req.body.id && req.params.id === req.body.id)) {
            const message =
            `Request path id (${req.params.id}) and request body id ` +
            `(${req.body.id}) must match`;
            console.error(message);
            return res.status(400).json({ message: message });
        }

        const toUpdate = {};
        const updateableFields = ["name", "borough", "cuisine", "address"];
        
        updateableFields.forEach(field => {
            if (field in req.body) {
            toUpdate[field] = req.body[field];
        }

        Book
            .findByIdAndUpdate(req.params.id, { $set: toUpdate })          
            ...
            ...
```

<br>

### STEP 5: Successful callback:
---------
If the promise is fufilled, send a 204 status code.

```JavaScript
    app.put("/books/:id", (req, res) => {
        if (!(req.params.id && req.body.id && req.params.id === req.body.id)) {
            const message =
            `Request path id (${req.params.id}) and request body id ` +
            `(${req.body.id}) must match`;
            console.error(message);
            return res.status(400).json({ message: message });
        }

        const toUpdate = {};
        const updateableFields = ["name", "borough", "cuisine", "address"];

        updateableFields.forEach(field => {
            if (field in req.body) {
            toUpdate[field] = req.body[field];
            }
        });

        Book
            .findByIdAndUpdate(req.params.id, { $set: toUpdate })
            .then(book => res.status(204).end())
            ...
    });
```

<br>

### STEP 6: If error, send error:
---------
If there is an error, send a server error message and send the message to the client.

```JavaScript
    app.put("/books/:id", (req, res) => {
        if (!(req.params.id && req.body.id && req.params.id === req.body.id)) {
            const message =
            `Request path id (${req.params.id}) and request body id ` +
            `(${req.body.id}) must match`;
            console.error(message);
            return res.status(400).json({ message: message });
        }

        const toUpdate = {};
        const updateableFields = ["name", "borough", "cuisine", "address"];

        updateableFields.forEach(field => {
            if (field in req.body) {
            toUpdate[field] = req.body[field];
            }
        });

        Book
            .findByIdAndUpdate(req.params.id, { $set: toUpdate })
            .then(book => res.status(204).end())
            .catch(err => res.status(500).json({ message: "Internal server error" }));
    });
```

<dl>
<dd>

<br>

## How do you DELETE a document using Mongoose?
To delete a document from the database collection, you simply need to use the ```findByIdAndRemove``` method.
<dl>
<dd>

```JavaScript
    app.delete("/books/:id", (req, res) => {
        Book.findByIdAndRemove(req.params.id)
            .then(book => res.status(204).end())
            .catch(err => res.status(500).json({ message: "Internal server error" }));
    });
```

</dd>
</dl>

<br>

## How do you handle non-existent endpoints?
There will undoubtedly be times when the client will submit a request to an enpoint that does not exist. In situations like these, you can create a catch-all middle-ware that will send a 404 status and message to the client.
<dl>
<dd>

```JavaScript
    app.use("*", function(req, res) {
        res.status(404).json({ message: "Not Found" });
    });
```

</dd>
</dl>