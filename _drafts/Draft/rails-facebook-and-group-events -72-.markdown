# Rails Facebook and Group Events

## Owners, We Don’t Need No Stinkin Owners

While working on the [BCN](https://github.com/asommer70/bcn) project we wanted to integrate Events our users are attending for local organizations that have their main “online calendar” in Facebook.  Turns out integrating Facebook Events is quite easy with the Facebook API and the [Koala](https://github.com/arsduo/koala) library.  

While setting up Events from Facebook Pages I used the **owner** field to match Organizations in our site.  Worked pretty good and I thought things were tip top.  That is until users reported that their Events weren’t coming through and there was an Organization in our site and everything.

Should be working right… not so fast sir.  If a Facebook user is attending/interested in an Event that was created by a Facebook Group there is no **owner** field.  Doh, I guess we can’t trigger off of that then.  Wait you say, there’s a **parent_group** field seems like that should work.  It does, but from my experience that field doesn’t always come through on an Event when using the API.

So we’ll have to do a search using Koala to find out if the owner of the Event is a Facebook Group, then search the Organizations setup in our site and compare them.

## Updating the Database

Facebook groups have an ID number associated with them and it’d be very handy to store that in the database (probably on the Organizations object).  Create a new migration with:

```
bin/rails add_facebook_group_to_organizations
```

Edit the new **db/migrate/$DATESTAMP_add_facebook_group_to_organizations.rb** file and add the following to the **change** method:

```
    add_column :organizations, :facebook_group, :string
```

Then perform the migration:

```
bin/rake db:migrate
bin/rake db:migrate RAILS_ENV=test
```

## Finding Facebook Groups

The **facebook_group** field needs to be populated when a new Organization is created.  Edit the **app/controllers/organizations_controller.rb** file and add the following to the **create** method:

```
      FacebookFindGroupJob.perform_now(current_user.facebook_token, current_user, @organization) if current_user.facebook_token
```

Next, create a new Active Job to find the Organization’s Facebook Group, if there is one, by creating a new file **app/jobs/facebook_find_group_job.rb** with:

```
class FacebookFindGroupJob < ActiveJob::Base
  queue_as :default
  include Sidekiq::Worker

  def perform(token, user, organization)

    @graph = Koala::Facebook::API.new(token)
    groups = @graph.get_connections("search?q='#{organization.name}'&type=group&limit=10", '')

    groups.each do |g|
      if organization.name.length == g['name'].length && organization.name == g['name']
        organization.facebook_group = g['id']
        organization.save
      end
    end

    FacebookGroupSyncJob.perform_now(user.facebook_token, user)
  end
end
```

So the user’s Facebook authentication token is used to query the Facebook API looking for **q=$GROUPNAME** and  **type=group** (also the first ten, but we’re really only going for the first one).  The results are then looped through and the Organization passed into the method via arguments is updated with the new **facebook_group** attribute. For good measure the **FacebookGroupSyncJob** is fired off to sync any outstanding Events.  

Let’s create the **app/jobs/facebook_group_sync_job.rb** and add:

```
class FacebookGroupSyncJob < ActiveJob::Base
  queue_as :default
  include Sidekiq::Worker

  def perform(token, user)

    organizations = user.organizations.where.not(facebook_group: nil)

    organizations.each do |organization|
      @graph = Koala::Facebook::API.new(token)
      events = @graph.get_connections(organization.facebook_group, 'events')

      events.each do |event|
        fb_event = @graph.get_connection(event['id'], nil,
                                         {'fields' => 'name,owner,start_time,end_time,cover,place,description'})

          start_date = Date.parse(event['start_time']) unless event['start_time'].nil?
          start_time = Time.parse(event['start_time']) unless event['start_time'].nil?
          end_date = Date.parse(event['end_time']) unless event['end_time'].nil?
          end_time = Time.parse(event['end_time']) unless event['end_time'].nil?
          image = fb_event['cover']['source'] unless fb_event['cover'].nil?
          #puts "fb_event: #{fb_event.inspect}"
          lat = fb_event['place']['location']['latitude'] if fb_event['place'] && fb_event['place']['location']
          lon = fb_event['place']['location']['longitude'] if fb_event['place'] && fb_event['place']['location']

          post = Post.find_by(title: event['name'])

          if post.nil?
            new_post = Post.create({
                                       title: event['name'],
                                       description: fb_event['description'],
                                       start_date: start_date,
                                       start_time: start_time,
                                       end_date: end_date,
                                       end_time: end_time,
                                       organization_id: organization.id,
                                       user: user,
                                       og_url: "https://www.facebook.com/events/#{event['id']}",
                                       og_title: event['name'],
                                       og_image: ActionController::Base.helpers.asset_path('facebook-social.svg'),
                                       image_url: image,
                                       communities: organization.communities
                                   })
            #new_post.image_url = image if image
            new_post.create_location({lat: lat, lon: lon})
            new_post.save
          else
            post.description = fb_event['description']
            post.start_date = start_date
            post.start_time = start_time
            post.end_date = end_date
            post.end_time = end_time
            post.og_url = "https://www.facebook.com/events/#{event['id']}"
            post.og_title = event['name']
            post.og_image = ActionController::Base.helpers.asset_path('facebook-social.svg')
            post.image_url = image if image
            post.create_location({lat: lat, lon: lon}) if lat && lon
            post.save
          end
      end
    end


  end
end
```

This job is basically the same as the **facebook_sync_job.rb** file, but adjusted to pull in Events for the Facebook Group.  There’s probably a very easy way to extract the duplicate code into one module, object, etc.  We’ll leave that for another day though.  Things are working and the people are happy.

## Conclusion

It’s awesome to be able to sync Events from both Pages and Groups now.  To execute the job for current Organizations (that have been updated with their Facebook Group ID), I created a quick rake task in **lib/tasks/facebook_group_events_sync.rake** with:

```
task :facebook_group_sync => :environment do
  user = User.find(7)
  FacebookGroupSyncJob.perform_now(user.facebook_token, user)
end
```

It simply uses a Facebook authenticated user to then perform the *FacebookGroupSyncJob*.  Boom!

Party On!