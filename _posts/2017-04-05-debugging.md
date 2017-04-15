---
layout: default
title: OMDB API using JavaScript
permalink: debugging
---

# Debugging your code

Debugging gives you a way to stop your programme while it is being executed and have a better look at what is going on. This is useful when you want to inspect something, like the results from your API call, or if you have a bug in your code and you want to see what is going wrong.

Your rails app has a debugging gem called [Byebug](https://github.com/deivid-rodriguez/byebug) automatically included. To use it, you can just call `byebug` at any point in your code.

For example, within your `search` method in your `MoviesController`, you can insert `byebug`

```
  def search
    q = params[:q]
    return unless q.present?

    require 'net/http'
    uri = URI.parse("http://www.omdbapi.com/?" + { s: q }.to_query)
    byebug
    json = Net::HTTP.get(uri)
  end
```

Then, when you reload the page or try to search, your programme will be interrupted, and you will see this in your terminal:

```
    2:   def search
    3:     q = params[:q]
    4:     return unless q.present?
    5:
    6:     require 'net/http'
    7:     uri = URI.parse("http://www.omdbapi.com/?" + { s: q }.to_query)
    8:     byebug
=>  9:     json = Net::HTTP.get(uri)
   10:   end
   11: end
(byebug)
```

The terminal will show you the location of your debugger, and then let you type at the byebug prompt.

At this point, you could have a look at the params or the uri that has been built

```
(byebug) params
{"utf8"=>"✓", "q"=>"dog", "commit"=>"Search", "controller"=>"movies", "action"=>"search"}
(byebug) uri
#<URI::HTTP http://www.omdbapi.com/?s=dog>
(byebug)
```

To let your programme continue to execute, type `continue`.

You can put byebugs in any of your functions, and even in your views:

```
<% if @results.present? %>
  <h1>Movies Found</h1>
  <ul>
    <% byebug %>
    <% @results.each do |movie| %>
      <li><%= movie["Title"] %></li>
    <% end %>
  </ul>
<% end %>
```

Your `@results` variable is available in this view, so you can have a closer look at it.

```
(byebug) @results
[{"Title"=>"Bottle Rocket", "Year"=>"1996", "imdbID"=>"tt0115734", "Type"=>"movie", "Poster"=>"https://images-na.ssl-images-amazon.com/images/M/MV5BMTI2NDY4Mzc1Nl5BMl5BanBnXkFtZTcwNzUyNDk5MQ@@._V1_SX300.jpg"}, ... }]
(byebug)
```

Dig deeper in the Byebug console and inspect the first movie in the `@results`

```
(byebug) @results[0]
{"Title"=>"Bottle Rocket", "Year"=>"1996", "imdbID"=>"tt0115734", "Type"=>"movie", "Poster"=>"https://images-na.ssl-images-amazon.com/images/M/MV5BMTI2NDY4Mzc1Nl5BMl5BanBnXkFtZTcwNzUyNDk5MQ@@._V1_SX300.jpg"}
(byebug)
```
(The first item in an array is at position 0, to get the second item in an array, you would use `@results[1]`, and so on)

Go further and retrieve individual attributes of the `@results`
```
(byebug) @results[0]["Poster"]
"https://images-na.ssl-images-amazon.com/images/M/MV5BMTI2NDY4Mzc1Nl5BMl5BanBnXkFtZTcwNzUyNDk5MQ@@._V1_SX300.jpg"
(byebug)
```

Try using the debugger to inspect objects you have saved in your database. For example, when you are making your favourite movie list, you have a Favourite model. In your debugger you can call `inspect` on any of your Favourite objects. 
```
(byebug) Favourite.first.inspect
  CACHE (0.0ms)  SELECT  "favourites".* FROM "favourites"  ORDER BY "favourites"."id" ASC LIMIT 1
"#<Favourite id: 1, title: \"Layer Cake\", created_at: \"2017-01-26 21:59:50\", updated_at: \"2017-01-26 21:59:50\">"
```
