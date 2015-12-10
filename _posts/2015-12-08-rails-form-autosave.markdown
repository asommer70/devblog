---
title:  "Form Auto-save with Rails"
date:   2015-12-08 13:05:00
categories: rails forms
layout: post
image: autosave.jpg
---

## LocalStorage to The Rescue

I’ve used/experimented with a few JavaScript auto-save libraries and some work better than others.  I guess that’s to be accepted.  I found that they worked the best when you weren’t doing a lot of other crazy JavaScript functions to the form.

At least I was able to easily break auto-save in some of my projects.

The answer to the problem was to roll my own solution.  I was heavily influenced by this excellent [post/video](https://css-tricks.com/video-screencasts/96-localstorage-for-forms/) on CSS Tricks where Chris details saving form data into localStorage.

Because I’m so into Rails lately I used CoffeeScript instead of straight up JavaScript.  I’ll go through the steps to add **auto-save** to our fun [Rails Forms](http://codepen.io/asommer70/post/rails-saving-form-data) app from previous posts.

<!--more-->

## CoffeeScript and localStorage

To enable auto-save edit the **app/assets/javascripts/inputs.coffee** file and at the beginning of the **ready_tables** function add:

```
  #
  # Setup some variables from the pathname to determine where to send data.
  #
  paths = window.location.pathname.split('/')
  action = paths[1]
  object_id = paths[2]
```

This will allow us to only fire certain functions if we’re on a certain page.  Next add an if statement before the **$.get** call to get the number of clicks for an add_row form:

```
  #
  # Click add-row the number of
  #
  if action == 'inputs' && object_id?
    $.get(window.location + '.json')
```

```
  #
  # Handle auto-save of Form data to localStorage if filling out a new form.
  #
  if action == 'forms'
    $form = $('.new_input')
    auto_save($form)
    restore_save($form)

auto_save = ($form) ->
  # Save the form data to localStorage.
  $form.find(':input').on 'blur', (e) ->
    form_data = $form.serializeArray()

    # Get the radio buttons and checkboxes.
    checkboxes = $form.find(':input').filter(':checkbox').serializeArray()
    radios = $form.find(':input').filter(':radio').serializeArray()
    form_data = $.merge(form_data, checkboxes)
    form_data = $.merge(form_data, radios)
    localStorage.setItem($form.attr('id'), JSON.stringify(form_data))

restore_save = ($form) ->
  # Restore Form data if there is some in localStorage.
  form_data = localStorage.getItem($form.attr('id'))
  if form_data?
    form_data = JSON.parse(form_data)

    # Have to loop the data twice to add dynamic rows then fill them in.
    for obj in form_data
      if obj.name.slice(-4) == '_rc]'
        # Find the add row button.
        obj_name = obj.name.split('[')[obj.name.split('[').length - 1]
        add_button_id = obj_name.substring(0, obj_name.length - 4) + "-add"
        $add_button = $('#' + add_button_id)

        # Click it the appropriate number of times.
        for i in [0...parseInt(obj.value)] by 1
          $add_button.trigger('click')

    # Fill in the fields.
    for obj in form_data
      $input = $("[name='#{obj.name}']")

      # Check for radio and checkboxes.
      if $input.is(':radio') and $input.val() == obj.value
        $input.attr('checked', true)
      if $input.is(':checkbox')
        if obj.value == 'on'
          $input.attr('checked', true)

      $input.val(obj.value)
```

The main part of the code is the **auto_save** and **restore_save** functions.  The **auto_save** function is fired by binding to the **blur** event of the input fields.  This will save the form data as JSON into local storage using the form *id* attribute.

Also, notice the **$.merge** function is used to save data for radio buttons and checkboxes.

The **restore_save** function checks localStorage for a key with the same name as the form *id* attribute and if it finds it creates an object from the JSON data.  The data is then looped and the value of the field is assigned by the key name of the JSON data.

Pretty straightforward and only a little more advanced than Chris’ great guide.


## Updating add_row

To get dynamic rows to save and restore we need update the **add_row** function in **app/assets/javascripts/inputs.coffee** to increment the **_rc** hidden field.  The **_rc** being the “row count” field used to keep track of how many dynamic rows have been added to a table.  Adjust the **app/views/forms/complex.html.erb** file changing the hidden field at the bottom of the table to:

```
            <input type="hidden" name="input[data][items_rc]" id="items_rc" data-table="items" />
```

In the **add_row** function in **app/assets/javascripts/inputs.coffee**  add these lines after the **count** variable has been set:

```
  # Update the _rc field.
  $('#items_rc').val(count)
```

And at the bottom of the **add_row** function add this code after the **$tbody.append** call:

```
  if window.location.pathname.split('/')[1] == 'forms'
    $form = $('.new_input')
    auto_save($form)
```

This will bind the **auto_save** functionality to the fields in the new row.

## Conclusion

Even though there are great auto-save libraries out there, sometimes it’s easier, or even necessary, to roll your own.

Also, a massive thanks to Chris for putting out such great content.

Party On!

Check out the example:

<p data-height="268" data-theme-id="0" data-slug-hash="wMBjYG" data-default-tab="result" data-user="asommer70" class='codepen'>See the Pen <a href='http://codepen.io/asommer70/pen/wMBjYG/'>Form Auto-Save</a> by Adam Sommer (<a href='http://codepen.io/asommer70'>@asommer70</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>
