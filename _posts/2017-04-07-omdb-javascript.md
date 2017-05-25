---
layout: default
title: Spotify API using JavaScript
permalink: api-javascript
---

# Search Spotify with JavaScript

In this tutorial, we're going to do all the same things we did in the Rails version but with JavaScript instead of Rails. The goal is to contextualise our knowledge of APIs by comparing and constrating the code.

## Steps overview

- Set up an `index.html` file and an `index.js` file and run a web server locally
- Create a search form
- Perform an AJAX request to the OMDB API when the JavaScript code is submitted
- Parse the JSON response
- Loop through the results and create HTML elements to show the results

## Basic setup

Make a directory for this project:

```
mkdir projects
cd projects
mkdir omdbjs
cd omdbjs
```

Create a basic `index.html`:

```html
<!doctype html>
<html>
  <head>
    <title>OMDB Search</title>
  </head>
  <body>
    <h1>OMDB Search</h1>

    <script src="https://code.jquery.com/jquery-3.2.0.min.js"></script>
    <script src="index.js"></script>
  </body>
</html>
```

Create a basic `index.js`:

```js
console.log("Hello world")
```

Since we've been using Rails, we should have Ruby installed. It comes with a built-in simple web server we can use to work on these files in our web browser. This is a lot like `rails s` except it only serves the files in the current directory. We don't have controllers, models, etc.

```
ruby -run -e httpd .
```

Output:

```
[2017-04-08 19:16:32] INFO  WEBrick 1.3.1
[2017-04-08 19:16:32] INFO  ruby 2.4.0 (2016-12-24) [x86_64-darwin15]
[2017-04-08 19:16:32] INFO  WEBrick::HTTPServer#start: pid=19254 port=8080
```

You should now be able to visit [http://localhost:8080](http://localhost:8080) and see a pretty empty HTML page.

If you view the browser's JavaScript console, you should see "Hello world". To access the JavaScript console in Chrome, you can use the menu: View -> Developer -> JavaScript Console. If you're using a different web browser, you might have to search for instructions or ask the person next to you or an instructor.

You may also have to reload the page to see "Hello world".

- ~~Set up an `index.html` file and an `index.js` file and run a web server locally~~
- Create a search form
- Perform an AJAX request to the OMDB API when the JavaScript code is submitted
- Parse the JSON response
- Loop through the results and create HTML elements to show the results

## Adding the form

We don't have Rails `.erb` files or templates or methods so we need to write this in plain HTML.

```html
    <form class="search-form">
      <label for="q">Search for:</label>
      <input type="text" name="q" id="q">
      <input type="submit" value="Search">
    </form>
```

We've given the form a class so we can find it from our JavaScript code.

- ~~Set up an `index.html` file and an `index.js` file and run a web server locally~~
- ~~Create a search form~~
- Perform an AJAX request to the OMDB API when the JavaScript code is submitted
- Parse the JSON response
- Loop through the results and create HTML elements to show the results

## Performing an AJAX request

AJAX stands for "Asynchronous JavaScript And XML". It's an archaic acronym that doesn't really mean anything anymore. In our case, it is going to do the same thing as `Net::HTTP` in Ruby - make an HTTP request to the OMDB API and return the results.

Let's update our `index.js` file:

```js
$(document).on("submit", ".search-form", function(event) {
  event.preventDefault();
  var q = $(event.target).find("[name=q]").val();

  $.ajax({
    url: "http://www.omdbapi.com/",
    data: { s: q },
    success: function(data) {
      console.log("success", data);
    },
    error: function(xhr, status, message) {
      console.log("error", status, message);
    }
  });
});
```

This might be a lot to take in. It's using the jQuery library to intercept the browser when it's about to submit the form. It prevents that from happening but runs our code instead of the normal form behaviour (sending an HTTP request to a server). Our behaviour is to perform an AJAX request using jQuery's `$.ajax()` function. We pass the OMDB API address and our query (in the variable `q`). When the request has finished and the response has come back, either the success or the error function will be called. Either way, we log the result on the JavaScript console which hopefully you still have open.

Phew.


- ~~Set up an `index.html` file and an `index.js` file and run a web server locally~~
- ~~Create a search form~~
- ~~Perform an AJAX request to the OMDB API when the JavaScript code is submitted~~
- Parse the JSON response
- Loop through the results and create HTML elements to show the results

## Parse the JSON response

If you click around in the JavaScript console, you might notice that the data that has come back from OMDB is already broken up into lots of little values like `Poster` and `Title`. That is because $.ajax() is so frequently used for APIs, that it assumes you're getting JSON back in the response and parses it for us. Done!

If jQuery hadn't done this step for us, we could also decode the response ourselves:

```js
var results = JSON.parse(jsonEncodedString);
```

- ~~Set up an `index.html` file and an `index.js` file and run a web server locally~~
- ~~Create a search form~~
- ~~Perform an AJAX request to the OMDB API when the JavaScript code is submitted~~
- ~~Parse the JSON response~~
- Loop through the results and create HTML elements to show the results

## Show the results

Add an element where we can put the results into:

```html
    <div class="results">
    </div>
```

Loop over the results from the API and make new HTML elements. This is the longer version of the code to try to show you everything that is happening:

```js
      var results = data["Search"];
      var resultsElement = $('.results');
      resultsElement.html("");
      for (var i = 0; i < results.length; i++) {
        var item = results[i];
        var posterElement = $("<img>").attr("src", item["Poster"]);
        resultsElement.append(posterElement);
      }
```

But jQuery's convenient functions like `$.map` lets us write a much shorter version:

```js
      $('.results').html(
        $.map(data["Search"], function(item) {
          return $("<img>").attr("src", item["Poster"]);
        })
      );
```

In most modern web browsers, we can make it even shorter using Array's map method and an arrow function:

```js
      $('.results').html(data["Search"].map(m => $("<img>").attr("src", m["Poster"])));
```

This is what mine looks like:

![Results being displayed]({{site.url}}/images/omdbjs-results.png)

- ~~Set up an `index.html` file and an `index.js` file and run a web server locally~~
- ~~Create a search form~~
- ~~Perform an AJAX request to the OMDB API when the JavaScript code is submitted~~
- ~~Parse the JSON response~~
- ~~Loop through the results and create HTML elements to show the results~~

## Checkpoint

- This version should feel faster than the Rails version. Why is that?
- What does `data["Search"]` do?
- In the code `$.map(data["Search"], ...)` and `data["Search"].map(...)`, what do you think "map" means? Check if you agree with the person next to you.
- Draw a diagram with boxes labelled:
  - Browser
  - OMDB API
- Now draw arrows starting at Browser, illustrating what happens when you click Search.
- Compare your drawing to the person next to you and discuss it.

## Now try

- What happens when you leave the field empty and click Search? What do you want to happen and how would you change the code to ensure this?
- Upload your code to a public web server and show it to your parents. (They might like the old movie posters.) Is this version easier to deploy and host than the Rails version? Why?
- Notice how there are 62 results for "kangaroo" but fewer than 62 are returned. Implement "next page" and "previous page" links.
