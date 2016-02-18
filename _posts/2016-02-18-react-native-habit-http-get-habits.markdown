---
title:  "React Native Habit HTTP GET Habits"
date:   2016-02-18 13:05:00
categories: react-native learning javascript
layout: post
image: rn_get.jpg
---

## Bringing Down the Habits

Since we now have a [server](https://github.com/asommer70/thehoick-habit-server) for our [Habit](https://github.com/asommer70/thehoick-habit-app) app it makes sense, at least in my mind balls, to be able to populate the Habits from the server as well.

Probably won't need to pull Habits every time, cause you might want to have certain habits only on certain devices.  That can always change, but it makes things simpler starting out.

<!--more-->

## HTTP GET

To populate the local Habits edit the **src/settings.js** file and add a new **saveSettings** method:

```javascript
saveSettings: function() {
    store.save('settings', this.state.settings);

    // Empty habits get data from server.
    fetch(this.state.settings.url + '/' + this.state.settings.username)
      .then((response) => response.text())
      .then((responseText) => {

        var habits = JSON.parse(responseText).habits;
          if (this.props.habits.length === 0) {
          store.save('habits', habits);
        }


        // Tell the Habit component on Main that we have some Habits, and all the other components.
        this.props.events.emit('got-server-habits', habits);
        this.props.events.emit('got-habits', habits);
      })
      .catch((error) => {
        this.popup.alert('Problem with the URL you entered could not get data.');
      });

    this.props.events.emit('settings-saved', this.state.settings);
    this.popup.alert('Settings saved...');
  },
```

Using the **fetch** API the Habits are retrieved from ```this.state.settings.url + '/' + this.state.settings.username``` to make things nice and scoped on the server.  The JSON response is then parsed into an object and if there are no local Habits they server Habits are saved to storage.

A new **got-server-habits** event is emitted, along with the **got-habits** event to tell the other components to rerender.  If there's an error getting the data from the server a popup is displayed to let you know.  This part could be more specific and helpful, but it'll work for now.

If things go well a popup is displaying letting you know that as well.

## got-server-habits

The **Habit** component defined in **src/components/habit.js** need to be modified to listen for the **got-server-habits** event.  Add the following to the **componentWillMount** method:

```javascript
    this.addListenerOn(this.props.events, 'got-server-habits', (habits) => {
      this.setHabits(habits);
    });
```

Also, extract the inner code from the **store.get('habits').then()** method inside the **componentDidMount** method and create a new **setHabits** method:

```
  setHabits: function(habits) {
    var habit = habits[habits.length - 1];
    var checked = this.checked(habit);
    this.setState({habit: habit, habits: habits, checked: checked, dataSource: this.state.dataSource.cloneWithRows(habits)}, function() {
      this.props.events.emit('got-habits', this.state.habits);
    });
  },
```

This bit of refactoring can replace the code from the **componentDidMount** and **new-habit** callback code with a call to **setHabits**:

```
      this.setHabits(habits);
```

Nice, now instead of repeating that code three times the new method can be called.  Yay for refactoring!

## Conclusion

With this new code things are pretty much locked in for the Habit app.

It's been a lot of fun building this app even if it's taken far longer than the original weekend project I started out building.  I guess I could have left it a lot sooner, but the features just kept coming.

I think after this it'll be on to the next project… or maybe a business.

Party On!
