# React Native Sharing Habits

## Go On and Tell The World Why Dontcha

So in [our](http://devblog.thehoick.com/react-native/learning/2016/01/05/react-native-habit-app.html) [little](http://devblog.thehoick.com/react-native/learning/2016/01/07/react-native-saving-habits.html) React Native app we can keep track of a Habit and even save the results for later.  Coming along nicely.  

The next feature that'd be nice to have is to be able to tell all our friends about our progress.  You know on social media.  You know for support and what not.

To that end we'll add a cool little share button at the bottom to pull up additional apps to create a post, tweet, email, etc and populate it with some text.

<!—more—>

## React Native Sharing

Turns out there is already a great library for implementing the type of share button that we're going to need.  The [react-native-share](https://github.com/EstebanFuentealba/react-native-share) npm is what we'll use.

The process of installing and configuring the library is a little involved, but if you've every cracked open Xcode, or developed an Android app in Java, then there isn't too many surprises.  The official docs stay to install the npm with:

```
npm i --save react-native-share
```

We're not going to do that though because in this [issue](https://github.com/EstebanFuentealba/react-native-share/issues/6) it says to use this in **package.json** instead:

```
    "react-native-share": "lifuzu/react-native-share"
```

So go ahead and add that to the file and then execute:

```
npm install
```

The issue is with React Native 0.16, so at some point the react-native-share npm will be updated and you can just fire off a ```npm install```, but in the mean time…

**Note:** If you have ```npm run start``` running in a console somewhere you might have to restart it after installing the new npm package.

Boom, all set!  Well not quite.  To add the library to Android we need to edit a few files.

### Android

First, edit **android/setting.gradle** replace the current **include ':app'** statement with:

```
include ':react-native-share', ':app'
project(':react-native-share').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-share/android')
```

This threw me the first time because I simply appended the two lines, but as you can see the **include** statement for ':app' is still there and I guess you only need it in there once (or the 'react-native-share' include needs to come before… either way). 

Next, edit **android/app/build.gradle** (pay close attention to the **android/app** part there is also a **build.gradle** in the **android** folder, but editing that file won't work) and add the following to the **dependencies**:

```
    compile project(':react-native-share')
```

Finally, add a couple of lines to the **android/app/src/main/java/$project_folders/MainActivity.java**.  At the beginning of the file after the last **import** statement (or at least that's where I blasted mine in) add:

```
import cl.json.RNSharePackage;
```

Then further down in the file inside the **MainActivity** class you'll see an **.addPackage(new MainReactPackage())** statement.  Add the following after it:

```
.addPackage(new RNSharePackage())
```

Pow! Now things should be ready for sharing in Android…. Well the library is setup anyway.

### IOS

The iOS instructions for react-native-share are a little simpler than for Android, but the current Readme docs might be a little confusing.  Here's what I did to enable the library:

* Open XCode.
* Right click on **Libraries** > Click Add Files To "YOUR PROJECT NAME".
* Browse to your project folder/node_modules/react-native-share and select RNShare.xcodeproj.
* This will add RNShare.xcodeproj under the Libraries and if you expand the library and expand the **Products** folder you should see the **libRNShare.a** file in red.
* Next, click the **project_name** at the top level in XCode.
* Click the **Build Phases** tab.
* Expand **Compile Sources** and click the **"+"** button.
* Choose the **libRNShare.a** file under **RNShare.xcodeproj**

Now rebuild the project and things should build successfully for iOS.

I hope you've been able to follow my instructions, but you should totally check out the [official docs](https://github.com/EstebanFuentealba/react-native-share) if you have any problems/questions.

## Share Button

With the installation and configuration out of the way we can get back to business and code up some React.  

Edit **src/main.js** and import the react-native-share module at the top of the file:

```
var Share = require('react-native-share');
```

Next, add a new Button component after the ScrollView in the **render** function:

```
        <Button text={'Share'} imageSrc={require('./img/share-icon.png')} onPress={this.onShare} textType={styles.shareText} buttonType={styles.shareButton} />
```

Now add a new **onShare** function to the component:

```
  onShare: function() {
    Share.open({
      share_text: 'Habit Progress',
      share_URL: 'For my ' + this.state.habit + ' habit I have done ' + this.state.days.length + ' days in a row.  Yay for progress! #thehoickhabitapp',
      title: 'For my ' + this.state.habit + ' habit I have done ' + this.state.days.length + ' days in a row.  Yay for progress! #thehoickhabitapp',
    },function(e) {
      console.log(e);
    });
  },
```

Now add some styles for the button to the styles object:

```
  shareButton: {
    borderColor: '#DFD9B9',
    borderRadius: 0
  },

  shareText: {
    textAlign: 'center',
    color: '#DFD9B9',
    paddingTop: 2
  },
```

## Share Icon

To add an image/icon to our Button component we need to make some minor tweaks.  Edit the **src/components/button.js** file and change the render function to:

```
  render: function() {
    var image;
    if (this.props.imageSrc) {
      image = <Image source={this.props.imageSrc} style={styles.shareIcon} />;
    } else {
      image = <View></View>;
    }

    return (
      <TouchableHighlight style={[styles.button, this.props.buttonType]} underlayColor={'gray'} onPress={this.props.onPress}>
        <View>
          {image}
          <Text style={[styles.buttonText, this.props.textType]}>{this.props.text}</Text>
        </View>
      </TouchableHighlight>
    )
  }
```

Then add some styles for the **shareIcon** (this should totally be made more dynamic, but for now…):

```
  shareIcon: {
    padding: 5,
    paddingBottom: 7,
    alignSelf: 'center',
    justifyContent: 'center',
  }
```

Now things should look pretty sweet!

## Conclusion

Sharing in the simulator isn't really a good test for the new Share button so I highly recommend whipping out a USB cable and installing the app on an actual device with some more sharing options then email.  In fact trying to share to email on iOS crashed a service on my Mac… Gmail app worked fine on Android though.

Overall the react-native-share library is great and getting things working wasn't too much of a pain.

Party On!