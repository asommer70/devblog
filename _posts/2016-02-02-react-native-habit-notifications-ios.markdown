---
title:  "React Native Habit Notifications iOS"
date:   2016-02-02 13:05:00
categories: react-native learning javascript
layout: post
image: rn_notify.jpg
---

## Remind Me Now…

The [Habit](https://github.com/asommer70/thehoick-habit-app) is so almost done it's not even funny.  Before wrapping up the major development there's one more feature I'd like to blast into it.  It would be awesome to get a reminder for a habit.  

It'd be even more awesome to only get the reminder if the Habit hasn't been checked off by the time of the reminder.  Not sure how to accomplish that without implementing some type of Push Notification system.  Don't think I want to do that for this app because then it would have to rely on a server for key functionality.

Instead I think we can add reminders to the built in Calendar/Reminder system for iOS and Android.  We'll have to do some code "branching" for this feature to use different components based on the device.  

Should be fun…

<!--more-->

## Date Picker iOS

// Setup a DatePickerIOS component triggered by a button in the Habits scene.
First thing we need is a way to choose a date, or really just the time, and fortunately React Native comes with a [DatePickerIOS](https://facebook.github.io/react-native/docs/datepickerios.html) component built in.  Let's start there.

Create a new **src/components/datepicker-ios.js** file with:

```
var React = require('react-native');
var {
  DatePickerIOS,
  StyleSheet,
  Text,
  TextInput,
  View
} = React;

module.exports = React.createClass({
  getDefaultProps: function () {
    return {
      date: new Date(),
      timeZoneOffsetInHours: (-1) * (new Date()).getTimezoneOffset() / 60,
    };
  },

  getInitialState: function() {
    return {
      date: this.props.date,
      timeZoneOffsetInHours: this.props.timeZoneOffsetInHours,
    };
  },

  onDateChange: function(date) {
    this.setState({date: date});
    this.props.events.emit('date-picked', date);
  },

  render: function() {
    return (
      <View>
        <DatePickerIOS
          date={this.state.date}
          mode="time"
          timeZoneOffsetInMinutes={this.state.timeZoneOffsetInHours * 60}
          onDateChange={this.onDateChange}
          minuteInterval={10}
        />
      </View>
    )
  }
});
```

Next, setup a React Native [Modal](https://facebook.github.io/react-native/docs/modal.html) to contain the Date Picker.  In **src/habits.js** import the *datepicker-ios* component:

```
var IOSDate = require('./components/datepicker-ios');
```

Adjust the **render** method adding the new **Set Reminder** button and the Modal components:

```
  render: function() {
    return (
      <View style={styles.container}>
        <Button text={'Back'} onPress={this.goBack} textType={styles.navText} buttonType={styles.navButton} />

        <View style={styles.wrapper}>
          <Button text={'Add Habit'} onPress={this.editHabit} textType={styles.navText} buttonType={styles.navButton} />
          <HabitForm habits={this.state.habits} events={this.props.events}/>

          <Text style={styles.heading}>Habits</Text>

          <View style={styles.hr}></View>

	<ScrollView style={[styles.mainScroll]} automaticallyAdjustContentInsets={true} scrollEventThrottle={200} showsVerticalScrollIndicator={false}>
            {this.habitComponents()}
          </ScrollView>
        </View>
        <Modal
          animated={true}
          transparent={false}
          visible={this.state.modalVisible}>
          <View style={styles.modal}>
            <View style={[styles.innerContainer]}>
              {React.Platform.OS == 'ios' ? <IOSDate events={this.props.events} /> : <View/>}
              <Button text={'Set Time'} onPress={this.closeModal.bind(this, false)} textType={styles.restartText} buttonType={styles.restartButton} />
              <Button text={'Remove Reminder'} onPress={this.removeReminder.bind(this, false)} textType={styles.deleteText} buttonType={styles.deleteButton} />
            </View>
          </View>
        </Modal>
      </View>
    )
  }
```

The **React.Platform.OS** attribute is used to determine if the device is an Android or iOS device.  For now the Android DatePicker is a View component, but we'll get back to that soon.  Also, notice the **Remove Reminder** button that will null out the Reminder for the Habit.

And the new styles:

```
  modal: {
    paddingTop: 25,
    backgroundColor: '#045491',
  },
```

Then add the Modal methods **openModal** and **closeModal**:

```
  openModal: function(habitIdx) {
    console.log('habits openModal... habitIdx:', habitIdx);
    this.setState({modalVisible: true, habitReminderIdx: habitIdx})
  },

  closeModal: function(visible) {
    console.log('habits closeModal... this.state.habitReminderIdx:', this.state.habitReminderIdx);
    this.setState({modalVisible: visible});
  },
```

The trick with the Modal is that the index of the habit (the one whose Set Reminder) button was clicked needs to be passed around so we can add a Reminder attribute to the Habit.  So we set the **this.state.habitReminderIdx**, in **openModal**, to use later.  

Add a **componentWillMount** method to listen to the **date-picked** event:

```
  mixins: [Subscribable.Mixin],

  componentWillMount: function() {
    this.addListenerOn(this.props.events, 'date-picked', (date) => {
      var habits = this.state.habits;
      var momentDate = moment(date);

      habits[this.state.habitReminderIdx].reminder = momentDate.format('hh:mm');

      this.setState({habits: habits}, () => {
        store.save('habits', this.state.habits);
        this.props.events.emit('new-habit', this.state.habits);
      });

      // Set the Habit Reminder from the Date Picker.
    });
```

Also, at the beginning of the file import **Moment.js** (Moment is tops!):

```
var moment = require('moment');
```

Here's where we'll get the Habit using the **this.state.habitReminderIdx** attribute and save the Reminder to storage after updating the state.

## Reminder Notifications

// Installing the reminder/notification packages for iOS and Android.

For Reminders on iOS the [react-native-calendar-reminders](https://github.com/wmcmahan/React-Native-CalendarReminders) package works great.  To install it execute:

```
npm install —save react-native-calendar-reminders
```

Then import the module in **src/habits.js**:

```
var RNCalendarReminders = require('react-native-calendar-reminders');
```

Like the instructions for **react-native-share** the **RNCalendarReminders** needs to be enabled in Xcode too.  To enable the library:

* Open XCode.
* Right click on **Libraries** > Click Add Files To "YOUR PROJECT NAME".
* Browse to your project folder/node_modules/react-native-calendar-reminders and select RNCalendarReminders.xcodeproj.
* This will add RNCalendarReminders.xcodeproj under the Libraries and if you expand the library and expand the **Products** folder you should see the **libRNCalendarReminders.a** file in red.
* Next, click the **project_name** at the top level in XCode.
* Click the **Build Phases** tab.
* Expand **Compile Sources** and click the **"+"** button.
* Choose the **libRNCalendarReminders.a** file under **RNCalendarReminders.xcodeproj**

Now rebuild the project and things should build successfully for iOS.


Authorize the iOS EventStore back in **src/habits.js** in the **date-picked** listener callback in the **componentWillMount** method add:

```
      RNCalendarReminders.authorizeEventStore((error, auth) => {
        console.log('authorizing EventStore...');
      });
```

Now the first time a Reminder is set the app will ask for authorization to add Reminders.

## Notification Form

Now whip in a button to open the Reminder modal by adding some components to the **habitComponents** method:

```
  habitComponents: function() {
    var habits = this.state.habits.map((habit, index) => {
      return (
        <View style={styles.habits} key={index}>
          <View style={styles.habitInfo}>
            <TouchableHighlight style={styles.habitButton} onPress={() => this.habitSelected(index)}>
              <Text style={styles.habitText}>{habit.name ? habit.name : ''}</Text>
            </TouchableHighlight>
            <LinkCount habit={habit} linkCountStyle={styles.linkCountText} events={this.props.events}/>
            <Text style={styles.linkCountText}>Reminder: {habit.reminder ? habit.reminder : 'No Reminder'}</Text>
          </View>

          <View style={styles.habitButtons}>
            <Button text={'Set Reminder'} onPress={() => this.openModal(index)} textType={styles.restartText} buttonType={styles.restartButton} />
            <Button text={'Restart Chain'} onPress={() => this.restartHabit(index)} textType={styles.restartText} buttonType={styles.restartButton} />
            <Button text={'Delete'} onPress={() => this.deleteHabit(index)} textType={styles.deleteText} buttonType={styles.deleteButton} />
          </View>
        </View>
      )
    });
    return habits;
  },
```

The the **Set Reminder** button will open the modal and the new Text component will display the Reminder time if there is one set.

## Actually Adding the Reminder

// Create the addReminder method checking the device the app is running on.
Back in the **date-picked** listener callback in the **componentWillMount** method add the following to create a new iOS Reminder:

```
      var habit = this.state.habits[this.state.habits.length - 1];
      RNCalendarReminders.saveReminder(habit.name, {
        location: '',
        notes: 'Reminder from The Hoick Habit App for Habit: ' + habit.name,
        startDate: date,
        alarms: [{
          date: -1 // or absolute date
        }],
        recurrence: 'daily'
      });
```

// Updating a Reminder
Awesome, we can now create new Reminders for our Habits.  If you experiment with the code at this point you'll notice that our code keeps adding new Reminders each time it's changed.  It should probably update an existing Reminder rather than adding new ones.  Adjust the Reminder code to be:

```
      // Search for the Reminder.
      RNCalendarReminders.fetchAllReminders(reminders => {
        // Find the Reminder ID.
        var reminderId;
        for (var i = 0; i < reminders.length; i++) {
          if (reminders[i].title == habit.name) {
            reminderId = reminders[i].id;
            break;
          }
        }

        // Update the Reminder, or create a new one.
        if (reminderId !== undefined) {
          RNCalendarReminders.saveReminder(habit.name, {
            id: reminders[i].id,
            location: '',
            notes: 'Reminder from The Hoick Habit App for Habit: ' + habit.name,
            startDate: date,
            alarms: [{
              date: -1 // or absolute date
            }],
            recurrence: 'daily'
          });
        } else {
          RNCalendarReminders.saveReminder(habit.name, {
            location: '',
            notes: 'Reminder from The Hoick Habit App for Habit: ' + habit.name,
            startDate: date,
            alarms: [{
              date: -1 // or absolute date
            }],
            recurrence: 'daily'
          });
        }
      });
```

On to removing a Reminder. At the end of the **removeReminder** method add:

```
  removeReminder: function(visible) {
    var habits = this.state.habits;
    habits[this.state.habitReminderIdx].reminder = null;

    this.setState({habits: habits, modalVisible: visible}, () => {
      store.save('habits', this.state.habits);
      this.props.events.emit('new-habit', this.state.habits);
    });

    // Remove the Reminder from iOS.
    RNCalendarReminders.fetchAllReminders(reminders => {
      for (var i = 0; i < reminders.length; i++) {
        if (reminders[i].title == this.state.habits[this.state.habitReminderIdx].name) {
          RNCalendarReminders.removeReminder(reminders[i].id);
        }
      }
    });
  },
```

## Conclusion

So the Recurrence part of this feature was a little more work than I'd originally thought.  Before the [React-Native-CalendarReminders](https://github.com/wmcmahan/React-Native-CalendarReminders) module didn't support recurrence.  After a quick search on Google the **EKReminder** class does support recurrence via the **recurrenceRules** attribute.  The functionality wasn't implemented probably because the recurrence rules can be more complicated.  

To create **daily**, **weekly**, **monthly**, and **yearly** reminders (though if you're building a habit I'm not sure why you'd want anything other than daily) I had to add some code to the **RNCalendarReminders.m** Objective-C file.  I've done a small amount of minor C coding in the past so I'm not a total newb to lower level programming.  What really helped me understand the crazy, or at least I think it's crazy and archaic, syntax of Objective-C was this [course](https://teamtreehouse.com/library/objectivec-basics) on Treehouse.

I have done some Swift iOS work, but never completed the app, and really wish that React Native components could be developed in Swift.  Overall though Objective-C isn't all that bad once you get a handle on the syntax.

Who knows maybe I'll do some more developing in Objective-C…

Party On!
