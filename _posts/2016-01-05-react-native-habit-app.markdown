---
title:  "Habit Chain App in React Native"
date:   2016-01-05 13:05:00
categories: react-native learning
layout: post
image: react_native.jpg
---

## Good Ol' React

When [React Native](https://facebook.github.io/react-native/) first came out I was all like "so what… I'm making web apps".  Then I had more and more ideas for Android and iOS apps that the framework started looking more attractive.

I did create a small app with the [Ionic](http://ionicframework.com/) framework, but the fact that React Native compiles down to native code and doesn't live inside of a Web View kind of makes it more attractive for me.

Plus there's the whole web developer going to mobile thing with JavaScript.

The most awesome thing about React Native is how closely the things I've learned with React (for the web) have translated to React Native.  It's pretty much the same thing…

It being the new year and all I'm trying to develop a meditation habit again, and I thought a great first React Native app would be to whip up an app to help keep track of executing on the habit each day.  Sort of like the whole Jerry Seinfeld thing of ["Don't Break the Chain"](http://lifehacker.com/281626/jerry-seinfelds-productivity-secret).

<!--more-->

## The Hoick Habit App

I've setup the project on Github [here](https://github.com/asommer70/thehoick-habit-app).  I started it three days ago, and it's pretty much ready for launch.  There's a couple of bugs that I'll address in another post, but for now let's dive into the project.

Installing React Native is pretty straight forward and I encourage you to follow the [install](https://facebook.github.io/react-native/docs/getting-started.html#content) and [setup](https://facebook.github.io/react-native/docs/android-setup.html) instructions to the letter.  

Ya, I tried building the Android package with the version of the Android SDK I downloaded with Android Studio, but I was unable to download the correct revision number, so I ended up using their instructions and installing another copy with **brew**.  Yay, for hard drive space.

## Creating the Project

To create a new React Native project use the command line utility:

```
react-native init thehoick.habit_app
```

Once the init process finishes you can run the app straight away with:

```
cd thehoick.habit_app
react-native run-android
```

That is if you have an Android emulator running, or an Android device with developer options enabled plugged into the USB.

To make things easier and be able to run the **run-android** command in the same window I use a split screen terminal and execute:

```
npm run start
```

This will build the React package and run the development server on port 8081 which the app connects to.

## One Codebase One Problem…

By default **react-native** creates an **index.ios.js** and **index.android.js** file so you can take advantage of the different features of each platform.  If you're going to be coding something pretty generic then it's easier to work from one codebase and just import it into the separate index files.

Edit **index.ios.js**, and **index.android.js**, and replace the contents with:

```
var React = require('react-native');
var { AppRegistry } = React;
var Main = require('./src/main');
AppRegistry.registerComponent('thehoick.habit_app', () => Main);
```

What this will do is use the exported module in **src/main.js** for our app.  This is done via the **AppRegistry** module, which I don't pretend to understand completely.  I just know that this works and that's enough for me.

## Main.js

Create a new directory for our source files named, you guessed it, **src** and inside there create a files named **main.js** with:

```
var React = require('react-native');
var {
  View,
  Text,
  StyleSheet,
  TextInput,
  TouchableWithoutFeedback,
  Image,
  ScrollView,
  TouchableHighlight
} = React;

var Button = require('./components/button');

var today = new Date();
var dayKey = today.getMonth().toString() + today.getDate().toString() + today.getFullYear().toString();
var day;


module.exports = React.createClass({
  getInitialState: function() {
    return {
      habit: '',
      checked: false,
      days: [],
      editHabit: true,
    }
  },

  saveHabit: function() {
    this.setState({habit: this.state.text, editHabit: false});
  },

  editHabit: function() {
    this.setState({editHabit: true})
  },

  addDay: function() {
    if (this.state.habit) {
      if (this.state.days.length != 0) {
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
      }
    } else {
      this.setState({editHabit: true});
    }
  },

  restartHabit: function() {
    this.setState({days: [], editHabit: false, checked: false});
  },

  cancelHabitEdit: function() {
    this.setState({editHabit: false});
  },

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
      input = <TextInput style={styles.input} onChangeText={(text) => this.setState({text})} value={this.state.text} />;
      save =  <Button text={'Save'} onPress={this.saveHabit} textType={styles.saveText} buttonType={styles.saveButton} />;
      cancel =  <Button text={'Cancel'} onPress={this.cancelHabitEdit} />;
      restart = <Button text={'Restart Chain'} onPress={this.restartHabit} textType={styles.restartText} buttonType={styles.restartButton} />;
    }

    var chains;
    if (this.state.habit) {
      chains =  <View style={styles.chains}>
                  {this.state.days.map(function(day, index) {
                    return <Image key={day.dayId} style={styles.icon}
                            source={index % 30 == 0 && index != 0 ? require('./img/chain-icon-green.png') : require('./img/chain-icon.png')} />;
                  })}
                </View>
    } else {
      chains = <View></View>;
    }

    return (
      <View style={styles.container}>
        <View style={styles.wrapper}>
          <View style={styles.shadow}>
            <TouchableWithoutFeedback onLongPress={this.editHabit} onPress={this.addDay}>
              <View style={[styles.habit, this.state.checked && styles.checked]}>
                <Text style={styles.habitText}>{this.state.habit ? this.state.habit : 'No habit configured...'}</Text>
              </View>
            </TouchableWithoutFeedback>
          </View>

          <View style={styles.formElement}>
            {label}
            {input}
            <View style={styles.editButtons}>
              {save}
              {cancel}
            </View>
            {restart}
          </View>

          <Text style={styles.days}>{this.state.days ? this.state.days.length : '0'} link{this.state.days.length == 1 ? '' : 's'} in the chain.</Text>
        </View>

        <ScrollView style={[styles.scroll]} automaticallyAdjustContentInsets={true} scrollEventThrottle={200}>
         {chains}
        </ScrollView>
      </View>
    )
  },
});

var styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    backgroundColor: '#045491',
  },

  wrapper: {
    marginTop: 40,
    justifyContent: 'center',
    alignItems: 'center',
  },

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
    elevation: 5,
  },

  habitText: {
    fontSize: 35,
    color: '#DFD9B9'
  },

  checked: {
    backgroundColor: '#4D9E7E',
  },

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

  days: {
    padding: 10,
    color: '#DFD9B9',
    fontSize: 16
  },

  icon: {
    padding: 0,
  },

  scroll: {
    height: 600,
  },

  chains: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    padding: 5,
    overflow: 'visible',
    borderColor: '#DFD9B9',
    borderWidth: 1
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
});
```

There's quite a bit of code in there and we'll take it a step at a time.

## Importing Components

At the top of the **main.js** the React modules are imported as well as a **Button** component that we will define in another file.

To create the component, make a new **components** directory in **src** and add a **button.js** file with:

```
var React = require('react-native');
var {
  Text,
  StyleSheet,
  TouchableHighlight,
  Image,
  View
} = React;

module.exports = React.createClass({
  render: function() {
    return (
      <TouchableHighlight style={[styles.button, this.props.buttonType]} underlayColor={'gray'} onPress={this.props.onPress}>
        <View>
          <Text style={[styles.buttonText, this.props.textType]}>{this.props.text}</Text>
        </View>
      </TouchableHighlight>
    )
  }
});

var styles = StyleSheet.create({
  button: {
    justifyContent: 'center',
    alignSelf: 'center',
    borderWidth: 1,
    padding: 5,
    borderColor: '#424242',
    margin: 10
  },

  buttonText: {
    flex: 1,
    alignSelf: 'center',
    fontSize: 20,
    color: '#424242'
  },
});
```

## Global Variables

Well they're not super global.  After importing components we setup some variables:

```
var today = new Date();
var dayKey = today.getMonth().toString() + today.getDate().toString() + today.getFullYear().toString();
var day;
```

* **today:** creates a new Date object.
* **dayKey:**  is used to hold the *id* of the "chain" entry for the day.
* **day:** will hold the results of the **findIndex** method later on.

## Main Component

Next, we setup the Main component that we imported into our **index** files above.  The first interesting thing is the **getInitialState** function where we start setting up the component:

```
  getInitialState: function() {
    return {
      habit: '',
      checked: false,
      days: [],
      editHabit: true,
    }
  },
```

So basically what we do in the rest of the app is set these options.

* **habit:** is a text string of the Habit that we want to track.
* **checked:** is used to determine the color of the Habit button.
* **days:** is an array of objects containing details about the chain.
* **editHabit:** is a boolean value that determines if the Habit form is visible or not.

We then have the *saveHabit*, *editHabit*, *restartHabit*, and *cancelHabit* habit functions.  These all set the state of different variables and as you can probably tell handle the Habit form.  

Except for **restartHabit**.  Clicking the **Restart** button will set the **this.state.days** back to an empty array as well as setting **this.state.checked** and **this.state.editHabit**.  Basically it puts things back to square one except for the actual Habit setting.

## Add Day

The next big piece of the pie is the **addDay** function.  This is where a **day** object is added to the **this.state.days** array:

```
addDay: function() {
    if (this.state.habit) {
      if (this.state.days.length != 0) {
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
      }
    } else {
      this.setState({editHabit: true});
    }
  },
```

First we check for a habit, then if **this.state.days** is not 0 the **findIndex** method is called and the **dayId** method of each object in the array is checked against the **dayKey** set up top.  If **day** is set not set then a **newDay** object is created with saves the **dayKey**, a timestamp, and the Habit name.  The new **day** object is then pushed onto **this.state.days**, or it's creates a new array.

## Render Function

The **render** function is where the magic happens in React.  The first part of the function is for setting up the Habit form:

```
    var input, save;

    if (this.state.editHabit !== true) {
      label = <View></View>;
      input = <View></View>;
      save = <View></View>;
      cancel = <View></View>;
      restart = <View></View>;
    } else {
      label = <Text style={styles.label}>Enter Habit</Text>;
      input = <TextInput style={styles.input} onChangeText={(text) => this.setState({text})} value={this.state.text} />;
      save =  <Button text={'Save'} onPress={this.saveHabit} textType={styles.saveText} buttonType={styles.saveButton} />;
      cancel =  <Button text={'Cancel'} onPress={this.cancelHabitEdit} />;
      restart = <Button text={'Restart Chain'} onPress={this.restartHabit} textType={styles.restartText} buttonType={styles.restartButton} />;
    }
```

If the **this.state.editHabit** is set then we setup some variables to hold React components for buttons and an text input.  If not then it sets up blank View components because they don't take up any space by default.

The **chains** element is then setup depending on if **this.state.habit** is set:

```
    var chains;
    if (this.state.habit) {
      chains =  <View style={styles.chains}>
                  {this.state.days.map(function(day, index) {
                    return <Image key={day.dayId} style={styles.icon}
                            source={index % 30 == 0 && index != 0 ? require('./img/chain-icon-green.png') : require('./img/chain-icon.png')} />;
                  })}
                </View>
    } else {
      chains = <View></View>;
    }
```

The cool thing, well I think it's cool, is the **this.state.days.map** function call.  You can blast in a fair amount of JavaScript inside React "{}" (well I guess it's JSX curly braces).  The **map** function creates a View component for each day object in the **this.state.days** array.  Inside the View component is an Image component whose icon is determined by it's index.  Every 30 days in the chain deserves a little something extra with a green icon.

Finally, the **return** statement combines the rest of the components:

```
    return (
      <View style={styles.container}>
        <View style={styles.wrapper}>
          <View style={styles.shadow}>
            <TouchableWithoutFeedback onLongPress={this.editHabit} onPress={this.addDay}>
              <View style={[styles.habit, this.state.checked && styles.checked]}>
                <Text style={styles.habitText}>{this.state.habit ? this.state.habit : 'No habit configured...'}</Text>
              </View>
            </TouchableWithoutFeedback>
          </View>

          <View style={styles.formElement}>
            {label}
            {input}
            <View style={styles.editButtons}>
              {save}
              {cancel}
            </View>
            {restart}
          </View>

          <Text style={styles.days}>{this.state.days ? this.state.days.length : '0'} link{this.state.days.length == 1 ? '' : 's'} in the chain.</Text>
        </View>

        <ScrollView style={[styles.scroll]} automaticallyAdjustContentInsets={true} scrollEventThrottle={200}>
         {chains}
        </ScrollView>
      </View>
    )
```

There are View components wrapping most of the main components in order to apply specific styles to them.

## Conclusion

The main conclusion from this post is that React Native is the shiz night!!!

I've had a lot of fun building this little app, and am looking forward to more posts on React Native and JavaScript.

Party On!
