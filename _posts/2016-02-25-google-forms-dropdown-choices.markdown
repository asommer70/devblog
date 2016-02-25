---
title:  "Google Form Dropdown Choices"
date:   2016-02-25 13:05:00
categories: google javascript
layout: post
image: google_sheets.jpg
---

## Forms Forms Forms

Seems like no matter how hard I try, I just can't seem to get away from forms.  Web forms, paper forms, mobile forms, etc… they're absolutely everywhere.

I guess that's what comes from the need to collect data.  And the whole Internet thing with browsers sending data back to the server and what not.

I recently brought on a new client that needs help with scheduling a series of appointments and adding them to a calendar.  Since they use Google Calendar, Gmail, etc I thought it'd be pretty easy to setup a simple [Google Form](https://www.google.com/forms/about/) backed by a [Google Sheet](https://www.google.com/sheets/about/) to collect data.

It was pretty simple once I dove in…

<!--more-->

## Setting Up the Form

First step in the process is to create a new form.  There are a plethora of ways to create a new form.  You can do it from the Google Form Chrome App, the Google Form page, or from the [Google Drive](https://www.google.com/drive/) site.  However you do it go ahead and create a new one.

Once you have a new Form created change the name of the Form in the title and the upper left hand corner (which is the actual Form file name) and change the first question to **Name** and a **Short Answer** type.  Or to whatever you'd like your first question to be.

Next, click the **Add Question** button (first one) in the little vertical icon menu down the right hand side.  Name the new question something descriptive like **Date Time** and set the type to **Dropdown**.

That's all there is to it for this simple form… well as far as questions and what not.

## Setting up the Sheet

With the Form all setup we can link it to a Spreadsheet (Google Sheet) and all the form responses will be stored there.  To link the Form to a Sheet we'll create a new one, but you can also link to an existing Google Sheet.

Inside the Form click on the **Responses** tab and click on the **Create Spreadsheet** icon in the right hand side of the menu (looks like a green square with a grid line in it).  In the **Select Response Destination** dialog that pops up choose a name for the new spreadsheet and click **Create**.

The new Spreadsheet will open up with a Sheet named **Form Responses 1**.  As you might guess you can link multiple forms to one spreadsheet.  

**Note:** Don't change the name of the Spreadsheet, or the Sheet, because if you do you'll have to relink them to the form.  

Next, create a new Sheet in the spreadsheet named **Available Dates**, or something similar, and in the first column add a bunch of date times like:

```
2/19/2016 8:00:00 GMT-0400 (EDT)
2/19/2016 9:00:00 GMT-0400 (EDT)
2/19/2016 10:00:00 GMT-0400 (EDT)
2/19/2016 11:00:00 GMT-0400 (EDT)
2/19/2016 12:00:00 GMT-0400 (EDT)
```

You can set the formatted date however you'd like.

## Customizing the Dropdown Choices

Now that we have a Form and it's linked to a Spreadsheet we can do some customizations with [Google Apps Script](https://developers.google.com/apps-script/).  App Script is a way to add on functionality, customizations, etc to Google Docs, Sheets, Forms, Gmail, etc using JavaScript.  It's pretty awesome!

We need a script to grab a range of dates in another sheet of our Spreadsheet and then add the dates to the **Date Time** drop down.

Open the App Script editor from inside the linked Sheet by clicking **Tools** then **Script Editor**.  This will open up the App Script editor for a new project.  Go ahead and set the title of the new project.

In the code editor change the name of **myFunction** to **getDates** or something similar.

Next, get the Form object via it's ID (the last part of the Form URL you see in the browser address bar) and current choices for the **Date Time** question:

```javascript
function getDates() {
  // Get the Form.
  var form = FormApp.openById('YOUR_FORM_ID');

  // Get the Date Time Item if it's there.
  var items = form.getItems();
  var datetimeItem;
  items.map(function(item, index) {
    if (item.getTitle() == 'Date Time') {
      datetimeItem = item;
    }
  });
}
```

Next the dates that are already taken need to be checked against the available dates.

```javascript
  // Get the list of chosen datetimes from the Form Responses sheet.
  var responseData = SpreadsheetApp.getActive().getSheets()[0];
  var takenDateValues = responseData.getDataRange().getValues();
  var takenDates = takenDateValues.map(function(value, index) {
    return value[2];
  });
  takenDates.shift();
```

With the **takenDates** array setup get the values from the **Available Dates** sheet:

```
// Get the Date Time Available sheet.
  var dateData = SpreadsheetApp.getActive().getSheets()[1];

  // Get the values from the cells.
  var range = dateData.getDataRange();
  var values = range.getValues();

// Create an array of dates from column A date time values.
  var dates = values.map(function(value, index) {

    // If the value hasn't been taken return it.
    var d = takenDates.checkDate(value[0]);
    if (d === false) {
      return value[0];
    } else {
      return 'Taken';
    }
  });
```

Notice the **takenDates.checkDate** method.  We'll add that below, but first add a the Date Time question if it's not there and add the choices:

```Javascript
  // Create the Date Time item and set it's choices to the dates array.
  if (datetimeItem === undefined) {
    datetimeItem = form.addListItem();
    datetimeItem.setTitle('Date Time')
  }

  dates.shift('Balls...');

  Logger.log(datetimeItem);
  datetimeItem.asListItem().setChoiceValues(dates);
```

Finally, below the **getDates** function add the **checkDates** method:

```javascript
//
// Compare dates.
//
Object.prototype.checkDate = function(dateValue) {
  var todayStamp = new Date().getTime() / 1000;
  var dateStamp = Date.parse(dateValue);

  for(var prop in this) {
    // Create timetamp from date strings and compare them.
    // Also check to make sure date isn't older than today.
    if (Date.parse(this[prop]) === Date.parse(dateValue)) {
        return true;
    } else if (dateStamp < Date.now()) {
      return true;
    }
  }
  return false;
}
```

Notice the method also checks for dates that have already happened.

## Running the App Script

The script can be run from the App Script Editor by selecting the **getDates** function in the **Select Function** dropdown in the top menu bar and clicking the **Run** button.

To have to script run, and update the choices, each time the form is submitted click **Resources** then **Current Project's Triggers** in the menu.  Click the link to setup a new trigger.

Run the **getDates** function, select **From Spreadsheet**, and then select **On Form Submit** and click the **Save** button.

Now, the choices will be updated each time someone chooses a date and the date they chose will show up as 'Taken' for the next users.

You might also want to add a trigger for the spreadsheet **On Edit** event because that will update the form when the dates in the **Available Dates** sheet are updated.

## Conclusion

Google Forms + Sheets + App Script is a powerhouse of for free data gathering.  I'm sort of surprised more people don't use it.  Then again maybe they do and I just don't know about it.

I guess one draw back is that you're sort of limited to using the Form styling and sending data to a Spreadsheet.  Sometimes you want to seriously brand your form.  Also, sending data to a database has it's advantages.

Party On!
