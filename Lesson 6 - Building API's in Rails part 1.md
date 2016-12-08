# Lesson 6 - Building API's in Rails part 1

## Intro and Agenda

In our last class we covered REST and HTTP

Tonight we'll continue that discussion by covering 

- API's... what exactly are they?
- Building API's in rails
- Active Model Serializers


## What exactly is an API

So, what exactly is an API?

If you're anything like me, I'm sure you've asked yourself this question at least a handful of times. Maybe you've even googled *what is an API* or, if you've mastered your google-fu, *what in the heck is an API and how do what should I know about it?*. Re

The term (I know I'm guilty of this) can be so loosly thrown around, that it can be hard to get a solid understanding. 

Like may other programming related concepts, API is an acronym that stands for Application Programming Interface. According to [Wikipedia](https://en.wikipedia.org/wiki/Application_programming_interface)

>  an **application programming interface (API)** is a set of subroutine definitions, protocols, and tools for building application software. A good API makes it easier to develop a computer program by providing all the building blocks, which are then put together by the programmer. An API may be for a web-based system, operating system, database system, computer hardware, or software library. An API specification can take many forms, but often includes specifications for routines, data structures, object classes, variables, or remote calls. 

So, this can give us a bit of perspective, but let's break this down. 

API's are essentially programs that communication with each other. An API can exist in many forms. Realistically, you could find yourself writing/working with API's in the form of: 

- web-based
- OS 
	- Windows
	- macOS
	- Linux
- Database
- 3rd party software  

That's not even a comprehensive list. 

When you read (or hear about) API's, people are most often referencing web-based API's. However, another example might help drive the point home. 

Let's say you wanted to create a desktop app for Windows. You would need to develop an application that communicates with the Windows Application Programming Interface. Essentially, Microsoft would give you certain guidelines in the documentation. By following those guidelines, you could take advantage of multiple facets of the windows operating system. The same could be said for macOS or Linux. 

You could also think of your Ruby classes as API's. You work with objects by calling methods that exist within the class itself. In that sense, your class is an API for your instantiated objects.

The focus of our class will be web-based REST API's. Which basically means that we could be developing a backend (server) for use with any number of different client applications... or... we might be the client utilizing another developer's API. 

This doesn't just pertain to front-end. For example, we could build a TodoApp that sends reminders to Google calendar. In that case, our application would be the client making requests to Google's API server. 


## Consuming API's

As stated in the previous class, before we build an API, I think it's a good idea to work with a couple of API's to get a lay of the land. 

Seeing how other people construct an API can serve as insight and direction for your own applications. Also, by working with other API's you begin to learn the importance of proper documentation. 


We can start by revisiting the [Star Wars API](https://swapi.co/).

At the home page, you can see a request bar (similar to Postman) that allows us to make requests to the API and review the output. This is definitely helpful, but let's travel a little further in. 

By clicking on Documentation, we can see a list of resources on the left-hand side. Clicking on each resource will give us an endpoint that we could query, and more details about how to work with it. 


Let's try querying a few things with postman. 


We could also use this with our own front-end to return the Starwars Opening Crawl by user search. Look at [Starwars Opening Crawl](https://github.com/shanebarringer/starwars_opening_crawl) for a basic idea. 



## Rails --api 

With the introduction of Rails 5, we now have an option for creating API's out of the box. This can be accomplished by using the `--api` flag. 

```ruby
$ rails new --api TodoApi -T
```

*note: we're using the `-T` flag to remove MiniTest in favor of Rspec*

This generates a new rails app without the middleware used for the view layer. If you navigate through the app, you'll notice that most everything looks the same. One difference you might notice is in the Application Controller. 

```ruby
class ApplicationController < ActionController::API

end
```

Here, ActionController is using a new module named API. This includes behaviour better suited for building API's

While we're in the app, let's add a couple of gems (that we'll discuss later).

```ruby
gem 'rack-cors'
gem 'active_model_serializers'
```

*note: don't forget to `bundle install`

Like any other rails app, we can start by building our first model

```shell
$ rails g model Task name priority:integer due_date:datetime && rake db:migrate
```

We can also go ahead and build out our Tasks controller

```shell
$ rails g controller Tasks
```

### Versioning our API

Before we start working on our controller actions, we need to discuss API versioning. Like most software, API's can have major releases that present breaking changes in the way the app previously worked. This includes deprecation of methods, changes to class structure, new migrations, just to name a few. 

Whenever you are building a standard Rails MVC application, this isn't really a major concern. Partially because you have tested your changes, and your views/UI supports the new changes. Once deployed, the user is no longer able to access the older version of the application. 

Remember, with API's other developers are utilizing your code. They may have designed entire interfaces around your response data. Once major enhancements are released, this could end up making breaking their application. 

Versioning your API is the easiest way to avoid the issue. This way developers can continue making calls to the older API, while updating their own application to work with your changes. 

In rails, API's are versions simply as `V1`, `V2`, `V3`, etc. 

In order to pull this off, we'll need to do some work in our controllers directory and route file. 

First create a new directory inside of controllers named `api`

```shell
$ mkdir ./app/controllers/api
```

Inside the api directory create a version directory

```shell
$mkdir .app/controllers/api/v1
```

Move the tasks controller inside your v1 directory

```shell
$ mv ./app/controllers/tasks_controller.rb ./app/controllers/api/v1/
```

Create a new file named `api_controller.rb` inside of v1

```shell
$ touch ./app/controllers/api/v1/api_controller.rb
```

This will be kind of like a Mini Application controller for our V1 API. Any code that might be used throughout all controllers can be placed here. Just like our application controller. Navigate to your newly created `api_controller` and add the following code. 

```ruby
module Api::V1
  class ApiController < ApplicationController

  end
end
```

Here, we have created a hierarchy for our API. Allowing our API module to access anything inside of V1. Also, we are placing our ApiController inside of the API module. 

Now, navigate to your Tasks controller and make the following changes. 

```ruby
module Api::V1
  class TasksController < ApiController 
    
  end
end
```

Here, we are wrapping our Tasks Controller in our V1 API and setting it's inheritance to our API controller. 

This may feel like a lot of work, but as your API grows, and especially when you start working on a new version, you'll be glad you took the time to things up this way. 

Now we can move to our Routes file.

Navigate to `config/routes.rb` and add the following code. 

```ruby
scope module: 'api' do
  namespace :v1 do
    resources :tasks
  end
end
```

It seems like there is a lot going on here, but I assure you, it's pretty simple. 

`scope` is entirely optional, whats happening here, is that we are scoping our Tasks Controller's actions to the `api` module. Meaning that HTTP requests now won't to have to include `/api`

This would be a good moment to run `rake routes`, let's do that.


`namespace` does exactly what it sounds like, this creates a namespace for our URI's. This means that all requests to V1 of our API, will need to be prefixed with `v1`.

`resources` as in a standard rails app, creates all of our RESTful routes.

## Challenge 1
- Create an API for ordering at a restaurant 
- Your API will ultimately have 3 models
	- Menu
	- Items
	- Orders
- For now, focus on creating the Menu part of the API
	- discuss the necessary attributes with your teammate 
	- generate the model
	- generate a controller
- Version your API
	- controllers
	- routes

**bonus:**	

- Try to write an index action 
- Fire up Postman and make a call to your API's menu index action<br>*note: for now, this should return an empty array*

## Challenge 1 Answer

```
Follow the notes and you should be fine :)
```

### Actions

Now that we have our API versioned, we can start working on implementing our controller actions. Let's start by creating the index action. 

```ruby
  def index
    @tasks = Task.all

    render json: @tasks
  end
```   

This will allow us to make a request to our index action and respond with JSON. As stated in the challenge above, If we test this out in postman we should get an empty array. Which is totally fine for now. 

Let's try it!

Now, we need to see if we can actually return some data. For the moment, let's hop into the rails console and add a task. (tests would be nice right now!)

```shell
$ rails c
```

```ruby
t = Task.create(name: 'build an API', priority: 1, due_date: DateTime.now)
```

Okay! Let's fire up the server and see what we get. 

```json
[
  {
    "id": 1,
    "name": "build an API",
    "priority": 1,
    "due_date": "2016-12-07T06:22:11.724Z",
    "created_at": "2016-12-07T06:22:11.733Z",
    "updated_at": "2016-12-07T06:22:11.733Z"
  }
]
``` 

Awesome! it works!

## Active Model Serializers

You may have noticed that we are returning all of the data. What if we only wanted to return: name, priority, and due_date? 

There are a [handful of ways](https://www.leighhalliday.com/responding-with-json-in-rails) that we can display this data correctly.

We could use the `to_json` method in our controller. 

```
def index
  @tasks = Task.all

  render json: @tasks.to_json(only: [:name, :priority, :due_date])
end
```

Send a new request to tasks index and you'll see this: 

```json
[
  {
    "id": 1,
    "name": "build an API",
    "priority": 1,
    "due_date": "2016-12-07T06:22:11.724Z",
 
  }
]
```

While that worked... it's not ideal, because handling more complex data will be cumbersome. Not to mention, our code will be much more prone to bugs, simply due to human error. 

Another way we could solve this problem is by using Jbuilder (or other similar tools) to create a JSON view. This essentially functions like an erb or haml view... just with json. While this is approach is fine for an application (with a view layer) that may occasionally return JSON, it's not ideal for an API. 

Here is a bit of code from the first snippet of the Jbuilder docs

```ruby
json.content format_content(@message.content)
json.(@message, :created_at, :updated_at)

json.author do
  json.name @message.creator.name.familiar
  json.email_address @message.creator.email_address_with_name
  json.url url_for(@message.creator, format: :json)
end
```

I do think Jbuilder is an excellent tool, so don't be afraid to use it in the future. I just think there is a more suitable way to display our data. 


The more suitable way is Active Model Serializers. Active Model Serializer is a tool that stresses convention over configuration that communicated directly with our controller. This allows us to to define how we want our data to returned. 

Let's take a look and see how it works. 

First, we'll need to create serializer. Thankfully we have a generator

```shell
$ rails g serializer task
```

*note: you can also create a model and serializer at the same time by using the resource generator: `$ rails g resource task name priority:integer due_date:datetime`*

This will create a new serializer directory inside of app. Navigate to `app/serializers/task_serializer.rb` and add the following:

```ruby
class TaskSerializer < ActiveModel::Serializer
  attributes :id, :name, :priority, :due_date
end
```

In ActiveModel Serializer `attributes` represent a whitelist of your model's attributes that will be serialized.

You may be asking yourself, what is serialization? 

At the core, serialization is simply the process of taking data (like an object) and converting it to a format that can be transported over HTTP or to saved to a database. In our situation, we are serializing a ruby object, by turning it into JSON so that it can be sent as a response over the web. 

This means that deserialization is the process of reconstructing an object from it's serialized state into it's inteded state. 

This probably makes sense for sending data over a wire, however, it may be a little fuzzy in relation to writing to a file or saving to the database. Check out the first answer in [this StackOverflow Post](http://stackoverflow.com/questions/2959661/rails-serializing-objects-in-a-database) for a great example of serialization data for your database. 




*note: don't forget to remove your previous `to_json` call in the `index` action*

Now, make another request through postman and you should get the expected output!

```json
[
  {
    "id": 1,
    "name": "build an API",
    "priority": 1,
    "due_date": "2016-12-07T06:22:11.724Z"
  }
]
```

Basically, we can tell our serializer how we want our data to be **serialized** and sent as a response. 

The interesting thing about this, is that we can also create methods for displaying data in a particular format. 

For example, the due_date comes back as a time-stamp. Let's think about how we could display it as something like Wed. Dec 7. 

Inside the serializer, add the following code: 

```ruby
class TaskSerializer < ActiveModel::Serializer
  attributes :id, :name, :priority, :converted_due_date

  def converted_due_date
    object.due_date.strftime('%A, %b %d')
  end
end
```

A couple things to note here...

We are using `object.due_date`, within the serializer, `object` represents the object being serialized! This would be the equivalent of storing an object in a variable and then operating on it. ie: `t = Task.find(1)`

Also, we are using `strftime` to convert the date. If you haven't used `strftime` before, it simply *string-i-fys* time. A great resource for this is [For a Good Strftime](http://www.foragoodstrftime.com/), which showcases lots of ways to disply your dates in string format. 

Now, we can send another request and review the response.

```json
[
  {
    "id": 1,
    "name": "build an API",
    "priority": 1,
    "converted_due_date": "Wednesday, Dec 07"
  }
]
```

Awesome! it worked! However, I don't like the attribute `converted_due_date`, we can change that by modifying our method name. Let's try this: 

```ruby
class TaskSerializer < ActiveModel::Serializer
  attributes :id, :name, :priority, :due_date

  def due_date
    object.due_date.strftime('%A, %b %d')
  end
end
```

Making another request, our response should be: 

```json
[
  {
    "id": 1,
    "name": "build an API",
    "priority": 1,
    "due_date": "Wednesday, Dec 07"
  }
]
```

Perfect! There is a lot more cool stuff you can do with Active Model Serializers, but the main takeaway is that this provides a nice layer of abstraction for determining the presentation of our data. 

## Challenge 2

- Continue building out the API from challenge 1
- Write the following controller Actions
	- index
	- create
	- show
- Using Postman, create the following 3 menus  
	- Appetizers
	- Drinks
	- Entrees
 
 
## Challenge 2 Answer
 
```
ha... you're on your own...
```

## Finishing up the Tasks Controller

To Finish out our controller, we need to add the rest of the actions.


```ruby
before_action :set_task, only: [:show, :update, :destroy]


def show
  render json: @task
end

def create
  @task = Task.new(task_params)

  if @task.save
    render json: @task, status: :created
  else
    render json: @task.errors, status: :unprocessable_entity
  end
end

def update
  if @task.update(task_params)
    render json: @task
  else
    render json: @task.errors, status: :unprocessable_entity
  end
end

def destroy
  @task.destroy
end

private
  def set_task
    @task = Task.find(params[:id])
  end

  def task_params
    params.require(:task).permit(:name, :priority, :due_date)
  end
```

Pretty straight-forward stuff here. We are simply creating our CRUD actions for a task and then responding with JSON (instead of html) 

You'll notice that we did not include a `new` or `edit` actions. This is because we aren't rendering a form for the user. Negating the need for those actions. 

## rack-cors

what it is, and how to configure

## Adding a User

Setting up associations within a Rails API is pretty straight-forward as well. Let's add a user model and associate it with Tasks.

First, we'll create a migration for associating users with tasks

```shell
$ rails g migration AddUsersToTasks user:references
```

Next we'll use the resource generator to create our model, controller and serializer, then migrate. 

```shell
$ rails g resource User email
$ rake db:migrate
```

Navigate to your User Model and add the association

```ruby
class User < ApplicationRecord
  has_many :tasks
end
```

Don't forget to associate your Task Model as well

```ruby
class Task < ApplicationRecord
  belongs_to :user
end
```

While we're adding associations, we can do the same with our serializers. Navigate to your Tasks serializer and add your association. 

```ruby
class TaskSerializer < ActiveModel::Serializer
  attributes :id, :name, :priority, :due_date
  belongs_to :user
  
  def due_date
    object.due_date.strftime('%A, %b %d')
  end
end
```

Now, navigate to your Users serializer and add your has_many association

```ruby
class UserSerializer < ActiveModel::Serializer
  attributes :id, :email
  has_many :tasks
end
```

Okay! Let's go ahead and add create, show and index actions for our User

```ruby
before_action :set_user, only: [:show, :update, :destroy]

def index
  @users = User.all

  render json: @users
end

def show
  render json: @user
end

def create
  @user = User.new(user_params)

  if @user.save
    render json: @user, status: :created
  else
    render json: @user.errors, status: :unprocessable_entity
  end
end

def update
  if @user.update(user_params)
    render json: @user
  else
    render json: @user.errors, status: :unprocessable_entity
  end
end

private
  def set_user
    @user = User.find(params[:id])
  end

  def user_params
    params.require(:user).permit(:email)
  end
```

Now we can make a post request to our user endpoint (`http://localhost:3000/v1/users`)to create a new user.

```json
{
	"user" : {
		"email" : "shane@techtalentsouth.com"
	}
}
```

Our response should look something similar to: 

```json
{
  "id": 1,
  "email": "shane@techtalentsouth.com",
  "tasks": []
}
```

Pretty neat right? Our serializer went ahead and added a tasks array for us. 

Now we can create a new task adding the user. Before we do that, we'll need to add the user to our task_params

```ruby
def task_params
  params.require(:task).permit(:name, :priority, :due_date, :user_id)
end
```

To create a new task, we simply need to add the user to the request body

```json
{
	"task" : {
		"name" : "set up associations for menu API",
		"priority": "1",
		"due_date": "10/12/16",
		"user_id" : "1"
	}
}
```

## Challenge 3

- Continue building out your API
	- Create your Item Resource
		- The Item model needs to have at least the following attributes
		- name
		- description
		- price 
	- Create several Items for each menu (yes items belong to menus)
	- Create an Order Resource
	- The order model needs at least the following attributes
		- Order Number (which could be ID)
		- Total
		- Order should belong to users and can have many items
	- Place some orders through postman :) 


## Homework

###**Due 12/12/16**

- Finish the Restaurant API challenge
    - Create a repo in our class GitHub using the naming convention `Restaurant_API_YOUR_TEAM_INITIALS_HERE`
    - each team member should create their own branch
    - Your API should
        - have 3 resources
            - Menu
            - Items
            - Orders
        - The user should be able to browse each menu 
            - being able to see all items on the menu
            - place an order
            - review their order
        - **extra credit:** 
            - write model tests
    - complete the work and push your branch to the repo
- Read [What is an API in english please](https://medium.freecodecamp.com/what-is-an-api-in-english-please-b880a3214a82#.lu4u9tx41)
- Read [To Serialize or not to Serialize](http://vaidehijoshi.github.io/blog/2015/06/23/to-serialize-or-not-to-serialize-activemodel-serializers/)
- Read [Using serialize option in Ruby on Rails](http://thelazylog.com/using-serialize-option-in-ruby-on-rails/) 
    - Write 1 takeaway from each 
    - turn in on slack
