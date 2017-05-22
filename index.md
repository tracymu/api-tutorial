---
layout: default
title: Let's learn about APIs
---

# Let's learn about APIs

## What is an API?

API stands for "Application Program Interfaces". In the context of the web, APIs are web sites that return data instead of web pages, allowing you to incorporate rich and interesting data and features into yourown websites. In this tutorial, we're going to use a the [Spotify API](https://developer.spotify.com/web-api/) to search for your favourite songs.

## Why learn APIs?

Learning to use APIs will allow you to write much more interesting websites full of content from the web. Other examples of powerful things you can do with APIs include using existing accounts like Facebook or GitHub to log into your website conveniently, providing relevant related photos from Flickr or Instagram, and showing data laid out on a map. For example, it only takes a few lines of JavaScript to [add a Google map in your application](https://developers.google.com/maps/documentation/javascript/adding-a-google-map). By using a map API like Google's, Google takes care of the complicated map functionality, leaving you to focus on just the functionality of your own application.

![API diagram]({{ site.url }}/images/api-diagram.png)

Your website will request information from an API using HTTP in much the same way as a web browser requests HTML using HTTP. If it goes right, you'll get data back. If something goes wrong in an API request it will return an error. For example, if you search for a non-existent record id, the API may return a 404 error (and you may have to display a nice error message to the person using your website saying you couldn't find what they were looking for). Reading over HTTP return codes can give you a feel of all the things that might go wrong if you're not careful. Some common status codes include:

![HTTP status codes]({{ site.url }}/images/http-status-codes.png)
[Click here to see full list of HTTP status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)


### An API request

Let's look at an example API by Spotify - [developer.spotify.com](https://developer.spotify.com/). We'll use the search query in their API, so we can search their music lists.

There are two main parts to a request for information from the Spotify API:

1. The URL to send all requests to
2. The parameters you include on the end of the URL

To make a search on Spotify, we're going to use their search endpoint for the URL to which we will send our requests:
```
https://api.spotify.com/v1/search
```
Parameters are included in a url after a `?` and are passed in as pairs. A parameter has a key and a value. The keys accepted by the Spotify APIs search endpoint are listed on the  [instructions page](https://developer.spotify.com/web-api/search-item/). These are the list of things you can ask the search endpoint for information about.

The parameter `q` is for your search query. For example, if you're looking for music with 'sunshine' in the title, you will append the key and value to the end of the url, like this:

[https://api.spotify.com/v1/search?q=sunshine](https://api.spotify.com/v1/search?q=sunshine)

If you make that request, you will see a response saying `"Missing parameter type"`. 
You can see from the instructions page there is a parameter `type`, which is required in every request. Valid types to use are album, artist, playlist and track.

You can include multiple parameters, separating them with `&`.

So to look for an album with the word sunshine, you can amend your request to be like this:

[https://api.spotify.com/v1/search?q=sunshine&type=album](https://api.spotify.com/v1/search?q=sunshine&type=album)


**To do:** Play around with the Spotify API page watching the changes in the request URL and the response. Build versions of these URLs yourself and play with them in your browser.

### An API response

APIs can give you a choice of format for the response. Spotify gives us JSON.

#### Understanding JSON

JSON stands for JavaScript Object Notation.

It is a way of sending information between a browser and a server, and has the following characteristics:

 - It is written as a collection of name-value pairs
 - It is contained in `{}`
 - Name and value are separated by colons, `:`
 - Name-value pairs are separated by commas `,`

An online [JSON editor](http://www.jsoneditoronline.org/) can be handy to both read JSON more easily and check that JSON you have written is formatted correctly.

This is a simplified example of JSON that might be returned by the Spotify API:

```json
{
  "name": "Sunshine & Whiskey",
  "artists": [
    {
      "name": "Frankie Ballard"
    }
  ]
}
```

An example of JSON in the interactive Ruby prompt `irb`. (`p` means "print")

```ruby
require 'json'
a = '{"album_type":"album", "name": "Sunshine & Whiskey", "artists":[{"name": "Frankie Ballard"}]}'
b = JSON.parse(a)
p b
p b["name"]
p b["artists"]
p b["artists"][0]["name"]
```

Output:

```ruby
{"album_type"=>"album", "name"=>"Sunshine & Whiskey", "artists"=>[{"name"=>"Frankie Ballard"}]}

"Sunshine & Whiskey"

[{"name"=>"Frankie Ballard"}]

"Frankie Ballard"
```
Note that the value of 'artists' is a list, which could contain more than one artist. To refamiliarise yourself with arrays in ruby, check out the [documentation](https://ruby-doc.org/core-1.9.3/Array.html).

### Checkpoint

- Which of these is JSON?

  ```plain
{"album_type":"album", "name": "Sunshine & Whiskey", "artists":[{"name": "Frankie Ballard"}]}
  ```

  or

  ```plain
{"album_type"=>"album", "name"=>"Sunshine & Whiskey", "artists"=>[{"name"=>"Frankie Ballard"}]}
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
rails new music
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
cd music
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

![New rails page]({{ site.url }}/images/new-rails.png)


### Add functionality specific to this app

To write some methods for searching and displaying music, we will need a controller, let's call it the music controller.

There are two actions we want to have this controller do

- search for songs and
- show songs,

so let's include those names in the generate command so that those methods will be created for us.

(It's easiest to leave your Rails server running and open a new Terminal or Command Prompt window to run this command. Just make sure you're in the directory where your rails server is. You might have to `cd projects/music`)

```
rails generate controller songs search
```

This means "Hey Rails, make a new controller called songs with a method called search".

Output:

```
      create  app/controllers/songs_controller.rb
       route  get 'songs/search'
      invoke  erb
      create    app/views/songs
      create    app/views/songs/search.html.erb
      invoke  test_unit
      create    test/controllers/songs_controller_test.rb
      invoke  helper
      create    app/helpers/songs_helper.rb
      invoke    test_unit
      invoke  assets
      invoke    coffee
      create      app/assets/javascripts/songs.coffee
      invoke    scss
      create      app/assets/stylesheets/songs.scss
```

We only really care about `search.html.erb` and `songs_controller.rb` in this tutorial. You can ignore the rest.

Have a look to see what additional files have been added to your app by this command.

If you look in the `config/routes.rb` file, you will see it has also added a route for you.

```ruby
Rails.application.routes.draw do
  get 'songs/search'

  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
end
```

Run your server, and you can visit those routes:

e.g. [http://localhost:3000/songs/search](http://localhost:3000/songs/search)

![placeholder]({{ site.url }}/images/placeholder.png)

This is the page where we want to put in our search box. So, open that view in your editor and read this page of the [Rails Guides](http://guides.rubyonrails.org/form_helpers.html#a-generic-search-form) to learn about making a simple search form.

Replace the code in your view file (`app/views/songs/search.html.erb`) with:

```erb
<%= form_tag(songs_search_path, method: :get) do %>
    <%= label_tag(:q, "Search for:") %>
    <%= text_field_tag(:q) %>
    <%= submit_tag("Search") %>
<% end %>
```

Which makes a simple search box that looks like this:

![search box]({{ site.url }}/images/search-box.png)

### Make a request to the Spotify API

In your songs controller (`app/controllers/songs_controller.rb`), update the `search` method.

```ruby
def search
  q = params[:q]
  return unless q.present?

  require 'net/http'
  uri = URI.parse("https://api.spotify.com/v1/search?" + { q: q, type: 'track' }.to_query)
  json = Net::HTTP.get(uri)
end
```

This will save your search term as a variable called `q`, create the URL you want to request, and perform the HTTP request to the API, storing the results in the variable `json`.

The last two steps are the same as what your web browser was doing earlier, except now this Ruby code is behaving like a web browser and saving the page to a variable.

The result comes back as a JSON list of artists and their details:

```plain
{
  "artists" : {
    "href" : "https://api.spotify.com/v1/search?query=sunshine&type=artist&offset=0&limit=20",
    "items" : [ {
      "external_urls" : {
        "spotify" : "https://open.spotify.com/artist/0CjWKoS55T7DOt0HJuwF1H"
      },
      "followers" : {
        "href" : null,
        "total" : 46277
      },
      "genres" : [ "gauze pop" ],
      "href" : "https://api.spotify.com/v1/artists/0CjWKoS55T7DOt0HJuwF1H",
      "id" : "0CjWKoS55T7DOt0HJuwF1H",
      "images" : [ {
        "height" : 640,
        "url" : "https://i.scdn.co/image/1c8b2d0aca7d482d08b30ab7879f6d8bf9c73c70",
        "width" : 640
      }, {
        "height" : 320,
        "url" : "https://i.scdn.co/image/ca29df2a6827ec401946ae8eaac16ca1f895f69b",
        "width" : 320
      }, {
        "height" : 160,
        "url" : "https://i.scdn.co/image/3566a7f24342fd62eb49d1b4a7ac7c3cffd7763d",
        "width" : 160
      } ],
      "name" : "Bipolar Sunshine",
      "popularity" : 72,
      "type" : "artist",
      "uri" : "spotify:artist:0CjWKoS55T7DOt0HJuwF1H"
    }, ...
```

### Parse the API results

The API result comes back as one giant string. This string is formatted as JSON. It has the information we need but our Ruby code needs to know how to break it apart into little keys and values so we can get the bits of information we want.

To do this, we "parse" it. In programming, parsing means interpreting a formatted string into a data structure we can use. Rails comes with the `JSON` library ready to go, so we can just use that. Add the following line into your `search` method:

```ruby
@results = JSON.parse(json)
```

This takes the big JSON-formatted string that the API sent us and turns it into a Ruby data structure we can work with. The Ruby data structure will look like this:

```ruby
{"tracks"=>{"href"=>"https://api.spotify.com/v1/search?query=sunshine&type=track&offset=0&limit=20", "items"=>[{"album"=>{"album_type"=>"album", "artists"=>[{"external_urls"=>{"spotify"=>"https://open.spotify.com/artist/0dvKgSdNB2U1gfp6ZcekYi"}, "href"=>"https://api.spotify.com/v1/artists/0dvKgSdNB2U1gfp6ZcekYi", "id"=>"0dvKgSdNB2U1gfp6ZcekYi", "name"=>"Frankie Ballard", "type"=>"artist", "uri"=>"spotify:artist:0dvKgSdNB2U1gfp6ZcekYi"}], "available_markets"=>["AD", "AR", "AT", "AU", "BE", "BG", "BO", "BR", "CA", "CH", "CL", "CO", "CR", "CY", "CZ", "DE", "DK", "DO", "EC", "EE", "ES", "FI", "FR", "GB", "GR", "GT", "HK", "HN", "HU", "ID", "IE", "IS", "IT", "JP", "LT", "LU", "LV", "MC", "MT", "MX", "MY", "NI", "NL", "NO", "NZ", "PA", "PE", "PH", "PL", "PT", "PY", "SE", "SG", "SK", "SV", "TR", "TW", "US", "UY"], "external_urls"=>{"spotify"=>"https://open.spotify.com/album/5VYrfPF3z9311jpQYEWjof"}, "href"=>"https://api.spotify.com/v1/albums/5VYrfPF3z9311jpQYEWjof", "id"=>"5VYrfPF3z9311jpQYEWjof", "images"=>[{"height"=>640, "url"=>"https://i.scdn.co/image/5a0fe5703a608ba905f278afd4b217f0a61418fa", "width"=>640}, {"height"=>300, "url"=>"https://i.scdn.co/image/7451ad190ae45dea04747c37c97e9edd1d8e2f94", "width"=>300}, {"height"=>64, "url"=>"https://i.scdn.co/image/3dd4bb8bf2b8c42e17aaedd964e54e7329162a0b", "width"=>64}], "name"=>"Sunshine & Whiskey", "type"=>"album", "uri"=>"spotify:album:5VYrfPF3z9311jpQYEWjof"},...
```

You can see that `@results` has a key of `"tracks"`, which contains two things - the URL of the search that took place, and a key called `"items"`.

The value of `"items"` is an array. Each song in that array has some attributes, including: `"album"`, `"artists"`, `"available_markets"`, `"name"`, `"duration_ms"` etc.

Let's change that last line in our search method to only return the array of songs. To do that, we specify the keys `"tracks"` and `"items"`.

```ruby
@results = JSON.parse(json)["tracks"]["items"]
```

[NOTE: If you are interested in what the `@results` object looks like, check out our lesson on how to use a [debugger](debugging)]

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
    @results = JSON.parse(json)["tracks"]["items"]
  end
end
```

### Let's show the results

We'd like to show the list of results in HTML like this:

![movie list]({{ site.url }}/images/movie-list.png)

Find the `search.html.erb` view file and add this code to the end:

```erb
<% if @results.present? %>
  <h1>Songs Found</h1>
  <ul>
    <% @results.each do |song| %>
      <li><%= song["name"] %></li>
    <% end %>
  </ul>
<% end %>
```

This code includes a loop, which will loop through each of the songs and pull out the value for the `"name"` key.

Note in your results you will see names of songs that don't necessarily include your keyword. This is because the search functionality is run on all fields of the songs - not just the name. Read the [documentation on the search endpoint](https://developer.spotify.com/web-api/search-item/) to find out how to restrict your search just to the name (or album title, or artist, etc.)

### Checkpoint

- Get a piece of paper and draw boxes on it, leaving some space around them. Label them:
  - Browser
  - Rails app
  - OMDB API
- Starting at Browser, imagine somebody presses the Search button. Now draw arrows connecting the boxes, illustrating what happens, in order from start to finish. Each arrow should represent some information or communication such as an HTTP request, or a response to an HTTP request.
- Label the arrows and try to describe what information is being passed. Parameters? JSON?
- Compare your diagram to the person next to you and discuss any differences.

### Now try

 - What happens when you replace `<li><%= song["name"] %></li>` with `<li><%= song %></li>`?
 - Can you display song attributes other than the title?
 - Can you search specifically on name or other attribute?
 - Can you add a `text_field_tag(:market)` to your form and update your controller so it searches by market also?

### Other tutorials:

 - [Compare favourite songs with a friend](/compare-favourite-songs.html)
 - [Try it again but in JavaScript](/omdb-javascript.html)
 - [More APIs to play with](/api-list.html)
 - [Debugging](/debugging.html)
