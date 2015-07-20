![General Assembly Logo](http://i.imgur.com/ke8USTq.png)

# Rails Routing and Control

## Objectives

By the end of this lesson, students should be able to:

* Use AJAX to get a list of movies a simple HTTP Server.
* Use Ajax to get a list of movies from a **pre-existing** simple Rails application.
* Create a new Rails application that will provide this list of movies.
* Access a list of movies and songs from this Rails API using HTTP clients, Browser and curl.
* Access a specific movie and a specific song.

## Prerequisites

- Ruby
- HTTP

## Routing and Control :: Front-End

Now that you've mocked up how an MVC app works, and explored the file structure of a Rails application, we'll take a look at how Routing and Control are actually implemented in a Rails.

![Rails HTTP Client Server](./images/http_client_server.png)

As you saw in the prior demonstration, the flow of our Rails app usually begins with a request from a client, usually a browser. You've started building front-end apps that use AJAX before, but in case you don't remember how to do it, we'll set one up together.

> Feeling fuzzy on HTTP? Check out this [video](https://www.youtube.com/watch?v=kGOpY2J31pI) on HTTP GET and POST, or review the lessons from week 3 on [HTTP](https://github.com/ga-wdi-boston/http-intro-08) and [AJAX](https://github.com/ga-wdi-boston/ajax-get-08).

First, let's create a directory to hold our new front-end app, which we'll call `movies_frontend`, and create an HTML file and a JS file inside it.

```bash
mkdir movies_frontend
cd movies_frontend
touch index.html
touch movies.js
```

Let's also create a file called `movies.json`. We'll start off by using this file to mock up the results we might get from a back-end.

```bash
touch movies.json
```

Next, let's add some content to these files.

##### index.html
  ```html
  <html>
  <head>
    <title>Movies</title>
  </head>
  <body>
    <h1>Movies</h1>
    <ul id='movies'>
    </ul>
    <button id='movies_button'>Get Movies</button>

    <script type="text/javascript" src="http://code.jquery.com/jquery-2.1.4.min.js"></script>
    <script type="text/javascript" src='movies.js'></script>
  </body>
  </html>
  ```

##### movies.js
  ```javascript
  $(document).ready(function(){
    var $moviesList = $('#movies');

    // Get the movies from the movies.json file served from
    // the web server listening on port 5000.
    var movies_url = 'http://localhost:5000/movies.json';

    $('#movies_button').on('click', function(event){
      $.ajax({
        url: movies_url,
        dataType: 'json'
      })
        .done(function(movies_json){
          var movies = JSON.parse(movies_json);
          movies.forEach(function(movie){
            $moviesList.append("<li>" + movie.name + "</li>");
          });
        })
        .fail(function(data){
          var errorMsg = 'Error: Accessing the URL' + movies_url;
          alert(errorMsg);
          console.log(errorMsg);
        });
    })
  });
  ```

##### movies.json

  ```json
  "[{\"name\":\"Affliction\",\"rating\":\"R\",\"desc\":\"Little Dark\",\"length\":123},{\"name\":\"Mad Max\",\"rating\":\"R\",\"desc\":\"Fun, action\",\"length\":154},{\"name\":\"Rushmore\",\"rating\":\"PG-13\",\"desc\":\"Quirky humor\",\"length\":105}]"
  ```
Next, we'll launch a simple server on our machines using `httpd` on port 5000. You may remember this from Week 3 as well.

```
ruby -run -e httpd . -p5000
```

Now, open up your browser and navigate to `http://localhost:5000`. This causes the browser to do the following things:

1. Send a HTTP Request from the browser to the server we've just created (listening on port 5000) to retrieve index.html from the server.
2. Send another HTTP Request from the browser to the `code.jquery.com` server, in order to access the jQuery library.
3. Send a HTTP Request to our new server asking for the `movies.js` file.
4. Run the contents of the movies.js file after the DOM is loaded.
5. When the button is clicked, send out a GET request to our new server, asking for the `movies.json` file.

We can actually see all of these requests with the `Network` tool.

What do we see when we click the button? All of the data from `movies.json` gets served up to `movies.js` as a response to the GET request; then, `movies.js` parses the JSON and renders some HTML based on it.

> In this example, HTML is being generated in the browser based on data from a JSON. This is generally referred to as 'client-side templating', and is a common paradigm these days; it's also the one we'll be focusing on most in class. However, before this became the norm, Rails apps would often serve up fully-constructed HTML pages as views; since the work of generating the HTML was being done on the server side, this was called 'server-side templating'. We'll look more deeply at templating later this week.


## Routing and Control :: Existing Movies API
Now let's plug our new front-end into an existing back-end API, rather than simply reading from a JSON file.

Go to [this repo](https://github.com/ga-wdi-boston/simple_rails_movies_api) and follow the directions given in the README.

> NOTE: **Be sure not to clone the repo into your `movies_frontend` folder.**

> ALSO NOTE: Don't forget to make sure that Postgres is running before running `rake db:create`.

Once you've cloned the repo, set up the database, and started the server, open your browser to `http://localhost:3000` - you should see Rails's "Welcome Aboard" page if your app is working properly.

Now that we've got a running back-end, let's update our the URL of our AJAX request; setting `movies_url` to `http://localhost:3000/movies` should allow our front-end app to find the back-end. We also should not need to explicitly parse the response that we get from the server, so let's update the `.done` handler accordingly.

```javascript
	 .done(function(movies){
        movies.forEach(function(movie){
          $moviesList.append("<li>" + movie.name + "</li>");
        });
      })
```

Does the app work as expected? Fantastic!


## Routing and Control :: Making a New Movies API
Let's go all the way and create a brand new back-end for our front-end to use. This new app will work the same way as the old movies API, but will have some additional features.

Navigate to a directory where you keep your projects, and create a new Rails app using the following command:

```bash
rails new movies_app -T --database=postgresql
```

> The `-T` flag means that we want Rails to skip setting up a testing framework.

Go into this new `movies_app` directory and take a look at the file structure; now that you've seen a few of these, perhaps it seems a little less foreign.

```bash
cd movies_app
subl .
```

Now start up the Rails server by running `rails server` (or just `rails s`, for short).

Let's try to access the default Rails URL. In your browser, go to *http://localhost:3000*

Uh oh! We hit an error.
```
>> ActiveRecord::NoDatabaseError
>> FATAL: database "movies_app_development" does not exist
```

Looks like we need to set up a database if we want to move forward.

> Actually, this is a standard requirement - all Rails apps must have a database of some kind in order to function.

We'll be learning more about how databases work soon, but for now, let's just run the command to create a new database. We'll be using a tool called `rake` for this (and many other things, as we go along). `rake` is a close cousin of `grunt` - its job is to allow us to automate tasks related to our project and run them on command.

```
rake db:create
```

Then, let's restart our servers by running `rails s`. If we open up the browser again and go to *http://localhost:3000*, we should see the Welcome Aboard page again - this means that Rails is running!

##### OK! We're done, right?

Well... no. Try going to *http://localhost:3000/movies*; instead of the JSON we'd expect, we instead get a routing error. Why? Because we haven't made any routes yet!

As you learned in the previous lesson, a route indicates which controller action will be triggered when a particular type of HTTP request arrives at a given URL. In this case, we want something to happen when we make a GET request at `/movies`.

1. **Make a route.**
  Open the file config/routes.rb and add the following text:

  ```ruby
  get 'movies', to: 'movies#index'
  ```

  We are creating a Rails route that will say to Rails "When you receive a GET request at the URL path '/movies', invoke the `index` method on the class MoviesController."

  Of course, our route alone is nothing without a controller action behind it; if we try to access *http://localhost:3000/movies*, we'll get another error:

  ```ruby
  >> uninitialized constant MoviesController
  ```

2. **Make a controller.**

  Go to `/app/controllers/` and create a new file called `movies_controller.rb`. Inside it, add the following code:

  ```ruby
  class MoviesController < ApplicationController
    def index
      render :text => "All my movies"
    end
  end
  ```

  This will create a new controller based on the default controller for our application, `ApplicationController`. In addition, this new controller now has a method called `index`, which should get triggered by the server in response to a GET request appearing at `/movies`. The only thing missing is the data - instead of giving us data on each movie, the `index` method (for now) only returns the text "All my movies".

3. **Access data.**
  In an actual Rails app, retrieving and manipulating data would be done through the model. However, for today, we'll simplify things by mocking up the model.

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

  This will cause MoviesController to return a JSON full of movie data, rather than just the text "All my movies".

### Your Turn - Create a Rails App for Songs

In pairs, create a new Rails app called `songs_app`, with a route at *http://localhost:3000/songs*, just like we did with `movies_app`. To do this, you will need to create a `SongsController` object with an `index` method.

Start by having your app render the text "All my songs" in response to a GET request at `/songs`; once that's working, create a (TEMPORARY) private method called `songs` which returns an array of hashes (one to represent each song). Each song will need to have a title, a duration, a price, and the name of the artist. Feel free to pick whatever songs you like.

Once you have the app working, draw out a diagram on your desks of the flow the application, starting with the HTTP request and ending with the view.
 
## Routing and Control :: Adding Back the Front ending

Now that we have a working back-end, let's create a new front-end for `songs_app`, based on the one that was created for `movies_app`. We should be able to copy the previous front-end, so long as we are sure to update the links.

```bash
mkdir songs_frontend
cd songs_frontend
touch index.html
touch songs.js
```

However, we soon run into an unexpected problem - our app isn't working, and we get a mysterious error in the console:

```javascript
XMLHttpRequest cannot load http://... . No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'null' is therefore not allowed access.
```

What does it mean? And what's this Access-Control-Allow-Origin stuff anyway?

You may not have realized this, but for security reasons, servers are (by default) only permitted to access their own files; if you have an app being hosted on `localhost:5000`, it only has the ability to see other files being hosted by the same server. So how do we allow our front-end app, hosted on one server (`localhost:5000`) to make AJAX requests to our Rails app, which is on another server (`localhost:3000`)?

The answer is CORS (cross-origin resource sharing), a system by which _some_ resources can be shared between different domains, without automatically sharing everything. In order to allow our front-end and back-end apps to work together, we need to specify a CORS policy for our back-end app that permits our front-end to interact with it.

First, add the following line of code to your Gemfile, and run `bundle install` to download the Gem.

```ruby
gem 'rack-cors', :require => 'rack/cors'
```

Then, edit a file in the `config` directory, `application.rb` to contain the following:
```
module SongsApp
  class Application < Rails::Application

    ...

    config.middleware.use Rack::Cors do
      allow do
        origins '*'
        resource '*', headers: :any, methods: [:get, :post, :patch, :put, :delete, :options]
      end
    end
  end
end
```

Finally, restart the Rails server by running `rails s`.

Your back-end's CORS policy should now be set up! Try running the AJAX request in your front end and see what happens.

## Additional Resources

- Part 1 of **[RailsGuides : Rails Routing from the Outside In](http://guides.rubyonrails.org/routing.html)**
