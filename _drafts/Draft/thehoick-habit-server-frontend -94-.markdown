# The Hoick Habit Server Frontend

## Data Manipulation

It didn't take long before the need to adjust the data on the [server](https://github.com/asommer70/thehoick-habit-server) sent by the [Habit App](https://github.com/asommer70/thehoick-habit-app) needed to be adjusted.

For whatever reason, ok there's a good reason and it's because I wasn't careful in sending data to the server, the habit data on the server I was working with was overwritten.  The feature I'm working on is to populate the Habits if the Settings for URL and Username are set and there aren't any configured.  We'll get to that in a minute though.

In the meantime whipping up some Node + Express + Jade templates was surprisingly very fun…

## Express Frontend

There's some additional setup to get Express ready for rendering templates.  First, install [Jade](http://jade-lang.com) with:

```
npm install --save jade
```

Next, edit **app.js** and add the following toward the top of the file, somewhere after the ```var app = express();``` statement:

```
app.set('view engine', 'jade');
```

This will tell Express that the app is using Jade for it's template engine.  Next, create a new **views** directory to hold our upcoming templates.

Before moving on, install the [PouchDB Upsert](https://github.com/pouchdb/upsert) plugin with:

```
npm install --save pouchdb-upsert
```

Then import the module in **app.js** after the *PouchDB* import:

```
PouchDB.plugin(require('pouchdb-upsert'));
```

Awesome, the app is ready to roll back documents to previous revisions and use cool cool templates.

## New Routes

The first new route is actually a re-do of the **/** index route.  Replace the static file serving **app.get('/', function(req, res){});** route with:

```
// GET /
// Return a simple welcome message.
app.get('/', function(req, res) {
  db.allDocs(function(error, docs) {
    res.render('index', {habits: docs, title: 'Habits'});
  });
});
```

The new index route uses the PouchDB **allDocs** method to get all the the docs, or at least an array of their IDs which can then be used to get the actual JSON documents.

Next, add a route to GET **/users/:username**:

```
// GET /users/:username
app.get('/users/:username', function(req, res) {
  db.get(req.params.username, {rev: req.query.rev, revs: true}, function(error, doc) {
    if (error) {
      res.status(error.status).render('error', {error: error});
    }

    res.render('user', {user: doc});
  });
});
```

This route will get actual documents based on the **:username** request parameter.  

The last new route is for rolling back documents to earlier revisions:

```
// GET /users/rollback/:username
app.get('/users/rollback/:username', function(req, res) {
  // Get the revision from the query string.
  db.get(req.params.username, {rev: req.query.rev}, function(error, oldDoc) {
    if (error) {
      res.status(200).render('error', {error: error});
    }

    db.upsert(oldDoc._id, function(doc) {
      doc.habits = oldDoc.habits || [];
      return doc;
    }).then((result) => {
      // success!
      res.redirect('/users/' + req.params.username);
    }).catch(function (err) {
      // error (not a 404 or 409)
    });
  });
```

This route will lookup a **:usernme** and if found use the **upsert** method, from the PouchDB Upsert plugin, to set the doc's attributes to the same as the revision passed via the request query string.  Upsert is great because it takes care of any update conflicts.  Not 100% sure why, but while developing this method I kept getting a bunch of conflicts… upsert to the rescue.

## Jade Templates

The [Jade](http://jade-lang.com) template engine is one of the most concise ways of writing HTML that I have ever used.  I was totally surprised how fun it was to code up some small pages using Jade.  It does kind of remind me of HAML, but I never did use that.

In the new route statements above notice the **res.render()** statements at the end.  The **render** method points to a template in the **views** directory.  As you can imagine we have some new files to create.  

First, create a **views/layout.jade** file:

```
doctype html
html(lang="en")
  head
    title The Hoick Habit Server | #{title}
    
    link(href="/assets/css/main.css", rel="stylesheet", type='text/css')
    link(href='https://fonts.googleapis.com/css?family=Carter+One|Noto+Serif|Dekko', rel='stylesheet', type='text/css')
  body
    .container
      .wrapper
        nav
          ul
            li
              a(href='/') Home
            li
              a(href='/habits') API
        hr.nav
        br
        
        block content
```

This file can then be included via the Jade **extends** method inside the rest of the templates.  So great to not have to repeat navigation elements in each template.

Now, create the **views/index.jade** file:

```
extends ./layout.jade  

block content
  h1 Welcome to The Hoick Habit Server
  h4 There is a JSON API.

  p See the <a href="/help">Help</a> page for more details.

  h3 Current Users:
  ul
    each habit in habits.rows
      li
        a(href='/users/#{habit.id}')= habit.id
```

This template is pretty simple.  It takes the **habits** object passed into the **res.render** method and loops over the **habits.rows** array returned by PouchDB.

Now for the big one.  Create the **views/user.jade** template:

```
extends ./layout.jade  

block content
  h3 Username: #{user.username}
  hr

  h3 Habits
  
  if user.habits && user.habits.length > 0
    ul
      each habit in user.habits
        li
          p= habit.name
          h4 Days:
          ul
            each day in habit.days
              li #[strong dayId:] #{day.dayId}
                br
                | #[strong created_at:] #{day.created_at}
                br
                | #[strong checked:] #{day.checked}
                
          h4 Reminder: #{habit.reminder}
            
  else
    p No habits at this time...

  hr
  br
  
  h3 Current Revision
  p= user._rev
  a(href='/users/rollback/#{user.username}?rev=#{user._rev}') Use This Revision?

  br
  br
  hr
  br
  
  h3 Revisions
  ul(class='revisions')
    each rev, idx in user._revisions.ids.reverse()
      li
        | #[p #{idx + 1}: #[a(href='/users/#{user.username}?rev=#{idx + 1}-#{rev}') #{rev}]]
```

This template lists each Habit in **habits**, then loops over the **Days** array for each Habit.  Notice that back in the **get('/users/:username')** route that the PouchDB **get** method used the **req.query.rev** attribute, if there was one.  Using this to get the document out of PouchDB will return the document at that revision.  This is a great feature of PouchDB (and CouchDB for that matter).  It's been a life saver for me at times…

At the bottom of the template the revisions for the document are looped over and links are created for each previous revision.

Lastly, there is a little **views/error.jade** to have something a little more friendly to say if PouchDB doesn't return a document, or there's some other type of error:

```
extends ./layout.jade  

block content
  h1 Houston You Have a Problem
  
  h3 A #{error.status} error has occurred.
  
  p 
    strong Error Details:
    br
    em name: #{error.name} 
    br
    em message: #{error.message} 
    br
    em reason: #{error.reason}
```

## Conclusion

Like I said, I hadn't planned on making a front-end to The Hoick Habit Server, but once you get started manipulating data via curl get's old fast.  

Whipping up this simple web app was actually a lot of fun.  I think getting to use Jade for a project was a lot of the fun because of how concise it allows you to write HTML.

Party On!