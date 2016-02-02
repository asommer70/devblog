# Developing React Native Android Calendar Provider

## No React Native Reminders on Android

To complete the reminder/notification feature for my little [Habit App](https://github.com/asommer70/thehoick-habit-app) I realized that there was no app, feature, etc that is 100% comparable to the iOS Reminder app.  

There is the [Calendar Provider](http://developer.android.com/guide/topics/providers/calendar-provider.html#manifest) API that does pretty much the same thing.  Knowing all that the next step is to find a React Native module for that API.

Unfortunately while developing The Hoick Habit App no such module existed.  Since I made a small contribution to the [react-native-calendar-reminders](https://github.com/wmcmahan/React-Native-CalendarReminders) module, the obvious conclusion is to develop something equivalent for Android.

Thus I started developing the [React Native Android Calendar Provider](https://github.com/asommer70/react-native-android-calendar-provider) module.


## Finding a Mentor

There is great [documentation](https://facebook.github.io/react-native/docs/native-modules-android.html#content) on developing your own React Native Modules, but the docs don't cover the basic step one stuff of creating needed files and directories.  I guess they assume that anyone whacky enough to develop new React Native modules already knows how to develop straight up Android apps.  While I have done a couple of Android Java apps, I am far from good/professional at it.

Still you have to start somewhere…

I found a couple of projects that were very similar to what I was trying to do and basically copied the file structure and files they used.  The first one is based on [this great](https://medium.com/@sejoker/writing-android-component-for-react-native-e34802bf3377#.dtcznycas) article by [Yevgen Safronov](https://medium.com/@sejoker).  I used his example [repo](https://github.com/sejoker/react-native-android-vitamio) mentioned in the article.

The problem with that project is that its a UI component and has slightly different need than what I'm planning.  So I used the [Awesome React Native](https://github.com/jondot/awesome-react-native) to find additional projects that implement similar "backend" functionality to what I'm planning.  The next project example, and much closer to my idea, is the [react-native-android-geolocation](https://github.com/garysye/react-native-android-geolocation) module.

This project should get me much closer to where I want to be.

## Setting Up the Project

Following those to projects I created the new repo on Github and cloned it locally.  It's refreshing to start a new project with only a README.md and LICENSE file.

### Files

The first file I created was the **package.json** file.  This will tell *npm* how to install the project, where to send bugs, etc.

After that comes **build.gradle**.  I'm not very familiar with [Gradle](http://gradle.org/), but I know that it is used to build Android projects and is sort of like Make, Rake, etc… only you know, for Java.  I pretty much copied the build.gradle from the **react-native-android-vitamio** project and just changed the React Native version.

The **.gitignore** file was also copied from react-native-android-vitamio and it seems pretty standard React Native gitignore so not big surprises  there.

Now **index.js** was totally copied from the **react-native-android-geolocation** project.  This file's job is pretty much to export the JavaScript module.  At least that's what I think it mainly does.

### Directories

With the root project files setup it's time to move on to the actual Java directories and files.

Following my example projects, and standard Java convention I'm pretty sure, I created the **src** directory and inside of that the **main/java/** directories.

After that you get to the whole reverse DNS naming convention and I created **com/thehoick/android_calendar_provider** directories.  At this point I'm not entirely sure if I should have used something like RNAndroidCalendarProvider, RN_Android_Calendar_Provider, or rn_android_calendar_provider.  Might have to go back and change things up once I have a better understanding of what I'm doing.

Also, inside the **main/java** directory is the **AndroidManifest.xml** file.  This file tells Android apps how things are setup and what to start first.  I followed the react-native-android-geolocation project's example for this file, but changed the **uses-permission** elements to match those needed for the [Android Calendar Provider API](http://developer.android.com/guide/topics/providers/calendar-provider.html#manifest).

And that pretty much brings us to the Java class files and the guts of the project.

## AndroidCalendarProviderModule.java
