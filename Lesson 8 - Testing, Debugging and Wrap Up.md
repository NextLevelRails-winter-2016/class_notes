# Lesson 8 

## Recap & Agenda

Wow... we've come a long way. We've covered most of the fundamentals (and even some advanced stuff) for building API's and testing with Rspec. 

Tonight we'll wrap up by discussing

- Debugging
- Request Specs
- Next Steps

Let's get started!

## Debugging with Pry

[Pry](http://pryrepl.org/) is an excellent gem that gives us a very useful REPL with enhanced debugging power. 

One of Pry's most useful features, is that it allows us to set _breakpoints_ in our code and then step through the execution. This becomes incredibly useful when getting unexpected results. 

Furthermore, pry can be used within all of our environments (including `test`)

I would recommend installing pry globally

```shell
$ gem install pry
```

Just like `irb` you can simply type `pry` to open on a REPL and start coding. While this is great, it does not offer a _lot_ more than IRB. Where pry really shines is in it's ability to become an interactive debugger. 

With a tiny bit of configuration, you can easily start using Pry within your ruby and/or rails apps. Let's take a look at a standard rails config. 


Similiar to rspec, we'll need to use the *rails-i-fied* version of pry. We can do this by adding pry-rails to our Gemfile.

```ruby
group :development, :test do
  # ... other gems ...
  
  gem 'pry-rails'
end
```

While we're in there, let's also go ahead and add [pry-byebug](https://github.com/deivid-rodriguez/pry-byebug) as well. 

```ruby
group :development, :test do
  # ... other gems ...
  
  gem 'pry-rails'
  gem 'pry-byebug'
end
```

pry-byebug gives us extra features for examining our code inluding: 

- `next`: allowing us to stop into the next line of code (within a method) with 
- `step`: allowing us to step to the next block of code
- `break` : allowing us to add additional breakpoints
- along with additional callstack navigation options

Now that we've added our gems, let's `$ bundle install`

To use pry, we simply add a breakpoint somewhere in our code. We can start with the index action in our tasks controller

```ruby
def index

  binding.pry
  
  # ... other code within action ...
end
```

We set the breakpoint by adding `binding.pry`

If issue a request, you'll notice that the execution stops and allows you to start typing within your terminal. 

Try it out by checking your params, asking for variables, declaring variables etc. 


Not only does this work within your development enviornment, you can also use pry within your specs.


## Request Specs

With the release of Rails 5, less focus is being placed on testing your controllers. In fact, both the Rails and Rspec teams recommend replacing your controller specs with better unit and integration tests. 

While Capybara is great for testing standard Rails apps, the creator of Capybara actually recommonds not using it for testing API's.

Instead, we can use Request Specs, which are especially great for testing API's. Mainly because we need to test our response codes and verify that the content matches our expectations. 

So, what are request specs? According to the [Rspec Documentation](https://www.relishapp.com/rspec/rspec-rails/docs/request-specs/request-spec)

> Request specs provide a thin wrapper around Rails' integration tests, and are
designed to drive behavior through the full stack, including routing
(provided by Rails) and without stubbing (that's up to you).

This means that we can test our requests and responses. Including controller actions, status codes, response body, redirects, authentication, and routes. 

While not as thorough as controller specs, with more focus on good unit tests and solid request specs, you can be very confident in your test suite. 

The good news is that Request specs aren't significantly different from Controller specs. Let's dig in and see an example. 

We can start by creating a our requests directory (if it doesn't already exist), and adding our first request spec. 

```shell
$ mkdir spec/requests
$ mkdir spec/request/api
$ mkdir spec/requests/api/v1
$ touch spec/requests/api/v1/tasks_spec.rb
```

Next, we'll go ahead and setup our spec file

```ruby
require 'rails_helper'

RSpec.describe "Task Requests", :type => :request do

end
```

For now, we can comment out our `authenticate_user` method in the Tasks Controller. We'll come back to testing authorization momentarily. 

We'll also need to create a Factories for our Tasks and Users

##### `spec/factories/users.rb`

```ruby
FactoryGirl.define do
  factory :user do
    email {FFaker::Internet.email}
    password 'Password1'

    factory :user_with_tasks do
      after(:build) do |user|
        [:email, :homework].each do |task|
          user.tasks << FactoryGirl.build(task, user: user)
        end
      end
    end
  end
end
```

##### `spec/factories/tasks.rb`

```ruby
FactoryGirl.define do
  factory :homework, class: Task  do
    association :user
    name  "complete homework"
    priority 1
    due_date {DateTime.now}
  end

  factory :email, class: Task do
    association :user
    name "reply to Zack's email"
    priority 2
    due_date { DateTime.now + 2.days}
  end

  factory :invalid_task, class: Task do
    name nil
    priority nil
    due_date nil
  end
end
```



Okay! Let's start testing! We can start by adding some test variables and testing our index


```ruby
RSpec.describe "Task Requests", :type => :request do

  describe "tasks API" do
    let(:homework){FactoryGirl.create(:homework)}
    let(:email){FactoryGirl.create(:email)}


    it 'returns a list of tasks' do
   	   task1 = homework
   	   task2 = email
   	  
      get v1_tasks_path
	
	   json = JSON.parse(response.body)
		
      expect(response).to be_success
      expect(json.length).to eq(2)
    end
  end
  
end
```

In this test we are: 

- creating two new tasks
- making a request to the `v1_tasks_path` ('v1/tasks')
- Using `JSON.parse` to 
	- parse our response body
	- store that value in a variable named json
- **expecting** our response status to be `200`
- **expecting** our response body to have 2 tasks


Obviously our passing code is: 

```ruby
    def index
      @tasks = Task.all

      render json: @tasks
    end
```


Next we'll test our indiviual tasks

```ruby
it 'returns the requested task' do
  get v1_task_path(homework.id)

  json = JSON.parse(response.body)
  expect(response).to be_success

  expect(json['name']).to eq("complete homework")
end
```

Again, nothing super crazy here. We are making a get request to our tasks `show` action and expecting the reponse to be successful and include the name of the task. 

The passing code will be: 

```ruby
def show
  render json: @task
end
```

### JSON Helper

You may have noticed that we've used the same variable declaration `json = JSON.parse(response.body)` twice at this point. You can imagine that we may end up using it a few more times. Let's go ahead and move this into a method to clean up our tests a bit. 

Navigate to the bottom of your current file and add this method: 

```ruby
def json
  JSON.parse(response.body)
end
```

This method will simply return the value we want to use. 

Now we can refactor our tests by simply removing the specific line containing `json = JSON.parse(response.body)` 

```ruby
it 'returns a list of tasks' do
   	   task1 = homework
   	   task2 = email
   	  
      get v1_tasks_path
			
      expect(response).to be_success
      expect(json.length).to eq(2)
    end
  end

it 'returns the requested task' do
  get v1_task_path(homework.id)

  expect(response).to be_success
  expect(json['name']).to eq("complete homework")
end

```

Pretty nifty eh? 


Alright, let's try creating a new task


```ruby
it 'creates a new task' do
  user =  FactoryGirl.create(:user)
  task_attributes = FactoryGirl.attributes_for(:email, user_id: user.id)

  expect {
    post "/v1/tasks", params: { task: task_attributes }
  }.to change(Task, :count).by(1)

  expect(response.status).to eq(201)
end
```

While this is different than our 2 previous specs, it's not incredibly different from what you would consider using for a controller test. 

In this test we are: 

- creating a user
- creating an attributes hash for tasks (and adding the user) 
- sending a `post` request to `/v1/tasks`, and passing our tasks hash as a param. 
- expecting
	- The task count (in the database) to increase by 1
	- the response status to be 201 (created) 

Our passing code for this will be what you normally expect

```ruby
def create
  @task = Task.new(task_params)

  if @task.save
    render json: @task, status: :created
  else
    render json: @task.errors, status: :unprocessable_entity
  end
end
```

For most Request specs, we want to stay on the *happy path* and only test expected outcomes. Leaving our edge case testing to unit tests. It probably wouldn't be a bad idea though, to go ahead and test for the appropriate response code for this type of request. 

```ruby
it 'returns a 422 when given invalid data' do
  invalid_task = FactoryGirl.attributes_for(:invalid_task)

  expect {
    post "/v1/tasks", params: { task: invalid_task }
  }.to_not change(Task, :count)

  expect(response.status).to eq(422)
end
```

This should pass given our the working code above, but it's always a good idea to make sure you go from red to green. Comment out the test, watch it fail. Then uncomment and run 1 more time. 


Now we can test delete to ensure our controller action is working properly

```ruby
it 'deletes task' do
  task = homework
  expect{
      delete "/v1/tasks/#{task.id}"
   }.to change(Task, :count).by(-1)
end
```

Not much to this one. We're simply creating a task and sending a delete request using the ID of the task. 



### Adding Authentication

We commented out authentication to build out our initial specs, but obviously we're going to need to address that. 

Let's start by writing an authentication test

```ruby
it 'unauthorized user is given 401' do
    get '/v1/tasks'
    expect(response.status).to eq(401)
end
```

Here we are testing to make sure unauthorized users are given a 401. 

This can be easily fixed by uncommenting our `authenticate_user` before_action. However, when we do this, all of our other tests will fail. 

For now, let's set those tests to pending and focus on our specific authentication test. 

We can start by creating another method that will return a valid token. 

```ruby
def authentication_token(user)
  post '/v1/user_token', params: {auth: {email: user.email, password: user.password}}
  json['jwt']
end
```

In the above code, we are issuing a `post` request to obtain a token and then parsing the body. This means that the value of the method will be our token. 


Next, we can create a couple of variables to create the token and give us a variables to work with. 

```ruby
  let(:user_with_tasks){FactoryGirl.create(:user_with_tasks)}
  let(:token) { authentication_token(user_with_tasks) }
```

In our spec, we can use the `headers` option like so:

```ruby
it 'returns a list of tasks' do
  get v1_tasks_path, headers: { AUTHORIZATION: "Bearer #{token}" }

  expect(response).to be_success
  expect(json.length).to eq(2)
end
```

You'll notice that `AUTHORIZATION` is in all caps. This is a requirement of our test envorionment. All of your headers will end up being capitalized. 


Okay! now, let's refactor our index test to expect that only a user's tasks will be shown

```ruby
it 'returns a list of tasks for current user' do
  task = homework
  get v1_tasks_path, headers: { AUTHORIZATION: "Bearer #{token}" }

  expect(response).to be_success
  expect(json.length).to eq(2)
end
```

We do this, by creating another task that does not belong to the user. we can then expect to get 2 tasks (instead of 3).

This will give us a failure. To pass the test we just need to modify our index action. 

```ruby
def index
   @tasks = Task.where(user: current_user)

   render json: @tasks
end
```

Now, we can use this same format (for headers) for all of our specs that require authentication, but that's a lot of typing, so I'd recommend that we create another variable for headers like so: 

```ruby
let(:headers) { {AUTHORIZATION: "Bearer #{token}"} }
```

This will allow for us to use the following syntax in our requests

```ruby
get v1_tasks_path, headers: headers
```


### Request Helper

Before we refactor the remainder of our specs, let's go ahead and create a request helper. Because undoubtably, we will end up using these methods in other places. 

```shell
$ mkdir spec/support
$ touch spec/support/request_helper.rb
```

Now, remove those methods from `tasks_spec` file and add them to your `request_helper`

```ruby
module Requests

  module JsonHelpers
    def json
      JSON.parse(response.body)
    end
  end

  module AuthHelpers
    def authentication_token(user)
      post '/v1/user_token', params: {auth: {email: user.email, password: user.password}}
      json['jwt']
    end
  end

end
```

Obviously, these could also live in their own files, but for now we'll keep them together. 

Next open up your `rails_helper` and include the modules within your `config` block

```ruby
RSpec.configure do |config|
  config.include Requests::JsonHelpers, type: :request
  config.include Requests::AuthHelpers, type: :request
  
  # ... other code
end
```

*note: also make sure your rails_helper knows to look into the support directory `Dir[Rails.root.join('spec/support/**/*.rb')].each { |f| require f }`

Run your specs again and they should still work! 


Finally, we can refactor the remainder of our specs to use authentication


```ruby
it 'returns the requested task' do
  get v1_task_path(homework.id), headers: headers

  expect(response).to be_success
  expect(json['name']).to eq("complete homework")
end
```

Our first spec is pretty straight forward. The others will require a bit more. 

```ruby
it 'creates a new task' do
  user =  FactoryGirl.create(:user)
  headers = {AUTHORIZATION: "Bearer #{authentication_token(user)}"}
  task_attributes = FactoryGirl.attributes_for(:email, user_id: user.id)

  expect {
    post "/v1/tasks", params: { task: task_attributes }, headers: headers
  }.to change(Task, :count).by(1)

  expect(response.status).to eq(201)
end
```

In this example, we aren't using the token from our user created in the `let` statement. Instead we are creating a user local to the method. This means that we'll also need to get a new token and assign that to the headers. 

We can use this same logic throughout the remainder of our specs. 

```ruby
it 'returns a 422 when given invalid data' do
  user =  FactoryGirl.create(:user)
  headers = {AUTHORIZATION: "Bearer #{authentication_token(user)}"}
  invalid_task = FactoryGirl.attributes_for(:invalid_task)

  expect {
    post "/v1/tasks", params: { task: invalid_task }, headers: headers
  }.to_not change(Task, :count)

  expect(response.status).to eq(422)
end

it 'deletes task' do
  user =  FactoryGirl.create(:user)
  headers = {AUTHORIZATION: "Bearer #{authentication_token(user)}"}
  task = homework
  expect{
      delete "/v1/tasks/#{task.id}", headers: headers
   }.to change(Task, :count).by(-1)
end
```

Obviously, this could be refactored, but I'll leave that for you. 

## Challenge 1

What? you didn't think you'd get out of here without another challenge :)

- Clone this repo
- Checkout a new branch
- Refactor the last 3 specs 

 
## Homework 

- Read the [Api on Rails Book](http://apionrails.icalialabs.com/book/frontmatter)
- Follow along with code samples
- Send me a completed link (in slack) to your Personal GitHub Repo
	-  Include your thoughts on the book when you submit the link

