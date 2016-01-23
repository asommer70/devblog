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

## Conclusion