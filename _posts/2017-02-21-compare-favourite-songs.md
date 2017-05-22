---
layout: default
title: Compare Favourite Songs
permalink: compare-favourite-songs
---

# Let's compare our favourite songs

In this tutorial we make a list of our favourite songs and compare it with a friend to see how "music-compatible" we are. You could use it to decide what to put on your road trip playlist!

We're going to:

* create a database where we can build up a collection of songs we like
* integrate our search functionality to add songs easily to this database
* add a page that shows us two people's favourite song collections, highlighting those in common

## Making a database

First we will create a Rails model called Favourite. For now we'll just save the title of the songs.

To create the model and associated files, generate a scaffold for a Favourite model with:

```
rails generate scaffold Favourite person:string spotify_id:string name:string preview_url:string
```

Have a look at all the files that were created then migrate your database to add the new favourites table.

```
rake db:migrate
```

Visit [http://localhost:3000/favourites/](http://localhost:3000/favourites/) and you should see that Rails has written the basic functionality for us, allowing us to list, create, edit and delete "favourite" records.

![Add form]({{ site.url }}/images/fav-song-add-form.png)

It's not very convenient yet, but it works. Let's revise our to do list.

* ~~create a database where we can build up a collection of songs we like~~
* integrate our search functionality to add songs easily to this database
* add a page that shows us two people's favourite song collections, highlighting those in common

## Integrating search

Let's add our search functionality back in. If you have a songs controller from the previous exercise, you can leave it alone, we'll make a new action for this app.

Change the `resources :favourites` line in your `config/routes.rb` file

```ruby
  resources :favourites do
    collection do
      get 'search'
    end
  end
```

Copy the search code from the last exercise into a new `search` method anywhere inside the class in `app/controllers/favourites_controller.rb`:

```ruby
  def search
    # Avoid requesting info from API if there was no search query
    q = params[:q]
    return unless q.present?

    # Request info from API
    require 'net/http'
    uri = URI.parse("https://api.spotify.com/v1/search?" + { q: q, type: 'track' }.to_query)
    json = Net::HTTP.get(uri)

    # Turn JSON-formatted string into Ruby data structure and make it available to the view
    @results = JSON.parse(json)["tracks"]["items"]
  end
```

Add a file named `app/views/favourites/search.html.erb`, and the content will be similar to the content of the songs search file in the last exercise. However, note that the first line uses `search_favourites_path` instead of `songs_search_path`.

Also, we've added a new attribute for each song - if a `preview_url` is present for the song, we'll open that up in a new tab.

```erb
<%= form_tag(search_favourites_path, method: :get) do %>
  <%= label_tag(:q, "Search for:") %>
  <%= text_field_tag(:q, params[:q]) %>
  <%= submit_tag("Search") %>
<% end %>

<% if @results.present? %>
  <h1>Songs Found</h1>

  <% @results.each do |song| %>
    <div>
      <h3><%= song["name"] %></h3>
      <% if song["preview_url"].present? %>
        <p><a href=<%= "#{song['preview_url']}" %> target="_blank" >Listen</a></p>
      <% end %>
    </div>
  <% end %>
<% end %>
```

![Search page version 1]({{ site.url }}/images/fav-songs-search1.png)

How do we save a search result to our database? Rails has many ways. Let's pick one.

What we want is a button next to each song that, when you click it, adds that song to the database and associates it with your name.

We'll need to let the user specify their name by adding some fields to the form:

```erb
  <%= label_tag(:person, "Your name:") %>
  <%= text_field_tag(:person, params[:person]) %>
```

And add a button next to each search result that will "post" the search result back into our own Rails application, setting all the fields necessary.

```erb
      <%= button_to favourites_path({
        favourite: {
          person: params[:person],
          spotify_id: song["id"],
          name: song["name"],
          preview_url: song["preview_url"],
        },
        method: :post
        }) do %>
        &#9829;
      <% end %>
```

`&#9829;` is an "HTML entity" which is jargon for a character or glyph, in this case a symbol.

![Search page version 2]({{ site.url }}/images/fav-songs-search2.png)

And after clicking the little button:

![Added from search]({{ site.url }}/images/fav-song-added.png)

Put a link on the index to our search page. Add this at the bottom of `app/views/favourites/index.html.erb`:

```erb
<br />
<%= link_to "Search for Favourites", search_favourites_path %>
```

One last change will make it much nicer, instead of those long preview addresses, let's allow the user to open the preview in a new window like we did on the search results. In the same index HTML template, replace:

```erb
<td><%= favourite.preview_url %></td>
```

With

```erb
<td>
  <% if favourite.preview_url.present? %>
    <a href=<%= favourite.preview_url %> target="_blank" >Listen</a>
  <% end %>
</td>
```

![Favourites index]({{ site.url }}/images/fav-songs-index.png)

Have a play around. It could certainly be improved in various ways, but it should let you get work done. Feel free to add any CSS in `app/assets/stylesheets/favourites.scss` if you feel like improving the layout a little, but it's not necessary for this tutorial.

Let's revise our to do list.

* ~~integrate our search functionality to add songs easily to this database~~
* add a page that shows us two people's favourite song collections, highlighting those in common

## Comparing songs

Earlier we added a page called `search`. Now we want to add a page called `compare`. The process is the same.

Add a line to `config/routes.rb`:

```ruby
  resources :favourites do
    collection do
      get 'search'
      get 'compare'
    end
  end
```

Add a link at the bottom of `app/views/favourites/index.html.erb`:

```erb
<br />
<%= link_to 'Compare Favourites', compare_favourites_path %>
```

Add a file named `app/views/favourites/compare.html.erb`:

Just like the search page, let's add a form at the top that lets the user specify two people's names:

```erb
<h1>Compare Favourites</h1>

<%= form_tag(compare_favourites_path, method: :get) do %>
  <%= label_tag(:person_a, "Person A") %>
  <%= text_field_tag(:person_a, params[:person_a]) %>
  <%= label_tag(:person_b, "Person B") %>
  <%= text_field_tag(:person_b, params[:person_b]) %>
  <%= submit_tag("Compare") %>
<% end %>
```

Now let's add a `compare` method anywhere inside the class in `app/controllers/favourites_controller.rb` to handle this information:

```ruby
  def compare
    @person_a = params[:person_a]
    @person_b = params[:person_b]
    return unless @person_a.present?
    return unless @person_b.present?
    @favourites_a = Favourite.where(person: @person_a)
    @favourites_b = Favourite.where(person: @person_b)
  end
```

I'll leave you to figure out how to show each person's list of favourite songs although if you get stuck, check the `search.html.erb` file. This is how mine looks:

![Compare page version 1]({{ site.url }}/images/fav-songs-compare1.png)

The last thing we want to do is highlight the favourite songs these two people have in common. Since you've come this far, I'm not going to tell you everything, but I'll give you one strategy that will work:

- Add a new array variable in the controller that will keep track of which Spotify ids are shared (maybe `@shared_favourite_ids = []`).
- Loop through both arrays in the controller and if you find any `spotify_id`s that are equal, add to this array (`@shared_favourite_ids.push(shared_spotify_id)`).
- Add a little code inside any loops in `compare.html.erb` to look up the new variable and determine whether to add an extra CSS class:

```erb
    <% css_class = if @shared_favourite_ids.include?(favourite.spotify_id) then "shared" else "unshared" end %>
```

You can then add this class to a `<span>` tag like this:

```erb
<li><span class="#{css_class}"><%= favourite.name %></span></li>
```

Then add some CSS to visually distinguish the shared songs.

This is what mine ended up looking like:

![Compare page version 2]({{ site.url }}/images/fav-songs-compare2.png)

Look at all the things we did today:

* ~~use our song search functionality to look up song~~
* ~~create a database where we can build up a collection of song we like~~
* ~~integrate our search functionality to add song easily to this database~~
* ~~add a page that shows us two people's favourite song collections, highlighting those in common~~

### Next steps

We've focused on getting something working, not doing it attractively, elegantly or efficiently. Now that it works, there's much we could improve.

- The suggested list of shared IDs could be made more efficient by replacing the array with a different data structure. Which method called on the array is inefficient? What data structure has a faster equivalent of this method?
- Everything is pretty ugly. Even 10 minutes of CSS will probably make everything more useable.
- This code doesn't handle errors well at all. What happens when somebody types in a name that doesn't exist? What happens when you don't fill in your name on the search page? Can you make these experiences better?
- The links for navigating around this application are a little basic. Can you rearrange them to help the user navigate to different pages more easily? For example, how do you want users to access your index page? Can you make `http://localhost:3000` go straight to the favourites list?
- What if you like a song only a little bit, or you want to hate watch a song? Update the application to allow either ratings out of 5 or text reviews or both. (TIP: You will need to add fields to your database, forms, and extra code inside your view loops to show the information).
- Can you deploy this to Heroku so you can show it to your parents?
