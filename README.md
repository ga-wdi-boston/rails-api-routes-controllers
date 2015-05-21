## Objectives
* Create a Rails application.
* Access a list of movies and songs.
* Access a specific movie and a specific song.
* Draw Diagrams that show the flow of a HTTP Requests and Responses.
* Create a simple front-end app that will use Ajax to get responses from the above Rails application.


## Demo: Create a Rails application

* Install Rails, yep it's just a ruby gem. *Note: this can be done from within any directory*  

```
gem install rails
```  
* Create a Rails application named movies_app. 

```
rails new movies_app -T --database=postgresql
```    

* Change into this movies_app directory and take a look at how Rails gives you a very clear location to place all the code you'll be writing. 


```
cd movies_app
subl .

```

No worries about what all this means, we'll get it over time.

* Start up rails.  

```
rails server
```

* In your browser, go to port 3000.

You should see this error. 
>> ActiveRecord::NoDatabaseError  
>> FATAL: database "movies_app_development" does not exist  

Oh,no Databases. We're going to see more about DB's later.


* Create a database for this rails app. *We always need a DB*. And restart rails.

```
rake db:create

rails server
```

* In your browser, go to port 3000.

Ya, you should see the Welcome Aboard page. Rails is running!!!


* Take a look at all your movies by going to *http://localhost:3000/movies*  

Bummer, you get a routing error, boo hoo, waa waa. Let's fix it.

But, lets talk about **routes** first. 

### Routes. Create a Route for /movies

Rails routes determine what will happen when we send a HTTP GET request to a specific URL. In the case above we trying to send a HTTP GET request to the route 'movies'.

But, we have told Rails what to do when we go to this URL? *Lets fix that.*

* open file config/routes.rb and add the below.  

```
get 'movies', to: 'movies#index'
```

* go to *http://localhost:3000/movies*

You should get the error *uninitialized constant MoviesController*

OK, whats going on here is that Rails saw the /movies URL and the ``get 'movies', to: 'movies#index'`` route said **when you see this /movies URL go to the MoviesController class and invoke the index method**

But there is no MoviesController class or index method?

### Controllers and Actions: 

* Create a Movies Controller with a index method/action.

1. Create a class app/controllers/movies_controller.rb.  
2. In this file add. 

```ruby
class MoviesController < ApplicationController
  def index
    render :text => "All my movies"
  end
end
```

Lets talk about what's going on here.

* Create a Movies Controller that will return JSON representation of a list of movies.  

```ruby
class MoviesController < ApplicationController
  # TEMPORARY ONLY!!
  def movies 
    [
      {name: 'Affliction', rating: 'R', desc: 'Little Dark', length: 123},
      {name: 'Mad Max', rating: 'R', desc: 'Fun, action', length: 154},
      {name: 'Rushmore', rating: 'PG-13', desc: 'Quirky humor', length: 105}
    ]
  end 

  # GET /movies
  def index
    render :json => movies.to_json
  end
end
```

## Lab:

Create a rails app that will respond to *http://localhost:3000/songs*

* It will show all the songs. 
* Create a route for */songs*  
* Create a songs controller.  
* Create a index action.  
* Create a method, named songs, that will return an array of hashes. Each hash will represent one song.  
* Each song will have a title, duration, price and artist name.

* Draw out a diagram of the flow of the HTTP Request and Response from the client/browser to the Rails server. 
	* Show the browser.  
	* Show the HTTP Request.  
	* Show the router.  
	* Show the controller.  
	* Show the HTTP Response.  


## Demo: Ajax 
### Setup Rails to allow Cross Browser HTTP Requests**

**Add this line to the Gemfile**
```
gem 'rack-cors', :require => 'rack/cors'
```

**Add these lines to the config/application.rb**

```
config.middleware.use Rack::Cors do
  allow do
    origins '*'
    resource '*', headers: :any, methods: [:get, :post, :patch, :put, :delete, :options]
  end
end
```

**Restart Rails**

```
rails server
```

### Create a frontend app

**Create a directory for a front-end application.**

```
mkdir movies_frontend
cd movies_frontend
touch index.html
touch movies.js
```

**In the index.html add**

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

**In the movies.js add**

```javascript
$(document).ready(function(){
  var $moviesList = $('#movies');

  $('#movies_button').on('click', function(event){
    $.ajax('http://localhost:3000/movies')
    .done(function(data){
      data.forEach(function(movie){
        $moviesList.append("<li>" + movie.name + "</li>");
      });
    })
    .fail(function(data){
      alert("Oops can't get from movies app!");
      console.log("Oops can't get from movies app!");
    })
  })
});
```

