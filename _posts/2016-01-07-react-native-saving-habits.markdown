---
title:  "React Native Saving Habits"
date:   2016-01-07 13:05:00
categories: react-native learning
layout: post
image: rn_store.jpg
---

## No Redos

The [Habit App](http://devblog.thehoick.com/react-native/learning/2016/01/05/react-native-habit-app.html) from last time is coming along nicely, but it's a little annoying to have to set the Habit each time we open, or refresh, the app.  To address this we'll use the [AsyncStorage](https://facebook.github.io/react-native/docs/asyncstorage.html) feature of React Native.

The AsyncStorage allows you to save key value pairs on both Android and iOS using their native backend storage facilities.  At least that's what I understand.

<!--more-->

## Simple Store

So the AsyncStorage documentations recommends "using an abstraction on top of AsyncStorage" and though they do give a good example of doing just that, I wasn't able to get things working in a timely fashion.

So I did what I usually do and did some searching of the Ol' Internets and found [react-native-simple-store](https://github.com/jasonmerino/react-native-simple-store) which wraps AsyncStorage for you.  Nice!

To install the library execute the following in the project directory:

```
npm install --save react-native-simple-store
```

The **—save** option will add an entry to the project's **package.json** file making it easier to install dependencies later.

As you can imagine react-native-simple-store has simple methods for CRUD operations save, get, update, and delete.  Let's start adding them to our project.

## Save The Habit

First, import the module at the top of the **src/main.js** file:

```
var store = require('react-native-simple-store');
```

Next, let's adjust **saveHabit** function to store the Habit string with a key of **habit**:

```
  saveHabit: function() {
    if (this.state.text) {
      store.save('habit', this.state.text).then(() => {
        this.setState({habit: this.state.text, editHabit: false});
      });
    } else {
      this.setState({editHabit: false});
    }
  },
```

Pow! The function now checks for **this.state.text** and if it's set from the Input component it will call **store.save** and in the callback function then sent **this.state.habit** via the **setState** method.  And if not will just close the form by setting **this.state.editHabit** to false.

## Getting The Habit

Now that we have the Habit saved (somewhere) we need to retrieve it and set the state attribute.  To do that before the **getInitialState** function executes we need to add a **componentDidMount** function to the component:

```
  componentDidMount() {
    // Get the habit from AsyncStorage.
    store.get('habit').then((data) => {
      if (data === null || data === undefined) {
        data = '';
        store.save('habit', '');
      }

      if (this.isMounted()) {
        this.setState({habit: data, editHabit: false, text: data});
      }
    });
  },
```

So we're using the **get** method from the simple-store to lookup the **'habit'** key which returns a promise.  Then we're changing a **.then** function to use the actual data.  If there is no habit key in storage it's going to save an empty string.  The function then checks to see if the competent is mounted via the **isMounted** method, and if so it will then set the state for the Habit, text, and editHabit attributes.

Great we can store and retrieve the Habit!

## Saving Days

The next order of business is to store the **days** array so we can keep track of things.  The new **addDay** function is:

```
  addDay: function() {
    if (this.state.habit) {
      if (this.state.days !== null) {
        day = this.state.days.findIndex(function(day, index, days) {
          if (day.dayId == dayKey) {
            return true;
          }
        });
      } else {
        day = -1;
      }

      if (day === -1) {
        var newDay = {dayId: dayKey, created_at: Date.now(), habit: this.state.habit};

        if (this.state.days === null) {
          this.setState({days: [newDay], checked: true});
        } else {
          this.state.days.push(newDay);
          this.setState({days: this.state.days, checked: true});
        }
        store.save('days', this.state.days);
      }
    } else {
      this.setState({editHabit: true});
    }
  },
```

The main change is this line:

```
        store.save('days', this.state.days);
```

Which saves a **days** key into storage of the **this.state.days**.

## Displaying Saved Days

Lastly, we need to add another **store.get** call to the **componentDidMount** function:

```
// Get the days array from AsyncStorage, and check if today has been checked.
    store.get('days').then((data) => {
      if (data === null || data === undefined) {
        store.save('days', []);
        data = [];
      }

      if (this.isMounted()) {
        this.setState({days: data});
      }

      day = data.findIndex(function(day, index, days) {
        if (day.dayId == dayKey) {
          return true;
        }
      });

      if (day !== -1) {
        this.setState({checked: true});
      }
    })
```

This bit of code looks up the **days** key in storage and in the **.then** function makes saves an empty array if there isn't any data returned.  If there is data and the component is mounted, **this.state.days** is set to the data from storage.  

We're also executing the **findIndex** method on the **data** array to check for an object matching the **dayKey** and if it finds it the **this.state.checked** attribute is set to true.

## Conclusion

This little app is now getting much more rounded out and useful.  There's some additional features that I think would be nice.  Number one being the ability to send the data to another computer somewhere.  It'd be great to back things up and to a web server and then be able to create some type of reporting dashboard, stats, etc on the data.

Hmmm… sounds like a fun Rails project.

Party On!
