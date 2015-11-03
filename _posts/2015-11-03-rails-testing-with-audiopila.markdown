---
title:  "Rails Testing with Audio Pila!"
date:   2015-11-03 13:05:00
categories: rails audiopila testing
layout: post
image: test_cover.jpg
---


## Test, Test 12, Test 1 2 3

For the longest time Test Driven Development, and the whole testing thing, was more than my mind balls could contain.  So I just didn’t deal with it.

I built some web apps and to be honest they weren’t as good as they could have been if I’d taken the time to learn what the whole **testing** thing was all about.

Finally about nine months or so ago buckled down and learned how to write some tests.  Oh man, the world has taken a turn for me.  The simple ability to not have to open a browser and clicky click click was, and is, tops!

<!--more-->

## Adjusting the Environment

Because we’re using the **rails_settings_cached** gem in our project the environment for the **test** and **development** needs to be adjusted to use memory for caching.  At the bottom of the **config/environments/development.rb** and **config/environments/test.rb** files add the following line:

```
config.cache_store = :memory_store
```

Now things should work the way we expect.

## Model Testing

Might as well start with the base of the pillar and test our Audio model.  The scaffold created an empty test for us in **test/models/audio_test.rb** so go ahead and edit it:

```
require 'test_helper'

class AudioTest < ActiveSupport::TestCase
  test 'an audio should have a name' do
    audio = Audio.new
    assert !audio.save
    assert !audio.errors[:name].empty?
  end

  test 'an audio should have a path' do
    audio = Audio.new
    assert !audio.save
    assert !audio.errors[:path].empty?
  end

  test 'an audio is valid with name and path' do
    audio = Audio.new(name: 'test.mp3', path: './test.mp3')
    assert audio.save
  end
end
```

You can run the test by executing:

```
ruby -Itest test/models/audio_test.rb
```

Things should fail at this point because we haven’t added any validations to the **app/models/audio.rb** file yet:

```
  validates :name, presence: true
  validates :path, presence: true
```

At this point a simple **presence** validation is fine.  We may come back later and do some more advanced validations.  Run the test again and things should pass.

Next, let’s do an even simpler test for our Settings model.  Create a new file named **test/models/settings_test.rb** with:

```
require 'test_helper'

class SettingsTest < ActiveSupport::TestCase
  test 'a setting is saved' do
    Settings.repositories = '/home/adam/Music'
    Settings.save

    assert !Settings.where(var: 'repositories').empty?
  end
end
```

This test creates a **repositories** setting, looks up that setting by checking the **var** field in the database for the matching record, and asserts that it is **not** empty.

Like the Audio test you can execute this test with:

```
ruby -Itest test/models/settings_test.rb
```

## Fixtures

Fixtures are a method for pre-populating the test environment with objects.  There is a fixture setup with the Audio scaffolding in **test/fixtures/audios.yml** edit the file to setup a correct Audio object:

```
test:
  name: 'test.mp3'
  path: '/home/adam/Music/test.mp3'
```

Now create a fixture for a Settings.repositories by creating a **test/fixtures/settings.yml** file with:

```
repo:
  var: 'repositories'
  value: "['~/Music']"
```

## Feature/Integration Tests

I like model tests, but to be perfectly honest I don’t see the value of creating tests for every specific view, form, etc.  If there is a super important page, or form, then I can understand adding a test for it.  My thought is why take the time to develop eighty little tests when you can develop a few tests that test the site as a whole.

Seems like most TDD super heroes do both anyway, but where’s the value of retesting things.  Models are different in my opinion because if the database isn’t working correctly then nothing really works correctly.  Also, you might not have a clear picture in your head what the relationship of your objects will be until you start developing them, and model tests help to solidify things and make sure they work the way you invasion.

So we’ll skip over the controller and view tests and skip right to the big dogs: Integration Tests.  Integration tests, also called Feature tests (at least I think they’re pretty much the same thing) will test everything in the stack of the web app.

For the first test create a new file **test/integration/settings_integration_test.rb** and we’ll test out adding a Repository location to via the web form:

```
require 'test_helper'

class SettingsIntegrationTest < ActionDispatch::IntegrationTest
  test 'adding a repository' do
    get '/settings'
    assert_response :success

    post_via_redirect '/settings', settings: { repositories: ['/Users/adam/Music'] }
    assert_equal '/settings', path
    assert_response :success
  end
end
```

Run the test with **rake**:

```
bin/rake test:integration
```

Things should work just peachy.  Now, setup  new test for the **deleting** of repositories.  Add the following to the **test/integration/settings_integration_test.rb** file:

```
  test 'removing a respository' do
    get '/settings'
    assert_response :success

    delete_via_redirect '/settings/repositories?index=1'
    assert_response :success

    assert_equal '/settings', path
    assert_equal 1, Settings.repositories.count
  end
```

Now that we can test the ability to add a *repository* to our Settings let’s make sure the Repository path is on the main page.  Create a new file **test/integration/audios_test.rb** with:

```
require 'test_helper'

class AudiosTest < ActionDispatch::IntegrationTest
  test 'that a repository is displayed' do
    get '/'

    assert_select '.repo-path', '~/Music'
  end
end
```

Next, test that the **sync_repo** method, and the Audios **create** method, are working.  Add the following to the Audios integration test file:

```
  test 'that repository will sync' do
    post_via_redirect "/audios", audio: { name: 'test2.mp3', path: '~/Music/test2.mp3' }

    get '/sync_repo/0'
    assert_response :redirect

    get '/'

    assert_select "ul.audio-list" do
      assert_select 'li', 2
    end
  end
```

This test will create a new Audio object name *test2.mp3*, get the **sync_repo** method, then go back to *’/‘*.  The test then uses the **assert_select** method to find the **audio-list** ul element and inside of it count the number of li elements.  It should be two since we have one from the *fixture* file.

Run the tests with ```bin/rake test:integration`` and everything should work fine.

Finally, run all the tests with ```bin/rake test``` and you should see some errors in the Audios controller tests created from the scaffold.  To fix those edit the **test/controllers/audios_controller_test.rb**.  Change the **one** fixture name to **test** in the **setup do** block:

```
    @audio = audios(:test)
```

In the **should create audio** test further down add a *name* and *path* to the **post** call:

```
      post :create, audio: { name: 'test3.mp3', path: '~/Music' }
```

One final adjustment to the controller.  Add the following line to the **should show audios** test to make sure there is an HTML5 audio element on the page:

```
    assert_select 'audio'
```

Run all the tests again and things should pass.

## Conclusion

I’ll be honest testing with the default Rails integrating with Test::Unit is a little awkward after getting used to developing and running tests with [RSpec](http://rspec.info/).  Also, the power of Rspec and [Capybara](http://jnicklas.github.io/capybara/) is awesome.  

Though it does seem that Capybara can be integrated with Test::Unit so maybe I’ll give that a try and stick it out with Test::Unit a little longer.  As long as things get tested I’m happy.

It does get frustrating spending a lot of time developing tests when you could be developing features…

Party On!
