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
    flexDirection: 'row',
    justifyContent: 'center'
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

## Settings Scene

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

I think that's good for our Settings component at the moment.  We'll come back to it when we're ready to configure a URL to POST the Habits to.  Maybe we'll get to that in the next post…

In the mean time.

## Move EventEmitter

Before we move the Form component to our new Settings scene there's a little more cleanup/adjustments to make.  The EventEmitter needs to be moved from **src/main.js** to **src/nav.js**.  Edit **src/nav.js** and at the top add:

```
var EventEmitter = require('EventEmitter');
var Subscribable = require('Subscribable');
```

Inside the component add the **mixins**:

```
  mixins: [Subscribable.Mixin],
```

Create the **eventEmitter** in **componentWillMount**:

```
componentWillMount: function() {
    this.eventEmitter = new EventEmitter();
  },
```

Next, add the **events** property to the Component in **renderScene**:

```
    return <Component route={route} navigator={navigator} events={this.eventEmitter} />;
```

Now adjust **src/main.js** changing the  components **events** property in the **render** method:

```
  render: function() {
    return (
      <View style={styles.container}>
        <ScrollView style={[styles.mainScroll]} automaticallyAdjustContentInsets={true} scrollEventThrottle={200} showsVerticalScrollIndicator={false}>
          <View style={styles.wrapper}>
            <Habit habits={this.state.habits} events={this.props.events}/>

            <LinkCount days={this.state.days} events={this.props.events}/>
          </View>

          <Chains habits={this.state.habits} events={this.props.events}/>
        </ScrollView>

        <View style={styles.buttonRow}>
          <Button text={'Settings'} onPress={this.openSettings} textType={styles.navText} buttonType={styles.shareButton} />
          <Button text={'Share'} imageSrc={require('./img/share-icon.png')} onPress={this.onShare} textType={styles.shareText} buttonType={styles.shareButton} />
        </View>
      </View>
    )
```

Awesome, the same EventEmitter is getting passed around to all components so they should all be able to see the same events.  Yay, woo!

## Habits Scene

We've got a Main scene and a Settings scene and now it's time to create a Habits scene where we can view the list of Habits we've added to the app.  It'd also be nice to be able to remove a Habit if we no longer want to track that information.

As with the Settings scene, create a new file **src/habits.js** with the following:

```
var React = require('react-native');
var {
  Text,
  View,
  ScrollView,
  StyleSheet
} = React;
var Subscribable = require('Subscribable');

var Button = require('./components/button');
var Form = require('./components/form');
var LinkCount = require('./components/link-count');

module.exports = React.createClass({
  mixins: [Subscribable.Mixin],

  componentWillMount: function() {
  },

  goBack: function() {
    this.props.navigator.pop();
  },

  render: function() {
    return (
      <View style={styles.container}>
        <Button text={'Back'} onPress={this.goBack} textType={styles.navText} buttonType={styles.navButton} />

        <View style={styles.wrapper}>
          <Text style={styles.white}>Habits</Text>
        </View>
      </View>
    )
  }
})
```

And the styles:

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

Basically it's the same as our Settings scene at this point.

### Nav

The Habits array needs to be passed to the Habits scene and one way to do that is to add it as a property through the Navigator.  Edit **src/nav.js** and setup the Nav component similar to how the Main component was setup after the **got-habits** event:

```
componentWillMount: function() {
    this.eventEmitter = new EventEmitter();

    this.addListenerOn(this.eventEmitter, 'got-habits', (habits) => {
      this.setState({habits: habits});
    });
  },

  getInitialState: function() {
    return {
      habits: []
    }
  },

  renderScene: function(route, navigator) {
    var Component = ROUTES[route.name];
    return <Component route={route} navigator={navigator} events={this.eventEmitter} habits={this.state.habits} />;
  },
```

Now Habits gets passed through the Nav component state once it's pulled from storage.  Don't forget to setup the Subscribable:

```
var Subscribable = require('Subscribable');
…
  mixins: [Subscribable.Mixin],
```

Also, add the new Habits component to our Nav:

```
var Habits = require('./habits');

var ROUTES = {
  main: Main,
  settings: Settings,
  habits: Habits
};
```

The **mixins** goes inside the component of course.  Now add the Habits to our Habits component in **src/habits.js**:

```
  getInitialState: function() {
    return {
      habits: this.props.habits,
      habit: this.props.habits[this.props.habits.length - 1]
    }
  },
```


### Render

Cool, adjust the Habits **render** method to make sure it's working:

```
      <View style={styles.container}>
        <Button text={'Back'} onPress={this.goBack} textType={styles.navText} buttonType={styles.navButton} />

        <View style={styles.wrapper}>
          <Text style={styles.white}>{this.state.habit.name}</Text>
        </View>
      </View>
```

Finally, to get to the Habits scene add another nav button to **src/main.js**:

```
        <View style={styles.buttonRow}>
          <Button text={'Settings'} onPress={this.openSettings} textType={styles.navText} buttonType={styles.shareButton} />
          <Button text={'Share'} imageSrc={require('./img/share-icon.png')} onPress={this.onShare} textType={styles.shareText} buttonType={styles.shareButton} />
          <Button text={'Habits'} onPress={this.openHabits} textType={styles.navText} buttonType={styles.shareButton} />
        </View>
```

And add the **openHabits** method:

```
  openHabits: function() {
    this.props.navigator.push({name: 'habits'});
  },
```

Great the app should be able to open both the Settings scene and the Habits scene.

## Habit Form

Now it's time to move the Form component from **src/main.js** to **src/habits.js**.  Move the component to the **render** method in settings:

```
        <View style={styles.wrapper}>
          <Text style={styles.white}>Settings... </Text>
          <Form habits={this.state.habits} events={this.props.events}/>
        </View>
```

Also, notice the change of **events** to **this.props.events** from **this.eventEmitter**.  Don't see the Form component?  Add a button for adding a Habit:

```
        <View style={styles.wrapper}>
          <Button text={'Add Habit'} onPress={this.editHabit} textType={styles.navText} buttonType={styles.navButton} />
          <Form habits={this.state.habits} events={this.props.events}/>

          <Text style={styles.white}>Habits</Text>
        </View>
```

Move the **editHabit** method from **src/components/habit.js** to **src/habits.js**:

```
  editHabit: function() {
    this.props.events.emit('edit-habit');
  },
```

Now when you add a new Habit, or change Habits, and go back to the Main scene you will see the Habit button change accordingly.  Makes me so glad we went through the refactoring process to get the components into their own files.

## Listing Habits

So we can add Habits and change which Habit is the main one, but we still can't see which Habits we have.  Time to add a list of Habits inside a React Native [ListView](https://facebook.github.io/react-native/docs/listview.html).  Edit the **render** method in **src/habits.js**:

```
          <ListView
            dataSource={this.state.dataSource}
            renderRow={(rowData, sectionId, rowId) =>
              <View style={styles.habits}>
                <View style={styles.habitInfo}>
                  <Text style={styles.habitText}>{rowData.name}</Text>
                  <LinkCount days={rowData.days} linkCountStyle={styles.linkCountText} events={this.props.events}/>
                </View>

                <View style={styles.habitButtons}>
                  <Button text={'Restart Chain'} onPress={() => this.restartHabit(rowId)} textType={styles.restartText} buttonType={styles.restartButton} />
                  <Button text={'Delete'} onPress={() => this.deleteHabit(rowId)} textType={styles.deleteText} buttonType={styles.deleteButton} />
                </View>
              </View>
            }
          />
```

To get the **this.state.dataSource** setup the **getInitialState** method needs some changes:

```
  getInitialState: function() {
    var ds = new ListView.DataSource({rowHasChanged: (r1, r2) => r1 !== r2});

    return {
      habits: this.props.habits,
      habit: this.props.habits[this.props.habits.length - 1],
      dataSource: ds.cloneWithRows(this.props.habits.reverse()),
    }
  },
```

The new **ds** variable is sets up the ListView DataSource and then it's set on state.  Also, notice the **reverse** method being called so that the current Habit is on top.  I didn't look too far into how all that works, but it does…

Now add some styles for the Habits list:

```
habits: {
    paddingTop: 10,
    paddingBottom: 10,
    flex: 2,
    borderColor: '#DFD9B9',
    borderWidth: 1,
    width: 300,
    flexDirection: 'row',
  },

  habitInfo: {
    alignSelf: 'flex-start'
  },

  habitText: {
    color: '#DFD9B9',
    fontSize: 20,
    borderColor: '#DFD9B9',
    paddingLeft: 10,
    width: 120
  },

  linkCountText: {
    color: '#DFD9B9',
    fontSize: 12,
    paddingLeft: 10
  },

  habitButtons: {
    alignSelf: 'flex-end',
    flexDirection: 'row',
    marginLeft: 5
  },

  deleteButton: {
    borderColor: '#CE4B41',
    marginLeft: 1
  },

  deleteText: {
    color: '#CE4B41',
    fontSize: 12
  },

  restartText: {
    textAlign: 'center',
    color: '#DFD9B9',
    fontSize: 12
  },

  restartButton: {
    borderColor: '#DFD9B9',
    borderRadius: 0,
    marginRight: 1
  },
```

Cool, now add the **deleteHabit** method:

```
  deleteHabit: function(habitIdx) {
    var habits = this.state.habits;
    habits.splice(habitIdx, 1);

    // Save the new Habits.
    this.setState({habits: habits, habit: habits[habits.length - 1], dataSource: this.state.dataSource.cloneWithRows(habits)}, () => {
      this.props.events.emit('new-habit', this.state.habits);
      store.save('habits', this.state.habits);
    })
  },
```

This method removes the Habit from the array based on the index passed in from the component via the **splice** method.  Then the state is updated and the **new-habit** event is triggered which will update the Habit button on the home page.  Lastly, the new Habits array is saved to storage.

The **restartHabit** method is similar to the method of the same name on the Form component:

```
  restartHabit: function(habitIdx) {
    var habits = this.state.habits;
    habits[habitIdx].days = [];

    this.setState({habits: habits, habit: habits[habits.length - 1], dataSource: this.state.dataSource.cloneWithRows(habits)}, () => {
      this.props.events.emit('got-habits', this.state.habits);
      this.props.events.emit('chain-restarted');
      store.save('habits', this.state.habits);
    });
  },

```

With this method we're resetting the **days** attribute to an empty array in place instead of pulling the Habit off changing it, then putting it back on the Habits array.  Much cleaner!

### LinkCount Update

Things don't quite work because we need to apply some changes to the LinkCount component in **src/components/link-count.js**.  First, adjust the **getInitialState**:

```
  getInitialState: function() {
    return {
      days: this.props.days
    }
  },
```

Now that we're using props to set the initial state also change the LinkCount in **src/main.js**:

```
            <LinkCount days={this.state.habit ? this.state.habit.days : []} events={this.props.events}/>
```

Back in **src/components/link-count.js** adjust the Text component to include style objects passed in via props:

```
      <Text style={[styles.days, this.props.linkCountStyle]}>
        {checkedDays} link{checks.length == 1 ? '' : 's'} in the chain.
      </Text>
```

Awesome, now our LinkCount in the Habits will look good and function correctly.

## Habit Button

// Change the longPress on the Habit button to be able to select Habits from a list instead of open the form.
Since we've moved all the functionality of the Habit **longPress** on the Main scene we can adjust that for something else.  I think being able to choose a habit on the Main scene would be cool.  

Change the **render** method in **src/components/habit.js** to:

```
render: function() {
    var habits;
    if (this.state.choosing) {
      habits = <View style={styles.habitsContainer}>
        <View style={styles.habitsHeader}>
          <Text style={styles.habitsHeaderText}>Choose a Habit</Text>
        </View>
        <View style={styles.habitsWrapper}>
          <ListView
              dataSource={this.state.dataSource}
              renderRow={(rowData, sectionId, rowId) =>
                <TouchableHighlight onPress={() => this.habitSelected(rowId)}>

                  <View style={styles.habits}>
                    <Text style={styles.habitsText}>{rowData.name ? rowData.name : ''}</Text>
                  </View>
                </TouchableHighlight>
              }
              renderSeparator={(sectionId, rowId, adjacentRowHighlighted) =>
                <View style={styles.separator} key={rowId} />
              }
            />
        </View>
      </View>
    } else {
      habits = <View/>
    }

    return (
      <View>
        <View style={styles.shadow}>
          <TouchableWithoutFeedback onLongPress={this.chooseHabit} onPress={this.addDay}>
            <View style={[styles.habit, this.state.checked && styles.checked]}>
              <Text style={styles.habitText}>{this.state.habit && this.state.habit.name != '' ? this.state.habit.name : 'No habit configured...'}</Text>
            </View>
          </TouchableWithoutFeedback>
        </View>
        {habits}
      </View>
    )
  }
})
```

We've also setup a ListView of Habits similar to the Habits component in the Habits scene (hmm maybe a chance for some refactoring… someday).  This list view also has a **rendorSeparator** property which will be displayed between items.  It's hard to make this look good, so maybe I'll revisit when I learn more about React Native design.

Next, add the **chooseHabit** method:

```
  chooseHabit: function() {
    this.setState({choosing: true});
  },
```

Like the **editHabit** method this sets the **this.state.choosing** which will display the Habits ListView.  Finally, we need to add the **habitSelected** method which is fired when a Habit is touched:

```
  habitSelected: function(habitIdx) {
    var habits = this.state.habits;
    var habit = habits.splice(habitIdx, 1);
    habits.push(habit[0])
    this.setState({habits: habits, habit: habits[habits.length -1], dataSource: this.state.dataSource.cloneWithRows(habits), choosing: false}, () => {
      this.props.events.emit('new-habit', this.state.habits);
      store.save('habits', this.state.habits);
    })
  },
```

## Form Cleanup

After all this we can clean up the **src/components/form.js** file by removing the Restart button in the **render** method:

```
  render: function() {
    var input, save;

    if (this.state.editHabit !== true) {
      label = <View></View>;
      input = <View></View>;
      save = <View></View>;
      cancel = <View></View>;
    } else {
      label = <Text style={styles.label}>Enter Habit</Text>;
      input = <TextInput style={styles.input} onChangeText={(text) => this.setState({text: text})} value={this.state.text} />;
      save =  <Button text={'Save'} onPress={this.saveHabit} textType={styles.saveText} buttonType={styles.saveButton} />;
      cancel =  <Button text={'Cancel'} onPress={this.cancelHabitEdit} />;
    }

    return (
      <View style={styles.formElement}>
        {label}
        {input}
        <View style={styles.editButtons}>
          {save}
          {cancel}
        </View>
      </View>
    )
  }
```

You can also remove the **restartHabit** method from that file as well.

## Conclusion

Well this was a whole bunch of new code and functionality.  It all started with a Settings scene and ended up with a whole Habits CRUD feature.  

It makes the app better though.  For the next post we'll probably definitely get into POSTing the Habits to a web server…

Party On!