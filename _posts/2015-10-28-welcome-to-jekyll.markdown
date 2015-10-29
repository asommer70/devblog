---
layout: post
title:  "Rails Playing Audio Files"
date:   2015-10-29 13:05:00
categories: rails audiopila
image: music_cover.jpg
---

## Jamming With Rails

Since getting into podcasts and recently purchasing some Spanish courses on CD there has been a rather sharp increase in audio files on my local network.  Similar to the [DVD Pila!](http://dvdpila.thehoick.com) project I’d like to whip up a web app to help manage the growing audio file pile…

Time for Audio Pila! (Rails version).  I’m thinking to setup a similar service that will list audio files in a directory structure and allow me to play them.  It’d also be very cool to be able to create playlists for them.

<!--more-->

## Rails Project

There is no hoops to jump through this time to get access to an API, so go ahead and create a new Rails project.  In a terminal enter:

```
rails new audiopila-rails
```

If you have a different name feel free to apply it to the command above.  Also, to get Ruby and Rails setup see [this post](http://devblog.boonecommunitynetwork.com/ruby-rails-and-passenger/).

### Database

[PostgreSQL](http://www.postgresql.org/) is so awesome that we’ll use it for this project, but if you feel good about SQLite or another database Rails is great either way.  To add PostgreSQL to this project edit the **config/database.yml** file:

```
default: &default
  adapter: postgresql
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: audiopila_dev

test:
  <<: *default
  database: audiopila_test

production:
  <<: *default
  database: audiopila
```

### Gems

Grab the Foundation, and pg, gems to make things look good by adding the following to the project’s **Gemfile**:

```
gem 'pg'
gem 'foundation-rails'
gem 'foundation-icons-sass-rails'
```

Now setup Foundation and install any additional gems:

```
rails g foundation:install
bundle install
```

We still haven’t created the databases yet, so go ahead and create them using **rake**:

```
bin/rake db:create db:migrate
bin/rake db:create db:migrate RAILS_ENV=test
```

Or at least the **dev** and **test** databases.  I’m not sure yet where I”ll “host” this web app so I’m going to hold off on setting the production database.  Why do things now that can be put off until later?

## Settings Model

The little we app we’re building is going to need some setting setup for different things.  The first thing it needs is the location of a directory that contains audio files.  I’m to going to refer to this location as an audio repository, or just repository, and we should probably be able to add more than one repository to our app.  

You never know when you’ll need to add a second hard drive to your machine in order to have enough space for all your audio content.

To make saving, updating, and reading (heh CRUD) functionality easy we’ll use the [rails-settings-cached](https://github.com/huacnlee/rails-settings-cached) gem.  To install the gem edit the **Gemfile** adding:

```
gem 'rails-settings-cached', '~> 0.4.0'
```

Generate the **settings** table using a migration:

```
rails g settings settings
```

Lastly, create the table with rake:

```
bin/rake db:migrate
bin/rake db:migarte RAILS_ENV=test
```

## Settings Route

Add the following line to create resource routes for our new Settings objects in **config/routes.rb**:

```
  resources :settings, except: [:show, :new, :edit]
```

Notice, we’re leaving off the **show** method because we’ll just list them all on a single page that will also serve as a form to create, update, and/or destroy settings.

If your curious you can see all the routes in the app using **rake**:

```
bin/rake routes
```

This command is also very useful for troubleshooting Rails routing errors.

## Settings Controller

The **rails-settings-cached** gem only provides a model and library code to manipulate setting data.  It’s up to us to provide a way to list, save, update, and destroy settings.

To handle those things we’ll create a controller **app/controllers/settings_controller.rb** to save the settings we’ll need:

```
class SettingsController < ApplicationController

  def index
    @repositories = Settings.repositories
  end

  def create
    if Settings.repositories
      Settings.repositories = Settings.repositories << settings_params[:repositories][0]
    else
      Settings.repositories = settings_params[:repositories]
    end

    respond_to do |format|
      @repositories = Settings.repositories

      flash[:success] = 'Setting successfully saved.'
      format.html { redirect_to settings_path, success: 'Setting was successfully saved.' }
      format.json { render :index, status: :created, location: settings_path }
    end
  end

  def destroy
    puts "params: #{params}"
    if params[:id] == 'repositories'
      Settings.repositories.delete_at(params[:index].to_i)
      Settings.repositories = Settings.repositories
    end

    Settings.save
    @repositories = Settings.repositories

    respond_to do |format|
      flash[:success] = 'Setting successfully deleted.'
      format.html { redirect_to settings_path, success: 'Setting was successfully deleted.' }
      format.json { render :index, status: :created, location: settings_path }
    end
  end

  private
    def settings_params
      params.require(:settings).permit(repositories: [])
    end
end
```

The controller is specific to the **repositories** setting at this time, but I’m sure we’ll go back and add some more methods (or refactor things to make them more generic) to handle additional setting values.

## Settings View

Only using an **index** view makes working with our Settings model simpler than other models.  To display the Settings for our web app create  new directory: **app/views/settings**, and inside the directory create: **app/views/settings/index.html.erb**, with:

```
<h3>Settings</h3>
<hr/>

<div class="row">
  <div class="columns large-6">
    <ul class="no-bullet">

      <li>
        <div class="row">
          <div class="columns large-4">
            <strong>repositories:</strong>
          </div>

          <div class="columns large-8">
            <ul class="repositories">
              <% @repositories.each_with_index do |repo, idx| %>
                <li>
                  <%= repo %>

                  &nbsp;&nbsp;&nbsp;&nbsp;

                  <%= link_to 'Delete', '/settings/repositories?index=' + idx.to_s, method: 'delete' %>
                </li>
              <% end %>
            </ul>
          </div>
        </div>
      </li>

    </ul>
  </div>
</div>

<hr/>

<%= form_for 'settings' do %>
  <div class="row">
    <div class="columns large-4">
      <%= label_tag 'settings_repositories_', 'Add Repository' %>
      <%= text_field_tag 'settings[repositories][]', '', placeholder: 'Music Repository path' %>
    </div>
  </div>

  <div class="row">
    <div class="columns small-4">
      <%= submit_tag 'Save Settings', class: 'button small' %>
    </div>
  </div>
<% end %>
```

The view is a simple form that let’s you append file paths onto the **Settings.repositories** array which is displayed above the form.

## Application Layout

I like to pretty up my layout a little and integrate the Foundation grid system from the start by adjusting **app/views/layouts/application.html.erb** like so:

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <title><%= content_for?(:title) ? yield(:title) : "foundation-rails" %></title>

    <%= stylesheet_link_tag    "application" %>
    <%= javascript_include_tag "vendor/modernizr" %>
    <%= javascript_include_tag "application", 'data-turbolinks-track' => true %>
    <%= csrf_meta_tags %>
  </head>

  <body>

    <% flash.each do |type, message| %>
      <div class="flash alert-box text-center <%= type %>", role="dialogalert", data-alert>
        <%= message %>
        <a href="#" class="close">&times;</a>
      </div>
    <% end %>

    <div class="row">
      <div class="small-12">
        <br/><br/>

        <%= yield %>
      </div>
    </div>
  </body>
</html>
```

This isn’t 100% necessary, but it does make things look good.

## Audio Scaffolding

For the Audio object start things off with some standard Rails objects by using the scaffold generator:

```
bin/rails g scaffold audios
```

This will create a default **controller**, **model**, **views**, and **routes** for us.  Might as well take advantage of all Rails has to offer.

## Audio Routes

Get things started by editting **config/routes.rb** and add the following (feel free to keep, or remove, the default documentation comments):

```
  root 'audios#index'
  resources :audios
```

The **index** method of the Audios controller will be the root, or index, route and using **resources** to take care of the rest.

## Add Audios Fields

We didn’t provide any field for your Audio object when we scaffolded things up, so let’s create a migration and add some more fields:

```
bin/rails g migration add_fields_to_audios
```

Edit the **db/migrate/$TIMESTAMP_add_fields_to_audios.rb** and add the following inside the **change** method:

```
    add_column :audios, :name, :string
    add_column :audios, :path, :string

    add_index :audios, :name
```

And execute the migration:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

## Audios Controller

When you open **app/controllers/audios_controller.rb** you’ll see the default methods added by the Rails scaffold.  What we need is a method to create database entries for the files in our Repositories.  Edit the Audios controller and add this method:

```
  # GET /sync_repo/:id
  def sync_repo
    repo = Settings.repositories[params[:id].to_i]

    files = `find #{repo} -name "*.mp3" -o -name "*.m4a" -o -name "*.ogg"`
    files = files.split("\n")

    files.each do |file|
      file_name = file.split('/')[-1]

      unless Audio.find_by_name(file_name)
        Audio.create(name: file_name, path: file)
      end
    end

    flash[:info] = 'Repository sync successful.'
    redirect_to audios_path
  end
```

To keep things simple we’re calling the Unix **find** command on the Repository file path.  Notice that the command is only looking for MP3, M4A, and OGG files.  If you have other file types feel free to add them (hrmm could probably be a good setting to configure… maybe later).  The method then grabs splits the output of **find** and loops through each file path generating the file name and creating it in the database if it’s not already there.

### sync_repo Route

There also needs to be a route for our **sync_repo** method, so edit **config/routes.rb** and add:

```
  get '/sync_repo/:id', to: 'audios#sync_repo', as: :sync_repo
```

## Audios Index

Edit the template for the Audios index method in **app/views/audios/index.html.erb** and go ahead and remove everything and replace it with:

```
<h2>Audio Files</h2>

<% @repositories.each_with_index do |repo, idx| %>
  <h3>Repository:</h3>

  <%= repo %>

  <br/><hr><br/>

  <% if @audios.blank? %>
    No audio files created from this repository...
    <br/><br/>
  <% end %>
  <%= link_to 'Sync Repository', sync_repo_path(idx), class: 'button small' %>

  <br/><hr><br/>


  <ul>
    <% @audios.each do |audio| %>
      <li>
        <%= link_to audio.name, audio_path(audio) %>
      </li>
    <% end %>
  </ul>

<% end %>
```

Notice the “Sync Repository” button (well anchor tag really) that will call our Audios Controller **sync_repo** method.  Click that and the database should start populating.  After you’ve synced with the repo you can click the link to view the Audio file’s details.

Not much there at the moment, well nothing really.  The “Edit” link doesn’t show anything either because we didn’t supply the fields during scaffolding.

## Audios Show

Let’s add the **show** template now.  Edit the **app/views/audios/show.html.erb** file and replace the contents with:

```
<h2><%= @audio.name %></h2>

<br/><hr/><br/>

<audio id="<%= @audio.id %>" controls>
  <source src="<%= stream_path(@audio.id) %>"></source>
</audio>

<br/><hr/><br/>


<%= link_to 'Edit', edit_audio_path(@audio), class: 'button small' %> |
<%= link_to 'Back', audios_path, class: 'button small' %>
```

## Audio stream

If you refresh the show page of an Audio now you will get a routing error.  That’s because we haven’t setup the **stream** route or Audio Controller **stream** method yet.  First, add the route in **config/routes.rb**:

```
  get '/stream/:id', to: 'audios#stream', as: :stream
```

Now edit **app/controllers/audios_controller.rb** and add the **stream** method:

```
  def stream
    audio = Audio.find(params[:id])
    if audio
      send_file audio.path
    end
  end
```

The magic of this method is the Rails **send_file** method.  This method sends whatever file you’d like to the browser.  Usually it’s used to send zip files, PDF files, etc.  Here we can use it to serve the audio file as part of the **source** element in the HTML5 audio tag.

Pretty cool huh?

## Audios Edit

To tie up some loose ends replace the contents of **app/views/audios/_form.html.erb** with:

```
<br/><br/>

<%= form_for(@audio) do |f| %>
  <% if @audio.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@audio.errors.count, "error") %> prohibited this audio from being saved:</h2>

      <ul>
      <% @audio.errors.full_messages.each do |message| %>
        <li><%= message %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="row">
    <div class="columns large-4">
      <%= f.text_field :name %>
    </div>
  </div>

  <div class="row">
    <div class="columns large-4">
      <%= f.text_field :path %>
    </div>
  </div>

  <div class="row">
    <div class="columns large-4">
      <%= f.submit 'Save Audio File', class: 'button small success’ %>
    </div>
  </div>
<% end %>

<br/><br/>
```

And if you’d like make the Edit template more aligned with the Foundation style by replacing **app/views/audios/edit.html.erb** with:

```
<h3>Editing <%= @audio.name %></h3>

<%= render 'form' %>

<%= link_to 'Show', @audio, class: 'button small' %> |
<%= link_to 'Back', audios_path, class: 'button small' %>
```

Also, why not replace the New template in **app/views/audios/new.html.erb**:

```
<h2>New Audio</h2>

<%= render 'form' %>

<%= link_to 'Back', audios_path, class: 'button small' %>
```

## Audios Parameters

One last thing before we sign off.

If you’ve tried to edit an Audio, or create a new one, you’ve probably run into the “ForbiddenAttributesError”.  This comes from eating fruit from certain trees.

Just kidding!

Update the audio_parameters private method in **app/controllers/audios_controller.rb** with the new fields we’ve added:

```
    def audio_params
      params[:audio].permit(:name, :path)
    end
```

Saving Audio updates and creating new Audios should now work like a charm.

## Conclusion

We covered a lot of ground with this post, or maybe it just feels that way to me, and it was a lot of fun.  Well I had fun at least.

My intention is to build out this app step by step in these posts and hopefully have a cool web app that is useful and fun to use at the end of it.  Might take a few months to write everything up and post things, but eventually we’ll get there.

**P.S.** Some of the instructions above assume you have the Rails development server running.  If you don’t have it running you can fire it up in a console by executing:

```
bin/rails s
```

**s** being the shorthand for ```rails serve```.


Party On!
