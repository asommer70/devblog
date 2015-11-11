# Audio Pila! Albums

## Group of Songs

The next logical feature for [Audio Pila!](https://github.com/asommer70/audiopila-rails) is to group audio files by Album.  This also gives us the opportunity to upload an image to display while playing an album, song, etc.

Personally I enjoy the experience of listening to a series of songs that are in a specific order by someone who has taken the time to craft a great album.  You just don’t get that when you download a single song.  Though downloading one or two great songs is also awesome. 

Cause there are a lot of bad albums out there.

## Scaffolding Albums

Like the Audios object we’ll use the **rails** command to generate the scaffold for your Albums class.  In a terminal enter:

```
bin/rails g scaffold album
```

You’ll notice that it creates a lot of new files.  Some we’ll use right away, some we’ll get to down the road, and others we might not use at all.  That’s okay cause we can always come back later and add functionality or more tests.

First thing we’ll do is delete the **app/assets/stylesheets/scaffolds.scss** file.  I like the Foundation styles as they are thank you very much.

## Database Migration

Create the **albums** table by editing the **db/migrate/TIMESTAMP_create_albums.rb** file adding the following inside the **change** method:

```
    create_table :albums do |t|
      t.string :name
      t.date :year
      t.string :artist
      t.string :genre
      t.references :audios, index: true

      t.timestamps null: false
    end

    add_reference :audios, :album, index: true, foreign_key: true
```

Then run the migration:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

## Model Updates

We have a scaffold of the Album model which is basically empty so edit **app/models/album.rb** and add:

```
class Album < ActiveRecord::Base
  validates :name, presence: true
  validates :artist, presence: true

  has_many :audios
end
```

I think for a basic album all you really need is a name and artist… good to know who made the album.  Now complete the association with the Audio model and edit **app/models/audio.rb** adding:

```
  belongs_to :album
```

Cool, now we can create an Album and assign a bunch of Audios to it.

## Albums Controller

I think the Albums controller is pretty good for the moment so let’s dive in and start making the view templates look/work.  Except for the **album_params** method which needs to permit the Album fields:

```
      params[:album].permit(:name, :artist, :year, :genre, audios: [])
```

Also, because we configure the **year** field as a Date object we need to create one from the string sent via the form.  Adjust the **update** method like so:

```
  def update
    year = Date.strptime(album_params[:year], '%Y') unless album_params[:year].blank?

    respond_to do |format|
      if @album.update(album_params)
        @album.year = year
        @album.save

        format.html { redirect_to @album, notice: 'Album was successfully updated.' }
        format.json { render :show, status: :ok, location: @album }
      else
        format.html { render :edit }
        format.json { render json: @album.errors, status: :unprocessable_entity }
      end
    end
  end
```

## Update the Navigation

Gotta know where you are to know where you want to go.  Update the **app/views/layouts/_nav.html.erb** partial with a link for the Albums index:

```
  <dd class="<%= 'active' if params[:controller] == 'albums' %>"><a href="/albums">Albums</a></dd>
```

I put it after the Audios link, but feel free to place it first or last if you think that works better.

## Adjust Album Templates

A good place to start is to replace the contents of the **app/views/albums/index.html.erb** file with:

```
<h1>Albums</h1>

<%= link_to 'New Album', new_album_path, class: 'button small' %>
<br/><br/>

<% unless @albums.blank? %>
  <ul class="album-list">
    <% @albums.each do |album| %>
      <li>
        <%= link_to album.name, album_path(album) %>
      </li>
    <% end %>
  </ul>

<% else %>
  <p>
    No albums... yet.  Create one <%= link_to 'here', new_album_path %>.
  </p>
<% end %>
<br>
```

It’s a little bit more minimal and cuts to the chase.

Next, edit the **app/views/albums/show.html.erb**:

```
<h2><%= @album.name %></h2>
<ul class="no-bullet">
  <li>
    <strong>Artist:</strong> <%= @album.artist %>
  </li>
  <li>
    <strong>Released:</strong> <%= @album.year.strftime('%Y') %>
  </li>
  <li>
    <strong>Genre:</strong> <span class="label warning"><%= @album.genre %></span>
  </li>
</ul>

<br/><hr/><br/>

<% unless @album.audios.empty? %>
  <ul class="no-bullet">
    <% @album.audios.each do |audio| %>
      <audio id="<%= audio.id %>" controls data-audio="<%= audio.id %>" class="player">
        <% if audio.path.index('http') %>
          <source src="<%= audio.path %>"></source>
        <% else %>
          <source src="<%= stream_path(audio.id) %>"></source>
        <% end %>
      </audio>
    <% end %>
  </ul>
<% end %>

<br/><hr/><br/>


<%= link_to 'Edit', edit_album_path(@album), class: 'button small' %> |
<%= link_to 'Back', albums_path, class: 'button small' %>
```

Now the **app/views/albums/_form.html.erb**:

```
<br/><br/>

<%= form_for(@album) do |f| %>
  <% if @album.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@album.errors.count, "error") %> prohibited this album from being saved:</h2>

      <ul>
      <% @album.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="row">
    <div class="columns large-4">
      <%= f.text_field :name, placeholder: 'Name' %>
    </div>
  </div>

  <div class="row">
    <div class="columns large-4">
      <%= f.text_field :artist, placeholder: 'Artist' %>
    </div>
  </div>

  <div class="row">
    <div class="columns large-4">
      <%= f.text_field :year, value: @album.year ? @album.year.strftime('%Y') : '', placeholder: 'Year' %>
    </div>
  </div>

  <div class="row">
    <div class="columns large-4">
      <%= f.text_field :genre, placeholder: 'Genre' %>
    </div>
  </div>


  <div class="actions">
    <%= f.submit 'Save Album', class: 'button small success' %>
  </div>
<% end %>

<% if action_name == 'edit' %>
  <%= link_to album_path(@album), method: :delete, id: 'delete', class: 'button tiny alert right' do %>
    <i class="fi-trash"></i>
  <% end %>
<% end %>

<br/><br/>
```

Might as well clean up the **app/views/albums/edit.html.erb**:

```
<h1>Editing Album</h1>

<%= render 'form' %>

<%= link_to 'Show', @album, class: 'button small' %> |
<%= link_to 'Back', albums_path, class: 'button small' %>
```

And **app/views/albums/new.html.erb**:

```
<h1>New Album</h1>

<%= render 'form' %>

<%= link_to 'Back', albums_path, class: 'button small' %>
```

If you’re thinking these are very similar to the Audio templates, then you’d be right.  Basically we’re getting the basic functionality down and we can come back later and add more customizations and features.


## Adding Audios to Albums


