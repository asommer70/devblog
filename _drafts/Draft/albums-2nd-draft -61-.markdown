# Albums 2nd Draft

## Albums The Complete Picture

There are some additional features needed for the Album object.  When outlining the features of the app creating Albums seemed like a very small step, but after the last post it turned out to be more complicated than I originally planned.  Not to mention much more time consuming.

Most of the delays had to do with getting a good auto-complete library setup.  I did try out a couple of others, but ended up going back with Chosen.  Which coincedently is the library I used in the [BCN](https://github.com/asommer70/bcn) project.  

Another sizable delay was getting the model setup correctly to “reference” back to the Audio object.  Creating the table is quite simple, but then you also have to create the “relationship” table that has ID numbers for Audios and their **belong_to** Album.

Once that was rocking things went much more smoothly…

## Album Covers

The first new feature for our Albums is going to be a cover image.  To make uploading image files easy we’ll use the [Dragonfly](http://markevans.github.io/dragonfly/) library and via gem.  This is a great library and makes it super easy to upload files and link to them in your view templaes.

### Installing Dragonfly

Edit the **Gemfile** and add:

```
gem 'dragonfly', '~> 1.0.12'
```

And bundle it:

```
bundle install
```

Next, execute:

```
rails generate dragonfly
```

You’ll notice that this creates the Dragonfly initializer in **config/initializers/dragonfly.rb**.  At this point Dragonfly is installed and we can now add it to our models.

### Adding Dragonfly to Albums

To add Dragonfly to a model we just add an accessor statement.  Edit **app/models/album.rb** and add the following line toward the top:

```
dragonfly_accessor :image
```

Next, the Album model needs to be updated with the new image fields.  Using the **rails** command create a new migration:

```
bin/rails g migration add_image_to_album
```

And in the **db/migrate/DATESTAMP_add_image_to_album.rb** file add the following to the **change** method:

```
    add_column :albums, :image_uid, :string
    add_column :albums, :image_uid, :string
    add_column :albums, :image_name, :string
```

Now apply the migration:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

### Updating the Album Form

Finally, adjust the **app/views/albums/_form.html.erb** file and add the following file input element:

```
  <div class="row">
    <div class="columns large-4">
      <%= f.label :image, 'Upload a cover image?'  %>
      <%= f.file_field :image %>
    </div>
  </div>

  <br/>
```

Also, don’t forget to update the **app/controllers/albums_controller.rb** file and add the **:image** filed to the permitted parameters in the **album_params** method:

```
      params[:album].permit(:name, :artist, :year, :genre, :image, :audio_ids => [])
```

### Displaying the Album Cover

To see what we’re working with as far as Album covers go edit the **app/views/albums/show.html.erb** file and add:

```
        <div class="row">
          <div class="columns large-6 text-center panel">
            <strong><%= audio.name %></strong>
            <br/>
            <%= image_tag @album.image.thumb('300x300').url if @album.image %>
            <audio id="<%= audio.id %>" controls data-audio="<%= audio.id %>" class="player">
              <% if audio.path.index('http') %>
                <source src="<%= audio.path %>"></source>
              <% else %>
                <source src="<%= stream_path(audio.id) %>"></source>
              <% end %>
            </audio>
          </div>
        </div>
```

Okay, this does a little more than just add an image tag to the page.  There’s a new row div and we’re wrapping the player Audio name and Album image in a Foundation panel div.  I think the affect looks pretty good.

The Audio show page should also have the Album image if it’s available. Edit **app/views/audios/show.html.erb** and change the player element to have:

```
<div class="row">
  <div class="columns large-6 text-center panel">
    <%= image_tag @audio.album.image.thumb('300x300').url if @audio.album && @audio.album.image %>
    <audio id="<%= @audio.id %>" controls data-audio="<%= @audio.id %>" class="player">
      <% if @audio.path.index('http') %>
        <source src="<%= @audio.path %>"></source>
      <% else %>
        <source src="<%= stream_path(@audio.id) %>"></source>
      <% end %>
    </audio>
  </div>
</div>
```

Now things are looking pretty good.