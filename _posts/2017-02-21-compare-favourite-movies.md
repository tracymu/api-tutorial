---
layout: default
title: Compare Favourite Movies
permalink: compare-favourite-movies
---

# Let's compare our favourite movies

In this tutorial we make a list of our favourite movies and compare it with a friend to see how "movie-compatible" we are. You could use it to decide what movie to watch this weekend.

We're going to

* ~~use our movie search functionality to look up movies~~
* create a database where we can build up a collection of movies we like
* integrate our search functionality to add movies easily to this database
* add a page that shows us two people's favourite movie collections, highlighting those in common

## Making a database

First we will create a Rails model called Favourite. For now we'll just save the title of the movies.

To create the model and associated files, generate a scaffold for a Favourite model with:

```
rails generate scaffold Favourite person:string imdbid:string title:string year:string poster:string
```

Have a look at all the files that were created then migrate your database to add the new favourites table.

```
rake db:migrate
```

Visit [http://localhost:3000/favourites/](http://localhost:3000/favourites/) and you should see that Rails has written the basic functionality for us, allowing us to list, create, edit and delete "favourite" records.

![Add form]({{ site.url }}/images/fav-movies-add-form.png)

It's not very convenient yet, but it works. Let's revise our to do list.

* ~~create a database where we can build up a collection of movies we like~~
* integrate our search functionality to add movies easily to this database
* add a page that shows us two people's favourite movie collections, highlighting those in common

## Integrating search

Let's add our search functionality back in. If you have a movies controller from the previous exercise, you can leave it alone, we'll make a new action for this app.

Add this line anywhere between `do` and `end` in `config/routes.rb`:

```ruby
  get "favourites/search"
```

Copy the search code from the last exercise into a new `search` method anywhere inside the class in `app/controllers/favourites_controller.rb`:

```ruby
  def search
    # Avoid requesting info from API if there was no search query
    q = params[:q]
    return unless q.present?

    # Request info from API
    require 'net/http'
    uri = URI.parse("http://www.omdbapi.com/?" + { s: q }.to_query)
    json = Net::HTTP.get(uri)

    # Turn JSON-formatted string into Ruby data structure and make it available to the view
    @results = JSON.parse(json)["Search"]
  end
```

Add a file named `app/views/favourites/search.html.erb` similar to the last exercise (note, the first line has the word favourites instead of movies):

```erb
<%= form_tag(favourites_search_path, method: :get) do %>
  <%= label_tag(:q, "Search for:") %>
  <%= text_field_tag(:q, params[:q]) %>
  <%= submit_tag("Search") %>
<% end %>

<% if @results.present? %>
  <h1>Movies Found</h1>

  <% @results.each do |movie| %>
    <div>
      <h3><%= movie["Title"] %></h3>
      <%= image_tag movie["Poster"], class: "poster" %>
    </div>
  <% end %>
<% end %>
```

![Search page version 1]({{ site.url }}/images/fav-movies-search1.png)

How do we save a search result to our database? Rails has many ways. Let's pick one.

What we want is a button next to each movie that, when you click it, adds that movie to the database and associates it with your name.

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
          imdbid: movie["imdbID"],
          title: movie["Title"],
          year: movie["Year"],
          poster: movie["Poster"],
        },
        method: :post
        }) do %>
        &#9829;
      <% end %>
```

`&#9829;` is an "HTML entity" which is jargon for a character or glyph, in this case a symbol.

![Search page version 2]({{ site.url }}/images/fav-movies-search2.png)

And after clicking the little button:

![Added from search]({{ site.url }}/images/fav-movies-added.png)

Put a link on the index to our search page. Add this at the bottom of `app/views/favourites/index.html.erb`:

```erb
<%= link_to "Search", favourites_search_path %>
```

One last change will make it much nicer, instead of those long poster addresses, let's show the actual movie posters. In the same index HTML template, replace:

```erb
        <td><%= favourite.poster %></td>
```

With

```erb
        <td><%= image_tag favourite.poster, class: "poster small" %></td>
```

Have a play around. It could certainly be improved in various ways, but it should let you get work done. Feel free to add any CSS in `app/assets/stylesheets/favourites.scss` if you feel like improving the layout a little, but it's not necessary for this tutorial.

This is what I added, FYI:

```css
.poster { max-width: 100px; }
.poster.small { max-width: 50px; }
```

Let's revise our to do list.

* ~~integrate our search functionality to add movies easily to this database~~
* add a page that shows us two people's favourite movie collections, highlighting those in common

## Comparing movies

Earlier we added a page called `search`. Now we want to add a page called `compare`. The process is the same.

Add a line to `config/routes.rb`:

```ruby
  get "favourites/compare"
```

Add a link at the bottom of `app/views/favourites/index.html.erb`:

```erb
<%= link_to 'Compare', favourites_compare_path %>
```

Add a file named `app/views/favourites/compare.html.erb`:

Just like the search page, let's add a form at the top that lets the user specify two people's names:

```erb
<h2>Compare</h2>

<%= form_tag(favourites_compare_path, method: :get) do %>
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

I'll leave you to figure out how to show each person's list of favourite movies although if you get stuck, check the `search.html.erb` file. This is how mine looks:

![Compare page version 1]({{ site.url }}/images/fav-movies-compare1.png)

The last thing we want to do is highlight the favourite movies these two people have in common. Since you've come this far, I'm not going to tell you everything, but I'll give you one strategy that will work:

- Add a new array variable in the controller that will keep track of which IMDB ids are shared (maybe `@shared_imdb_ids = []`).
- Loop through both arrays in the controller and if you find any IMDB ids that are equal, add to this array (`@shared_imdb_ids.push(shared_imdb_id)`).
- Add a little code inside any loops in `compare.html.erb` to look up the new variable and determine whether to add an extra CSS class:

```erb
    <% css_class = if @shared_imdb_ids.include?(favourite.imdbid) then "shared poster" else "unshared poster" end %>
```

You can then add this class to an `<img>` tag like this:

```erb
      <%= image_tag favourite.poster, class: css_class %>
```

Then add some CSS to visually distinguish the shared movies.

This is what mine ended up looking like:

![Compare page version 2]({{ site.url }}/images/fav-movies-compare2.png)

Look at all the things we did today:

* ~~use our movie search functionality to look up movies~~
* ~~create a database where we can build up a collection of movies we like~~
* ~~integrate our search functionality to add movies easily to this database~~
* ~~add a page that shows us two people's favourite movie collections, highlighting those in common~~

### Next steps

We've focused on getting something working, not doing it attractively, elegantly or efficiently. Now that it works, there's much we could improve.

- The suggested list of shared IDs could be made more efficient by replacing the array with a different data structure. Which method called on the array is inefficient? What data structure has a faster equivalent of this method?
- Everything is pretty ugly. Even 10 minutes of CSS will probably make everything more useable.
- This code doesn't handle errors well at all. What happens when somebody types in a name that doesn't exist? What happens when you don't fill in your name on the search page? Can you make these experiences better?
- The links for navigating around this application are a little basic. Can you rearrange them to help the user navigate to different pages more easily? For example, how do you want users to access your index page? Can you make `http://localhost:3000` go straight to the favourites list?
- What if you like a movie only a little bit, or you want to hate watch a movie? Update the application to allow either ratings out of 5 or text reviews or both. (TIP: You will need to add fields to your database, forms, and extra code inside your view loops to show the information).
- Can you deploy this to Heroku so you can show it to your parents?
