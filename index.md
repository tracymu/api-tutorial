---
layout: default
title: Let's learn about APIs
---

# Let's learn about APIs

## What is an API?

APIs stands for "Application Program Interfaces". APIs are web sites and protocols that allow you to incorporate rich and interesting data and features into your websites. In this tutorial, we're going to use an API to search a database for your favourite movies.

## Why learn APIs?

Teaching you how to use APIs will let you write much more interesting websites full of content from the web. Other examples of powerful things you can do with APIs include using existing accounts like Facebook or GitHub to log into your website conveniently, providing relevant related photos from Flickr or Instagram, and showing data laid out on a map. For example, it only takes a few lines of JavaScript to [add a Google map in your application](https://developers.google.com/maps/documentation/javascript/adding-a-google-map). By using a map API like Google's, Google takes care of the complicated map functionality, leaving you to focus on just the functionality of your own application.

![](/images/api-diagram.png)

Your website will request information from an API using HTTP in much the same way as a web browser requests HTML using HTTP. If it goes right, you'll get data back. If something goes wrong in an API request it will return an error. For example, if you search for a non-existent record id, the API may return a 404 error (and you may have to display a nice error message to the person using your website saying you couldn't find what they were looking for). Reading over HTTP return codes can give you a feel of all the things that might go wrong if you're not careful. Some common status codes include:

![](/images/http-status-codes.png)
[Click here to see full list of HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)


### An API request

Let's look at an example API by the Open Movie Database - [omdbapi.com](omdbapi.com).

There are two main parts to a request for information from the Open Movie Database API:

1. The URL to send all requests to
2. The parameters you include on the end of the URL

Parameters are included in a url after a `?` and are passed in as pairs. A parameter has a key and a value. The keys accepted by the OMDB API are listed on the API instructions page. These are the list of things you can ask the API for information about.

e.g. if you include the parameter key `s` that means you are searching for a title.

The value of the parameter is what you pass in as the search term. For example, if you're looking for movies with "dog" in the title, you will append the key and value to the end of the url like this:

[http://www.omdbapi.com/?t=dog](http://www.omdbapi.com/?t=dog)

You can include multiple parameters, separating them with `&`

e.g. the OMDB API allows you to request the year of the movie with `y`
So your request for "dog" movies from "2012" would be

[http://www.omdbapi.com/?t=dog&y=2012](http://www.omdbapi.com/?t=dog&y=2012)


**To do:** Play around with the OMDB API page watching the changes in the request URL and the response. Build versions of these URLs yourself and play with them in your browser.

### An API response

APIs can give you a choice of format for the response. OMDB gives us either XML or JSON. Today we will be focusing on JSON.

#### Understanding JSON

JSON stands for JavaScript Object Notation.

It is a way of sending information between a browser and a server, and has the following characteristics:

 - It is written as a collection of name-value pairs
 - It is contained in `{}`
 - Name and value are separated by colons, `:`
 - Name-value pairs are separated by commas `,`

An online [JSON editor](http://www.jsoneditoronline.org/) can be handy to both read JSON more easily and check that JSON you have written is formatted correctly.

This is a simplified example of JSON that might be returned by the OMDB:

```json
{"Title": "Dog Day Afternoon", "Year": "1975", "Rated": "R"}
```

An example of JSON in the interactive Ruby prompt `irb`. (`p` means "print")

```ruby
require 'json'
a = '{"Title": "Dog Day Afternoon", "Year": "1975", "Rated": "R"}'
b = JSON.parse(a)
p b
p b["Title"]
p b["Year"]
```

Output:

```ruby
{"Title"=>"Dog Day Afternoon", "Year"=>"1975", "Rated"=>"R"}
"Dog Day Afternoon"
"1975"
```

### Checkpoint

- Which of these is JSON?

  ```plain
  {"Title": "Dog Day Afternoon", "Year": "1975", "Rated": "R"}
  ```

  or

  ```plain
  {"Title"=>"Dog Day Afternoon", "Year"=>"1975", "Rated"=>"R"}
  ```
    
- In the Ruby code example above, what is the type of the variable `a`? What is the type of the variable `b`? (You can check your answer by entering `a.class` or `b.class` into `irb`.)
- Why do we have two versions of the same data?
- If you guessed wrong or didn't know, ask the person next to you or an instructor.


## Let's build something using an API

We're going to build a small rails app to get information from the OMDB API via a search box and display the results.

### Set up a simple rails app

1. Make a new directory where you want your app to be saved.

```
mkdir projects
cd projects
```

2. Inside that directory make a new rails project.

```
rails new movies
```

Its output will look something like this:

```
      create
      create  README.md
      create  Rakefile
      create  config.ru
      create  .gitignore
      create  Gemfile
      create  app
      create  app/assets/config/manifest.js
      create  app/assets/javascripts/application.js

... (abbreviated!) ...

Using coffee-rails 4.2.1
Using jquery-rails 4.3.1
Using web-console 3.5.0
Using rails 5.0.2
Using sass-rails 5.0.6
Bundle complete! 15 Gemfile dependencies, 62 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
         run  bundle exec spring binstub --all
* bin/rake: spring inserted
* bin/rails: spring inserted`
```

3. Run your new project.

```
cd movies
rails s
```

Output:

```
=> Booting Puma
=> Rails 5.0.2 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
Puma starting in single mode...
* Version 3.8.2 (ruby 2.4.0-p0), codename: Sassy Salamander
* Min threads: 5, max threads: 5
* Environment: development
* Listening on tcp://localhost:3000
Use Ctrl-C to stop
```

4. View your app

Open [http://localhost:3000](http://localhost:3000) in a web browser.

You should see this image:

![](/images/new-rails.png)


### Add functionality specific to this app

To write some methods for searching and displaying movies, we will need a controller, let's call it the movies controller.

There are two actions we want to have this controller do

- search for movies and
- show movies,

so let's include those names in the generate command so that those methods will be created for us.

(It's easiest to leave your Rails server running and open a new Terminal or Command Prompt window to run this command. Just make sure you're in the directory where your rails server is. You might have to `cd projects/movies`)

```
rails generate controller movies search
```

This means "Hey Rails, make a new controller called movies with a method called search".

Output:

```
      create  app/controllers/movies_controller.rb
       route  get 'movies/search'
      invoke  erb
      create    app/views/movies
      create    app/views/movies/search.html.erb
      invoke  test_unit
      create    test/controllers/movies_controller_test.rb
      invoke  helper
      create    app/helpers/movies_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/movies.coffee
      invoke    scss
      create      app/assets/stylesheets/movies.scss
```

We only really care about `search.html.erb` and `movies_controller.rb` in this tutorial. You can ignore the rest.

Have a look to see what additional files have been added to your app by this command.

If you look in the `config/routes.rb` file, you will see it has also added a route for you.

```ruby
Rails.application.routes.draw do
  get 'movies/search'

  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

Run your server, and you can visit those routes:

e.g. [http://localhost:3000/movies/search](http://localhost:3000/movies/search)

![placeholder](images/placeholder.png)

This is the page where we want to put in our search box. So, open that view in your editor and read this page of the [Rails Guides](http://guides.rubyonrails.org/form_helpers.html#a-generic-search-form) to learn about making a simple search form.

Add this code to your template file:

```erb
<%= form_tag(movies_search_path, method: :get) do %>
    <%= label_tag(:q, "Search for:") %>
    <%= text_field_tag(:q) %>
    <%= submit_tag("Search") %>
<% end %>
```

Which makes a simple search box that looks like this:

![search box](images/search-box.png?raw=true)

### Make a request to the OMDB API

In your movies controller, update the `search` method.

```ruby
def search
  q = params[:q]
  return unless q.present?

  require 'net/http'
  uri = URI.parse("http://www.omdbapi.com/?" + { s: q }.to_query)
  json = Net::HTTP.get(uri)
end
```

This will save your search term as a variable called `q`, create the URL you want to request, and perform the HTTP request to the API, storing the results in the variable `results`.

The last two steps are the same as what your web browser was doing earlier, except now this Ruby code is behaving like a web browser and saving the page to a variable.

The result comes back looking like it did in the web browser:

```plain
{"Search":[{"Title":"Dog Day Afternoon","Year":"1975", ... }]}
```

### Parse the API results

The API result comes back as one giant string. This string is formatted as JSON. It has the information we need but our Ruby code needs to know how to break it apart into little keys and values so we can get the bits of information we want.

To do this, we "parse" it. In programming, parsing means interpreting a formatted string into a data structure we can use. Rails comes with the `JSON` library ready to go, so we can just use that. Add the following line into your `search` method:

```ruby
@results = JSON.parse(json)
```

This takes the big JSON-formatted string that the API sent us and turns it into a Ruby data structure we can work with. The Ruby data structure will look like this:

```ruby
{"Search"=>[{"Title"=>"Dog Day Afternoon", "Year"=>"1975", ... }, { Title"=>"Alpha Dog", "Year"=>"2006", "imdbID"=>"tt0426883" ... }, ...]
```

You can see that `@results` has a key of `"Search"`, whose value is an array. Each movie has a `"Title"`, `"Year"`, etc.

Let's change that last line in our search method to only return the array of movies. To do that, we specify the key `"Search"`.

```ruby
@results = JSON.parse(json)["Search"]
```

[NOTE: If you are interested in what the `@results` object looks like, ask an instructor about using a debugger]

This is what the code looks like now:

```ruby
class MoviesController < ApplicationController
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
end
```

### Let's show the results

We'd like to show the list of results in HTML like this:

![movie list](images/movie-list.png)

Find the `search.html.erb` view file and add this code to the end:

```erb
<% if @results.present? %>
  <h1>Movies Found</h1>
  <ul>
    <% @results.each do |movie| %>
      <li><%= movie["Title"] %></li>
    <% end %>
  </ul>
<% end %>
```

This code includes a loop, which will loop through each of the movies and pull out the value for the `"Title"` key.

### Checkpoint

- Get a piece of paper and draw boxes on it, leaving some space around them. Label them:
  - Browser
  - Rails app
  - OMDB API
- Starting at Browser, imagine somebody presses the Search button. Now draw arrows connecting the boxes, illustrating what happens, in order from start to finish. Each arrow should represent some information or communication such as an HTTP request, or a response to an HTTP request.
- Label the arrows and try to describe what information is being passed. Parameters? JSON?
- Compare your diagram to the person next to you and discuss any differences.

### Now try

 - What happens when you replace `<li><%= movie["Title"] %></li>` with `<li><%= movie %></li>`?
 - Can you display movie attributes other than the title?
 - Can you add a `text_field_tag(:year)` to your form and update your controller so it searches by year also?

### Other tutorials:

 - [Save a list of your favourite movies](favourites_list)
 - Twitter api
 - make it pretty
 - geoff to do a weather one?
 - javascript module - e.g. carousel
 - other available apis to work on  - list
 - do this exact same exercise but with front-end JavaScript code using jQuery and AJAX.

