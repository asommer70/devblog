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

Moving on, create a new file **src/components/form.js** and move the following code from **src/main.js** into it:

```
var React = require('react-native');
var {
  View,
  Text,
  StyleSheet,
  TextInput
} = React;
var store = require('react-native-simple-store');

var Button = require('./button');
var Subscribable = require('Subscribable');

module.exports = React.createClass({
  mixins: [Subscribable.Mixin],

  getInitialState: function() {
    return {
      text: '',
      editHabit: false,
      habits: this.props.habits
    }
  },
```

As with the **habit.js** component, this one starts by importing the **Subscribable** module and adding the **mixins** attribute.  The **getInitialState** method returns an object with more of the original main component attributes.

```
  componentWillMount: function() {
    this.addListenerOn(this.props.events, 'edit-habit', () => {
      this.setState({editHabit: true})
    });

    this.addListenerOn(this.props.events, 'got-habits', (habits) => {
      this.setState({habits: habits});
    });
  },
```

This component needs to listen to the **got-habits** event that is fired once the Habit component retrieves the Habits array from storage.  Also, the component listens for the **edit-habit** event which determines if the Form is displayed or not.

```
  saveHabit: function() {
    // Check this.state.habits for a habit.name matching this.state.text.
    var habitIdx = this.state.habits.findIndex( (habit, index, habits) => {
      if (habit.name == this.state.text) {
        return true;
      }
    });

    if (habitIdx !== -1) {
      // Move old habit to last (current Habit).
      var habits = this.state.habits;
      var storedHabit = habits.splice(habitIdx, 1);
      habits.push(storedHabit[0]);

      this.setState({habits: habits, habit: storedHabit[0], editHabit: false}, function() {
        this.props.events.emit('new-habit', this.state.habits);
        store.save('habits', this.state.habits);
      });
    } else {
      // Create new Habit.
      var habit = {name: this.state.text, days: []};
      var habits = this.state.habits;
      habits.push(habit);

      this.setState({habits: habits, habit: habit, editHabit: false}, function() {
        this.props.events.emit('new-habit', this.state.habits);
        store.save('habits', this.state.habits);
      })
    }
  },
```

The **saveHabit** method is the largest in this component.  Like the *addDays* method it's pretty much the same as it was in the Main component.

```
  cancelHabitEdit: function() {
    this.setState({editHabit: false});
  },

  restartHabit: function() {
    var habit = this.state.habits.pop();
    habit.days = [];

    var habits = this.state.habits
    habits.push(habit);

    this.setState({habits: habits, habit: habit, editHabit: false, checked: false}, function() {
      store.save('habits', this.state.habits);
      this.props.events.emit('got-habits', this.state.habits);
      this.props.events.emit('chain-restarted');
    });
  },
```

The **cancelHabit** is pretty much the same and **restartHabit** not only sets some state also emits the **chain-restarted** event.  This event will be listened for on the LinkCount and Chains components.  Also, notice the **got-habits** event is emitted after storing the new **habits**.  This will cause the rest of the components listening for that event to re-render.

```
  render: function() {
    var input, save;

    if (this.state.editHabit !== true) {
      label = <View></View>;
      input = <View></View>;
      save = <View></View>;
      cancel = <View></View>;
      restart = <View></View>;
    } else {
      label = <Text style={styles.label}>Enter Habit</Text>;
      input = <TextInput style={styles.input} onChangeText={(text) => this.setState({text: text})} value={this.state.text} />;
      save =  <Button text={'Save'} onPress={this.saveHabit} textType={styles.saveText} buttonType={styles.saveButton} />;
      cancel =  <Button text={'Cancel'} onPress={this.cancelHabitEdit} />;
      restart = <Button text={'Restart Chain'} onPress={this.restartHabit} textType={styles.restartText} buttonType={styles.restartButton} />;
    }

    return (
      <View style={styles.formElement}>
        {label}
        {input}
        <View style={styles.editButtons}>
          {save}
          {cancel}
        </View>
        {restart}
      </View>
    )
  }
});
```
Like the Habit component moving the render logic to this component makes the Main component that much cleaner.

Finally, add some styles:

```
var styles = StyleSheet.create({
  input: {
    padding: 4,
    height: 40,
    borderWidth: 1,
    borderColor: '#424242',
    borderRadius: 3,
    margin: 5,
    width: 200,
    alignSelf: 'center',
  },

  formElement: {
    backgroundColor: '#eeeeee',
    margin: 5,
  },

  label: {
    alignSelf: 'center',
    justifyContent: 'center',
    fontSize: 18,
    marginTop: 10,
  },

  restartButton: {
    borderColor: '#CE4B41',
  },

  restartText: {
    color: '#CE4B41',
  },

  saveButton: {
    borderColor: '#4D9E7E',
  },

  saveText: {
    color: '#4D9E7E',
  },

  editButtons: {
    flexDirection: 'row',
    flex: 2,
    alignSelf: 'center',
    justifyContent: 'center',
  },
})
```

Then replace the Form in the Main component **render** with:

```
            <Form habits={this.state.habits} events={this.eventEmitter}/>
```

## Link Count

Extract the code for the link count sentence next.  Create a new file **src/components/link-count.js** with:

```
var React = require('react-native');
var {
  View,
  Text,
  StyleSheet,
} = React;
var Subscribable = require('Subscribable');

module.exports = React.createClass({
  mixins: [Subscribable.Mixin],

  componentDidMount: function() {
    this.addListenerOn(this.props.events, 'got-habits', (habits) => {
      this.setState({days: habits[habits.length - 1].days});
    });

    this.addListenerOn(this.props.events, 'day-added', (habits) => {
      this.setState({days: habits[habits.length - 1].days});
    });

    this.addListenerOn(this.props.events, 'chain-restarted', () => {
      this.setState({days: []});
    });

    this.addListenerOn(this.props.events, 'new-habit', (habits) => {
      this.setState({days: habits[habits.length - 1].days});
    });
  },

  getInitialState: function() {
    return {
      days: []
    }
  },
```

Like the other new components the **Subscribable** model is imported.  This component is listening for several events in the **componentDidMount** method.  Basically whenever the **Habit** component is changed from the Form, or loaded from storage, the **this.state.days** attribute is set based on the **habits** argument passed along with the event.

Which brings us to the **render** method:

```
  render: function() {
    var checkedDays;
    var checks;
    if (this.state.days.length >= 1) {

      // Need an array of checked days starting with today going back to the first unchecked day.
      checks = [];
      for (var i = this.state.days.length; i > 0; i--) {
        if (this.state.days[i - 1].checked) {
          checks.push(this.state.days[i]);
        } else {
          break;
        }
      }
      checkedDays = checks.length;
    } else {
      checkedDays = '0';
      checks = [];
    }

    return (
      <Text style={styles.days}>
        {checkedDays} link{checks.length == 1 ? '' : 's'} in the chain.
      </Text>
    )
  }
})
```

Whip in some styles:

```
var styles = StyleSheet.create({
  days: {
    padding: 10,
    color: '#DFD9B9',
    fontSize: 16
  },
})
```

Then adjust the Main component's **render** method:

```
            <LinkCount days={this.state.days} events={this.eventEmitter}/>
```

## Chains

Very similar to the LinkCount component create  new component in **src/components/chains.js** for the Chains ScrollView:

```
var React = require('react-native');
var {
  View,
  Text,
  ScrollView,
  Image,
  StyleSheet,
} = React;
var Subscribable = require('Subscribable');

module.exports = React.createClass({
  mixins: [Subscribable.Mixin],

  componentDidMount: function() {
    this.addListenerOn(this.props.events, 'got-habits', (habits) => {
      var habit = habits[habits.length - 1];
      this.setState({habit: habit, days: habit.days});
    });

    this.addListenerOn(this.props.events, 'day-added', (habits) => {
      var habit = habits[habits.length - 1];
      this.setState({habit: habit, days: habit.days});
    });

    this.addListenerOn(this.props.events, 'chain-restarted', () => {
      this.setState({days: []});
    });

    this.addListenerOn(this.props.events, 'new-habit', (habits) => {
      var habit = habits[habits.length - 1];
      this.setState({habit: habit, days: habit.days});    });
  },

  getInitialState: function() {
    return {
      days: [],
      habit: {name: '', days: []}
    }
  },
```

Here we're listening for the same events as the LinkCount component and setting a **days** and **habit** attributes on state.

Adding a **render** method:

```
  render: function() {
    var chains;

    var chainIcons = this.state.days.map(function(day, index) {
      var icon;
      if (index % 30 == 0 && index !=0) {
        icon = require('../img/chain-icon-green.png');
      } else {
        icon = require('../img/chain-icon.png');
      }

      if (day.checked == false) {
        icon = require('../img/broken-chain-left-icon.png');
      }

      return <Image key={day.dayId} style={styles.icon}
              source={icon} />;
    });

    if (this.state.habit.name != '') {
      chains =  <View style={styles.chains}>
                  {chainIcons}
                </View>
    } else {
      chains = <View></View>;
    }

    return (
      <ScrollView style={[styles.scroll]} automaticallyAdjustContentInsets={true} scrollEventThrottle={200}>
       {chains}
      </ScrollView>
    )
  },
});
```

Notice on that the paths to the icon files have changed.  The **./img/$icon.png** is now **../img/** because we're importing the icon from the **src/components/** directory and not directly from the **src** directory where our **main.js** file is.  Other than that the Chains component hasn't really changed much.

Now make things look good with styles:

```
var styles = StyleSheet.create({
  scroll: {
    height: 600,
  },

  icon: {
    padding: 0,
  },

  chains: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    padding: 5,
    overflow: 'visible',
    borderColor: '#DFD9B9',
    borderWidth: 1
  },
})
```

In the Main component's **render** method replace the Chains component with:

```
          <Chains habits={this.state.habits} events={this.eventEmitter}/>
```

## Conclusion

That's a lot of changes to digest, but it's well worth it.  So much easier to track down problems with individual components when all the code for the component is in a separate file… and that file isn't four hundred lines long.

[Here's](https://github.com/asommer70/thehoick-habit-app/blob/master/src/main.js) the complete **src/main.js** in it's refactored goodness.  So much cleaner…

The next feature I'd like to develop is saving the Habits to a web service somewhere.

Party On!