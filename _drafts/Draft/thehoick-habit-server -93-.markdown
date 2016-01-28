# The Hoick Habit Server

## Serving Up Some Habits

Since we've baked in the ability for the [Habit App](https://github.com/asommer70/thehoick-habit-app) to send Habit data to a web server via HTTP POST, it only stands to reason that the next step is to develop some type of server to get the habits.

Since I've been learning a lot about [Node.js](https://nodejs.org/en/) and React Native is sort of based on Node, well it uses **npm** anyway, I said to myself "self, why not whip up a Node server to save the Habit data?".

I started a new [server](https://github.com/asommer70/thehoick-habit-server) to accept HTTP POST, GET, and DELETE methods.  The server will save Habit data into a [PouchDB](http://pouchdb.com) database.  

Using PouchDB might give additional functionality down the road, and it's very simple to use with JavaScript.

## Setting up the Project

The first step in the process was to create a new npm project with ```npm init``` then fill in the little questionnaire.  That sets up the **package.json** file to manage npm dependencies.  Very nice!

Next, I created the **app.js** file and the **public** directory.  From my reading and this [course](https://www.udemy.com/understand-nodejs) the public directory is the [Express](http://expressjs.com/) convention for serving static files.

I then whipped up a simple **public/welcome.html** file:

```
<!DOCTYPE html>
<html>
  <head>
    <title>The Hoick Habit Server</title>

    <link href='https://fonts.googleapis.com/css?family=Carter+One|Noto+Serif|Dekko' rel='stylesheet' type='text/css'>

    <style>
      html, body {
        background-color: #045491;
        color: #424242;
      }

      .container {
        max-width: 700px;
        margin: 0 auto;
      }

      .wrapper {
        margin-top: 100px;
        background-color: #DFD9B9;
        padding: 40px;
      }

      h1 {
        font-family: 'Carter One', sans-serif;
      }

      h4 {
        font-family: 'Noto Serif', serif;
      }

      p {
        font-family: 'Ubuntu', sans-serif;
      }

      a {
        color: #F4AA31;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="wrapper">
        <h1>Welcome to The Hoick Habit Server</h1>
        <h4>There is a JSON API.</h4>

        <p>
          See the <a href="/help">Help</a> page for more details.
        </p>
      </div>
    </div>
  </body>
</html>
```

Totally could use the Express tempting system, but since the major part of this app will be the JSON API I figured static HTML was good enough for now.  The other static files **public/404.html** and **public/help.html** are pretty much the same, but with some adjusted content in the *wrapper* div.

## Additional Modules

With the static files taken care of we can install the rest of the needed Node modules via npm:

```
npm install --save express body-parser pouchdb
```

Now in **app.js** import the modules and start setting things up:

```
var express = require('express');
var fs = require('fs');
var bodyParser = require('body-parser')

var PouchDB = require('pouchdb');
var db = new PouchDB('habits');
var remoteCouch = false;

var app = express();
app.use(bodyParser.urlencoded({ extended: false }))
app.use(bodyParser.json())

// GET /assets (static files).
app.use('/assets', express.static(__dirname + '/public'));
```

The express, fs, body-parer, and pouchdb modules are imported. Then the **app** is created as an **express()** instance.

The **bodyParser** is setup to be able to get the JSON values off of HTTP POST bodies and the **public** directory is setup to serve static files via the **/assets** route.

## Listing Habits

Now, add a route to list all Habits in the database:

```
// GET /habits
app.get('/habits', function(req, res) {
  db.allDocs(function(error, docs) {
    res.json(docs);
  });
});
```

Each document object in the database is returned by the **docs** argument in the callback then Express serves it with **res.json()**.

## Creating Habits

Next, handle HTTP POST requests:

```
// POST /habits/
app.post('/habits', function(req, res) {
  // Look for the document in the database.
  db.get(req.body.username, (error, doc) => {
    // If there is no document for that user create one.
    if (error && error.status == 404) {
      // Use username for _id.
      req.body._id = req.body.username;

      // Create new doc.
      db.put(req.body, (error, newDoc) => {
        if (error) {
          res.status(error.status).json(error);
        }

        res.json(newDoc);
      });
    } else {
      // Update the doc.
      db.put(req.body, doc._id, doc._rev, (error, updatedDoc) => {
        if (error) {
          res.status(error.status).json(error);
        }

        res.json(updatedDoc);
      });
    }
  })
});
```

Here the server performs a PouchDB **get** for the **username** attribute passed into the request body.

If the document for the user isn't found a new one is created via **put** after setting the **req.body._id** attribute as the username.  This is necessary for CouchDB otherwise the **_id** will be auto-generated.  This wouldn't be a big deal, but I think it's easier to lookup docs (habits) by username.

If there is already a document with the key of username the document is updated with a **put**.  The updated, or new, document is then returned via JSON.

## GET and DELETE

The **get** and **delete** routes/methods are much simpler then the **post**:

```
// GET /habits/:username
app.get('/habits/:username', function(req, res) {
  // Get habits array from database.
  db.get(req.params.username, function (error, doc) {
    if (error) {
      res.status(error.status).json(error);
    }
    res.json(doc);
  });
});

// DELETE /habits/:username
app.delete('/habits', function(req, res) {
  // Lookup username.
  db.get(req.body.username, function (error, doc) {
    if (error) {
      res.status(error.status).json(error);
    }

    // Remove the doc and send status message.
    db.remove(doc);
    res.json({username: req.params.username, message: 'Removed from database.'});
  });
});
```

The **/habits/:username** route gets the document from the database, or returns an error which will be a 404 if the document isn't found.  **Deleting** habits is similar to creating them in that a **get** is performed on the database first, and if it's not found the error is sent in the response, then the document is deleted with the **remove()** method and a status sent via JSON.

## Conclusion

I had originally envisioned a Rails app to store the data from the Habit app, but I'm glad I whipped this up with Node.  Using another platform is quite fun from time to time.

Also, using all the JavaScript skills I've been building up is great.  Actually understanding what I'm doing is awesome!  At least I think I know what I'm doing…

Party On!