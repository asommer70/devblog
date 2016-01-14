# React Native Habit Navigation

##  Getting Around

I know that [last time]() I ended with wanting to be able to upload the Habits the [app]() has been tracking to a web server of some type, but once I started diving into how that would actually work I realized that it would clutter the main screen somewhat to add another form.

Instead of mucking around with more forms and buttons on the main screen we'll use React Native [Navigator](https://facebook.github.io/react-native/docs/navigator.html) to create a new Settings screen where we can add a URL and such for saving our habit's progress.

Might also move the Habit form to it…

## Nav Component

I'm not sure I understand the React Native Navigator to the fullest extent possible, in fact I'm 100% sure I don't understand Navigator all that well, but I understand it enough to feel confident in using some code that navigates.

Create a new files **src/nav.js** and add the following:

```
var React = require('react-native');
var {
  StyleSheet,
  Navigator
} = React;

var Main = require('./main');

var ROUTES = {
  main: Main,
};
```

As you can see we're importing React and our Main component.  After that a **ROUTES** object is setup that creates a route for each main/screen component in the app.

```
module.exports = React.createClass({
  componentWillMount: function() {
  },

  renderScene: function(route, navigator) {
    var Component = ROUTES[route.name];
    return <Component route={route} navigator={navigator} />;
  },

  render: function() {
    return (
      <Navigator
        style={styles.container}
        initialRoute={{name: 'main'}}
        renderScene={this.renderScene}
        configureScene={() => { return Navigator.SceneConfigs.FloatFromRight; }}
        />
    );
  }
});

var styles = StyleSheet.create({
  container: {
    flex: 1
  }
});
```

Next the React Native component is created.  The main parts are the **renderScene** method which returns the component we pass to it and adds the Navigator to the **props**.  At least that's how I understand it.

Inside the **render** method the Navigator component is setup and the default/start screen is set to **main** in the **initalRoute** property.

Next, we need to adjust the **index.android.js** and **index.ios.js** files to call the **src/nav.js** component instead of the **src/main.js**.  Edit the index files to have:

```
var React = require('react-native');
var { AppRegistry } = React;
var Nav = require('./src/nav');
AppRegistry.registerComponent('thehoick.habit_app', () => Nav);
```

Now when you run the app nothing should really be changed.  But we now have navigation and that's cool.

## Settings Button

Edit **src/main.js** and add a new button to open our Settings screen:

```
        <View style={styles.buttonRow}>
          <Button text={'Settings'} onPress={this.openSettings} textType={styles.navText} buttonType={styles.shareButton} />
          <Button text={'Share'} imageSrc={require('./img/share-icon.png')} onPress={this.onShare} textType={styles.shareText} buttonType={styles.shareButton} />
        </View>
```

Notice that we're wrapping the Share button in a View so that we can put the Settings button next to it.  Go ahead and add these styles now:

```
  buttonRow: {
    flexDirection: 'row'
  },

  navText: {
    textAlign: 'center',
    color: '#DFD9B9',
    fontSize: 18
  },
```

This will give us a smaller button for Settings to the left of the Share button.

Back in **src/nav.js** import the Settings component:

```
var Settings = require('./settings');
```

Now add the Settings route:

```
var ROUTES = {
  main: Main,
  settings: Settings,
};
```

If you check the app now it will have a render error because we haven't setup the **src/settings.js** file yet.  We'll do that next though.

## Settings Screen

Create a **src/settings.js** files with:

```
var React = require('react-native');
var {
  Text,
  View,
  ScrollView,
  StyleSheet
} = React;

var Button = require('./components/button');

module.exports = React.createClass({
  goBack: function() {
    this.props.navigator.pop();
  },

  render: function() {
    return (
      <View style={styles.container}>
        <Button text={'Back'} onPress={this.goBack} textType={styles.navText} buttonType={styles.navButton} />

        <View style={styles.wrapper}>
          <Text style={styles.white}>Settings... </Text>
        </View>
      </View>
    )
  }
})
```

For now this is a simple View with a Text component and a Back button.  It's great that the Navigator is like a stack, or array, and you can push and pop screen/scene components on and off to move around the app.  To go back to the Main component we do **this.props.navigator.pop()** and Pow! Back to the main screen.

Don't forget to add the styles:

```
var styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: 15,
    backgroundColor: '#045491',
  },

  wrapper: {
    marginTop: 40,
    justifyContent: 'center',
    alignItems: 'center',
  },

  navText: {
    textAlign: 'center',
    color: '#DFD9B9',
    fontSize: 18
  },

  navButton: {
    borderColor: '#DFD9B9',
    borderRadius: 0,
    alignSelf: 'flex-start'
  },

  white: {
    color: '#DFD9B9'
  },
})
```

## Habit Form

// Move the Habit form from the Main screen to settings.

// Open the Settings screen if there are no habits configured.

## Habits Screen

// Create habits.js with a scrolling ListView for each Habit and display how many days in the chain it has.

// Delete a habit from the Habits array in storage.

## Habit Button

// Change the longPress on the Habit button to be able to select Habits from a list instead of open the form.

## Conclusion