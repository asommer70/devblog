# Rails Audio Player with React

## React.js and Rails

I’ve used [React.js](https://facebook.github.io/react/) with Rails for a couple of little projects, but when it came time to add Album/Playlist player controls to a “header” element similar to other online music playing sites, I thought that React could be better at determining if a collection of audios was being played or not.

It’s turned out harder than I originally thought to get everything migrated, but I think I’m about 90% there.  So I thought it’d be great to write up the first part of the “Great React Migration” and detail the steps to use React for HTML5 audio playback.

## Setting Up React with Rails

As before, we’ll use the immensely useful [react-rails](https://github.com/reactjs/react-rails) gem.  To install the gem edit you **Gemfile** and add:

```
gem 'react-rails', '~> 1.5.0'
```

Then run the install script in a terminal:

```
rails g react:install
```

Finally, add React build variant to the environment by editing **config/environments/development.rb** and add:

```
  config.react.variant = :development
```

And for **config/environments/production.rb**:

```
  config.react.variant = :production
```

## The Player Component

The React Player class will have basically the same functions as the CoffeeScript version, but it’s a little cleaner because we’ll use the component state to contain the HTML5 audio element.

Create a new file **app/assets/javascripts/components/audio.js.jsx** with the following:

```
var Audio = React.createClass({

  getInitialState: function() {
    return {
      player: false
    }
  },

  componentDidMount: function() {
    var $player = $('#' + this.props.id);

    /**
      * Have to add media event listeners here.
      *
    */
    $player.on('play', (e) => {
      e.preventDefault();
      this.playLocation();
    });

    $player.on('pause', (e) => {
      e.preventDefault();
      this.pause();
    });

    $player.on('ended', (e) => {
      e.preventDefault();
      this.ended();
    });

    $(document).on('keydown', (e) => {
      // Move currentTime forward and backward via arrow keys and play/pause via spacebar.
      if (e.keyCode == 39) {
        this.state.player.currentTime += 1;
      } else if (e.keyCode == 37) {
        this.state.player.currentTime -= 1;
      } else if (e.keyCode == 32 && this.state.player.paused == true) {
        e.preventDefault();
        this.state.player.play();
      }  else if (e.keyCode == 32 && this.state.player.paused == false) {
        e.preventDefault();
        this.state.player.pause()
      }
    });

    $player.on('wheel', (e) => {
      e.preventDefault();
      e.stopPropagation();
      // $player.focus();
      if (e.originalEvent.wheelDelta > 0) {
        this.state.player.currentTime += 1;
      } else {
        this.state.player.currentTime -= 1;
      }
    });
  },

  componentWillUnmount: function() {
    var $player = $('#' + this.props.id);
    $player.off('play');
    $player.off('pause');
    $(document).off('keydown')
    $player.off('wheel')
  },

  getPlaybackTime: function(time) {
    var hours = Math.floor(time / 3600);
    var minutes = Math.floor(time / 60);
    if (minutes > 59) {
      minutes = Math.floor(time / 60) - 60;
    }

    var seconds = Math.round(time - minutes * 60);
    if (seconds > 3599) {
      seconds = Math.round(time - minutes * 60) - 3600;
    }

    return time;
  },

  playLocation: function() {
    this.setState({player: $('#' + this.props.id)[0]}, function() {
      var playbackTime = this.getPlaybackTime(this.state.player.currentTime);

      $.get('/audios/' + this.props.id + '.json').then( (data) => {
        this.state.player.currentTime = data.playback_time;
        this.state.player.play();
      });
    });
  },

  pause: function() {
    var playbackTime = this.getPlaybackTime(this.state.player.currentTime);

    // Do the putting.
    $.ajax({
      url: '/audios/' + this.props.id  + '.json',
      method: 'put',
      data: 'audio[playback_time]=' + playbackTime
    });
  },

  ended: function() {
    // Set playback_time to 0.
    $.ajax({
      url: '/audios/' + this.props.id  + '.json',
      method: 'put',
      data: 'audio[playback_time]=' + 0
    });

    $(document).trigger('playback-ended');
  },

  render: function() {
    return <audio id={this.props.id} controls className="player" preload="false">
      <source src={this.props.audio.path.indexOf('http') ? '/stream/' + this.props.id : this.props.audio.path} />
    </audio>
  }
});
```

You’ll notice the same functionality for the **playLocation** and **pause** methods.  Pause will send an HTTP PUT to the audio object and **playLocation** does a GET for the last saved **playback_time** then starts playing the audio.

The keyboard and mouse controls are setup inside the **componentDidMount** method because we need to add listeners for events outside the normal React built in ones.

Also, a cool thing is in the **render** method we’re using a ternary if statement to determine the **src** attribute for the audio element.  I always find ternary operators fun… maybe it’s the word “ternary”, heh funny word.

## Mount the Component

Lastly, we need to replace the **erb** audio tag with our new React component.  Edit **app/views/audios/show.html.erb** and replace the audio element with:

```
    <%= react_component 'Audio', id: @audio.id, action: controller_name, audio: @audio %>
```

Might as well replace the audio elements on Album and Playlist pages by editing **app/views/layouts/_collections.html.erb**:

```
                    <%= react_component 'Audio', id: audio.id, action: controller_name, audio: audio %>
```

Finally, to make sure everything is working with your React component and not the old CoffeeScript delete the **app/assets/javascripts/audios.coffee** file.

## Conclusion

Here is an example of the React audio player, but this Pen saves the **playback_time** to localStorage instead of sending it to a Rails server:



It’s been a while since I’ve posted an [Audio Pila!](https://github.com/asommer70/audiopila-rails) article and it’s great to get back to that project.  The U.S. Holiday season is a busy time for me, but I have found some time here and there to hack on this little app.

I for one am looking forward to using React in more of my Rails apps.

Party On!