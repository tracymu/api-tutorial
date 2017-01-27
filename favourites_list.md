# Let's save our favourite movies

Since we have made our movie search a Rails app, we can create a Favourite model and save a list of our favourite movies from our searches.

First we will create a Rails model called Favourite. For now we'll just save the title of the movies.

To create the model and associated files, generate a scaffold for a Favourite model with:
`rails generate scaffold Favourite title:string `

Have a look at all the files that were created then migrate your database to add the new favourites table.

`rake db:migrate`

To add movies to our favourites list, we need to create a form around each movie returned in the search results. 

We need to make changes to this section of your search results view:

```
  <% @movies.each do |movie| %>
  <li><%= movie['Title'] %></li>
  <% end %>
 ```

Around each of the movies in the list, we'll add a form for a new Favourite. Each form will: 
  - show the movie title
  - have a `hidden_field` pre-populated with the value of that movie title to send with the form
  - a button to submit the form, which will add the favourite to your list.

```
  <% @movies.each do |movie| %>
  <%= form_for Favourite.new, url: {action: "create", controller: "favourites"} do |f| %>
    <li>
      <%= movie['Title'] %>
      <%= hidden_field(:favourite, :title, :value => movie['Title']) %>
      <%= f.submit "Add to favourites" %>
    </li>
    <% end %>   
  <% end %>
```

![movie_forms](movie_forms.png)

The forms will submit via a route that was added to your app when you created the Favourites scaffold. You can look at the file `routes.rb` to see which routes have been created for your app, and for the full list in your terminal you can run `rake routes`.

 If you look in the `favourites_controller.rb` file, you can see all the actions currently available to your Favourite model, which includes the `create` action that will handle the creation of the new Favourite. 

 If you add any of these movies to your favourites list, you will see that you are redirected to a `show` page for that favourite. This is because of the `redirect_to @favourite` in the `create` action.

 We don't want our use to lose their search results every time they add a favourite, so let's change that behaviour so that we stay on the search results page.

 



The scaffold also created an action and route for an `index` page for your favourites. An index page lists all the records for that model.

If you visit `localhost:3000/favourites` you can see the movies you have added to your favourites list.

![favourites_list](favourites_list.png)


###Next steps:
 - Format the movie search results page to better display the results - you might want to consider implementing a HTML table
 - Create and remove links in your app to help the user navigate to different pages - for example, how do you want users to access your index page?
 - Use the edit and update actions to add reviews to your movies. (TIP: you will also need to add fields to your database to hold these reviews).