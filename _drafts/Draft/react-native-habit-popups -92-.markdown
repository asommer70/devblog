# React Native Habit Popups

## Are You Sure?

The main features of the [Habit](https://github.com/asommer70/thehoick-habit-app) app are all in place.  There are a couple of things that aren't sitting well with me though.  

One deleting a habit, reminder, etc happens instantly with no way to back out.  Not so good if you accidentally touch your phone in the wrong place with the app open.  To fix this we'll set things up to use a confirmation popup before destroying any user data.

## React Native Popup

Hurray for the Open Source community.  The [react-native-popup](https://github.com/beefe/react-native-popup) module will totally accomplish the majority of the work we need to implement confirmations.  Even better this module is a React Native component written in straight JavaScript (well JSX, ES6, etc) and can be installed with a quick:

```
npm install --save react-native-popup
```

After that the component needs to be imported using the **ES6** syntax.  Edit **src/habits.js** and add the following below the other imports at the top:

```
import Popup from 'react-native-popup';
```

Things like this remind me that I totally need to learn the new ES6 syntax.

Now inside the **render** method above the Modal component add:

```
        <Popup ref={(popup) => { this.popup = popup }}/>
```

Great now everything is ready for popup greatness.

## Confirm Remove Reminder

First, create a new method **confirm** for launching the confirmation popup:

```
  confirm: function(content, callbackFunc, callbackArgs) {
    // Close the Modal if it's open.
    if (this.state.modalVisible) {
      this.setState({modalVisible: false});
    }

    this.popup.confirm({
        content: content,
        ok: {
            callback: () => {
                this[callbackFunc](callbackArgs);
            },
        },
    });
  },
```

This method will create execute **popup.confirm** method with an object that configures the popup.  In this case when the **Ok** button is pressed inside the popups callback execution we'll execute the supplied **callbackFunc** argument sending the **callbackArgs**.  I think this is a "JavaScripty" way to do things…

Next, the first popup we'll add is for removing the Reminder by clicking the Reminder text.  Change the TouchableHighlight for the Reminder text to:

```
            <TouchableHighlight underlayColor={'gray'} style={styles.habitButton} onPress={
                () => this.confirm('Really remove remove reminder?', 'removeReminder', index)
            }>
              <Text style={styles.reminderText}>Reminder: {habit.reminder ? habit.reminder : 'No Reminder'}</Text>
            </TouchableHighlight>
```

Great, now when the button is pressed the new **confirm** method will be called passing in some text letting the user know what's about to happen, the **removeReminder** method, and the **index** of the Habit will be passed to this.removeReminder via the popup callback.

## More Confirmations

Now do the same for the rest of the deleting methods.  Change the **Remove Reminder** button inside the Modal to:

```
              <Button text={'Remove Reminder'} onPress={this.confirm.bind(this, 'Really remove remove reminder?', 'removeReminder', false)} textType={styles.deleteText} buttonType={styles.deleteButton} />
```

Inside the **habitComponents** method change the **Restart Chain** button:

```
            <Button imageStyle={styles.iconImage} imageSrc={require('./img/reload-icon.png')} onPress={() => this.confirm('Really restart chain?', 'restartHabit', index)} buttonType={styles.restartButton} />
```

And finally, change the Delete Habit button to:

```
            <Button imageStyle={styles.iconImage} imageSrc={require('./img/trash-icon.png')} onPress={() => this.confirm('Really delete habit?', 'deleteHabit', index)} buttonType={styles.deleteButton} />
```

## Refactoring Settings

It's time to get back to something that has been slowly tweaking my mellon for the past few days.  The **Setttings** scene is kind of a mess with the hidden forms containing the URL and Username.  What's the point of hiding the forms if the whole point of going to the Settings page is to change them?

I don't know, but I think it was a design decision based on the **Add Habit** form… at least that's my story, and I'm sticking to it.

So let's change up the Settings scene to something a little more friendly.  First, edit **src/settings.js** and at the top replace the **setting-form.js** import with the **react-native-popup** import:

```
import Popup from 'react-native-popup';
```

Next, remove the **componentWillMount**, **showUrlForm**, and **showUsernameForm** methods.  Change the **getInitialState** method to be:

```
  getInitialState: function() {
    return {
      settings: {
        url: '',
        username: ''
      }
    }
  },
```

And the **componentDidMount** method is the same because we still need to get the Settings from storage.  Now for the major overhaul, change the **render** method:

```
  render: function() {
    return (
      <View style={styles.container}>
        <ScrollView style={[styles.mainScroll]} automaticallyAdjustContentInsets={true} scrollEventThrottle={200} showsVerticalScrollIndicator={false}>

        <Button imageSrc={require('./img/arrow-left-icon.png')} onPress={this.goBack} imageStyle={styles.backImage} buttonType={styles.navButton} />


        <View style={styles.wrapper}>
          <View style={styles.formWrapper}>

            <View style={styles.formElement}>
              <Text style={styles.label}>Server URL:</Text>
              <TextInput
                style={styles.input}
                onChangeText={(text) => this.setState({settings: {url: text, username: this.state.settings.username}})}
                value={this.state.settings.url ? this.state.settings.url : ''} />
            </View>

            <View style={styles.formElement}>
              <Text style={styles.label}>Username:</Text>
              <TextInput
                style={styles.input}
                onChangeText={(text) => this.setState({settings: {username: text, url: this.state.settings.url}})}
                value={this.state.settings.username ? this.state.settings.username : ''} />
            </View>
          </View>

          <Button
            text={'Save'}
            imagePos={styles.rowButton}
            imageStyle={styles.saveImage}
            imageSrc={require('./img/save-icon.png')}
            onPress={this.saveSettings}
            textType={styles.saveText}
            buttonType={styles.saveButton} />
        </View>

        </ScrollView>
        <Popup ref={(popup) => { this.popup = popup }}/>
      </View>
    )
  }
```

The "showForm" buttons have been removed and replaced by two TextInput components with a Text label.  The state the value of the TextInputs are set from state and when the fields are changed the state for both is updated via the **setState** method.  This will probably need to change in the future if there are more Settings added because this version updates all settings each time one is changed.

Also, notice the **Save** button at the bottom. This will save the value of both fields to storage, but to do that add the **saveSettings** method:

```
  saveSettings: function() {
    store.save('settings', this.state.settings);
    this.popup.alert('Settings saved...');
  },
```

Again, yay for popups!

Blast in some new styles:

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
    alignSelf: 'center',
    flex: 1,
  },

  formWrapper: {
    backgroundColor: '#DFD9B9'
  },

  saveText: {
    color: '#DFD9B9',
    fontSize: 30
  },

  navButton: {
    borderColor: '#DFD9B9',
    borderRadius: 0,
    alignSelf: 'flex-start'
  },

  backButton: {
    borderColor: '#DFD9B9',
    borderRadius: 0,
    flexDirection: 'row'
  },

  saveButton: {
    borderColor: '#DFD9B9',
    borderRadius: 0,
  },

  rowButton: {
    flexDirection: 'row'
  },

  saveImage: {
    marginRight: 5
  },

  backImage: {
    padding: 10,
    margin: 5
  },

  input: {
    padding: 4,
    height: 40,
    borderWidth: 1,
    borderColor: '#424242',
    borderRadius: 3,
    margin: 5,
    width: 200,
    alignSelf: 'flex-end',
    color: '#424242'
  },

  formElement: {
    backgroundColor: '#DFD9B9',
    margin: 5,
    flexDirection: 'row'
  },

  label: {
    alignSelf: 'center',
    justifyContent: 'center',
    fontSize: 18,
    width: 90
  },
})
```

Finally, update **src/components/button.js** for some additional styles passed through props:

```
module.exports = React.createClass({
  render: function() {
    var image;
    if (this.props.imageSrc) {
      image = <Image source={this.props.imageSrc} style={[styles.shareIcon, this.props.imageStyle]} />;
    } else {
      image = <View></View>;
    }

    return (
      <TouchableHighlight style={[styles.button, this.props.buttonType]} underlayColor={'gray'} onPress={this.props.onPress}>
        <View style={this.props.imagePos}>
          {image}
          {this.props.text ? <Text style={[styles.buttonText, this.props.textType]}>{this.props.text}</Text> : <View/>}
        </View>
      </TouchableHighlight>
    )
  }
});
```

## Conclusion

This little app has come a long way and taken quite a bit longer than the weekend I originally thought it would take.  I guess my JavaScript skill aren't quite up to whipping up an app in a few days.  Though I did have to contribute to two additional projects to get the features I wanted.

Now time for some backend server coding…

Party On!