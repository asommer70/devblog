# Album/Playlist Player With React and Rails

## Almost There

In the [last post](http://devblog.thehoick.com/rails/react/audiopila/2015/12/10/audiopila-react-player.html) we migrated the HTML5 audio element and associated JavaScript playback handling to React.  This time we’re ready to migrate the Album/Playlist (or Collections, as I’ve come to think of them) to React.  

Having both of this functionality in React will be great for integrating the two and hopefully making the code more maintainable.  At least I think the code looks a lot cleaner in React.  CoffeeScript is great, but like regular JavaScript, too many jQuery event bindings in one file can get pretty hairy pretty quickly.

## Player Component

To migrate Collection playback to React create a new React component in **app/assets/javascripts/components/player.js.jsx**:

```
var Player = React.createClass({

  getInitialState: function() {
    return {
      buttonColor: 'warning',
      playing: false,
      status: 'fi-play',
      currentAudio: this.props.currentAudio,
      collection: this.props.audios,
      looper: false,
      shuffle: false
    }
  },

  componentDidMount: function() {
    /**
     * Handle shuffle and looping when audio ends.
    */
    $(document).on('playback-ended', () => {
      if (this.state.shuffle) {
          var random = this.getRandomPos();
          var currentPlayer = $('#' + this.state.collection[random].id)[0];

          this.setState({currentAudio: this.state.collection[random], currentPlayer: currentPlayer, playing: true}, () => {
            //this.play();
            this.state.currentPlayer.play();
            this.updateCollection();
          });
      } else {
        // If the audio is the last one and looping is on then start the first one.
        if (this.state.currentAudio.album_order == this.state.collection.length) {
          if (this.state.looper) {
            this.changeAudio(1);
          } else {
            // If it is the last audio and album/playlist is not set to loop just return and stop playback.
            return;
          }
        } else {
          // If not last audio advance to next.
          this.changeAudio(1);
        }
      }
    });
  },

  findCurrentAudio: function() {
    var current = undefined;

    for (i in this.state.collection) {
      if (this.state.collection[i].id == this.state.currentAudio.id) {
        current = parseInt(i);
      }
    }
    return current;
  },

  getRandomPos: function() {
    var random = Math.floor(Math.random() * this.state.collection.length);

    // Don't let the current audio play back to back.
    var current = this.findCurrentAudio();
    while (random == current) {
      random = Math.floor(Math.random() * this.state.collection.length)
    }
    return random;
  },

  updateCollection: function() {
    if (this.props.action == 'albums') {
      $('#playing_audio').html(this.state.currentAudio.name)
      field = 'album[current_audio]='
    } else {
      $('#playing_audio').html(this.state.currentAudio.name)
      field = 'playlist[current_audio]='
    }

    $.ajax({
      url: '/' + this.props.action + '/' + this.props.id + '.json',
      method: 'put',
      data: field + this.state.currentAudio.id
    })
  },

  play: function() {
    if (this.state.playing) {
      this.setState({buttonColor: 'warning', status: 'fi-pause', playing: false});
      this.pause();
    } else {
      this.setState({buttonColor: '', status: 'fi-play'});

      if (this.state.currentPlayer === undefined) {
        if (this.state.currentAudio === null) {
          this.setState({currentAudio: this.state.collection[0], currentPlayer: $('#' + this.state.collection[0].id)[0]}, () => {
            $('#' + this.state.currentAudio.id)[0].play()
          });
        } else {
          this.setState({ currentPlayer: $('#' + this.state.currentAudio.id)[0] });
          $('#' + this.state.currentAudio.id)[0].play()
        }
      } else {
        this.state.currentPlayer.play();
      }
      this.setState({playing: true})
    }

    this.updateCollection();
  },

  pause: function() {
    if (this.state.currentAudio !== undefined) {
      $('#pause').toggleClass('warning')
      if (!$('#play').hasClass('warning')) {
        $('#play').toggleClass('warning')
      }

      if (this.state.currentPlayer !== undefined) {
        this.state.currentPlayer.pause();
        this.setState({playing: false});
      }
    }
  },

  changeAudio: function (direction) {
    this.state.currentPlayer.pause();

    if (this.state.shuffle) {
      var random = this.getRandomPos();

      var currentAudio = this.state.collection[random]
      var currentPlayer = $('#' + this.state.collection[random].id)[0];
    } else {
      // Get next Audio and play it.
      var currentPos = this.findCurrentAudio() + direction;

      // Handle going backwards from the last Audio.
      var currentAudio = this.state.collection[currentPos];
      if (currentPos >= this.state.collection.length) {
        currentAudio = this.state.collection[0];
      } else if (currentPos == -1) {
        currentAudio = this.state.collection[this.state.collection.length - 1]
      }
      var currentPlayer = $('#' + currentAudio.id)[0];
    }

    this.setState({currentAudio: currentAudio, currentPlayer: currentPlayer}, function() {
      this.state.currentPlayer.play();
      this.updateCollection();
    });
  },

  looper: function() {
    $('#looper').toggleClass('warning')
    if (this.state.looper === true) {
      this.setState({ looper: false });
    } else {
      this.setState({ looper: true },function() {
        if (this.state.playing === false) {
          this.play();
        }
      });
    }
  },

  shuffle: function() {
    $('#shuffle').toggleClass('warning')
    if (this.state.shuffle === true) {
      this.setState({ shuffle: false });
    } else {
      this.setState({ shuffle: true },function() {
        if (this.state.playing === false) {
          this.play();
        }
      });
    }
  },

  render: function() {
    return <div className="controls">
      <button className="warning large control" onClick={this.changeAudio.bind(this, -1)}><i className="fi-previous"></i></button>&nbsp;
      <button id="play" className={"large control " + this.state.buttonColor} onClick={this.play}><i className={this.state.status}></i></button>&nbsp;
      <button className="warning large control" onClick={this.changeAudio.bind(this, 1)}><i className="fi-next"></i></button>&nbsp;
      <br/>
      <button id="looper" className="warning small control" onClick={this.looper}><i className="fi-loop"></i></button>&nbsp;
      <button id="shuffle" className="warning small control" onClick={this.shuffle}>
        <i className="fi-shuffle"></i>
      </button>
    </div>
  }
})
```

Pow!  

There’s kind of a lot going on so let’s take a look at some bits and pieces.

Starting at the bottom we have the required render function that will display our button controls in their own little div.  Notice that we only have the one button for play/pause.  Kinda makes sense and having one button for play/pause seems to be the convention out there on the Internets.

 The **play()** method is the next big piece:

```
play: function() {
    if (this.state.playing) {
      this.setState({buttonColor: 'warning', status: 'fi-pause', playing: false});
      this.pause();
    } else {
      this.setState({buttonColor: '', status: 'fi-play'});

      if (this.state.currentPlayer === undefined) {
        if (this.state.currentAudio === null) {
          this.setState({currentAudio: this.state.collection[0], currentPlayer: $('#' + this.state.collection[0].id)[0]}, () => {
            $('#' + this.state.currentAudio.id)[0].play()
          });
        } else {
          this.setState({ currentPlayer: $('#' + this.state.currentAudio.id)[0] });
          $('#' + this.state.currentAudio.id)[0].play()
        }
      } else {
        this.state.currentPlayer.play();
      }
      this.setState({playing: true})
    }

    this.updateCollection();
  },
```

As you can see the playing state attribute is used to determine if the collection is currently playing.  So if it is currently playing the **currentAudio** is paused.

The **pause()** method is pretty simple (and similar to the CoffeeScript version), the currentPlayer is paused and the **state.playing** attribute is updated.  Also, because we’re using the React Audio component when the element is paused the playback position is saved via HTTP PUT.

The **updateCollection()** method is where some magic happens:

```
  updateCollection: function() {
    if (this.props.action == 'albums') {
      $('#playing_audio').html(this.state.currentAudio.name)
      field = 'album[current_audio]='
    } else {
      $('#playing_audio').html(this.state.currentAudio.name)
      field = 'playlist[current_audio]='
    }

    $.ajax({
      url: '/' + this.props.action + '/' + this.props.id + '.json',
      method: 'put',
      data: field + this.state.currentAudio.id
    })
  },
```

This is called to update the collection object with the **current_audio** so we’ll know where we left off.

The next big method is the **changeAudio** method:

```
  changeAudio: function (direction) {
    this.state.currentPlayer.pause();

    if (this.state.shuffle) {
      var random = this.getRandomPos();

      var currentAudio = this.state.collection[random]
      var currentPlayer = $('#' + this.state.collection[random].id)[0];
    } else {
      // Get next Audio and play it.
      var currentPos = this.findCurrentAudio() + direction;

      // Handle going backwards from the last Audio.
      var currentAudio = this.state.collection[currentPos];
      if (currentPos >= this.state.collection.length) {
        currentAudio = this.state.collection[0];
      } else if (currentPos == -1) {
        currentAudio = this.state.collection[this.state.collection.length - 1]
      }
      var currentPlayer = $('#' + currentAudio.id)[0];
    }

    this.setState({currentAudio: currentAudio, currentPlayer: currentPlayer}, function() {
      this.state.currentPlayer.play();
      this.updateCollection();
    });
  },
```

This method pauses the current audio, then if the shuffle button is pressed it will get a random audio from the collection using the **getRandomPos()** method, if the collection is not shuffling the next audio will be determined by the **findCurrentAudio()** method setting the currentAudio to the first one if the currentOne is last and last if the first one is current and going backwards, and finally setting the state and playing the **currentPlayer**.

Lastly, the **componentDidMount()** method is a React method to execute code after the component has been mounted and added to the page:

```
componentDidMount: function() {
    /**
     * Handle shuffle and looping when audio ends.
    */
    $(document).on('playback-ended', () => {
      if (this.state.shuffle) {
          var random = this.getRandomPos();
          var currentPlayer = $('#' + this.state.collection[random].id)[0];

          this.setState({currentAudio: this.state.collection[random], currentPlayer: currentPlayer, playing: true}, () => {
            //this.play();
            this.state.currentPlayer.play();
            this.updateCollection();
          });
      } else {
        // If the audio is the last one and looping is on then start the first one.
        if (this.state.currentAudio.album_order == this.state.collection.length) {
          if (this.state.looper) {
            this.changeAudio(1);
          } else {
            // If it is the last audio and album/playlist is not set to loop just return and stop playback.
            return;
          }
        } else {
          // If not last audio advance to next.
          this.changeAudio(1);
        }
      }
    });
  },
```

Before the great React migration looping and shuffling were handled in the **audios.coffee** file when the **ended** event is fired on an HTML5 audio element.  The event binding then checked the **media_control** object for all the needed info like currently playing audio (or just ended actually), the next audio, etc.

Since the React Audio component doesn’t know about all that information the code for looping and shuffling has been moved into the Player component and in **componentDidMount()** our ‘playback-ended’ event is bound and a function is called to call **changeAudio()** if the collection is looping, or get a random audio if shuffling.  

Also, the method is what plays the next audio even if looping and shuffling are not set.  If they’re not set the playback will stop after the last audio is played.

## Mounting the Player

Like our Audio component we need to use the **react_component** method in our template.  Edit the **app/views/layouts/_collections.html.erb** file and replace the controls div with:

```
    <%= react_component 'Player', id: @collection.id,
        action: controller_name,
        audios: @collection.audios,
        currentAudio: @last_audio %>
```

## Conclusion

Migrating this functionality to React has seemed a lot harder than it should have been.  I think the actual problem is my brain hangover from the Thanksgiving holiday though.  It’s very interesting the effect that eating a bunch of crappy food for half a week will have on you.  I never really noticed it before, but maybe it’s a sign that I’m getting old…

There’s a couple of other things to update to complete our React migration and get back to the functionality we had before.  So stay tuned.

Party On!