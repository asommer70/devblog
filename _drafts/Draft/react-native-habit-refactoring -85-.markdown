# React Native Habit App Refactoring

## Intro

The **main.js** file for our humble [Habit](https://github.com/asommer70/thehoick-habit-app) app has gotten pretty crowded and unwieldy.  At least I'm starting to think so cause tracking down bugs is harder and harder and I think some of the cause is the amount paging I have to do through the file to find a chunk of code.

Not to mention actually understanding how all the different code and functions interact with each other.  Time for some clean up… cause I might want to add some more features down the road.

## Events

To make it easy for our components to communicate with each other we'll use the EventEmitter from Node.js to pass around, and subscribe to, events when certain actions happen.  Like pressing a button for example.

In **src/main.js** at the top add this require for EventEmitter:

```
var EventEmitter = require('EventEmitter');
```

Next, add this new **componentWillMount** method:

```
  componentWillMount: function() {
    this.eventEmitter = new EventEmitter();
  },
```

Awesome, this will add an **EventEmitter** object to our main component and we can then pass it to the other components (the ones that need it) via props.

## Habit

The largest component to refactor, I say largest because of the **addDay** method, is the Habit component itself. The addDay method looks complicated, but once you dive into it it's not too bad.  I'm sure there's a bunch of ways to make it better, but for now it'll do.

Create a new file in **src/components/habit.js** and add:

```
var React = require('react-native');
var {
  View,
  Text,
  StyleSheet,
  TouchableWithoutFeedback,
} = React;
var store = require('react-native-simple-store');
var Subscribable = require('Subscribable');
var moment = require('moment');

var today = moment();
// var today = moment().add(1, 'days');
// var today = moment().add(7, 'days');
// var today = moment().add(8, 'days');
var dayKey = today.format('MMDDYYYY');
```

As you can see the **today** and **dayKey** variables are moved here.  Also, note that we're importing the **Subscribable** module in order to add listeners to events that blast out of the EventEmitter.

Next, add the **React.createClass**:

```
module.exports = React.createClass({
  mixins: [Subscribable.Mixin],

  componentWillMount: function() {
    this.addListenerOn(this.props.events, 'new-habit', (habits) => {
      var habit = habits[habits.length - 1];
      var checked = this.checked(habit);

      this.setState({habits: habits, habit: habit, checked: checked})
    });

    this.addListenerOn(this.props.events, 'chain-restarted', () => {
      this.setState({checked: false});
    });
  },

  getInitialState: function() {
    return {
      checked: false,
      habit: {name: '', days: []},
      habits: [],
    }
  },

  componentDidMount: function() {
    // Get the habits from AsyncStorage and set the current habit to the last one.
    store.get('habits').then((data) => {
      var habit;
      var checked;

      habit = data[data.length - 1];
      checked = this.checked(habit);

      if (this.isMounted()) {
        this.setState({habit: habit, habits: data, checked: checked}, function() {
          this.props.events.emit('got-habits', this.state.habits);
        });
      }
    });
  },
```

This is the beginning of our component and there's a lot going on.  First, notice the **mixins** attribute whose value is an array consisting of the **Subscribabl.Mixin** method.  To be honest I didn't dig into why the **mixins** attribute is needed, but I know that subscribing to an event doesn't work without it.  I pulled the line from an example on using events in React Native.

Next, we add a new **componentWillMount** method which has some **listeners** created via the ** **addListenerOn** method (I guess that the mixins attribute let's us call the **addListenerOn** method on **this**… maybe).  The two events we're looking for are the **new-habit** and **chain-restarted**.  Those events will be *emitted* from another component, so for now note that inside the callback for the listeners the Habit component's state is set.

Like before in our monolithic main component the **componentDidMount** method is where we pull the **habits** form storage.  Once we have the **habits** array the state is set and afterwards the **got-habits** event is emitted.  Other components will listen for this event and update their state with the **habits** passed as an argument.

Move the **editHabit** and **checked** methods to the new file:

```
  editHabit: function() {
    this.props.events.emit('edit-habit');
  },

  checked: function(habit) {
    var day = habit.days.findIndex(function(day, index, days) {
      if (day.dayId == dayKey) {
        return true;
      }
    });

    if (day !== -1) {
      return true;
    }  else {
      return false;
    }
  },
```

Because there will be a new **Form** component **editHabit** emits a **edit-habit** event which will determine if the form is displayed or not.  Basically it's the same as the **this.state.editHabit** boolean used before, but now it'll be on another component and that component will react to the event triggered by the Habit component.

Now, migrate the **addDay** method:

```
  addDay: function() {
    if (this.state.habit.name != '') {
      // Find out if there is an entry in days for today.
      var day = this.state.habit.days.findIndex(function(day, index, days) {
        if (day.dayId == dayKey) {
          return true;
        }
      });

      // If no entry create one.
      if (day === -1) {
        var newDay = {dayId: dayKey, created_at: today.unix(), habit: this.state.habit.name, checked: true};
        var habit = this.state.habits.pop();

        if (habit) {
          // Find the number of days between today and the last day recorded.
          var lastDay = habit.days[habit.days.length - 1];

          if (lastDay !== undefined) {
            var momentLastDay = moment.unix(lastDay.created_at);
            var diffOfDays = today.diff(momentLastDay, 'days');

            if (diffOfDays > 1) {
              // Do diffOfDays - 1 to exclude the lastDay entry from being added inside the loop.
              for (var i = diffOfDays - 1; i > 0; i--) {
                var momentBetweenDay = today.subtract(i, 'days');

                var betweenDay = {dayId: momentBetweenDay.format('MMDDYYYY'), created_at: momentBetweenDay.unix(), habit: this.state.habit.name, checked: false }
                habit.days.push(betweenDay);
              }
            }
          }

          habit.days.push(newDay);

          // Update this.state.habits with the new Habit.
          var habits = this.state.habits;
          habits.push(habit);

          // Update state.
          this.setState({habits: habits, habit: habit, checked: true});

          // Store the new habits.
          store.save('habits', this.state.habits);

          this.props.events.emit('day-added', this.state.habits);
        } else {
          this.setState({editHabit: true});
        }
      }
    } else {
      this.setState({editHabit: true});
    }
  },
```

This method is pretty much the same as before.  We've set things up very similar so with the Habit component so there's nothing to change really.  It's a big method so it looks better on it's own.

Now the **render** method:

```
  render: function() {
    return (
      <View style={styles.shadow}>
        <TouchableWithoutFeedback onLongPress={this.editHabit} onPress={this.addDay}>
          <View style={[styles.habit, this.state.checked && styles.checked]}>
            <Text style={styles.habitText}>{this.state.habit.name != '' ? this.state.habit.name : 'No habit configured...'}</Text>
          </View>
        </TouchableWithoutFeedback>
      </View>
    )
  }
})
```

Finally, migrate the **styles** from the main component:

```
var styles = StyleSheet.create({
  habit: {
    justifyContent: 'center',
    alignItems: 'center',
    padding: 20,
    borderWidth: 2,
    borderColor: '#DFD9B9',
  },

  shadow: {
    shadowColor: '#424242',
    shadowOffset: {width: 0, height: 3},
    shadowOpacity: 0.7,
    shadowRadius: 3,
    elevation: 3
  },

  habitText: {
    fontSize: 35,
    color: '#DFD9B9'
  },

  checked: {
    backgroundColor: '#4D9E7E',
  },

});
```

Back in the **src/main.js** file replace the Habit component in **render** with:

```
            <Habit habits={this.state.habits} events={this.eventEmitter}/>
```

So we're getting the **this.props.events** from the **eventEmitter** setup on the main component.  Also, don't forget to import the Habit module at the top of main.js:

```
var Habit = require('./components/habit');
```

You can now remove all the methods, styles, etc that we've migrated to the Habit component.

## Form

// Extract the Habit form to form.js.

## Link Count

// Extract the Link Count sentence into link-count.js.

## Chains

// Extract the Chains ScrollView into it's own chains.js file.

