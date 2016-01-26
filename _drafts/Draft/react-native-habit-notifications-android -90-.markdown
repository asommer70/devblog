# React Native Habit Notifications Android

## Reminders on Android

Getting reminder notifications on iOS for the [Habit](https://github.com/asommer70/thehoick-habit-app) app was more complicated than I thought, but learning some Objective-C and contributing to another Open Source project was fun.  Always fun to learn new things.

Now it's time to create another platform specific component for Android reminders.

## Upgrading React Native

During the development of this little app a new version of React Native has been released, and to get the date picker module for Android to work React Native needs to be upgraded.  Fortunately the process as outlined in the [documentation](https://facebook.github.io/react-native/docs/upgrading.html) is pretty straight forward.  

I'll cover the steps I did, but they may more may not be all that useful depending on your project.

Since I was on version 0.17 the first step to upgrade to 0.18 is:

```
npm install --save react-native@0.18
```

Then as the documentation states, I updated they template files with:

```
react-native upgrade
```

This reset the build.gradle, settings.gradle, and MainActivity.java files that we modified for the [react-native-share](https://github.com/EstebanFuentealba/react-native-share) module.  Fortunately getting the settings back in was pretty much the same as with the old version of React Native.

I'll cover the changes while documenting installing the [react-native-date](https://github.com/nucleartux/react-native-date) module for Android.

## React Native Date

Starting off with **react-native-date** install it in a terminal by entering:

```
npm install --save react-native-date
```

Update **android/settings.gradle** to inclue react-native-date:

```
include ':react-native-share', ':reactdate', ':app'
project(':react-native-share').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-share/android')
project(':reactdate').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-date')
```

Notice that this example also includes the **react-native-share** module from [earlier](http://devblog.thehoick.com/react-native/learning/2016/01/12/react-native-sharing-habits.html).

Next, update the **android/app/build.gradle** file dependencies:

```
    compile project(':react-native-share')
    compile project(':reactdate')
```

Now update **android/app/source/main/java/com/thehoick/habit_app/MainActivity.java** (replace the path after *../java/..* with the name of your own app) and import the package at the top:

```
import me.nucleartux.date.ReactDatePackage;
import cl.json.RNSharePackage;
```

Further down the file add change the **getPackages()** method line to be:

```
    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
        new MainReactPackage(), new ReactDatePackage(this), new RNSharePackage());
    }
```

The **react-native-date** and **react-native-share** modules are now ready for some JavaScript.  Well the react-native-share module is already setup.

Remember to run the project on Android use: ```react-native run-android```.

## DateAndroid showTimepicker

I didn't realize it at the time, but the React Native [Modal](https://facebook.github.io/react-native/docs/modal.html#content) component is an iOS only feature at this time.  There is a third party module that creates modals that work on iOS and Android, but after digging into things the **react-native-date** component appears in it's own overlay window type thing.  

In other words you just press a button and things look good.

So in **src/habits.js** the only thing to do to get an Android date picker setup is to adjust the **openModal** method:

```
  openModal: function(habitIdx) {
    console.log('habits openModal... habitIdx:', habitIdx);
    if (React.Platform.OS == 'ios') {
      this.setState({modalVisible: true, habitReminderIdx: habitIdx})
    } else {
        NativeModules.DateAndroid.showTimepicker(function() {}, (hour, minute) => {
          var habits = this.props.habits;

          habits[habitIdx].reminder = hour + ":" + minute;
          this.setState({habits: habits}, () => {
            store.save('habits', this.props.habits);
            this.props.events.emit('new-habit', this.props.habits);
          });
        });
    }
  },
```

The **React.Platform.OS** attribute is checked and if it's not an iOS device then the **react-native-date** component is setup.  Yay, for things that are simpler.

## Set Reminders

According to [Google](http://developer.android.com/guide/topics/providers/calendar-provider.html) inserting, updating, and viewing calendar events is best done through the Intent system.  The alternative being that you can develop your app to be a Calendar Provider.  Being a Calendar Provider has the advantage of not having to open another app to complete a task.  The disadvantage is that there are a lot more features that need implemented to account for all the things a Calendar can do.

The the Habit App the Intent system will work great, and there's a great module [react-native-send-intent](https://github.com/lucasferreira/react-native-send-intent) that allows us to open the Calendar and pre-populate the Event fields with settings of our choice.  

## React Native Send Intent

Installation of the react-native-send-intent module is pretty much the same as other third party React Native Android modules.  First, install the npm:

```
npm install react-native-send-intent --save
```

Add it to the project in the **android/settings.gradle** (the following shows all the additions we've made so far):

```
include ':react-native-share', ':reactdate', ':RNSendIntentModule', ':app'
project(':react-native-share').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-share/android')
project(':reactdate').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-date')
project(':RNSendIntentModule').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-send-intent')
```

Make sure the dependencies get compiled by adding the following to **android/app/build.gradle**:

```
    compile project(':RNSendIntentModule')
```

Finally, register the module in **MainActivity.java**:

```
        new MainReactPackage(), new ReactDatePackage(this), new RNSharePackage(), new RNSendIntentPackage());
```

Whammy! The react-native-send-intent module is ready to be used in JavaScript.  Well after you restart **npm run start** if it's still running and rebuild the Android app with ```react-native run-android```.

## Adding Events with Intent

Intention is such a big part of life.  To create new Calendar Events via the Intent mechanism add the following code in **src/habits.js** inside the **openModal** method (hmmm, maybe time for some refactoring of that method…) and inside the **NativeModules.DateAndroid.showTimepicker** callback (because we'll need the hour and minute selected from the picker to create new event):

```
       NativeModules.DateAndroid.showTimepicker(function() {}, (hour, minute) => {

          var habits = this.props.habits;
          minute = Math.round(minute / 10) * 10;

          habits[habitIdx].reminder = hour + ":" + minute;
          store.save('habits', this.props.habits);

          this.setState({habits: habits}, () => {
            this.props.events.emit('new-habit', this.props.habits);

            // Get the endDate using Moment based on Moment object for the startDate and adding 30 minutes.
            var startDate = moment().format('YYYY-MM-DD') + ' ' + habits[habitIdx].reminder;
            var startMoment = moment(startDate);
            var endMoment = startMoment.add(30, 'm');
            var endDate = endMoment.format('YYYY-MM-DD hh:mm');

            // Create the Calendar Intent.
            SendIntentAndroid.sendAddCalendarEvent({
              title: habits[habitIdx].name,
              description: 'Reminder from The Hoick Habit App for Habit: ' + habits[habitIdx].name,
              startDate: startDate,
              endDate: endDate,
              recurrence: 'daily'
            });
          });
        });
```

So what we're doing here is rounding the **minute** returned from the picker to the nearest 10 minutes.  Then saving the **hh:mm** formatted time to storage.  The **this.state.habits** attribute is updated and after that the **new-habit** event is emitted.  Moment.js is used to get today's date formatted into I think some version of the ISO standard along with the time from the time picker.  That information is then used along with the habit's name attribute to create a new Event and send it to the Calendar.

Awesome!  You can open an Event directly with Intents, but to do so you need the **id** of the Event and I wasn't sure how to get that back through the React Native modules.  Might be a great feature for later.

In the mean time the **react-native-send-intent** module can open the Calendar.  That should give us a good enough method of editing Reminders.  Wrap the **NativeModules.DateAndroid.showTimepicker** in the following if-else statement:

```
      if (habits[habitIdx].reminder === undefined || habits[habitIdx].reminder === null) {
      } else {
        // Open Calendar for editing reminder event.
        SendIntentAndroid.sendOpenCalendar();
      }
```

So now if there is a **reminder** attribute of the habit saved in storage the Calendar will be opened when the **Set Reminder** button is pressed.

## Removing Reminders

There's now an issue of removing/resetting the reminder the app has in storage.  In the iOS code we have a **Remove Reminder** button inside the modal, but since there is no modal for Android we can add that functionality to the Reminder text itself.

Adjust the **habitComponents** method to wrap the Reminder <Text> in a <TouchableHighlight>:

```
            <TouchableHighlight style={styles.habitButton} onPress={() => this.removeReminder(index)}>
              <Text style={styles.linkCountText}>Reminder: {habit.reminder ? habit.reminder : 'No Reminder'}</Text>
            </TouchableHighlight>
```

Now, change the **removeReminder** method:

```
removeReminder: function(visible) {
    var habits = this.props.habits;

    if (React.Platform.OS == 'ios') {
      habits[this.state.habitReminderIdx].reminder = null;

      this.setState({habits: habits, modalVisible: visible}, () => {
        store.save('habits', this.props.habits);
        this.props.events.emit('new-habit', this.props.habits);
      });

      // Remove the Reminder from iOS.
      RNCalendarReminders.fetchAllReminders(reminders => {
        for (var i = 0; i < reminders.length; i++) {
          if (reminders[i].title == this.props.habits[this.state.habitReminderIdx].name) {
            RNCalendarReminders.removeReminder(reminders[i].id);
          }
        }
      });
    } else {
      habits[visible].reminder = null;

      this.setState({habits: habits}, () => {
        store.save('habits', this.props.habits);
        this.props.events.emit('new-habit', this.props.habits);
      });
    }
  },
```

First we get check the platform and if it's not **ios** the **visible** argument variable will be the index of the habit selected and not **false**.  That index is then used to set the Habit's reminder to **null** and the **state** is updated.  After which the Habits are saved to storage and the **new-habit** event is emitted.

## Conclusion

It's an interesting job to get similar features in Android and iOS.  The two operating systems are very alike as far as feature parity, but the implementations are quite different in some areas.

The Android Intent system is really great in my opinion and though the whole "Use this App by default" dialog can get old pretty quick, opening one app from another to add additional functionality to your app is genius.

It was also a lot of fun to contribute to two Open Source projects that add some great functionality to  React Native.  Also, the whole React Native framework is a lot of fun developing apps for both Android and iOS at the same time.

Party On!