# React Native Habit Scopes

## More Than One Habit

It'd be great to be able to store information about more than one habit at a time.  That won't work the way the app is currently storing the Habit as a straight up string and the days as an array.  It'd probably be better to have a Habits array that has a Habit object which includes a days array unique to that Habit.

I sort of had the thought when I created the day object with a habit attribute pointing to the habit that it is tied to, but I really didn't take it far enough.

Anyway, time to update things…

## componentDidMount()

Since it's the first function in the component might as well start with the **componentDidMount** function.  Edit **src/main.js** and adjust the function as so:

```
componentDidMount() {
    // Get the habits from AsyncStorage and set the current habit to the last one.
    store.get('habits').then((data) => {
      console.log('habits data:', data);

      var habit;
      var checked = false;

      if (data === null || data === undefined || data.length == 0) {
        data = [];
        habit = {name: '', days: []};
        store.save('habits', []);
      } else {
        habit = data[data.length - 1];

        day = habit.days.findIndex(function(day, index, days) {
          if (day.dayId == dayKey) {
            return true;
          }
        });

        if (day !== -1) {
          checked = true;
        }
      }

      if (this.isMounted()) {
        this.setState({habit: habit, habits: data, editHabit: false, text: habit.name, checked: checked});
      }
    });
  },
```

As you can see if there is no data the function goes ahead and stores a **'habits'** key with an object consisting of a **name** and **days** attribute.  If it does find a **habits** in storage it uses the last **habit** object in the array and sets state once the component is mounted.

Should probably pull the **checked** code out into it's own function…

While we're up here adjust the **getInitialState** function:

```
  getInitialState: function() {

    return {
      habits: [],
      habit: {name: '', days: []},
      text: '',
      checked: false,
      editHabit: true,
    }
  },
```

It's basically the same except we're adding the new **habits** array and the **habit** attribute is an empty object instead of an empty string.

## saveHabit

 We also need to change up the **saveHabit** function:

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

      // Determine if the habit has been checked.
      var day = storedHabit[0].days.findIndex(function(day, index, days) {
        if (day.dayId == dayKey) {
          return true;
        }
      });

      var checked;
      if (day !== -1) {
        checked = true;
      } else {
        checked = false;
      }
      this.setState({habits: habits, habit: storedHabit[0], editHabit: false, checked: checked}, function() {
        store.save('habits', this.state.habits);
      });
    } else {
      // Create new Habit.
      var habit = {name: this.state.text, days: []};
      var habits = this.state.habits;
      habits.push(habit);

      this.setState({
        habits: habits,
        habit: habit,
        editHabit: false,
        checked: false
      }, function() {
        store.save('habits', this.state.habits);
      })
    }
  },
```

There's quite a bit more to this function then the earlier version.  First, we find the current **habit** index via the **findIndex** method.

If the habit is in the habits array it is blasted off of it via the **split** method and pushed onto the end of **this.state.habits** via the new **habits** variable (cause you don't want to modify the state attributes outside of setState).  The habit is checked for a day matching today and then the new **habits** array, habit, etc are saved back to the state.

If the habit is not found in **this.state.habits** a new Habit object is created and pushed onto the **habits** array then the whole shebang is saved back to the store.

Hrrmmmmm, bunch of cleanup possible with this function too…

## addDay

Now edit the **addDay** function:

```
  addDay: function() {
    if (this.state.habit) {
      day = this.state.habit.days.findIndex(function(day, index, days) {
        if (day.dayId == dayKey) {
          return true;
        }
      });

      if (day === -1) {
        // Create a new day.
        var newDay = {dayId: dayKey, created_at: Date.now(), habit: this.state.habit.name};

        // Update the Habit
        var habit = this.state.habits.pop();
        habit.days.push(newDay);

        // Update this.state.habits with the new Habit.
        var habits = this.state.habits;
        habits.push(habit);

        // Update state.
        this.setState({habits: habits, habit: habit, checked: true});

        // Store the new habits.
        store.save('habits', this.state.habits);
      }
    } else {
      this.setState({editHabit: true});
    }
  },
```

Like with the **saveHabit** function we're finding the habit in habits then checking for an entry for today, if there isn't a **day** for today create one, create a habit and habits variables from the state attribute, push the new day onto the habit, push the habit back on habits, update state, and store the new state.

## restartHabit

Reseting the chain is a little more complicated now too.  Edit the **restartHabit** function with:

```
  restartHabit: function() {
    var habit = this.state.habits.pop();
    habit.days = [];

    var habits = this.state.habits
    habits.push(habit);

    this.setState({habits: habits, habit: habit, editHabit: false, checked: false});
    store.save('habits', this.state.habits);
  },
```

Here the last habit is popped off of **this.state.habits**, the habit.days is reset to an empty array, the new habit is pushed back to the habits variable, then the state is saved and then stored.

## Updating the Component

With all the supporting functions (or are they methods?…) adjusted we can adjust the **render** function.  The form logic and components are okay, but we need to change most everything after them:

```
    var chains;
    if (this.state.habit.name != '') {
      chains =  <View style={styles.chains}>
                  {this.state.habit.days.map(function(day, index) {
                    return <Image key={day.dayId} style={styles.icon}
                            source={index % 30 == 0 && index != 0 ? require('./img/chain-icon-green.png') : require('./img/chain-icon.png')} />;
                  })}
                </View>
    } else {
      chains = <View></View>;
    }

    return (
      <View style={styles.container}>
      <ScrollView style={[styles.mainScroll]} automaticallyAdjustContentInsets={true} scrollEventThrottle={200} showsVerticalScrollIndicator={false}>
        <View style={styles.wrapper}>
          <View style={styles.shadow}>
            <TouchableWithoutFeedback onLongPress={this.editHabit} onPress={this.addDay}>
              <View style={[styles.habit, this.state.checked && styles.checked]}>
                <Text style={styles.habitText}>{this.state.habit.name != '' ? this.state.habit.name : 'No habit configured...'}</Text>
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

          <Text style={styles.days}>{this.state.habit.days ? this.state.habit.days.length : '0'} link{this.state.habit.days.length == 1 ? '' : 's'} in the chain.</Text>
        </View>

        <ScrollView style={[styles.scroll]} automaticallyAdjustContentInsets={true} scrollEventThrottle={200}>
         {chains}
        </ScrollView>
        </ScrollView>

        <Button text={'Share'} imageSrc={require('./img/share-icon.png')} onPress={this.onShare} textType={styles.shareText} buttonType={styles.shareButton} />
      </View>
    )
```

Basically we need to change **this.state.days** to **this.state.habit.days** and things will be good to go.

## Conclusion

Fun times, and good stuff.  Though it's not recommended to try and change more than one habit at a time (something about not being able to focus on more than one thing at a time) I think it's cool that the app can have a history of different habits that been worked on.

Like they say, the habit makes the man… or is that clothes?

Party On!