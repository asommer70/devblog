---
title:  "Audio Pila! Albums"
date:   2015-11-12 13:05:00
categories: rails audiopila
layout: post
image: album_cover.jpg
---

## Group of Songs

The next logical feature for [Audio Pila!](https://github.com/asommer70/audiopila-rails) is to group audio files by Album.  This also gives us the opportunity to upload an image to display while playing an album, song, etc.

Personally I enjoy the experience of listening to a series of songs that are in a specific order by someone who has taken the time to craft a great album.  You just don’t get that when you download a single song.  Though downloading one or two great songs is also awesome.

Cause there are a lot of bad albums out there.

<!--more-->

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
      t.integer :audios, index: true

      t.timestamps null: false
    end

    create_table :albums_audios do |t|
      t.belongs_to :album, index: true
      t.belongs_to :audio, index: true
    end
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
      params[:album].permit(:name, :artist, :year, :genre, audio_ids: [])
```

Also, because we configure the **year** field as a Date object we need to create one from the string sent via the form.  Add a new method in the **private** section:

```
    # Need to create a Date object before year is saved to the database.
    def modify_date
      year = Date.strptime(album_params[:year], '%Y') unless album_params[:year].blank?
      params[:album][:year] = year
    end
```

And at the top of the file add a **before_filter** for the *create* and *update* actions:

```
  before_filter :modify_date, :only => [:create, :update]
```

Now the **year** field should be saved as a Date object and we can do some date math to find albums in a certain time frame down the road.

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
    <strong>Released:</strong> <%= @album.year.strftime('%Y')  if @album.year %>
  </li>
  <li>
    <strong>Genre:</strong> <span class="label warning"><%= @album.genre %></span>
  </li>
</ul>

<br/><hr/><br/>

<% unless @album.audios.empty? %>
  <ul class="no-bullet">
    <% @album.audios.each do |audio| %>
      <li>
        <strong><%= audio.name %></strong>
        <br/>

        <audio id="<%= audio.id %>" controls data-audio="<%= audio.id %>" class="player">
          <% if audio.path.index('http') %>
            <source src="<%= audio.path %>"></source>
          <% else %>
            <source src="<%= stream_path(audio.id) %>"></source>
          <% end %>
        </audio>
      <% end %>
    </li>
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
  <%= link_to @album, method: :delete, id: 'delete', class: 'button tiny alert right' do %>
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

## Tests

A whole slew of tests were created when the Album object was scaffolded.  I think keeping the same layout/coverage for Albums as we have for the Audios object will work fine.  

To make things line up better I deleted the **spec/requests**, **spec/routing**, **spec/helpers**, **spec/views**, and **spec/controllers** directories.  Looking back might have been less hassle to skip creating tests for the Album… I think there’s a way to do that with the scaffolding command.

Either way we should now be ready to edit the **spec/models/album_spec.rb** file and add some tests:

```
require 'rails_helper'

RSpec.describe Album, type: :model do
  let(:valid_attributes) {
    {
        name: 'Rubber Factory',
        artist: 'The Black Keys',
    }
  }

  context 'validations' do
    it 'has name and path' do
      should allow_value('Rubber Factory').for(:name)
      should allow_value('The Black Keys').for(:artist)
      should_not allow_value('').for(:name)
      should_not allow_value('').for(:artist)
    end
  end

  context 'creation' do
    it 'can create repositories settings' do
      Album.create(valid_attributes)
      expect(Album.last).to be_truthy
      expect(Album.last.name).to eq('Rubber Factory')
    end
  end
end
```

Awesome, I feel confident that our Albums will be saved to the database.  Next, create a new directory **spec/features/albums** and a new file for index tests **spec/features/albums/index_spec.rb**:

```
require 'rails_helper'

describe 'Displaying Albums' do

  let!(:album) { Album.create(name: 'Rubber Factory', artist: 'The Black Keys') }

  it 'displays a repository' do
    visit albums_path

    expect(page).to have_content('Rubber Factory')
  end
end
```

Go ahead and run the test:

```
bin/rspec spec/features/albums/index_spec.rb
```

Things should work out fine and the test should pass.  If not there’s probably a problem somewhere (heh, I’m funny).

Now let’s test creating an Album.  Create a new file **spec/features/albums/create_spec.rb**:

```
require 'rails_helper'

describe 'Create Albums' do

  it 'creates a new album' do
    visit albums_path

    click_link 'New Album'

    fill_in 'Name', with: 'Rubber Factory'
    fill_in 'Artist', with: 'The Black Keys'
    fill_in 'Year', with: '2004'
    fill_in 'Genre', with: 'Rock'
    click_button 'Save Album'

    expect(page).to have_content('Album was successfully created.')
    expect(page).to have_content('Rubber Factory')
  end
end
```

To round things out create a **spec/features/albums/update_spec.rb** file to test editing an Album:

```
require 'rails_helper'

describe ‘Update Album' do
  let!(:album) { Album.create(name: 'Rubber Factory', artist: 'The Black Keys', genre: 'Rock') }

  it 'creates a new album' do
    visit albums_path

    click_link 'Rubber Factory'
    click_link 'Edit'
    fill_in 'Genre', with: 'Blues Rock'
    click_button 'Save Album'

    expect(page).to have_content('Album was successfully updated.')
    expect(page).to have_content('Blues Rock')
  end
end
```

And finally, create a test for destroying an Album in **spec/features/albums/destroy_spec.rb**:

```
require 'rails_helper'

describe 'Destroy Album' do
  let!(:album) { Album.create(name: 'Rubber Factory', artist: 'The Black Keys', genre: 'Rock') }

  it 'creates a new album' do
    visit albums_path

    click_link 'Rubber Factory'
    click_link 'Edit'

    find('#delete').click

    expect(page).to have_content('Album was successfully destroyed.')
  end
end
```

Awesome, that should cover the base CRUD (Create Read Update Delete) functionality of our Albums.

## Adding Audios to Albums

Obviously the next step is to actually be able to add an Audio to an Album.  There are a number of ways to accomplish this amazing feat of brilliance, and I’m sure we’ll do at least a couple of them during the life of this project, but to make it quick we’ll use an auto-complete select element in the Audio and Album forms.

There are a number of great auto-complete libraries out there and for this task we’ll use the [Chosen](https://github.com/harvesthq/chosen) library.  There is a gem for integrating Chosen with Rails, but for whatever reason I couldn’t get it to work with the version of Rails I was running so we’re going to download the files and use the Vendor asset pipeline to integrate them.  You can download Chosen [here](https://github.com/harvesthq/chosen/releases).

Once you have it downloaded, unzip the archive file and copy the **chosen.jquery.js** file to **vendor/assets/javascripts** and **chosen.css** to **vendor/assets/stylesheets**.

Also, copy the **chosen-sprite.png** and **chosen-sprite@2x.png** files to **app/assets/images/**.


Include the JavaScript for Chosen by adding the following to **app/assets/javascripts/application.js**:

```
//= require chosen-jquery
```

And the stylesheets by adding this line to **app/assets/stylesheets/application.css.scss**:

```
*= require chosen
```

Great, now Chosen is installed and ready to do some completing.  The next step is to add a Chosen (select field) to the Album form in **app/views/albums/_form.html.erb**:

```
  <div class="row">
    <div class="columns large-4">
      <%= f.label :audios %>
      <%= f.select :audio_ids,
                   Audio.all.map { |a| [a.name, a.id] },
                   { include_blank: true },
                   { class: 'chosen-select',
                     multiple: true,
                     data: { 'placeholder' => 'Start typing audio file name...' }
                   }
      %>
    </div>
  </div>

  <br/>
```

I added the field under the **genre** input element, but you can add it anywhere you’d like.

Now to make the chosen input field more Foundationy I created a new SASS stylesheet in **app/assets/stylesheets/foundation_chosen.scss** to override some of the default Chosen styles:

```
@import "chosen";
@import "foundation_and_overrides";

.chosen-container {
  font-size: 13px;
}

.chosen-container .chosen-drop {
  border: 1px solid #cccccc;
}

.chosen-container .search-choice .group-name, .chosen-container .chosen-single .group-name {
  color: rgba(0, 0, 0, 0.75);
}

.chosen-container-single .chosen-single {
  border: 1px solid #cccccc;
}
.chosen-container-single .chosen-default {
  color: rgba(0, 0, 0, 0.75);
}

.chosen-container-single .chosen-single abbr {
  background: image-url('chosen-sprite.png') -42px 1px no-repeat;
}

.chosen-container-single .chosen-single div b {
  background: image-url('chosen-sprite.png') no-repeat 0px 2px;
}

.chosen-container-single .chosen-search input[type="text"] {
  border: 1px solid #cccccc;
  background: white image-url('chosen-sprite.png') no-repeat 100% -20px;
  background: image-url('chosen-sprite.png') no-repeat 100% -20px;
  font-size: 14px;
}

.chosen-container .chosen-results li {
  font-size: 14px;
}

.chosen-container .chosen-results li.active-result {
  font-size: 14px;
}
.chosen-container .chosen-results li.disabled-result {
  font-size: 14px;
}

.chosen-container .chosen-results li.highlighted {
  background-image: none;
  color: #fff;
  font-size: 14px;
}

.chosen-container-multi .chosen-choices {
  border: 1px solid #cccccc;
}

.chosen-container-multi:active .chosen-choices:active {
  margin-bottom: 0;
}

.chosen-container-multi .chosen-choices li.search-field input[type="text"] {
  background: inherit;
}

.chosen-container-multi .chosen-choices li.search-choice {
  border-radius: none;
  background-image: none;
  color: rgba(0, 0, 0, 0.75);
}

.chosen-container-multi .chosen-choices li.search-choice .search-choice-close {
  background: image-url('chosen-sprite.png') -42px 1px no-repeat;
}

.chosen-container-multi .chosen-choices li.search-choice-disabled {
  background-image: none;
  background-color: inherit;
}

.chosen-container-active .chosen-single {
  border: none;
  box-shadow: none;
}

.chosen-container-active.chosen-with-drop .chosen-single {
  border: 1px solid #cccccc;
  background-image: none;
  border-radius: none;
  box-shadow: 0 1px 0 #fff inset;
}

.chosen-rtl .chosen-search input[type="text"] {
  background: white image-url('chosen-sprite.png') no-repeat -30px -20px;
  background: image-url('chosen-sprite.png') no-repeat -30px -20px;
}

@media only screen and (-webkit-min-device-pixel-ratio: 1.5), only screen and (min-resolution: 144dpi), only screen and (min-resolution: 1.5dppx) {
  .chosen-rtl .chosen-search input[type="text"],
  .chosen-container-single .chosen-single abbr,
  .chosen-container-single .chosen-single div b,
  .chosen-container-single .chosen-search input[type="text"],
  .chosen-container-multi .chosen-choices .search-choice .search-choice-close,
  .chosen-container .chosen-results-scroll-down span,
  .chosen-container .chosen-results-scroll-up span {
    background-image: image-url('chosen-sprite@2x.png') !important;
  }
}
```

Finally, the Chosen jQuery needs to be applied add the following to **app/assets/javascripts/albums.coffee**:

```
ready_albums = ->

  $('.chosen-select').chosen
    allow_single_deselect: true
    no_results_text: 'No results matched'

$(document).ready(ready_albums)
$(document).on('page:load', ready_albums)
```

Super, we can add Audio files to Albums from the Album form.

## Adding an Album to an Audio

To make things that more convenient, and to get more use out of the Chosen library, let’s add the ability to add an Audio to an Album from the Audio form.

Edit **app/views/audios/_form.html.erb**:

```
  <div class="row">
    <div class="columns large-4">
      <%= f.label :album %>
      <%= f.select :album_id,
                   Album.all.map { |a| [a.name, a.id] },
                   { include_blank: true },
                   { class: 'chosen-select'}
      %>
    </div>
  </div>

  <br/>
```

Now add the **album_id** parameter to the **audio_params** method in **app/controllers/audios_controller.rb**:

```
      params[:audio].permit(:name, :path, :playback_time, :album_id)
```

Finally, display the Album on the Audio show page by editing **app/views/audios/show.html.erb**:

```
<p>
  <strong>Album:</strong> <%= link_to @audio.album.name, @audio.album if @audio.album %>
</p>
```

I put the p tag under the *h2* element, but it could go anywhere on the page.

## Conclusion

It was quite a bit of work to get Albums setup in the app… or at least it felt that way.  There’s some more advanced features I’d like to add, but we’ll have room for that in the next post.

Party On!
