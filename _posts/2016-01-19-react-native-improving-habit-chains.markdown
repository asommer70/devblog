---
title:  "React Native Improving Habit Chains"
date:   2016-01-19 13:05:00
categories: react-native learning
layout: post
image: rn_improvements.jpg
---

## Intro

The next feature for the [Habit App](https://github.com/asommer70/thehoick-habit-app) is to track weather you executed on the habit every day or not.  That means we'll need to have an entry for every day no matter what.

The way I implemented this is to add a **checked** boolean attribute to our **day** objects that are attached to a **habit**.  We can then assign an appropriate icon based on that attribute.

The tricky part is adding entries for missed days if the app hasn't been used for a while.  For that we'll need to do some date math.

<!--more-->

## Momement.js

A very popular (or at least I think it's popular) date library in JavaScript is [Moment.js](http://momentjs.com/).  This library is great because it allows you to easily add, subtract, format, create, etc dates.  Date maths are often a pain to deal with, but thankfully we'll be sticking to figuring out the number of days from one date to another.

To install moment.js execute:

```
npm install —save moment
```

Remember if you have ```npm run start``` running in a terminal window you'll need to restart it after installing moment.js.

Once things are installed edit the **src/main.js** file and adjust the variables at the top of the file:

```
var today = moment();
var dayKey = today.format('MMDDYYYY');
var day;
```

Setting **today** to a Moment object will allow us to do some manipulations down the line.

## Adjust addDay

Now, update the **addDay** function:

```
  addDay: function() {
    if (this.state.habit) {
      // Find out if there is an entry in days for today.
      day = this.state.habit.days.findIndex(function(day, index, days) {
        if (day.dayId == dayKey) {
          return true;
        }
      });

      // If no entry create one.
      if (day === -1) {
        var newDay = {dayId: dayKey, created_at: today.unix(), habit: this.state.habit.name, checked: true};
        var habit = this.state.habits.pop();

        if (habit) {
          // Find the number of days between today and the last day recorded.
          var lastDay = habit.days[habit.days.length - 1];

          if (lastDay !== undefined) {
            console.log('lastDay:', lastDay);
            var momentLastDay = moment.unix(lastDay.created_at);
            var diffOfDays = today.diff(momentLastDay, 'days');

            if (diffOfDays > 1) {
              // Do diffOfDays - 1 to exclude the lastDay entry from being added inside the loop.
              for (var i = diffOfDays - 1; i > 0; i--) {
                var momentBetweenDay = today.subtract(i, 'days');

                var betweenDay = {dayId: momentBetweenDay.format('MMDDYYY'), created_at: momentBetweenDay.unix(), habit: this.state.habit.name, checked: false }
                habit.days.push(betweenDay);
              }
            }
          }

          habit.days.push(newDay);

          // Update this.state.habits with the new Habit.
          var habits = this.state.habits;
          habits.push(habit);

          console.log('habits:', habits);

          // Update state.
          this.setState({habits: habits, habit: habit, checked: true});

          // Store the new habits.
          store.save('habits', this.state.habits);
        } else {
          this.setState({editHabit: true});
        }
      }
    } else {
      this.setState({editHabit: true});
    }
  },
```

My solution to the "adding back days" problem is to check for missing days when a new day is added.  To do that we're grabbing the last day and creating a Moment object from the **created_at** attribute (though we could now use the dayId), then using the Moment **diff** method we determine the number of days from today until the last day entered.

If there are more then one days without an entry they get added by looping backwards through the array and creating a new **day** object for each one.  Notice the use of the Moment **subtract** method.  This will create a Moment object for the day however many days since today.  

The new **day** objects are then pushed onto the **habit.days** array which is then used to update state and eventually saved to the store.

## Chains Components

The Chains components (the ScrollView that displays the link icons) needs to be updated to account for the now three options for an icon.  Edit the **render** function and replace the **this.habit.days.map** call with:

```
    var chains;
    var habitDays = this.state.habit.days;

      var chainIcons = habitDays.map(function(day, index) {
      var icon;
      if (index % 30 == 0 && index !=0) {
        icon = require('./img/chain-icon-green.png');
      } else {
        icon = require('./img/chain-icon.png');
      }

      if (day.checked == false) {
        icon = require('./img/broken-chain-left-icon.png');
      }

      return <Image key={day.dayId} style={styles.icon}
              source={icon} />;
    });

    if (this.state.habit.name != '') {
      chains =  <View style={styles.chains}>
                  {chainIcons}
                </View>
    } else {
      chains = <View></View>;
    }
```

This is basically the same as before only the **this.state.habit.days.map** call is pulled out of the JSX "{}" and placed into it's own section where we can then have more space to manipulate icon require.  

## Update the Link Count

Below the new Chains section add this code to determine the Link count statement:

```
    var checkedDays;
    var checks;
    if (habitDays.length >= 1) {

      // Need an array of checked days starting with today going back to the first unchecked day.
      checks = [];
      for (var i = habitDays.length; i > 0; i--) {
        if (habitDays[i - 1].checked) {
          checks.push(habitDays[i]);
        } else {
          break;
        }
      }
      checkedDays = checks.length;
    } else {
      console.log('balls...');
      checkedDays = '0';
      checks = [];
    }
```

This new code creates an array of days that have been **checked** until it comes to one that hasn't been.  Like before  the code loops backwards through the array until it comes to a "unchecked" day.  That new array is then used in the component.

Finally, update the count component inside the **return** statement:

```
          <Text style={styles.days}>
            {checkedDays} link{checks.length == 1 ? '' : 's'} in the chain.
          </Text>
```

To further test things out you can manipulate the **today** variable at the top:

```
// var today = moment();
var today = moment().add(1, 'days');
// var today = moment().add(2, 'days');
// var today = moment().add(7, 'days');
//var today = moment().add(8, 'days');
```

To make sure things are working I uncomment/comment one today for another.  The Moment **add** method is awesome for quickly finding out tomorrow, the next day, or a week from now.

## Conclusion

Quite a lot of features have been added since I set out to build a "simple" habit tracking app.  I guess when you start to think about all the possibilities of what an app can do it quickly becomes more than expected.  

I think there's some type of term for that… something to do with scopes???

Also, this project is due for a massive refactor to clean up some of the cruft, duplicate code, and make thins overall better organized.

Party On!
