# React Native Habit HTTP POST

## Syncing Habits

Totally ready to backup the data from the [Habit](https://github.com/asommer70/thehoick-habit-app) so we can maybe develop some cool dashboard app later on.  To do this we'll send the Habits array to a server via HTTP POST every time it's saved.

It'd also be good to check the server for some Habits if there aren't any configured and there is a URL for a sync location setup.  Which means the Habits data will need to be scoped to a User object of some type before being sent to the server.

## Renaming Form Component

Before we get started with adding a new URL Form, the previous Form component should probably be renamed.  Having a component named Form and another named UrlForm might be a little confusing when glancing at the project's file tree.

To fix things up rename the **src/components/form.js** file to **src/components/habit-form.js**.  Then edit **src/habits.js** and change the module import:

```
var HabitForm = require('./components/habit-form');
```

Then change the component name in the **render** function:

```
          <HabitForm habits={this.state.habits} events={this.props.events}/>
```

Overall not too bad, and now things will be a lot more organized.

## Setting Form

For the new SettingForm component create a new file **src/components/setting-form.js** with:

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
    }
  },

  cancelSetting: function() {
    this.props.events.emit('cancel-' + this.props.setting);
  },

  saveSetting: function() {
    var settings = this.props.settings;
    settings[this.props.setting] = this.state.text;

    // Set the state and save the URL to storage.
    this.props.events.emit('new-settings', settings);
    store.save('settings', settings);
  },

  render: function() {
    return (
      <View style={styles.formElement}>
        <Text style={styles.label}>Enter {this.props.setting.charAt(0).toUpperCase() + this.props.setting.slice(1)}</Text>
        <TextInput style={styles.input} onChangeText={(text) => this.setState({text: text})} value={this.props.val} />
        <View style={styles.editButtons}>
          <Button text={'Save'} onPress={this.saveSetting} textType={styles.saveText} buttonType={styles.saveButton} />
          <Button text={'Cancel'} onPress={this.cancelSetting} />
        </View>
      </View>
    )
  }
})
```

As you can see this component is very similar to our HabitForm component.  It's a little more generic in that the label Text component is set from **this.props.setting** (with a capitalized first letter), and the Input component's value is set from **this.props.val**. 

When the Save button is pushed the **saveSetting** adds/replaces attributes to object passed to **this.props.settings** then emits a **new-settings** event passing the new settings.  The **cancelSetting** method emits a **cancel-$setting** event that is customized to the Setting for the form.  This means we'll have to listen for a particular string event, but it will enable the opening and closing of particular forms.

And the styles:

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
    backgroundColor: '#DFD9B9',
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

## Updating Settings

Back to the Settings scene we mocked up in the previous post.  Edit the **src/settings.js** file and import the new SettingForm component:

```
var SettingForm = require('./components/setting-form');
```

Next, add a **componentWillMount** method to to listen for the events:

```
  componentWillMount: function() {
    this.addListenerOn(this.props.events, 'cancel-url', () => {
      this.setState({urlForm: false});
    });

    this.addListenerOn(this.props.events, 'cancel-username', () => {
      this.setState({usernameForm: false});
    });

    this.addListenerOn(this.props.events, 'new-settings', (settings) => {
      this.setState({settings: settings, urlForm: false, usernameForm: false});
    });
  },
```

Get the Settings from storage in **componentDidMount** and set initial state in **getInitialState**:

```
  componentDidMount: function() {
    store.get('settings').then((data) => {
      console.log('data:', data);
      if (data === null) {
        data = {};
      }
      this.setState({settings: data});
    })
  },

  getInitialState: function() {
    return {
      settings: {},
      urlForm: false,
      usernameForm: false,
    }
  },
```

As with the HabitForm add some methods for displaying each SettingForm:

```
  showUrlForm: function() {
    this.setState({urlForm: true});
  },

  showUsernameForm: function() {
    this.setState({usernameForm: true});
  },
```

Finally, adjust the **render** method:

```
render: function() {
    var url;
    if (this.state.urlForm) {
      urlForm = <SettingForm events={this.props.events} val={this.state.settings.url} setting={'url'} settings={this.state.settings} />;
    } else {
      urlForm = <View/>;
    }

    if (this.state.usernameForm) {
      usernameForm = <SettingForm events={this.props.events} val={this.state.settings.username} setting={'username'} settings={this.state.settings} />;
    } else {
      usernameForm = <View/>;
    }

    return (
      <View style={styles.container}>
        <ScrollView style={[styles.mainScroll]} automaticallyAdjustContentInsets={true} scrollEventThrottle={200} showsVerticalScrollIndicator={false}>

        <Button text={'Back'} onPress={this.goBack} textType={styles.navText} buttonType={styles.navButton} />

        <View style={styles.wrapper}>

          <Text style={styles.heading}>Send Data URL</Text>
          <View style={styles.hr}></View>

          <Text style={styles.whiteText}>{this.state.settings.url ? this.state.settings.url : 'No URL configured at this time.'}</Text>

          <View style={styles.wrapper}>
            <Button text={'Set URL'} onPress={this.showUrlForm} textType={styles.navText} buttonType={styles.navButton} />
            {urlForm}
          </View>

          <View style={styles.hr}></View>

          <Text style={styles.heading}>Username</Text>
          <View style={styles.hr}></View>

            <Text style={styles.whiteText}>{this.state.settings.username ? this.state.settings.username : 'No username configured at this time.'}</Text>

            <View style={styles.wrapper}>
              <Button text={'Set Username'} onPress={this.showUsernameForm} textType={styles.navText} buttonType={styles.navButton} />
              {usernameForm}
            </View>
          </View>
        </ScrollView>
      </View>
    )
  }
```

So each SettingForm is dependent on the state of the component and each ones props are set to configure the actual Form components inside the SettingForm.  It's a little convoluted, and the end result doesn't look all that visually appealing, but it works for now.

Also, update the styles:

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

  heading: {
    color: '#DFD9B9',
    fontSize: 30
  },

  hr: {
    flex: 1,
    width: 300,
    marginTop: 10,
    marginBottom: 10,
    padding: 1,
    backgroundColor: '#DFD9B9'
  },

  whiteText: {
    color: '#DFD9B9',
    fontSize: 16
  }
})
```

## POSTing to the URL

Alright, the app is ready to send some HTTP POSTs.  First, we need to get the Settings out of storage and onto the Main component.  Edit **src/main.js** and add a **componentDidMount** method (basically the same as the Settings componentDidMount):

```
  componentDidMount: function() {
    store.get('settings').then((data) => {
      console.log('data:', data);
      if (data === null) {
        data = {};
      }
      this.setState({settings: data});
    })
  },
```

Now the Main component needs to listen to event from other components to determine when to actually send the data.  I think at this time the two events we want to listen for are **new-habit** and **day-added**.  That should make sure the important stuff is saved to a server.

Edit the **componentWillMount** method:

```
    this.addListenerOn(this.props.events, 'new-habit', () => {
      this.sendData();
    });

    this.addListenerOn(this.props.events, 'day-added', () => {
      this.sendData();
    });
```

And create the **sendData** method which will use the [fetch](https://facebook.github.io/react-native/docs/network.html) API to perform the POST:

``` 
sendData: function() {
    if (this.state.settings.url !== undefined &&
        this.state.settings.url != '' &&
        this.state.settings.username !== undefined &&
        this.state.settings.username != '') {
      fetch(this.state.settings.url, {
        method: 'POST',
        headers: {
          'Accept': 'application/json',
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({username: this.state.settings.username, habits: this.state.habits})
      })
    }
  },
```

I really like how they chose to use the standard fetch API in React Native.  Makes things super easy…. Yay, Woo!

## Conclusion

We now have a way to blast our Habits data to a server over HTTP and the data will be "scoped" to a username.  Since there's not authentication, or forced SSL, I recommend only using this feature on a local LAN.

Then again is the fact that you performed some arbitrary habit that big of a secret?  I guess the real issue is that you wouldn't want to setup a server live on the Internet where any geek off the street could POST some data too.

It'd also be cool to save the data to a Google Sheet, or another online database of some type.  Also, **GETting** Habits from the server will have to wait until later… when I actually code up a server to save this data.

Party On!