---
layout: default
title: Using a Debugger
permalink: debugging
---

# Debugging your code

Debugging gives you a way to stop your programme while it is being executed and have a better look at what is going on. This is useful when you want to inspect something, like the results from your API call, or if you have a bug in your code and you want to see what is going wrong.

Your rails app has a debugging gem called [Byebug](https://github.com/deivid-rodriguez/byebug) automatically included. To use it, you can just call `byebug` at any point in your code.

For example, within your `search` method in your `SongsController`, you can insert `byebug`

```ruby
  def search
    # Avoid requesting info from API if there was no search query
    q = params[:q]
    return unless q.present?

    # Request info from API
    require 'net/http'
    uri = URI.parse("https://api.spotify.com/v1/search?" + { q: q, type: 'track' }.to_query)
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
    7:     uri = URI.parse("https://api.spotify.com/v1/search?" + { q: q, type: 'track' }.to_query)
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
{"utf8"=>"✓", "q"=>"rain", "commit"=>"Search", "controller"=>"songs", "action"=>"search"}
(byebug) uri
#<URI::HTTPS:0x007fed68a0b088 URL:https://api.spotify.com/v1/search?q=rain&type=track>
(byebug)
```

To let your programme continue to execute, type `continue`.

You can put byebugs in any of your functions, and even in your views:

```erb
<% if @results.present? %>
  <% byebug %>
  <h1>Songs Found</h1>

```

Your `@results` variable is available in this view, so you can have a closer look at it. When you do, you will see that the structure of `@results` is an array of albums. The first item in an array is at position 0, so you can get just the first item with `@results[0]`. To get the second item in an array, you would use `@results[1]`, and so on.

```
(byebug) @results[0]
{"album"=>{"album_type"=>"single", "artists"=>[{"external_urls"=>{"spotify"=>"https://open.spotify.com/artist/7KUri7klyLaIFXLcuuOMCd"}, "href"=>"https://api.spotify.com/v1/artists/7KUri7klyLaIFXLcuuOMCd", "id"=>"7KUri7klyLaIFXLcuuOMCd", "name"=>"Stargate", "type"=>"artist", "uri"=>"spotify:artist:7KUri7klyLaIFXLcuuOMCd"}, {"external_urls"=>{"spotify"=>"https://open.spotify.com/artist/1KCSPY1glIKqW2TotWuXOR"}, "href"=>"https://api.spotify.com/v1/artists/1KCSPY1glIKqW2TotWuXOR",...
```

This is still a lot of data. 

You probably want to dig deeper to get more fine grained information. Your `@results` contains both hashes and arrays, nested inside each other. You can get information out of `@results` by using the keys of the hashes, and positions in arrays. You might like this [tutorial](https://learnrubythehardway.org/book/ex39.html) to learn more about getting information out of hashes and arrays.

For example, if I wanted to get the name of the second artist of the first album in results:

```
(byebug)  @results[0]["album"]["artists"][1]["name"]
"P!nk"
```

You can also try using the debugger to inspect objects you have saved in your database. For example, when you are making your favourite song list, you have a Favourite model. In your debugger you can call `inspect` on any of your Favourite objects. 

```
(byebug) Favourite.last.inspect
  Favourite Load (0.2ms)  SELECT  "favourites".* FROM "favourites"  ORDER BY "favourites"."id" DESC LIMIT 1
"#<Favourite id: 4, person: \"tracy\", spotify_id: \"3aWhZ1zeqy1kdjXiyMh22T\", name: \"What They Want\", preview_url: \"https://p.scdn.co/mp3-preview/ddbb456112c441e4d239...\", created_at: \"2017-05-25 22:29:55\", updated_at: \"2017-05-25 22:29:55\">"
(byebug)
```
