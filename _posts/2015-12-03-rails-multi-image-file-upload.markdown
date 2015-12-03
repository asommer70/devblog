---
title:  "Rails Multiple File Upload"
date:   2015-12-03 13:05:00
categories: rails foundation
layout: post
image: upload.jpg
---

## Uploading Fun Time Images

A request came in for the [BCN](https://github.com/asommer70/bcn) project to be able to upload multiple images to a Post.  This seems reasonable, and I know that it’s possible to upload multiple files at a time.  You can even do it via Ajax if you’re feeling saucy.

I decided to not use an Ajax method, or to create a file drag ’n drop area on the page to keep things simple.  If users request that feature then I’ll probably go back and add it later.  I guess the biggest reason I didn’t include that was because most of the current libraries I found applied their own styles, and I wanted to use the [Foundation Clearing](http://foundation.zurb.com/sites/docs/v/5.5.3/components/clearing.html) feature to create a little web gallery as a preview.

<!--more-->

## Adding Models

To apply multiple images to a Post there needs to be a new model with a **has_many** relationship.  In this case I chose to call it Photo.  Create a new migration:

```
bin/rails g migration create_photos
```

Add the following to the **change** method in **db/migrate/DATESTAMP_create_photos.rb**:

```
    create_table :photos do |t|
      t.string   "image", index: true
      t.string   "image_uid"
      t.string   "image_name", index: true

      t.timestamps null: false
    end

    add_reference :photos, :post, index: true, foreign_key: true

    create_table "photos_posts" do |t|
      t.belongs_to :photo, index: true
      t.belongs_to :post, index: true
    end
```

Execute the migration:

```
bin/rails db:migrate
bin/rails db:migrate RAILS_ENV=tests
```

Now create the new model for Photos in **app/models/photo.rb**:

```
class Photo < ActiveRecord::Base
  dragonfly_accessor :image

  attr_accessor :image_web_url

  validates :image, presence: true

  validates_property :ext, of: :image, in: ['jpeg', 'jpg', 'png', 'gif', 'svg', 'svgz'], if: :image_changed?
  validates_property :mime_type, of: :image,
                     in: ['image/jpeg', 'image/png', 'image/gif', 'image/svg+xml', 'image/svg'],
                     if: :image_changed?
  validates_property :format, of: :image, in: ['jpeg', 'png', 'gif', 'svg', 'svgz'], if: :image_changed?

  belongs_to :post
end
```

It’s basically all the checks from the Post Photo attribute, which we’ll leave basically alone to maintain backwards compatibility.

Finally, update the **app/models/post.rb** file with the new relationship:

```
  has_many :photos
```

This should get things setup to be able to add multiple Photos to a Post.

## Updating the Form

The Post form needs to be adjusted for multi-file upload and to display the images in a Clearing gallery.  Edit the **app/views/posts/_form.html.erb** file, change the **form** element at the top first:

```
<%= form_for(@post, multipart: true, data: {'abide' => true})  do |f| %>
```

This will allow multi-part uploads.  Now change the Photo fields below:

```
            <div class="columns large-12">
              <span>
                Hold Ctrl, or Command on Mac, to select more than one file.
              </span>
            </div>
          </div>
          <div class="row">
            <div class="large-12 columns">
              <h4 class="pics-label hide">Pics to be uploaded:</h4>
              <ul id="photos_clearing" class="clearing-thumbs" data-clearing>
              </ul>
              <br/>
              <label for="photos">Add some pics?</label>
              <input type="file" name="photos[]" id="photos" multiple />
            </div>

          <br/><br/><br/><br/><br/>
```

Notice the **photos_clearing** ul element.  This will be used to display a gallery preview, but to do that we need to adjust the JavaScript/CoffeeScript that makes the current single preview work.

## CoffeeScript Changes

To add our soon to be uploaded images to a Clearing preview gallery edit **app/assets/javascripts/posts.coffee** and add the **multiPhotoDisplay** function:

```
@multiPhotoDisplay = (input) ->
  #
  # Read the contents of the image file to be uploaded and display it.
  #
  if (input.files && input.files[0])
    for file in input.files
      reader = new FileReader()

      reader.onload = (e) ->
        image_html = """<li><a class="th" href="#{e.target.result}"><img width="75" src="#{e.target.result}"></a></li>"""

        $('#photos_clearing').append(image_html)

        if $('.pics-label.hide').length != 0
          $('.pics-label').toggle('hide').removeClass('hide')

        $(document).foundation('reflow')

      reader.readAsDataURL(file)

    window.post_files = input.files
    if window.post_files != undefined
      input.files = $.merge(window.post_files, input.files)
```

The Clearing magic happens in the **image_html** li element and the call to **$(document).foundation(‘reflow’)**.   This will reapply the Foundation JavaScripts to the page after the new elements are added.

The major difference from previewing one image is that the **input.files** are looped over and each files is read and displayed.  All thanks to the magic of HTML5.

Now, add a *onclick* handler toward the top of the file:

```
    $('#photos').on 'change', (e) ->
      multiPhotoDisplay(this);
```

There now when the files are attached the **multiPhotoDisplay** function will be called and we’ll be able to see them.

Check out the results in this Pen:

<p data-height="268" data-theme-id="0" data-slug-hash="yeBLxb" data-default-tab="result" data-user="asommer70" class='codepen'>See the Pen <a href='http://codepen.io/asommer70/pen/yeBLxb/'>Multi Image Upload with Gallery</a> by Adam Sommer (<a href='http://codepen.io/asommer70'>@asommer70</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

## Conclusion

Uploading multiple files is great, but doing it easily is still a hassle.  Or at least I think holding down a key on the keyboard and selecting multiple things will be a hassle for the average user.

The good thing about splitting the Photos into their own model is that they can also be added one at a time by editing the post.  I tried to come up with a good way to “cache” the photos before uploading them, but there are necessary safe guards in JavaScript to not allow arbitrary file upload.

Party On!
