---
title:  "DVD Pila! Remote Project Setup"
date:   2016-03-01 13:05:00
categories: react-native javascript
layout: post
image: rn_remote.jpg
---

## Remote Control with Web Sockets

After learning a little bit of [Socket.io](http://socket.io) and adding [web sockets](http://devblog.thehoick.com/rails/dvdpila/2016/02/23/dvdpila-web-sockets.html) to [DVD Pila!](https://github.com/asommer70/dvdpila) I've been working on a React Native app to be a remote control for DVD Pila!.

The thought being that the remote app will connect to the server's web socket and send messages for controlling the playback of videos.

Seems like a good theory…

<!--more-->

## Setting Things Up

Fire up a new React Native project with:

```
react-native init dvdpila_remote
```

The layout of the project will have a couple of scenes so we'll need to get navigator involved.  My thought is to do the same type of project as [The Hoick Habit App](https://github.com/asommer70/thehoick-habit-app) and use components that will be universal across both iOS and Android.  

Doesn't have to look too good as long as it works.

## The Code

First off, replace the contents of **index.ios.js** with:

```javascript
import React, { AppRegistry } from 'react-native';

import Main from './src/main';
AppRegistry.registerComponent('dvdpila_remote', () => Main);
```

Next, create a **src** directory and add a file **src/main.js**:

```javascript
import React, { Component, StyleSheet, Navigator } from 'react-native';

import Settings from './settings';
import Controls from './controls';
import Dvds from './dvds';

var ROUTES = {
  controls: Controls,
  settings: Settings
};


class Main extends Component {
  constructor(props) {
    super(props);
  }

  renderScene(route, navigator) {
    var Component = ROUTES[route.name];
    return <Component route={route} navigator={navigator} />;
  }

  render() {
    return (
      <Navigator
        style={styles.container}
        initialRoute={{name: 'controls'}}
        renderScene={this.renderScene}
        configureScene={() => { return Navigator.SceneConfigs.FloatFromRight; }}
        />
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1
  }
});

export default Main;
```

The app will have to *"scenes"* (or whatever you call the major view components) and the **Navigator** component is setup to be able to switch between them.  The **Controls** scene is the starting scene and it'll be where the playback buttons are housed.

Settings will consist of a simple TextInput element and a button that will save the value of the input to local storage using the [react-native-simple-store](https://github.com/jasonmerino/react-native-simple-store), the same one used in The Hoick Habit App, cause well… it's simple.

## Controls Scene

Create the **src/controls.js** file to create the layout for the DVD Pila! Buttons:

```javascript
import React, { Component, View, Text, Image, StyleSheet } from 'react-native';
import store from 'react-native-simple-store';

import ControlButton from './components/control_button';
import Button from './components/button';

class Controls extends Component {
  constructor(props) {
    super(props);
    this.props = props;

    this.state = {settings: {host: ''}};

    store.get('settings').then((data) => {
      if (data === null) {
        data = {};
      } else {

        this.setState({settings: data});
      }
    });
  }

  settings() {
    this.props.navigator.push({name: 'settings'});
  }

  render() {

    if (this.state.status == 'stop' || this.state.status == 'pause') {
      playControl = <ControlButton src={require('./img/play-white-icon.png')} onPress={this.changePlay.bind(this)} />;
    } else {
      playControl = <ControlButton src={require('./img/pause-white-icon.png')} onPress={this.changePlay.bind(this)} />;
    }

    return (
      <View style={styles.container}>
        <Text style={styles.title}>DVD Pila! Remote</Text>

        <View style={styles.buttons}>
          <ControlButton src={require('./img/previous-white-icon.png')} />

	<ControlButton src={require('./img/play-white-icon.png')} />;

          <ControlButton src={require('./img/next-white-icon.png')} />
        </View>

        <View style={styles.divider} />
        <View style={styles.row}>
          <Button buttonStyle={styles.navButton} text={'Settings'} src={require('./img/gear-white-icon.png')} onPress={this.settings.bind(this)} />
        </View>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#475333',
  },

  buttons: {
    marginTop: 20,
    flexDirection: 'row',
  },

  smallerButton: {
    marginBottom: 15,
    marginTop: 15,
    padding: 10,
  },

  title: {
    fontSize: 38,
    color: 'white',
    marginBottom: 30,
    marginTop: 10
  },

  divider: {
    borderBottomWidth: 1,
    padding: 1,
    width: 300,
    borderColor: '#9F4115',
    marginTop: 45,
    marginBottom: 5
  },

  row: {
    flexDirection: 'row'
  },

  navButton: {
    marginLeft: 15,
    marginRight: 15
  }
});

export default Controls;
```

This is simple component that has some buttons defined in imported files, a text element for letting people know what app they're using and a button at the bottom to open the Settings scene.

## Settings Scene

Now setup the **Settings** scene by creating the **src/settings.js** file:

```javascript
import React, { Component, StyleSheet, View, Text, TextInput } from 'react-native';
var store = require('react-native-simple-store');
import Button from './components/button';

class Settings extends Component {
  constructor(props) {
    super(props);
    this.props = props;

    this.state = {settings: {host: ''}};
    store.get('settings').then((data) => {
      if (data === null) {
        data = {};
      }
      this.setState({settings: data});
    });
  }

  goBack() {
    this.props.navigator.pop();
  }

  save() {
    store.save('settings', this.state.settings);
  }

  render() {
    return (
      <View style={styles.container}>
        <View style={styles.nav}>
          <Button src={require('./img/left-arrow-white.png')} onPress={this.goBack.bind(this)} buttonStyle={styles.backButton} />
        </View>
        <View style={styles.divider} />

        <View style={styles.wrapper}>
          <Text style={styles.title}>Setttings</Text>

          <View style={styles.formElement}>
            <Text style={styles.label}>DVD Pila! Host:</Text>
            <TextInput
              style={styles.input}
              onChangeText={(text) => this.setState({settings: {host: text}})}
              value={this.state.settings.host ? this.state.settings.host : ''} />
          </View>
          <Button text={'Save'} src={require('./img/save-icon.png')}
            onPress={this.save.bind(this)}
            viewStyle={styles.row}
            textStyle={styles.saveText}
            imageStyle={styles.saveImage} />
        </View>
      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#475333',
  },

  nav: {
    flexDirection: 'row',
    // marginBottom: 100,
    marginTop: 20,
    marginLeft: 5
  },

  wrapper: {
    justifyContent: 'center',
    alignItems: 'center'
  },

  title: {
    fontSize: 38,
    color: 'white',
    marginBottom: 50,
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
    backgroundColor: '#ECECD1',
    margin: 5,
    flexDirection : 'row',
    padding: 5
  },

  label: {
    alignSelf: 'center',
    justifyContent: 'center',
    fontSize: 18,
    width: 90
  },

  backButton: {
    paddingBottom: 0
  },

  saveText: {
    fontSize: 35,
    paddingRight: 40,
    paddingLeft: 10
  },

  saveImage: {
    marginLeft: 40,
    marginTop: 5
  },

  row: {
    flexDirection: 'row'
  },

  divider: {
    borderBottomWidth: 1,
    padding: 1,
    width: 400,
    borderColor: '#9F4115',
    marginBottom: 100,
    marginTop: 5
  }
});

export default Settings;
```

This file practically the same as the **settings.js** file from The Hoick Habit App.  Feels great to reuse code.  So there's the TextInput component and a Save button that will store the string value of the TextInput into simple storage via the **store** setup at the top of the file.  

Also, check out the super cool back button at the top of the scene.  It's fun to click it.

## Buttons

Since this project has a couple of different types of buttons, or actually cause the control buttons are a major feature of the Controls scene I broke the different types out into separate files.

First, create a **src/components** directory and the first buttons files **src/components/buttons.js**:

```javascript
var React = require('react-native');
var {
  View,
  Text,
  StyleSheet,
  TouchableHighlight,
  Image
} = React;

module.exports = React.createClass({
  render: function() {
    var image, text;
    if (this.props.src) {
      image = <Image source={this.props.src} style={[styles.image, this.props.imageStyle]} />;
    } else {
      image = <View/>;
    }

    if (this.props.text) {
      text = <Text style={[styles.buttonText, this.props.textStyle]}>{this.props.text}</Text>;
    } else {
      text = <View/>;
    }

    return (
      <TouchableHighlight
        style={[styles.button, this.props.buttonStyle]}
        underlayColor={'#eeeeee'}
        onPress={this.props.onPress}
        >
        <View style={this.props.viewStyle}>
          {image}
          {text}
        </View>
      </TouchableHighlight>
    );
  }
});

var styles = StyleSheet.create({
  button: {
    justifyContent: 'center',
    alignItems: 'center',
    borderWidth: 1,
    borderRadius: 5,
    padding: 5,
    borderColor: '#9F4115',
    marginTop: 10,
    backgroundColor: '#9F4115',
  },

  buttonText: {
    flex: 1,
    alignSelf: 'center',
    fontSize: 10,
    color: 'white'
  },

  image: {
    alignSelf: 'center',
    marginBottom: 5
  }
});
```

This button component is the standard type and like Settings very similar to the *buttons.js* in The Hoick Habit App.

The next type of button is a **ControlButton** defined in **src/components/control_button.js**:

```javascript
import React, { Component, View, Text, TouchableHighlight, Image, StyleSheet } from 'react-native';

class ControlButton extends Component {
  constructor(props) {
    super(props);
    this.props = props;
  }

  render() {
    return (
      <TouchableHighlight
        style={[styles.button, this.props.buttonStyle]}
        underlayColor={'#ECECD1'}
        onPress={this.props.onPress}
        >
        <Image source={this.props.src} />
      </TouchableHighlight>
    );
  }
}

const styles = StyleSheet.create({
  button: {
    justifyContent: 'center',
    alignItems: 'center',
    borderWidth: 1,
    borderRadius: 5,
    padding: 20,
    margin: 5,
    borderColor: '#9F4115',
    backgroundColor: '#9F4115',
  },
  buttonText: {
    flex: 1,
    alignSelf: 'center',
    fontSize: 20,
    color: '#222324'
  }
});

export default ControlButton;
```

This class component defines bigger buttons that are used on the Controls scene and will eventually control Play/Pause, Previous, and Advance functionality.

## Conclusion

It's fun to get up and running on a new project quickly.  I guess that's the benefit of having gone through it a couple of times now.  React Native gets more and more fun each app I create.

Don't know how many I'll whip up, but so far this native apps in JavaScript thing is a lot of fun.

Party On!
