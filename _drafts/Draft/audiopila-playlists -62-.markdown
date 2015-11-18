# Playlists Are The New Mix Tape

## Playlists For One and All

It’s so frustrating to have to recreate your favorite playlists in a new music service.  I’m hoping that implementing the feature in [Audio Pila!](https://github.com/asommer70/audiopila-rails) will be the last time I’ll have to migrate them.  To be honest I don’t have all that many, but like a good mix tape a good playlist is great for using someone else’s art to express your emotions.

Or something like that…

## Some Tests First

Before whipping up some playlists, we should go back and add some tests for the Album objects.  Specifically I’d like to test adding Audios to Albums from the Album edit page and adding an Album to an Audio on the Audio edit page.

For our first test edit the **spec/features/albums/create_spec.rb** and add:

```
it 'creates a new album and adds an audio', js: true do
    visit '/audios/new'

    fill_in 'Name', with: 'When the Lights Go Out.mp3'
    fill_in 'Path', with: '/Users/adam/Music/When the Lights Go Out.mp3'
    click_button 'Save Audio File'
    audio = Audio.last

    visit albums_path

    click_link 'New Album'

    fill_in 'Name', with: 'Rubber Factory'
    fill_in 'Artist', with: 'The Black Keys'
    fill_in 'Year', with: '2004'
    fill_in 'Genre', with: 'Rock'

    audios_field = find('input.default')
    audios_field.set(audio.name)
    find('.active-result').click
    click_button 'Save Album'

    album = Album.last

    expect(page).to have_content('Album was successfully created.')
    expect(page).to have_content('Rubber Factory')
    expect(album.audios.count).to eq(1)
  end
```

This tests adds creating an Audio then adding it to a new Album.   The reason the Audio creation steps are in there is because the test uses JavaScript (the Poltergeist browser driver) to execute the test.  If you create the Audio in the test it won’t be available when creating the Album because the “browser” is running in a different thread.  At least that’s what I understand about it.  At some point we might go back and use a fixture library like [Factory Girl](https://github.com/thoughtbot/factory_girl) to create things.  Again, I think that’s what it’s for from my understanding.

So let’s test the “opposite” functionality open **spec/features/audios/create_spec.rb** and add:

```
  it 'creates an audio with an album', js: true do
    visit albums_path

    click_link 'New Album'
    fill_in 'Name', with: 'Rubber Factory'
    fill_in 'Artist', with: 'The Black Keys'
    fill_in 'Year', with: '2004'
    fill_in 'Genre', with: 'Rock'
    click_button 'Save Album'
    album = Album.last

    visit '/audios/new'

    fill_in 'Name', with: 'When the Lights Go Out.mp3'
    fill_in 'Path', with: '/Users/adam/Music/When the Lights Go Out.mp3'
    #album_field = find('input.default')
    #album_field.set(album.name)
    find('.chosen-single.chosen-default').click
    find('li.active-result.highlighted').click
    click_button 'Save Audio File'

    audio = Audio.last

    expect(page).to have_content('Audio was successfully created.')
    expect(page).to have_content('When the Lights Go Out.mp3')
    expect(page).to have_content('Rubber Factory')
    expect(audio.album).to eq(album)
  end
```

This new test create an Album then the Audio adding the Album from the dropdown field.  Coooooolllll…

## Playlist Model

Instead of using the scaffold for creating our Playlist object we’ll just copy the setup for Albums since they’re basically the same thing.  Well mostly.

I’m sure there’s a better way of doing things like using polymorphic tables or something, but having a separate entity for Playlists and Albums I think is easier to deal with mentally.  At least for my mind balls.

We will use a Rails migration to create the **playlists** table though.  Create the migration with:

```
bin/rails g migration create_playlists
```

Then edit the **db/migrate/DATESTAMP_create_playlists.rb** file and add this inside the **change** method:

```
    create_table :playlists do |t|
      t.string :name
      t.string :description
      t.string :image
      t.string :image_uid
      t.string :image_name
      t.integer :current_audio index: true

      t.timestamps null: false
    end

    create_join_table :audios, :playlists
```

And apply the migration:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

Finally, copy the **app/models/album.rb** file to **app/models/playlist.rb** and edit it:

```
class Playlist < ActiveRecord::Base
  validates :name, presence: true

  has_and_belongs_to_many :audios, -> { order 'album_order asc' }

  dragonfly_accessor :image
end
```

As you can tell basically the word Album was changed to **Playlist** and the **has_many** association for Audios is a **has_and_belongs_to_many** for Playlists.  Cause you know you might want to add an Audio to multiple Playlists.

Speaking of which, edit the **app/model/audio.rb** file and add:

```
  has_and_belongs_to_many :playlists
```

This will setup the other side of the Playlist - Audio relationship.

## Playlist Controller

Like with the model copy the **app/controllers/albums_controller.rb** file to **app/controllers/playlists_controller.rb** and edit it changing “Album” to “Playlist” (case intensively):

```
class PlaylistsController < ApplicationController
  before_action :set_playlist, only: [:show, :edit, :update, :destroy]

  # GET /playlists
  # GET /playlists.json
  def index
    @playlists = Playlist.all
  end

  # GET /playlists/1
  # GET /playlists/1.json
  def show
    @last_audio = Playlist.find(@playlist.current_audio) if @playlist.current_audio
  end

  # GET /playlists/new
  def new
    @playlist = Playlist.new
  end

  # GET /playlists/1/edit
  def edit
  end

  # POST /playlists
  # POST /playlists.json
  def create
    @playlist = playlist.new(playlist_params)

    respond_to do |format|
      if @playlist.save
        format.html { redirect_to @playlist, notice: 'Playlist was successfully created.' }
        format.json { render :show, status: :created, location: @playlist }
      else
        format.html { render :new }
        format.json { render json: @playlist.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /playlists/1
  # PATCH/PUT /playlists/1.json
  def update
    respond_to do |format|
      if @playlist.update(playlist_params)
        format.html { redirect_to @playlist, notice: 'Playlist was successfully updated.' }
        format.json { render :show, status: :ok, location: @playlist }
      else
        format.html { render :edit }
        format.json { render json: @playlist.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /playlists/1
  # DELETE /playlists/1.json
  def destroy
    @playlist.destroy
    respond_to do |format|
      format.html { redirect_to playlists_url, notice: 'Playlist was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_playlist
      @playlist = Playlist.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def playlist_params
      params[:playlist].permit(:name, :description, :image, :current_audio, :audio_ids => [])
    end
end
```

## Update the Nav

Now is a good time to add the **resource** routes for Playlists to **config/routes.rb**:

```
  resources :playlists
```

Next, edit **app/views/layouts/_nav.html.erb**:

```
  <dd class="<%= 'active' if params[:controller] == 'playlists' %>"><a href="/playlists">Playlists</a></dd>
```


## Playlist Views

For the Playlist templates copy the whole **app/views/albums** directory to **app/views/playlists** to get started.  Next, edit the **app/views/playlists/index.html.erb** file and change things up for Playlists:

```
<h1>Playlists</h1>

<%= link_to 'New Playlist', new_playlist_path, class: 'button small' %>
<br/><br/>

<% unless @playlists.blank? %>
  <ul class="album-list">
    <% @playlists.each do |playlist| %>
      <li>
        <%= link_to playlist.name, playlist_path(playlist) %>
      </li>
    <% end %>
  </ul>

<% else %>
  <p>
    No playlists... yet.  Create one <%= link_to 'here', new_playlist_path %>.
  </p>
<% end %>
<br>
```

Since we’ve now linked to the New Playlist page, edit the **app/views/playlists/new.html.erb** file:

```
<h1>New Playlist</h1>

<%= render 'form' %>

<%= link_to 'Back', playlists_path, class: 'button small' %>
```

Next, the **app/views/playlists/_form.html.erb**:

```
<br/><br/>

<%= form_for(@playlist) do |f| %>
  <% if @playlist.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@playlist.errors.count, "error") %> prohibited this playlist from being saved:</h2>

      <ul>
      <% @playlist.errors.full_messages.each do |message| %>
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
      <%= f.text_field :description, placeholder: 'Description' %>
    </div>
  </div>

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

  <div class="row">
    <div class="columns large-4">
      <%= f.label :image, 'Upload a cover image?'  %>
      <%= f.file_field :image %>
    </div>
  </div>

  <br/>

  <div class="actions">
    <%= f.submit 'Save Playlist', class: 'button small success' %>
  </div>
<% end %>

<% if action_name == 'edit' %>
  <%= link_to @playlist, method: :delete, id: 'delete', class: 'button tiny alert right' do %>
    <i class="fi-trash"></i>
  <% end %>
<% end %>

<br/><br/>
```

Now the **app/views/playlists/show.html.erb**:

```
<h2><%= @playlist.name %></h2>
<ul class="no-bullet">
  <li>
    <strong>Description:</strong> <%= @playlist.description %>
  </li>
</ul>

<div class="row">
  <div class="columns large-6">
    <h4>Play Playlist</h4>
    <div class="controls">
      <button class="warning large control change" data-function="-1" data-playlist="<%= @playlist.id %>"><i class="fi-previous"></i></button>
      <button id="play" class="warning large control" data-function="play" data-playlist="<%= @playlist.id %>"><i class="fi-play"></i></button>
      <button id="pause" class="warning large control" data-function="pause" data-playlist="<%= @playlist.id %>"><i class="fi-pause"></i></button>
      <button class="warning large control change" data-function="1" data-playlist="<%= @playlist.id %>"><i class="fi-next"></i></button>
      <br/>
      <button id="looper" class="warning small control" data-function="looper" data-playlist="<%= @playlist.id %>"><i class="fi-loop"></i></button>
      <button id="shuffle" class="warning small control" data-function="shuffle" data-playlist="<%= @playlist.id %>">
        <i class="fi-shuffle"></i>
      </button>
    </div>
    <div class="current-song">
      <h5>Current Audio</h5>
      <span id="playing_audio">
        <% if @last_audio %>
          <strong><%= @last_audio.name %></strong> was last played.
        <% else %>
          Don't know what was last played...
        <% end %>
      </span>
    </div>
  </div>
</div>


<br/><hr/><br/>

<% unless @playlist.audios.empty? %>
  <ul class="no-bullet">
    <% @playlist.audios.each do |audio| %>
      <li>

        <div class="row">
          <div class="columns large-6 text-center panel">
            <strong><%= link_to audio.name, audio %></strong>
            <br/>
            <%= image_tag @playlist.image.thumb('300x300').url if @playlist.image %>
            <audio id="<%= audio.id %>" controls data-audio="<%= audio.id %>" class="player">
              <% if audio.path.index('http') %>
                <source src="<%= audio.path %>"></source>
              <% else %>
                <source src="<%= stream_path(audio.id) %>"></source>
              <% end %>
            </audio>
          </div>
        </div>

      <% end %>
    </li>
  </ul>
<% end %>

<br/><hr/><br/>


<%= link_to 'Edit', edit_playlist_path(@playlist), class: 'button small' %> |
<%= link_to 'Back', playlists_path, class: 'button small' %>
```

Don’t forget to adjust the **app/views/playlists/edit.html.erb** file too:

```
<h1>Editing Playlist</h1>

<%= render 'form' %>

<%= link_to 'Show', @playlist, class: 'button small' %> |
<%= link_to 'Back', playlists_path, class: 'button small' %>
```

Also, change the **app/views/playlists/index.json.jbuilder** and **app/views/playlists/show.json.jbuilder** files:

```
json.array!(@playlists) do |playlist|
  json.extract! playlist, :id
  json.url playlist_url(playlist, format: :json)
end
```

```
json.extract! @playlist, :id, :name, :description, :audios, :current_audio, :created_at, :updated_at
```

Whoa, Pow!!! That was quite quick to get up and running with Playlists.  Well okay there’s still some work to do with actually playing a playlist, but we can save that for another post.

## Playlist Tests

But before we sign off let’s get some base tests going for Playlists.  Create a new file **spec/models/playlist_spec.rb** file with:

```
require 'rails_helper'

RSpec.describe Playlist, type: :model do
  let(:valid_attributes) {
    {
        name: 'Fun Stuff',
        description: 'Stuff that does not suck.',
    }
  }

  context 'validations' do
    it 'has name' do
      should allow_value('Fun Stuff').for(:name)
      should_not allow_value('').for(:name)
    end
  end

  context 'creation' do
    it 'can create playlists' do
      Playlist.create(valid_attributes)
      expect(Playlist.last).to be_truthy
      expect(Playlist.last.name).to eq('Fun Stuff')
    end
  end
end
```

Now create a new directory **spec/features/playlists** and inside it create **spec/features/playlists/create_spec.rb** (or you can copy the albums features directory and edit the files):

```
require 'rails_helper'

describe 'Create Playlists' do

  it 'creates a new playlist' do
    visit playlists_path

    click_link 'New Playlist'

    fill_in 'Name', with: 'Fun Stuff'
    fill_in 'Description', with: 'Stuff that does not suck.'
    click_button 'Save Playlist'

    expect(page).to have_content('Playlist was successfully created.')
    expect(page).to have_content('Fun Stuff')
  end

  it 'creates a new playlist and adds an audio', js: true do
    visit '/audios/new'

    fill_in 'Name', with: 'When the Lights Go Out.mp3'
    fill_in 'Path', with: '/Users/adam/Music/When the Lights Go Out.mp3'
    click_button 'Save Audio File'
    audio = Audio.last

    visit playlists_path

    click_link 'New Playlist'

    fill_in 'Name', with: 'Fun Stuff'
    fill_in 'Description', with: 'Stuff that does not suck.'

    audios_field = find('input.default')
    audios_field.set(audio.name)
    find('.active-result').click
    click_button 'Save Playlist'

    playlist = Playlist.last

    expect(page).to have_content('Playlist was successfully created.')
    expect(page).to have_content('Fun Stuff')
    expect(playlist.audios.count).to eq(1)
  end
end
```

Create/edit **spec/features/playlists/destroy_spec.rb**:

```
require 'rails_helper'

describe 'Destroy Playlist' do
  let!(:playlist) { Playlist.create(name: 'Fun Stuff', description: 'Stuff that does not suck.') }

  it 'destroys a playlist' do
    visit playlists_path

    click_link 'Fun Stuff'
    click_link 'Edit'

    find('#delete').click

    expect(page).to have_content('Playlist was successfully destroyed.')
  end
end
```

For the **spec/features/playlists/index_spec.rb**:

```
require 'rails_helper'

describe 'Displaying Playlists' do

  let!(:playlist) { Playlist.create(name: 'Good Stuff', description: 'Stuff that does not suck.') }

  it 'displays playlists' do
    visit playlists_path

    expect(page).to have_content('Good Stuff')
  end
end
```

Finally, edit/create the **spec/features/playlists/update_spec.rb**:

```
require 'rails_helper'

describe 'Update Playlist' do
  let!(:playlist) { Playlist.create(name: 'Good Stuff', description: 'Stuff that does not suck.') }

  it 'creates a new playlist' do
    visit playlists_path

    click_link 'Good Stuff'
    click_link 'Edit'
    fill_in 'Description', with: 'Stuff that mostly does not suck.'
    click_button 'Save Playlist'

    expect(page).to have_content('Playlist was successfully updated.')
    expect(page).to have_content('mostly')
  end
end
```

## Conclusion

Can’t express how great Rails is for taking care of the CRUD operations for you.  Totally makes things very fast to setup and get going.  Especially if you have models with similar attributes.

Playing the Audios in a Playlist may be a little more tricky than playing an Album, but then again maybe not so much different.  Get ready for some CoffeeScript…

Party On!