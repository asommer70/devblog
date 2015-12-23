---
title:  "Rails Sorting"
date:   2015-12-24 13:05:00
categories: rails learning
layout: post
image: sorting.jpg
---

## Sorty Sort Sort

Sorting with the ActiveRecord **order** method seems pretty simple, and to be honest the simple sorts I’ve used it for so far it’s been great.  Things do get a little complicated once you want to sort on a field in an associated object though.

For my larger forms project I’ve been working on sorting Input forms by Form name and the Input created_at field, and in this post we’ll walk through setting up some sorts in our [Rails Forms](http://codepen.io/asommer70/post/rails-saving-form-data).

I used this great [Rails Cast](http://railscasts.com/episodes/228-sortable-table-columns) as the basis of my solution.  The Forms project isn’t exactly the same since we’re not sorting a simple table of data, but using specific attributes of the objects.  Still a very good walk through and it also introduced me to a better way of using **helper_methods** in controllers to set “global” variables.

<!--more-->

## Inputs Controller

Jumping right in edit the **app/controllers/inputs_controller.rb** file and add this line at the top:

```
  helper_method :sort_direction
```

Then inside the **private** section add the **sort_direction** method:

```
    def sort_direction
      %w[asc desc].include?(params[:direction]) ? params[:direction] : 'asc'
    end
```

And in adjust the **@inputs** in the **index** method to like so:

```
  def index
    if params[:sort] == 'name'
      @inputs = Input.all.order("form_name #{sort_direction}")
    elsif params[:sort] == 'created_at'
      @inputs = Input.all.order("created_at #{sort_direction}")
    else
      @inputs = Input.all
    end
  end
```

Instead of the clever, and simple, database query used in the Rails Cast example we’re using a slightly more cluttered *if, elsif* statement to check the **params[:sort]** value.

## Index Template

We need some buttons to determine the sort order so edit the **app/views/inputs/index.html.erb** file and at the beginning add:

```
<br/>
<div class="row">
  <div class="columns large-12">
    <ul class="button-group">
      Sort by: &nbsp;
      <li><%= sortable 'Form', 'name', 'asc' %></li>
      <li><%= sortable 'Created Date', 'created_at', 'asc' %></li>
    </ul>
  </div>
</Div>
<br/>
```

The **sortable** method will be defined in a helper so don’t go refreshing anything quite yet.

## Sortable Helper

Open the **app/helpers/application_helper.rb** file and add this **sortable** method:

```
  def sortable(title, column, direction)
    direction = sort_direction == "asc" ? "desc" : "asc"
    icon = sort_direction == 'desc' ? 'down' : 'up'

    link_to inputs_path, class: 'tiny button secondary', id: column do
      "#{title} &nbsp; <i class=fi-arrow-#{icon}></i>".html_safe
    end
  end
```

This helper will create the links for the sort buttons which then add the query parameters for the sort options.

## Application.scss

We’re finally using the Foundation icons so if you haven’t (like I hadn’t) rename the **app/assets/stylesheets/application.css** file to **app/assetts/stylesheets/application.scss** and add the following line to enable icons:

```
@import 'foundation-icons';
```

## Conclusion

There’s a couple of other things we could add, some adjustment to the applied sort button for example, but we’ll leave that for another time.  Mostly because my young son has woken up and it’s time for me to stop writing on this post.

Also, multiple sort criteria would be awesome.  I know that I’ll need to add some sorting for Posts in the [BCN](https://github.com/asommer70/bcn) project soon so I’ll probably cover more advanced sorting then… or whatever solution I come up with.

Party On!
