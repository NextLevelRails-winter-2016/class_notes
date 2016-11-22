# Lesson 3 - Tesing Rails Controllers and Features

## Recap & Agenda

Last class we discussed: 

- Rails Application Setup/Config with Rspec and friends
- Testing our Models
- Factory Girl
- Using FFaker in our factories

Tonight we'll cover:

- Controller tests
	- Anatomy of a controller test
	- GET
	- POST
	- PUT/PATCH
	- DELETE
- Unit vs Integration Tests
- Database Cleaner Configuration 
- Capybara and Feature Specs


## Controller Tests

Due to the fact that we will shortly move into creating API's, learning how to test your controllers is important. 

As you know, our controllers handle Requests from the Client and Responses from the Server. 

At first glance, controller specs can appear to be confusing on the surface. Let's look at a snippet from a basic todo app

```ruby
describe "GET #index" do
    it "assigns all tasks as @tasks" do
      task = Task.create!(valid_attributes)
      get :index
      expect(assigns(:tasks)).to eq([task])
    end
  end

  describe "GET #show" do
    it "assigns the requested task as @task" do
      task = Task.create! valid_attributes
      get :show, params: {id: task.to_param}
      expect(assigns(:task)).to eq(task)
    end
  end

  describe "GET #new" do
    it "assigns a new task as @task" do
      get :new, params: {}
      expect(assigns(:task)).to be_a_new(Task)
    end
  end
```

However, once you learn the bits of new syntax, you'll be able to write controller specs in your sleep. 

We can start by thinking about what our controller should do, and the things we should test. 

- Does the controller redirect to the expected view or render the expected template?
- Does the controller give the appropriate http response code?
- What type of content should we respond with?
- Does the controller need to render flash messages?
- Was any information added/updated/deleted?

With this in mind, we can start to dig in and write some specs for an application. 

First, we'll need to generate a Tasks controller

```shell
$ rails generate controller Tasks
```

Take a moment to review the files created by our generator. Specifially, you'll notice that a spec file was created for us in the `spec/controllers` directory.

Go ahead and open that file 


### Redirect and Render

To start, we can test our *http GET* requests and then move into the other various http actions

Let's write an initial test to determine if our index action redirects to the appropriate view/template

```ruby
describe "GET #index" do

  it 'renders the index template' do
    get :index
      
      expect(response).to render_template(:index)
  end
end
```

Let's start with the familiar points

You'll notice that our `describe` and `it` blocks are very similar. Additionally, aside from a new *actual value* (response) and a new *render_template* matcher  our `expect` block looks exactly the same. 

For the new syntax. 

We are declaring `get :index` 

Within Rspec each HTTP verb has its own method. In this case our method is `get`, which takes a controller action as the first argument. As we'll see shortly, you can pass additional arguments that take the place of params.

Response is an object provided in our controller specs that represents the finished product from our controller action. 

In this case we are *expecting* our **response** to render the index view

Running this spec will present the following error: 

```shell
Failures:

  1) TasksController GET #index renders the index template
     Failure/Error: get :index

     ActionController::UrlGenerationError:
       No route matches {:action=>"index", :controller=>"tasks"}
       
```

This is telling us that we are missing a route. Specifically a route for `"tasks#index`

We can address this error by navigating to `config/routes.rb` and adding: 

```ruby
get 'tasks' => 'tasks#index'
```

By running `$ rake routes` you'll see that an index route has been created. 

Running the spec again, we'll get the following error:

```shell
Failures:

  1) TasksController GET #index renders the index template
     Failure/Error: get :index

     ActionController::UnknownFormat:
       TasksController#index is missing a template for this request format and variant.

       request.formats: ["text/html"]
       request.variant: []

       NOTE! For XHR/Ajax or API requests, this action would normally respond with 204 No Content: an empty white screen. Since you're loading it in a web browser, we assume that you expected to actually render a template, not… nothing, so we're showing an error to be extra-clear. If you expect 204 No Content, carry on. That's what you'll get from an XHR or API request. Give it a shot.
       
```

This test is letting us know that we are missing an index template. 

Let's go ahead and create that and run our test again

```shell
$ touch app/views/index.html.erb
```

Running our test again, the spec will automatically pass. Why? Well, the default behavior of our `index` action is to `render` the `index` view/template. 

So, yes, this test is not completely necessary, however, now you understand `render_template` yay!

### index

Now our spec should pass. Let's add an additional spec to this describe block.  

The next spec will look like this: 

```ruby
it 'returns all tasks for user' do
   user = create(:user)

   get :index
      
   expect(assigns(:tasks)).not_to be_nil
end
```

This test will: 

- create a user
- make a `get` request for the tasks/index view
- expect that a `@tasks` instance variable is created and that it will *not* be nil. 

#### `assigns()`

Let's take a moment and look at `expect(assigns())`

`assigns` references instance variables that you expect to be created within your controller action. ie: 

- `@tasks = Task.all`
- `@task = Task.new`

In the example of:<br>
`expect(assigns(:tasks)).not_to be_nil`, we are expecting an instance variable of `@tasks` to not be `nil`

When we run our test we should get the following error:

```shell
Failures:

  1) TasksController GET #index returns all tasks for user
     Failure/Error: expect(assigns(:tasks)).not_to be_nil

       expected: not nil
            got: nil
            
```

To address this spec we can simply add the following code to our `index` action

```ruby
def index

	@tasks = Task.all
	
	# ... respond_to ...
	
end
```

### new

Now, lets move on to our `new` action

We can start with familiar code to test that the appropriate template is being served up. 

```ruby
describe 'GET #new' do
  it 'renders the new template' do
    get :new
    expect(response).to render_template(:new)
  end
end
```

Which should result in the following error: 

```shell
Failures:

  1) TasksController GET #new renders the new template
     Failure/Error: get :new

     ActionController::UnknownFormat:
       TasksController#new is missing a template for this request format and variant.

       request.formats: ["text/html"]
       request.variant: []

       NOTE! For XHR/Ajax or API requests, this action would normally respond with 204 No Content: an empty white screen. Since you're loading it in a web browser, we assume that you expected to actually render a template, not… nothing, so we're showing an error to be extra-clear. If you expect 204 No Content, carry on. That's what you'll get from an XHR or API request. Give it a shot.
       
```

To address this failure, we simply need to add a *new* html view

```shell
$ touch app/views/tasks/new.html.erb
```

Running the test again should result in success.

Next we'll write a spec to test our New action is behaving as expected. 

```ruby
describe '#GET new` do 

# ... other spec ...

  it "assigns a new task to @task" do
    get :new
    expect(assigns(:task)).to be_a_new(Task)
  end
end
```

Here we are: 

- making a `get` request to our `new` action
- expecting the `@task` variable to be a *new* instance of the Task class

Our failure should be: 

```ruby
Failures:

  1) TasksController GET #new assigns a new task to @task
     Failure/Error: expect(assigns(:task)).to be_a_new(Task)
       expected nil to be a new Task(id: integer, name: string, priority: integer, created_at: datetime, updated_at: datetime, due_date: datetime, user_id: integer)
       
```

As you can see, we got a `nil` value for `@task` and our expected value is a new Task object:<br>`Task(id: integer, name: string, priority: integer, created_at: datetime, updated_at: datetime, due_date: datetime, user_id: integer)`

Our passing code will be written in the new action within our controller: 

```ruby
def new
  @task = Task.new
end
```

We only have a couple more `get` requests to test 

Let's move on to `show`

This time, we're going to invert our specs and start with testing the value of `@task`

```ruby
it "assigns the requested task as @task" do
    task = create(:homework)
      
    get :show, params: {id: task.to_param}
      
    expect(assigns(:task)).to eq(task)
end
```

Our new syntax in this spec is found in our `get` request where we are passing params.<br>
By using the syntax `params: {}` we can pass a params hash with our request. This is followed by assigning an `id` to `task.to_param`

`to_param` converts our newly created task into a url parameter. 

#### using `raise` and `.inspect` to inspect our values

You can check this out by *temporarily* adding the following code to your test

```ruby

it "assigns the requested task as @task" do
    task = create(:homework)
      
    get :show, params: {id: task.to_param}
      
    raise task.to_param.inspect
      
    expect(assigns(:task)).to eq(task)
end

```

Which will give us an stringified ID. 

and 

```ruby
it "assigns the requested task as @task" do
    task = create(:homework)
      
    get :show, params: {id: task.to_param}
    
    raise task.inspect
      
    expect(assigns(:task)).to eq(task)
end
```

Which will yield an entire task object. 

Going back to our original spec, we should get the following error 

```shell
Failures:

  1) TasksController GET #show assigns the requested task as @task
     Failure/Error: expect(assigns(:task)).to eq(task)

       expected: #<Task id: 5, name: "complete homework", priority: 1, created_at: "2016-11-20 20:13:43", updated_at: "2016-11-20 20:13:43", due_date: "2016-11-20 20:13:43", user_id: 11>
            got: nil

       (compared using ==)
```

Our passing code could simply be: 

```ruby
def show
  @task = Task.find(params[:id])
end
```

However, following best practice let's go ahead and write a `before_action`

```ruby
class TasksController < ApplicationController
  before_action :set_task, only: [:show, :edit, :update, :destroy]

  # ... actions ...
  
  private
  
  def set_task 
    @task = Task.find(params[:id])
  end
end
```

Next we'll write our render spec

```ruby
it "renders the :show template" do
   task = create(:email)
   get :show, params: { id: task.to_param }
   
   expect(response).to render_template :show
end
```

This should automatically pass, however, it's nice to know that we have the render action covered by tests. 

Finally, we'll write our edit spec which is very similar to show. For the sake of time, we're going to move on. The tests/code can be found below:


##### spec

```ruby
  describe 'GET #edit' do
    it "assigns the requested task as @task" do
      task = create(:homework)

      get :edit, params: {id: task.to_param}

      expect(assigns(:task)).to eq(task)
    end

    it "renders the :edit template" do
      task = create(:email)
      get :edit, params: { id: task.to_param }
      expect(response).to render_template :edit
    end
  end
```

##### code

```ruby
def edit
end
```

`$ touch app/views/tasks/edit.html.erb`


### POST

Now you have a pretty solid foundation for writing most of your `get` based requests. Testing `post` requests is a little more involved, but once you break down the specs, you'll be up and running in no time! 

Create will be a good place to start, so let's go ahead and scratch out our tests. 

```ruby
describe "POST #create" do
  context "with valid attributes" do
    it 'persists new contact' 
      
    it 'redirects to show page'
  end

  context 'with invalid attributes' do
    it 'does not persist contact' 

    it 're-renders :new template'
  end
end
```
#### `attributes_for`

In this scenario, there are 2 things worty of note: 

1. we are now going to test for valid and invalid attributes. Logically, this is necessary as there are 2 natural outcomes for trying to create a new task 
2. We are using `context`, which works exactly like `describe`. This is a convention used by many Ruby/Rspec developer to enhance readability within our specs.

Before we write our first spec, let's go ahead and make use of `let`

```ruby
describe "POST #create" do    
  let(:user) {create(:user)}
  let(:valid_attributes) { attributes_for(:email, user_id: user.id) }
  let(:invalid_attributes) { attributes_for(:invalid_task)}

```

Above we have created a set of **valid attributes** based on our `:homework` factory, and a set of **invalid attributes** based on our `:invalid_task` factory

We've also snuck in another bit of new syntax in the form of `attributes_for`

`attributes_for` is a factory_girl method that simply creates a `hash` (as opposed to an object). Consider it to be a cousin of `build(:factory_name)`, however, instead of creating an object in memory, a hash of attributes (matching the object's attributes) is created. 

This allows us to test the parameters being submitted by the client, but does come with a slightly confusing drawback. `attributes_for` does not build an association. Thus, we've also created a user object and added it to the attributes of our valid task hash. 

This will also, require us to slightly reconfigure our User Factory. 

```ruby
FactoryGirl.define do
  factory :user do
    firstname {FFaker::Name.first_name}
    lastname {FFaker::Name.last_name}
    email {FFaker::Internet.email}

    factory :user_with_tasks do

      after(:build) do |user|
        [:email, :homework].each do |task|
          user.tasks << FactoryGirl.build(:email, user: user)
        end

      end
    end
  end
end
```

Above, we have added an additional factory named `users_with_tasks` this factory will inherit `firstname`, `lastname` and `email` from our standard `user` factory. Then only this new factory will use the `after(:build)` method. Allowing us more flexibility in our tests. 

As you would expect, this will cause a few of our previous tests to fail. Let's go ahead and run our whole test suite and get those specs passing again. 

Navigate to `spec/models/user_spec.rb` 

```ruby
RSpec.describe User, type: :model do

  let(:user) { build(:user) }
  let(:user_with_tasks) { build(:user_with_tasks) }
  
  # ... specs
  
  it 'has many tasks' do
    expect(user_with_tasks.tasks.length).to eq(2)
  end

 it 'returns tasks due today' do
   task = user_with_tasks.tasks.first
   task.update(due_date: DateTime.now)
   expect(user_with_tasks.due_today.length).to eq(1)
 end
end
```

Here we have modified the two specs that test for associations, and made our tests green again. 


Now, we can move on to writing our persistance test for the `create` method. Our spec is going to look slightly different, but hang in there with me, it will make sense shortly. 

```ruby
it 'persists new task' do
  expect{
     post :create, params: {task: valid_attributes}
  }.to change(Task, :count).by(1)
end
```

Unlike other specs, we are not instantiating an object. Instead, we are going to pass the full http request to our expect block. 

Using `{}` for our `expect`ation is new. So let's break that down...

The HTTP request is being passed in as a `Proc` object. <br>Yes, expect can take a block/prock. Allowing us to execute a code within the block and evaluate it afterwards.

In this case, we are: 

- passing a hash of task attributes to our create method
- expecting the count of Tasks (in the database) to increase by 1

The failure for this spec should look similar to this: 

```shell
Failures:

  1) TasksController POST #create with valid attributes persists new task
     Failure/Error:
       expect{
         post :create, params: {task: valid_attributes}
       }.to change(Task, :count).by(1)

       expected #count to have changed by 1, but was changed by 0

```

Our implemetation of working code is going to be slightly more complex as well. We can start with the following code 

```ruby
  def create
    @task = Task.new(task_params)
    @task.save
  end
```

This will present another failure that we need to address: 

```shell
Failures:

  1) TasksController POST #create with valid attributes persists new task
     Failure/Error: @task = Task.new(task_params)

     NameError:
       undefined local variable or method `task_params' for #<TasksController:0x007fa194412c28>
       Did you mean?  task_path
       
```

Up until now, we have not defined our task parameters. This can be easily addressed by adding the very familiar rails controller parameter code. 

```ruby
def task_params
  params.require(:task).permit(:name, :priority, :due_date, :user_id)
end
```

Running our spec again, we should find that all tests are passing. 

```ruby
it 'redirects to show page' do
  post :create, params: { task: valid_attributes }
  expect(response).to redirect_to(assigns(:task))
end
```

Here we are using a new matching of `redirect_to` this is very similar to the `render_template` method, with the exception that it kicks off a whole new MVC cycle. 

This test should look/feel familiar. We are creating a new task and expecting to be redirect to the show page. Our failing spec should resemble this: 

```shell
Failures:

  1) TasksController POST #create with valid attributes redirects to show page
     Failure/Error: expect(response).to redirect_to(assigns(:task))
       Expected response to be a <3XX: redirect>, but was a <204: No Content>
       
```

We need to choose how to respond if our attempt to create a task is successful. 

Our passing code will be: 

```ruby
def create
  @task = Task.new(task_params)
  
  respond_to do |format|
    if @task.save
      format.html { redirect_to task_path(@task) }
    end
  end
end
```

As stated eariler, we also need to write some specs for the event that the user's attempt to create a task fails. 

We'll go ahead and write both specs for this particular as they are both dependent upon the same code. 

```ruby
context 'with invalid attributes' do
  it 'does not persist contact' do
    expect{
      post :create, params: {task: invalid_attributes}
    }.not_to change(Task, :count)
  end

  it 're-renders :new template' do
    post :create, params: {task: invalid_attributes}
    expect(response).to render_template(:new)
  end
end
```

Our failure will look like this: 

```ruby
Failures:

  1) TasksController POST #create with invalid attributes does not persist contact
     Failure/Error:
       respond_to do |format|
         if @task.save
           format.html { redirect_to task_path(@task) }
         end

     ActionController::UnknownFormat:
       ActionController::UnknownFormat
     # ./app/controllers/tasks_controller.rb:25:in `create'
     # ./spec/controllers/tasks_controller_spec.rb:86:in `block (5 levels) in <top (required)>'
     # ./spec/controllers/tasks_controller_spec.rb:85:in `block (4 levels) in <top (required)>'

  2) TasksController POST #create with invalid attributes re-renders :new template
     Failure/Error:
       respond_to do |format|
         if @task.save
           format.html { redirect_to task_path(@task) }
         end

     ActionController::UnknownFormat:
       ActionController::UnknownFormat
     # ./app/controllers/tasks_controller.rb:25:in `create'
     # ./spec/controllers/tasks_controller_spec.rb:91:in `block (4 levels) in <top (required)>'
     
```

This is letting us know that we don't have any kind of mechanism for handling failure within our create action. Our passing code simply needs to include an `else` statement re-rendering the `new` template.

```ruby
  respond_to do |format|
      if @task.save
        format.html { redirect_to task_path(@task) }
      else
        format.html { render :new }
      end
    end
```

Now our spec should be passing. 

### PUT/PATCH update

At this point, controller testing should be starting to feel accessible to you. Thankfully, testing our update method is very similar to create. with a few minor modifications. Below you'll find the code for all of our update specs, along with the passing code and a brief explanation at the end. 

#### Full Spec

```ruby
describe '#PATCH #update' do
  let(:task) { create(:email) }
  let(:new_attributes) { attributes_for(:homework) }
  let(:invalid_attributes) { attributes_for(:invalid_task)}

  context 'with valid params' do
    it 'updates the selected task' do
      patch :update, params: { id: task.to_param, task: new_attributes }

      task.reload

      expect(task.name).to eq('complete homework')
      expect(task.priority).to eq(1)
    end

    it 'redirects to the task' do
      patch :update, params: { id: task.to_param, task: new_attributes }
      task.reload

      expect(response).to redirect_to(task)
    end
  end

  context 'with invalid params' do
    it 'does not update the task' do
      patch :update, params: {id: task.to_param, task: invalid_attributes}
      expect(assigns(:task)).to eq(task)
    end

    it 're-renders the edit template' do
      patch :update, params: {id: task.to_param, task: invalid_attributes}
      expect(response).to render_template(:edit)
    end
  end
end
```

#### Controller Code

```ruby
  def update
    respond_to do |format|
      if @task.update(task_params)
        format.html { redirect_to task_path(@task)}
      else
        format.html { render :edit }
      end
    end
  end
```

The first thing to notice is that we've modified our `let` blocks and included: 

- task (that is already persisted to the db)
- a new hash of attributes 
- an invalid hash 
- we don't need to create a user object in this instance, because our association is added through our `let(:task)` statement. 

we've also used a new method in the form of `task.reload` this will reload the task from the database and allow us to verify that the details have been changed. 

Our invalid params specs are very similiar to the create specs. With the exception that we are testing that the user is sent back to the `:edit` view/template. 

#### DELETE #destroy

Last but not least, we'll need to test our `destroy` action. Can you guess what the spec will look like? 

```ruby
describe "DELETE #destroy" do
  let(:task) {build(:homework)}

  it "destroys the requested task" do
    task.save
    expect {
      delete :destroy, params: {id: task.to_param}
    }.to change(Task, :count).by(-1)
  end

  it "redirects to the tasks list" do
    task.save
    delete :destroy, params: {id: task.to_param}
    expect(response).to redirect_to(tasks_path)
  end
end
```

While there are several ways you can write this spec. For me, the above is the most readable. 

We are building a task, saving, then destroying it. 

Our passing code will be: 

```ruby
def destroy
  if @task.destroy
    respond_to do |format|
      format.html { redirect_to tasks_path }
    end
  end
end
```

## Integration testing with Capybara

Now that we have pretty solid coverage with unit tests, we can move into some basic feature testing. This is also known as integration testing. 

The major differences between Unit and Integration tests, is that integration tests check for how our models, controllers, etc. interact with multiple scenarios. Requiring us to zoom out a bit and think about the big picture of how users will interact with our application. 

When building a normal Rails application with Models Controllers *and Views*, the preferred way of perfoming integration tests is with a tool by the name of Capybara. While there are other options available, we will focus primarily on using/understanding Capybara. 

Capybara's strength is that it allows us to simualate real-world use cases within our application. This is accompished by Capybara creating a virtual DOM and running through our view layers. Allowing us to continue working on our development cycles and spend much less time clicking around, filling out forms and waiting on page reloads. 

The specs are written in very similar style to Rspec, so, while we will be introducing new syntax. You should catch on pretty quickly. 


We have already set our application up to use Capybara, by adding it to our Gemfile. However, before we get started let's go ahead and add another useful Gem by the name of Database Cleaner

### Database Cleaner

Add Database Cleaner to the `:test` group in your Gemfile. 

```ruby
gem 'database_cleaner'
```

```shell
$ bundle install
```

Next, we'll use Avdi Grimm's [wonderful configuration](http://www.virtuouscode.com/2012/08/31/configuring-database_cleaner-with-rails-rspec-capybara-and-selenium/)

I would encourage you to read through this explanation. 

Step 1 - Navigate to `spec/spec_helper.rb` and change the following line of code (from true to false

```ruby
config.use_transactional_fixtures = false
```

Step 2 - create a `support` directory and a `database_cleaner` file

```shell
$ mkdir spec/support
$ touch spec/support/database_cleaner.rb
```

Step 3 - Add the following code to `database_cleaner.rb`

```ruby
RSpec.configure do |config|
 
  config.before(:suite) do
    DatabaseCleaner.clean_with(:truncation)
  end
 
  config.before(:each) do
    DatabaseCleaner.strategy = :transaction
  end
 
  config.before(:each, :js => true) do
    DatabaseCleaner.strategy = :truncation
  end
 
  config.before(:each) do
    DatabaseCleaner.start
  end
 
  config.after(:each) do
    DatabaseCleaner.clean
  end
 
end
```

Step 4 - Require database cleaner within our `rails_helper`

Navigate to `spec/rails_helper.rb`
add the `require 'support/database_cleaner` below `require 'rspec/rails`

```ruby
require 'spec_helper'
require 'rspec/rails'
require 'support/database_cleaner
```

This will now tear down our database between specs, which will give us a clean slate and cause our specs to run just a little bit faster. 

Otherwise, we would occasionally need to run `$ rake db:reset` to keep our databse lean. 



On to Feature Specs! Capybara's specs live within `spec/features`, you may need to create this directory. 


```shell
$ mkdir spec/features
```

Next, you'll want to create a feature spec

```shell
$ touch spec/features/tasks_spec.rb
```

Now, we can write our first feature spec. A very basic example will be to test our `index` view. 

```ruby
require 'rails_helper'

feature "tasks/index" do
  scenario "renders a list of tasks" do
    create(:homework)
    create(:email)

    visit tasks_path

    expect(page).to have_content('complete homework')
    expect(page).to have_content("reply to Zack's email")
  end
end
```


Here, you'll see two new keywords `feature` and `scenario`. These keywords work exactly like `describe` and `it`

In this test we are going to do several things: 
- Create 2 new tasks
- visit the tasks path (the index view)
- expect the page to show our newly created tasks. 

Running the spec, we should get the following failure

```shell
Failures:

  1) tasks/index renders a list of tasks
     Failure/Error: expect(page).to have_content('complete homework')
       expected to find text "complete homework" in ""
       
```

This is letting us know that our expected text was not found. In order to fix this, navigate to `app/views/tasks/index.html.erb` and add the following code: 

```html
<% @tasks.each do |task| %>
  <p> <%= task.name %> </p>
  <p> <%= task.priority %></p>
<% end %>
```

Here we are perfoming a basic iteration that displays the name and priority of each task. 

Next, we'll create a basic spec that tests adding a new task

```ruby
feature 'New Task' do
  scenario 'user adds a new task' do
    user = create(:user)
    visit tasks_path

    expect{
      click_link 'New Task'
      fill_in 'Name', with: 'Learn Rspec'
      fill_in 'Priority', with: 1
      fill_in 'Due date', with: DateTime.now
      select(user.email, from: 'task_user_id')
      click_button('Create Task')
    }.to change(Task, :count).by(1)

    expect(current_path).to eq(task_path(Task.last.id))
    expect(page).to have_content('Learn Rspec')

  end
end
```

This test is slightly more involved, so let's walk through it by running our specs and writing passing code. 

```shell
Failures:

  1) New Task user adds a new task
     Failure/Error: click_link 'New Task'

     Capybara::ElementNotFound:
       Unable to find link "New Task"
       
```

We are visiting the index view and expecting to click a link with the content of "New Task". 

```html
<%= link_to "New Task", new_task_path %>
```

Running our tests, we get a new failure. Let's rinse and repeat all the way through this spec. 

#### failure

```shell
Failures:

  1) New Task user adds a new task
     Failure/Error: fill_in 'Name', with: 'Learn Rspec'

     Capybara::ElementNotFound:
       Unable to find field "Name"
       
```

#### code

We need to navigate to `app/views/tasks/new.html.erb` and add a Name field (which means we'll need a form)

```html
<%= form_for @task do |f| %>
   <div>
    <%= f.label :name %>
    <%= f.text_field :name %>
  </div>
<% end %>
```

Great! now on to our next part of the spec

#### failure

```shell
Failures:

  1) New Task user adds a new task
     Failure/Error: fill_in 'Priority', with: 1

     Capybara::ElementNotFound:
       Unable to find field "Priority"
       
```

#### code

```html
 <div>
    <%= f.label :priority %>
    <%= f.text_field :priority %>
 </div>
``` 

#### failure

```shell
Failures:

  1) New Task user adds a new task
     Failure/Error: fill_in 'Due date', with: DateTime.now

     Capybara::ElementNotFound:
       Unable to find field "Due date"
 
```

#### code

```html
  <div>
    <%= f.label :due_date %>
    <%= f.text_field :due_date %>
  </div>
```

The next bit of syntax is different. This is really only temporary until we add User authentication. For now, we are going to offer a dropdown with a list of user emails for the user to select from. We will then pass the user ID back to the controller's create action

#### failure

```shell
Failures:

  1) New Task user adds a new task
     Failure/Error: select(user.email, from: 'task_user')

     Capybara::ElementNotFound:
       Unable to find select box "task_user"
       
```

#### code

```html
  <div>
    <%= f.label :user_id, "Select your email address" %>
    <%= f.collection_select :user_id, User.all, :id, :email %>
  </div>
```

Now, we have a new type of failure

#### failure 

```shell
Failures:

  1) New Task user adds a new task
     Failure/Error: click_button('Create Task')

     Capybara::ElementNotFound:
       Unable to find button "Create Task"
```

We simply need to add a submit button (similar to our link from earlier)

```html
 <%= f.submit %>
```

At this point, we'll get another error that might need a bit of explanation. 

```shell
Failures:

  1) New Task user adds a new task
     Failure/Error: expect(page).to have_content('Learn Rspec')
       expected to find text "Learn Rspec" in ""
       
```

This is due to the fact that our Controller's create action is redirecting us to the `show` view. 

To fix this, navigate to `app/views/tasks/show.html.erb` and add the following: 

```html
<p> <%= @task.name %> </p>
<p> <%= @task.priority %> </p>
<p> <%= @task.due_date %> </p>
```

Sweet! all of our tests are passing

Let's add an edit spec as well

```ruby
feature 'Edit Task' do

  let(:task) { create(:homework)}

  scenario 'User edits task' do
    visit task_path(task)
    expect(page).to have_content('complete homework')

    click_link("Edit")

    fill_in 'Name', with: 'Master Rspec'
    fill_in "Priority", with: 1
    fill_in "Due date", with: DateTime.now
    select(task.user.email, from: 'task_user_id')
    
    click_button('Update Task')

    expect(current_path).to eq(task_path(task.id))

    expect(page).to have_content('Master Rspec')
  end
end
```

In this `scenario` we are:

- creating a task
- navigating to that task's show page
- clicking Edit
- updating the task 
- insuring that our task was updated

#### failure

```shell
Failures:

  1) Edit Task User edits task
     Failure/Error: click_link("Edit")

     Capybara::ElementNotFound:
       Unable to find link "Edit"
       
```

We are already on the show page, which expects a link to edit


#### code

```html
<%= link_to "Edit", edit_task_path(@task) %>
```

Our next failure will be: 

```shell
Failures:

  1) Edit Task User edits task
     Failure/Error: fill_in 'Name', with: 'Master Rspec'

     Capybara::ElementNotFound:
       Unable to find field "Name"
       
```

At this point, it might be tempting to go ahead and duplicate our code in the edit view. Let's code smart though and create a form partial and load that partial in our New and Edit views. 

#### code

```shell
$ touch app/views/tasks/_form.html.erb
```

Copy/Paste your code from new into the new form parital

```html

<%= form_for @task do |f| %>
  <div>
    <%= f.label :name %>
    <%= f.text_field :name %>
  </div>

  <div>
    <%= f.label :priority %>
    <%= f.text_field :priority %>
  </div>

  <div>
    <%= f.label :due_date %>
    <%= f.text_field :due_date %>
  </div>

  <div>
    <%= f.label :user_id, "Select your email address" %>
    <%= f.collection_select :user_id, User.all, :id, :email %>
  </div>

  <%= f.submit %>
<% end %>
```

in `edit.html.erb` render your form partial

```html
<%= render 'form' %>
```

Run your specs and they should be passing!

Now remove the form code from `new.html.erb`, add the form partial and run your entire test suite!


Capybara has some other features that are very compelling that I will leave for you to explore at your leisure. 

## Conclusion

Congrats. You've just built an application using Test Driven Development. Yes, there is more to be done, which we will cover in the next class. Just think about how much faster this will increase your speed of development once you have a bit of practice!


## Exercise && Homework

###**Due 11/28/16**

- Complete Steps 3 and 4 on the [todo_with_rspec](https://github.com/NextLevelRails-winter-2016/todo_with_rspec#test-order) app 
- Read [Mastering Ruby Blocks in Less than 5 minutes](http://mixandgo.com/blog/mastering-ruby-blocks-in-less-than-5-minutes)
	- write a sentence or two about your biggest takeaway
	- submit through DM on slack
- continue building blog app you started last week
	- write a full suite of controller specs
	- write a feature specs for the following:
		- Index
		- New Post
		- Edit/Update Post 
- Read Avid Grimm's [explanation of Database Cleaner config](http://www.virtuouscode.com/2012/08/31/configuring-database_cleaner-with-rails-rspec-capybara-and-selenium/)
	- Pick out 1 part of the configuration and describe your understanding of it. 
	- Submit through Slack DM 
