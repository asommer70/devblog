---
title:  "Rails Helpers"
date:   2015-12-17 13:05:00
categories: rails bcn
layout: post
image: helpers.jpg
---

## Help Me, Help Help Me

I while back, well quite a while actually, blasted through a [Treehouse course](https://teamtreehouse.com/library/rails-partials-and-helpers) on Rails Helpers and I guess it didn’t really sink in.  I think at the time I was working on a simple project and didn’t see the need for “overly complicating” things with additional code in some other folder other than the *views* directory.

Ah sigh, it would have been useful to at least keep [Rails Helpers](http://mixandgo.com/blog/the-beginner-s-guide-to-rails-helpers) in mind while I was adding a ton of if statements and other logic to my templates.

As it turns out Rails Helpers are actually quite simple to understand and add a lot of power to the tempting system.

<!--more-->

## Moving Post Image to a Helper

For the [BCN](https://github.com/asommer70/bcn) project we have the ability to add a photo to a Post and have it displayed prominently on the show page and as a sort of “icon” for the Post on the index page.  As you might imagine there are a few options for this.  The image can be manually uploaded by the user creating the post, the image can be part of the Open Graph protocol for a link shared in the Post, or the image can be a default icon.

Options, Options, Options…

Previously the Post image was determined in the **app/views/layouts/_posts_index_list.html.erb** file (and a couple of others).  The code was repeated in each file and was somewhat convoluted.  To clean things up I moved the logic to **app/helpers/posts_helper.rb** and created the **post_list_image** method:

```
def post_list_image(post)

    if post.organization && !post.organization.communities.empty?
      border = "border: 5px solid #{post.organization.color}"
    else
      border = ''
    end

    if post.image
      link_to post do
        image_tag post.image.thumb('75x75#').url, style: border
      end
    elsif post.og_image && post.og_image != '//:0' && !post.og_image.blank?
      link_to post do
       image_tag post.og_image, style: border
      end
    elsif post.start_date
      link_to post do
        image_tag 'calendar-icon.svg', size: '75x75', alt: post.title, style: border
      end
     else
      link_to post do
        image_tag 'bcn_logo_black.svg', size: '75x75', alt: post.title, style: border
       end
    end
  end
```

The great thing about Helpers is that you can use all the View methods you normally would like **link_to**, but since it’s not an actual template you don’t need all the escape characters (<%= %>).

Then the new method need to replace the old logic in the **app/views/layouts/_posts_index_list.html.erb** file:

```
<%= post_list_image(post) %>
```

Pow! And Bob’s Your Uncle!

## Conclusion

This was a shorter post and we’ve drifted away from [Audio Pila!](https://github.com/asommer70/audiopila-rails) again and it’s mostly because I’m traveling and wanted to get something written up before hand that would be easy to post from the road.

Rails Helpers are an awesome feature and I know I’ll be going back and cleaning up the massive logic I’ve embedded in some of my templates.

Yay for learning!

Party On!
