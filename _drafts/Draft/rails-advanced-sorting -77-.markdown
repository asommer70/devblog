# Rails Advanced Sorting

## Well More Advanced Then Last Time

In the [last post](http://devblog.thehoick.com/rails/learning/2015/12/24/rails-sorting.html) we explored a simple sorting option for Rails.  In this post we’ll work on more advanced sorting with multiple models (or tables if you prefer) and some additional advanced options.

To be clear this really isn’t all that advanced, but it is more advanced then simply sorting on one attribute of a model.  I’ve added the advanced sorting to the [BCN](https://github.com/asommer70/bcn) project and will go over the changes that were made to add this functionality.

## Sortable Helper

Let’s start off this post by editing the **app/helpers/post_helpers.rb** file.  Add the following **sortable** method:

```
  def sortable(title, column, type)
    if (type == 'events')
      path = events_path
    end

    path = path + "?sort=#{column}"

    color = sort_column == column ? 'secondary' : ''

    link_to path, class: "button small #{color}", id: column do
      "#{title}".html_safe
    end
  end
```

This **sortable** method is somewhat simpler than the our previous version because we’re not worried about the sorting direction.  I figured if someone wants to see the first Posts created they can use the pagination at the bottom of the page.

The other advancement is the **color** variable.  This is set based on the **params[:sort]** sent to the controller.  It’s great to be able to set the button color without having to hack up some JavaScript.

## Events Template

Next, cause we’re starting with Events, edit the **app/views/posts/events.html.erb**:

```
<div class="small-12 large-12 columns">

  <h1>Events</h1>
  <p class="grey">What's going on in the Boone community...</p>
  <hr/>
  <br/>

  <div class="row">
    <div class="columns large-2 no-left-padding">
      <%= link_to new_post_path, class: 'button small' do %>
        <%= image_tag 'plus-icon.svg', class: 'ty-icon' %>
        &nbsp;
        New Post
      <% end %>
    </div>

    <div class="columns large-10 left">
      <span>Sort Events By:</span> &nbsp;
      <ul class="button-group sorter">
        <li><%= sortable 'Start Date', 'start_date', action_name %></li>
        <li><%= sortable 'Organization', 'organization', action_name %></li>
        <li><%= sortable 'Community', 'community', action_name %></li>
      </ul>
    </div>
  </div>

  <br/>

  <ul class="posts no-bullet">
    <% @events.each do |post| %>
        <%= render 'layouts/posts_index_list', post: post %>
    <% end %>
  </ul>

  <br/><br/>

  <%= will_paginate @events, renderer: FoundationPagination::Rails %>
  <br/><br/>
</div>
```

I’ve included the entire file here because it’s not too large and I’m not sure if everyone has been following along.  Notice the call to **sortable** in the button group *ul > li* elements.  The call supplies the link title, sort column, and type in this case it’s set to **action_name** so that we can use **index** for the regular Posts index page.

## Posts Controller

Now edit the **app/controllers/posts_controller.rb** file and adjust the **events** method like so:

```
    case params[:sort]
    when 'organization'
      @events = Post.where(['start_date = ? or start_date > ?', DateTime.now, DateTime.now]).includes(:organization).order('organizations.name asc')
        .order(:start_date).paginate(:page => params[:page], :per_page => 20)
    when 'community'
      @events = Post.where(['start_date = ? or start_date > ?', DateTime.now, DateTime.now]).includes(:communities).order('communities.name asc')
        .order(:start_date).paginate(:page => params[:page], :per_page => 20)
    else
      @events = Post.where(['start_date = ? or start_date > ?', DateTime.now, DateTime.now]).order(:start_date).paginate(:page => params[:page], :per_page => 20)
    end
```

This is where the magic truly happens.  We now have two **case** statements in this method.  One for the **params[:sort]** and one for the **params[:events]** (not shown, but it’s used on the home page… trust me).  Inside the **params[:sort]** case statement we’re checking to sort by Organization, Community, and with a default of Post start_date.

The thing to really pay attention to is the **includes** method.  From my understanding this method lets you include additional tables in your query.  Not sure where you’d use that exactly, but another thing the includes method let’s you do is sort the results by columns in the included table using the **order** method.

The thing I had to get my head around is using the singular **:organization** (object name) in the **includes** method and the plural table name in the **order** method.   Also, notice the chained double **order** method calls.  Just cause we want to sort by the name of an Organization, or Community, doesn’t mean we still don’t want to order by Post start_date as well.

Finally, at the bottom of the **app/controllers/posts_controller.rb** inside the **private** section add this method:

```
    def sort_column
      params[:sort] if params[:sort]
    end
```

And back at the beginning of the file add:

```
  helper_method :sort_column 
```

The **sort_column** method simply returns the **params[:sort]** if it’s there.  This is then made available to our **sortable** method in the **app/helpers/post_helpers.rb** file.  We’ve now come full circle.

## Conclusion

Hopefully this more advanced, though still not too advanced, dive into Rails sorting is helpful.  It was a bunch of fun for me to add sorting to multiple Rails projects and get better and better each time.

I guess that’s how programming works though isn’t it…

Party On!