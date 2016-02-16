![General Assembly Logo](http://i.imgur.com/ke8USTq.png)

# Rails Routing and Control

## Objectives

By the end of this lesson, students should be able to:

*   Explain the roles of a router and a controller within a Rails application
*   Create a table mapping methods and paths to standard CRUD actions
*   Write basic routes for all standard CRUD actions
*   Generate standard CRUD routes using "resource routing"
*   Limit which routes "resource routing" generates, using `only:` and `except:`
*   Generate a controller to handle some category of requests
*   Create actions on a controller to handle specific requests

## Prerequisites

*   Ruby
*   MVC
*   HTTP
*   JavaScript
*   jQuery
*   AJAX

## Rails APIs from the Outside

Now that you've mocked up how an MVC app works,
and explored the file structure of a Rails application,
we'll take a look at how Routing and Control are
actually implemented in a Rails.

![Rails HTTP Client Server](./images/http_client_server.png)

As you saw in the prior demonstration,
the flow of our Rails app usually begins with a request from a client,
usually a browser.

> Feeling fuzzy on HTTP?
> Check out this [video](https://www.youtube.com/watch?v=kGOpY2J31pI)
> on HTTP GET and POST, or review the material from Unit 1 on
> [HTTP](https://github.com/ga-wdi-boston/http-study),
> and AJAX [GET](https://github.com/ga-wdi-boston/jquery-ajax-get-delete),
> [POST](https://github.com/ga-wdi-boston/jquery-ajax-post),
> and [PATCH](https://github.com/ga-wdi-boston/jquery-ajax-patch).

### Code-Along : Working with an Existing API

Let's start off by hitting an existing Rails API with `curl` requests
and looking at what we get back.

Go to [this repo](https://github.com/ga-wdi-boston/simple_rails_movies_api)
and follow the directions given in the README.

> NOTE: Be sure not to clone the `simple_rails_movies_api` repo
> inside _this repo_.

Once you've cloned `simple_rails_movies_api`,
run each of the following commands:

*   `bundle install`
Install all of the Gems for our Rails project.

> Ordinarily, we would next run (a) `rake db:create`,
> which uses Rails's in-built task runner, `rake`, to create a new database.
> Additionally, we might use other custom `rake` tasks to do things like
> populate the database with some example data for testing and debugging.
> However, this particular app is so simple
> that it doesn't even have a database -
> all of the data is hard-coded.
> This is _extremely_ atypical.

*   `rails s`
Launch the Rails server.

Finally, open your browser to `localhost:3000` -
if your app is working properly, you should see Rails's "Welcome Aboard" page.

According to the README inside the `simple_rails_movies_api` repo,
the API does two things.
If you make a GET request to `http://localhost:3000/movies`,
it will return a list of movies, in JSON format.
If you make a GET request to `http://localhost:3000/movies/<some number>`,
it will try to return one movie -
specifically, the movie whose ID matches the number in the URL.

If we hit the API with this GET request:

`curl -w "\n" http://localhost:3000/movies`

we should see the following JSON response

```JSON
[{"id":3,"name":"Affliction","rating":"R","desc":"Little Dark","length":123},{"id":7,"name":"Mad Max","rating":"R","desc":"Fun, action","length":154},{"id":10,"name":"Rushmore","rating":"PG-13","desc":"Quirky humor","length":105}]
```

Now let's try requesting a single movie.

`curl -w "\n" http://localhost:3000/movies/7`

should give us the following response:

```JSON
{"id":7,"name":"Mad Max","rating":"R","desc":"Fun, action","length":154}
```

In an actual project, we would probably be accessing this JSON using AJAX,
but CURL provides us a convenient way of testing our API
without needing to build a front-end.

## Routing and Control :: Making a New Movies API

Let's go all the way and create a brand new back-end for our front-end to use.
This new app will work the same way as the old movies API,
but will have some additional features.

Navigate to a directory where you keep your projects,
and create a new Rails app using the following command:

```bash
rails new movies_app -T --database=postgresql
```

> The `-T` flag means that we want Rails to skip setting up a testing framework.

Go into this new `movies_app` directory and take a look at the file structure;
now that you've seen a few of these, perhaps it seems a little less foreign.

```bash
cd movies_app
subl .
```

Now start up the Rails server by running `rails server`
(or just `rails s`, for short).

Let's try to access the default Rails URL.
In your browser, go to `http://localhost:3000`

Uh oh! We hit an error.

```rails
>> ActiveRecord::NoDatabaseError
>> FATAL: database "movies_app_development" does not exist
```

Looks like we need to set up a database if we want to move forward.

> Actually, this is a standard requirement -
> all Rails apps must have a database of some kind in order to function.

We'll be learning more about how databases work soon,
but for now, let's just run the command to create a new database.
We'll be using a tool called `rake` for this
(and many other things, as we go along).
`rake` is a close cousin of `grunt` -
its job is to allow us to automate tasks related to our project
and run them on command.

```bash
rake db:create
```

Then, let's restart our servers by running `rails s`.
If we open up the browser again and go to `localhost:3000`
we should see the Welcome Aboard page again - this means that Rails is running!

**OK! We're done, right?**

Well... no. If we try going to `localhost:3000`,
instead of the JSON we'd expect, we instead get a routing error.
Why? Because we haven't made any routes yet!

As you learned in the previous lesson,
a route indicates which controller action will be triggered
when a particular type of HTTP request arrives at a given URL.
In this case,
we want something to happen when we make a GET request at `/movies`.

1.  **Make a route.**
Open the file config/routes.rb and add the following text:

```ruby
get '/movies', to: 'movies#index'
```

We are creating a Rails route that will say to Rails
"When you receive a GET request at the URL path '/movies',
invoke the `index` method on the class MoviesController."

Of course, our route alone is nothing without a controller action behind it;
if we try to access `localhost:3000/movies`, we'll get another error:

```ruby
>> uninitialized constant MoviesController
```

2.  **Make a controller.**

Go to `/app/controllers/` and create a new file called `movies_controller.rb`.
Inside it, add the following code:

```ruby
class MoviesController < ApplicationController
  def index
    render :text => "All my movies"
  end
end
```

This will create a new controller based on
the default controller for our application, `ApplicationController`.
In addition, this new controller now has a method called `index`,
which should get triggered by the server in response to
a GET request appearing at `/movies`.
The only thing missing is the data -
instead of giving us data on each movie,
the `index` method (for now) only returns the text "All my movies".

3.  **Access data.**
In an actual Rails app,
retrieving and manipulating data would be done through the model.
However, for today, we'll simplify things by mocking up the model.

Edit your MoviesController to have the following code:

```ruby
class MoviesController < ApplicationController
  def index         # GET /movies
    render :json => movies.to_json
  end

  private
  def movies        # TEMPORARY ONLY!!
    [
      {name: 'Affliction', rating: 'R', desc: 'Little Dark', length: 123},
      {name: 'Mad Max', rating: 'R', desc: 'Fun, action', length: 154},
      {name: 'Rushmore', rating: 'PG-13', desc: 'Quirky humor', length: 105}
    ]
  end
end
```

This will cause MoviesController to return a JSON full of movie data,
rather than just the text "All my movies".

### Handle Cross Browser HTTP Requests

However, we soon run into an unexpected problem -
our app isn't working, and we get a mysterious error in the console:

```javascript
XMLHttpRequest cannot load http:...
No 'Access-Control-Allow-Origin' header is present on the requested resource.
Origin 'null' is therefore not allowed access.
```

What does it mean? And what's this Access-Control-Allow-Origin stuff anyway?

You may not have realized this, but for security reasons,
servers are (by default) only permitted to access their own files;
if you have an app being hosted on `localhost:5000`,
it only has the ability to see other files being hosted by the same server.
So how do we allow our front-end app, hosted on one server (`localhost:5000`)
to make AJAX requests to our Rails app,
which is on another server (`localhost:3000`)?

The answer is CORS (cross-origin resource sharing),
a system by which _some_ resources can be shared between different domains,
without automatically sharing everything.
In order to allow our front-end and back-end apps to work together,
we need to specify a CORS policy for our back-end app
that permits our front-end to interact with it.

First, add the following line of code to your Gemfile,
and run `bundle install` to download the Gem.

```ruby
gem 'rack-cors', :require => 'rack/cors'
```

Then, edit a file in the `config` directory,
`application.rb` to contain the following:

```ruby
module MoviesApp
  class Application < Rails::Application

    ...

    config.middleware.use Rack::Cors do
      allow do
        origins '*'
        resource '*', headers: :any, methods: [:get, :post, :patch, :put, :delete, :options]
      end
    end # end of CORS configuration
  end
end
```

Finally, restart the Rails server by running `rails s`.

Your back-end's CORS policy should now be set up!
Try running the AJAX request in your front end and see what happens.

### Your Turn - Create a Rails App for Songs

In pairs, create a new Rails app called `songs_app`,
with a route at `localhost:3000/songs`,
just like we did with `movies_app`.
To do this,
you will need to create a `SongsController` object with an `index` method.

Start by having your app render the text "All my songs"
in response to a GET request at `/songs`;
once that's working, create a (TEMPORARY) private method called `songs`
which returns an array of hashes (one to represent each song).
Each song will need to have a title, a duration, a price,
and the name of the artist. Feel free to pick whatever songs you like.

Once you have the app working,
draw out a diagram on your desks of the flow the application,
starting with the HTTP request and ending with the view.

## Routing and Control :: Adding Back the Front ending

Now that we have a working back-end,
let's create a new front-end for `songs_app`,
based on the one that was created for `movies_app`.
We should be able to copy the previous front-end,
so long as we are sure to update the links.

```bash
mkdir songs_frontend
cd songs_frontend
touch index.html
touch songs.js
```

## Additional Resources

*   Part 1 of **[RailsGuides : Rails Routing from the Outside In](http://guides.rubyonrails.org/routing.html)**
