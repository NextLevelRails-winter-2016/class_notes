# Lesson 4 - rspec-mocks, coverage and authentication.md

## Recap & Agenda

Last class we discussed:

- Testing Controllers with Rspec
- Unit vs Integration Tests
- Capybara and Feature Specs
- Database Cleaner

Tonight we are going to dig into:

- Mocks, Stubs and Test Doubles
- Code Coverage
- Testing with Devise



## Mocks, Stubs, and Test Doubles 

We are going to start tonight by introducing mocks and stubs. Which, like many other topics in this class, is deserving of much more time. 

According to the [Rspec Documentation](https://www.relishapp.com/rspec/rspec-mocks/docs)...

> rspec-mocks helps to control the context in a code example by letting you set known return
values, fake implementations of methods, and even set expectations that specific messages
are received by an object.

Meaning that we can use `rspec-mocks` to simulate objects, methods, return values, etc. in our tests 


In order to take advantage of mocks, we need to start with **Test Doubles**. 

The Rspec Documentation states that: 
> A test double is an object that stands in for another object in your system during a code
example

We could also say that `test doubles` are essentially fake representations of real objects... a stunt double, if you will... 

Method Stubs are: 

> an instruction to an object (real or test double) to return a
known value in response to a message:

In other words, `stubs` are fake methods that return a pre-determined value (by us, the developer)

I know this presents several questions, so let me explain the benefits and then we'll dig into some code. 

Utilization of mocks and stubs can result in 2 major benefits: 

- *encapsulation* - tests become independent of other classes/objects by:
	- removing dependencies on other classes and objects 
	- allow focus on implementation of the specific model/class/unit
- *performance* - specs can run faster
	- take less time to setup/teardown
	- do not: 
		- persist to the database
		- require complex associations

While, other benefits could be discussed, the above are what should be most interesting to us. 

Let's look at some code, for this example we are going to start with a basic Ruby application for budgeting/banking. 

Let's go ahead and run through the basic setup: 

```shell
$ mkdir budget_app 
$ cd budget_app
$ rspec --init
$ touch bank_account.rb
$ touch spec/bank_account_spec.rb
```

Now that we have our initial structure, let's go ahead and configure our spec files. 

Navigate to `spec/spec_helper.rb` and add:

```ruby
require_relative '../bank_account'
```

Next, navigate to `spec/bank_account_spec.rb` and add the following: 

```ruby
require 'spec_helper'
```

Alright! Now we can start thinking about implementation. 

For the sake of time, let's assume that we've already decided on instatiation of our BankAccount object. This will be similar to our Record Collection object from class 1. 

BankAccount will contain an array of transactions... let's go ahead and add that to `bank_account.rb`

```ruby
class BankAccount

   def initialize(transactions)
      @transactions = transactions
   end
 
end
```

Let's write out our two specs and then work through the implementation. 

```ruby

require 'spec_helper'
require 'date'

describe BankAccount do

   it "returns an array of only today's transactions" do

   end

   xit "displays the sum for only today's transactions" do

   end
end
```

In the above example there are 3 items of note: 

- `require 'date'` we are bringing Ruby's Date Library into our spec
- return an array of ONLY today's transactions
- show the sum of those transactions

While, theoretically, we could do all of that with one spec, it would be better to test them individually. 

Now we can start using rspec-mocks. We know that will eventually need a Transaction class with it's own attributes, for now though, we can simply create a test double for transaction. 

```ruby
it "returns an array of only today's transactions" do
    
  transaction1 = double('transaction', purchase_date: Date.today) 

end
```

Above we are using `double` and then passing a string as our first argument. Technically, this all we need to create a *test double*. (that will stand in for our Transaction class.)<br> We are, however passing an additional argument of `purchase_date` which `stub`'s out a transaction date.


In summary, we've created a *representation* of our Transaction object that can stand in while we are building out our bank account class. 


We can go ahead and add a few more test doubles. 

```ruby
it "returns an array of only today's transactions" do
  transaction1 = double('transaction', purchase_date: Date.today)
  transaction2 = double('transaction', purchase_date: Date.today)
  transaction3 = double('transaction', purchase_date: Date.today.next_day)
end
```

Next we'll instantiate a BankAccount object 

```ruby
it "returns an array of only today's transactions" do
  # ... test doubles here ...
  
  bank_account = BankAccount.new([transaction1, transaction2, transaction3])
end
```

And finally, we'll add our expectation 

```ruby
it "returns an array of only today's transactions" do
  # ... test doubles here ...
  
  bank_account = BankAccount.new([transaction1, transaction2, transaction3])
  
  expect(bank_account.filter_todays_transactions).to match([transaction1, transaction2])

end 
```

For our expecation, we are saying that our *actual value*, which is a method named **filter_todays_transactions** should match the expected value of an array with our first two transaction test doubles. 


Running our specs we should get a pretty standard error: 

```shell
Failures:

  1) BankAccount returns an array of only today's transactions
     Failure/Error: expect(bank_account.filter_todays_transactions).to match([transaction1, transaction2])

     NoMethodError:
       undefined method `filter_todays_transactions' for #<BankAccount:0x007ffdfa9d72e0>
       
```

Adding our method in the BankAccount class should now return the following error: 

```shell
Failures:

  1) BankAccount returns an array of only today's transactions
     Failure/Error: expect(bank_account.filter_todays_transactions).to match([transaction1, transaction2])
       expected nil to match [#<Double "transaction">, #<Double "transaction">]
       
```

This looks different, but if you take a moment to examine the error it will make sense. 

Our actual value is `nil` and our expected value is an array with 2 of our test doubles. 

The following code should make our spec pass. 

```ruby
def filter_todays_transactions
  @transactions.select{ |t| t.purchase_date == Date.today }
end
```

Here we are simply returning an array with only items that match a purchase_date of Today. yay!

Now we can move on to our second test. however, let's go ahead and place these test doubles in `let` statemenents. 

```ruby

let(:transaction1) { double('transaction', purchase_date: Date.today) }
let(:transaction2) { double('transaction', purchase_date: Date.today) }
let(:transaction3) { double('transaction', purchase_date: Date.today.next_day) }

it "returns an array of only today's transactions" do

 bank_account = BankAccount.new([transaction1, transaction2, transaction3])

 expect(bank_account.filter_todays_transactions).to match([transaction1, transaction2])

end
```

Now, let's write out our second spec

```ruby
it "displays the sum for only today's transactions" do

  allow(transaction1).to receive(:amount).and_return(25.00)
  allow(transaction2).to receive(:amount).and_return(50.00)
  allow(transaction3).to receive(:amount).and_return(100.00)

  bank_account = BankAccount.new([transaction1, transaction2, transaction3])

  expect(bank_account.total_spent_today).to eq(75)
end
```

There is a bit of new syntax here, so let's take a moment to break it down. 

`allow` is the rspec syntax responsible for setting **message expectations** for our objects. So, here we are saying: 

- allow our test double to 
	- `receive` a message named: `amount` 
	- `return` a value of `25`

Essentially, we are simulating the invocation of a method and assigning the expected value. 

In our expectation, we are declaring that our *actual value* should be a method named **total_spent_today** which returns a value that matches the *expected value* of 75 

Assuming we've already stepped through creating a method, we should have the following failure.

 

```shell
Failures:

  1) BankAccount displays the sum for only today's transactions
     Failure/Error: expect(bank_account.total_spent_today).to eq(75)

       expected: 75
            got: nil

       (compared using ==)
              
```

Our passing code will be something like this: 

```ruby
def total_spent_today
  filter_todays_transactions.inject(0){ |sum, t| sum + t.amount }
end
```

Here we are invoking the `filter_todays_transactions` method and getting the sum of the array item's values. 

Now our specs should pass, and we've made use of rspec-mocks to build out the functionality of our BankAccount class. 



#### Factory Girl mocks

Similarly, FactoryGirl has a method by the name of `build_stubbed` which instantiates a fake object, assigns an id, and acts as if the object was persisted to the database. The stubbed/mocked object, however, is not actually persisted. 

One of the main benefits of using build_stubbed is speeding up our specs. 

To see how this works in action we can build a stubbed/mocked FactoryGirl object in our User Model spec by swapping out `let(:user) { build(:user) }` for `let(:user) { build_stubbed(:user)}`

```ruby
RSpec.describe User, type: :model do

  let(:user) { build_stubbed(:user)}
  let(:user_with_tasks) { build(:user_with_tasks)}

   # ... tests ...

end
```

While running our specs you'll notice that all but 1 test still passes. 

```shell
Failures:

  1) User is invalid without unique email
     Failure/Error: user.save

     RuntimeError:
       stubbed models are not allowed to access the database - User#save()

```

The error message is pretty straight forward. We have _mocked_ a persisted object. This test actually depends on an object being previously persisted to the database. Thus, our mocked object won't work for this specific test. <br> The easist way to fix this, is to actually persist an object, by swapping `user.save` for `user = create(:user)`

```ruby
it 'is invalid without unique email' do
  user = create(:user)

  other_user = build(:user, email: user.email)

  expect(other_user.save).to eq(false)
end
```

Now, we are creating 1 real object within our tests and allowing the remainder of our tests to use a mock object. 

We can further increase the speed of our tests by using `build_stubbed` in our `user_with_tasks` factory's `after(:build)` method.

 
```ruby
factory :user_with_tasks do
after(:build) do |user|
  [:email, :homework].each do |task|
    user.tasks << FactoryGirl.build_stubbed(task, user: user)
  end
end
```

doing this, will result in a new, but similar failure. 

```shell
Failures:

  1) User returns tasks due today
     Failure/Error: task.update(due_date: DateTime.now)

     RuntimeError:
       stubbed models are not allowed to access the database - Task#save()
       
```

Let's bring in a `stub` to fix this

##### previous spec
```ruby
 it 'returns tasks due today' do
   task = user_with_tasks.tasks.first
   
   task.update(due_date: DateTime.now)
   
   expect(user_with_tasks.due_today.length).to eq(1)
 end
```

##### new spec

```ruby
it 'returns tasks due today' do
   task = user_with_tasks.tasks.first

   allow(task).to receive(:due_date).and_return(DateTime.tomorrow)
   
   expect(user_with_tasks.due_today.length).to eq(1)
end
```

As you can see, we are simply stubbing our first task object's due_date to have the value of tomorrow (instead of today)

For more on `build_stubbed` check out [Josh Clayton's](https://robots.thoughtbot.com/use-factory-girls-build-stubbed-for-a-faster-test) excellent blog post. 

Now that we've made these changes, let's run our full test suite...<br>
Yay! all of our tests are passing



Mocking/Stubbing is not without it's drawbacks. The main one being that we are assuming/implying that our other object's methods and implementation details work as expected. As you journey down the road of using this technique, make sure that you are taking time along the way to run your full test suite and refactor as you go. 


My final thought is that Mocking/Stubbing is not a necessity for testing. In fact, you can go extremely far with what you've alreadly learned (prior to tonight's material). 

## Code Coverage

Given that we are now covering our application with tests, it would be helpful to see areas that might be missing test coverage. 

Thankfully, there is an excellent gem we can use named [SimpleCov](https://github.com/colszowka/simplecov)

While SimpleCov offers many configuration options, initial setup/config is pretty straight-forward

Start by adding SimpleCov to your Gemfile and `$ bundle install`

```ruby
gem 'simplecov', require: false, group: :test
```

Next, navigate to `spec/spec_helper.rb` and require/start SimpleCov at the top of the file

```ruby
require 'simplecov'
SimpleCov.start 'rails'
```

*note: it's also a good idea to add `coverage` to our `.gitignore` file.*

Now, when we run our specs we will get a coverage report that looks similar to:

```shell
Coverage report generated for RSpec to /PATH_TO_APP/APP_NAME/coverage. 47 / 61 LOC (77.05%) covered.
```

We can also launch a web-view with detailed reports by running: 

```shell
$ open coverage/index.html
```

Within this view, we can see additional details about our code coverage. 


## Adding Devise

In our previous rails examples, we did not add any kind of user authentication. Let's go ahead and destroy our User Model and then re-build using Devise.

First, we'll need to add devise to our gemfile

```ruby
gem 'devise'
```

Next, we can run through the appropriate setup/config steps

```shell
$ bundle install
$ rails g devise:install
$ rails d model User
$ rails g devise User
$ rake db:drop
$ rake db:migrate
```

Here, we

- installed devise
- destroyed our previous User model
- created our User model through devise
- Dropped the databse (instead of rolling back)
- Migrated the database

At this point, we should probably go ahead and run our full test suite... Go ahead, I'll wait. 

ug... that's a lot of failing tests. If you take a moment to read the errors though, a solution should quickly come to your mind. 

The major error we are hitting, is that our User object is failing to be created, leading to a failed association. 

The easy fix? Let's go ahead and create a User factory that addresses the built-in Devise validations. 

##### `spec/factories/users.rb

```ruby
FactoryGirl.define do
  factory :user do
    email {FFaker::Internet.email}
    password 'Password1'
    encrypted_password 'Password1'
  end
```

This is the minimum amount of code needed to instantiate an associated object. Running our suite again should lead to passing tests! 

Now, we can focus on testing our newly created User Model. 

The easiest thing to do, would be to grab our existing code from a previous commit and add it back in. _yay git!_

##### `spec/models/user_spec.rb`

```ruby
require 'rails_helper'

RSpec.describe User, type: :model do
  let(:user) { build_stubbed(:user)}
  let(:user_with_tasks) { build(:user_with_tasks)}

  it 'has a valid factory' do
    expect(user).to be_valid
  end

  it 'is invalid without firstname' do
   user.firstname = nil
   expect(user).not_to be_valid
  end

  it 'is invalid without lastname' do
   user.lastname = nil
   expect(user).not_to be_valid
  end

  it 'is invalid without email' do
   user.email = nil
   expect(user).not_to be_valid
  end

  it 'is invalid without unique email' do
    user = create(:user)

    other_user = build(:user, email: user.email)

    expect(other_user.save).to eq(false)
  end

  it 'has many tasks' do
   expect(user_with_tasks.tasks.length).to eq(2)
  end

  it 'returns tasks due today' do
    task = user_with_tasks.tasks.first

    allow(task).to receive(:due_date).and_return(DateTime.tomorrow)
   
    expect(user_with_tasks.due_today.length).to eq(1)
  end
  
end
```

Now, we can run our tests and start building out our User model

We'll notice that our first 2 failures are related to the first and last name attributes. You may recall that devise does not give us those fields 'out of the box', so we will need to create a migration and add them.

```shell
$ rails g migration AddFirstnameAndLastnameToUsers firstname lastname && rake db:migrate
```

This alone, won't cause our tests to pass, but it will help us to avoid some unnecessary errors. 

Next, we will want to update our Factory

```ruby
FactoryGirl.define do
  factory :user do
    firstname {FFaker::Name.first_name}
    lastname {FFaker::Name.last_name}
    email {FFaker::Internet.email}
    password 'Password1'
    encrypted_password 'Password1'
  end
end
```

Now, we can add our validations

##### `app/models/user.rb`

```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  validates :firstname, presence: true
  validates :lastname, presence: true
end
```

This should cause our validation specs to pass. While we're at it, we can also remove the email validations from our tests, because Devise tests email/password validations internally. 

Alright! Our last set of failing specs is up! We can start by adding a `:user_with_tasks` factory

```ruby
FactoryGirl.define do
  factory :user do
    firstname {FFaker::Name.first_name}
    lastname {FFaker::Name.last_name}
    email {FFaker::Internet.email}
    password 'Password1'
    encrypted_password 'Password1'
    
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

Running this test will present a new failure, which is a pretty easy solve. We need to set up our assocation to tasks in the User model 

```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable

  validates :firstname, presence: true
  validates :lastname, presence: true

  has_many :tasks
end
```

Unfortunately, our spec still fails. This, however, is also a pretty simple fix

```ruby
def due_today
  self.tasks.select{ |t| t.due_date.to_date == DateTime.now.to_date }
end
```

Woohoo! now we're back to green! 

Now, we can start setting up for User authentication 

Navigate to your tasks controller and add: `before_action :authenticate_user!`

```ruby
class TasksController < ApplicationController

  before_action :set_task, only: [:show, :edit, :update, :destroy]
  before_action :authenticate_user!

  # ... controller actions ...
end
```

This will result in quite a few failures, due to the fact that now our Tasks controller is expecting a User to be logged in. 

There are a number of ways you could go about solving this particular issue. However, I believe the easiest is to review the Devise documentation and use some very small modifications. 


### Devise helper method config
Devise, gives us several helper methods for our tests, (including `sign_in(user)` we just need to include these helpers in our tests. 

We'll begin by navigating to `spec/rails_helper` and requiring devise (towards the top of the file)

```ruby
require 'spec_helper'
require 'rspec/rails'
require 'support/database_cleaner'
require 'devise'
```

Next, scroll down to somewhere inside of the `config` block and include the Devise Test Controller Helpers 


```ruby
RSpec.configure do |config|

  config.include FactoryGirl::Syntax::Methods
  config.include Devise::Test::ControllerHelpers, :type => :controller
  
  # ... other config options ...
  
end
```

also, while we're in here go ahead and add your Integration helper, so that Devise can be used inside of your Capybara specs

```ruby
RSpec.configure do |config|
	
  # ... other code ... 
  
  config.include Devise::Test::IntegrationHelpers, type: :feature
  
end
```

Okay, to keep things super simple, let's just add a basic `before` block

```ruby
RSpec.describe TasksController, type: :controller do
  
  before { sign_in(create(:user)) }
  
  # ... tests ... 
  
end
```

Now, our controller specs should be green! For good measure, let's also test that our unathenticated redirect works as well. 

```ruby
  describe 'unathenticated' do
    it 'redirects user to login page when not signed in' do

      sign_out(:user)
      get :index
      expect(response).to redirect_to(new_user_session_path)
    end
  end
```

This could obviously be tested more rigerously, but let's move on for now. 

### Integration Specs with Authentication

For our feature specs to pass, we will need Sign the user in before clicking around the site.

This can be done by updating our specs to the following: 

```ruby
feature "tasks/index" do
  scenario "renders a list of tasks" do
    user = create(:user)
    sign_in(user)

    create(:homework, user: user)
    create(:email, user: user)

    visit tasks_path

    expect(page).to have_content('complete homework')
    expect(page).to have_content("reply to Zack's email")
  end
end
```

The real key here, is that we create a user, and then sign them in. 

Our second feature spec can be updated using the same logic. 

```ruby
feature 'New Task' do
  scenario 'user adds a new task' do
    user = create(:user)
    sign_in(user)
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

In our last spec, let's see what it looks like to use `let` to instantiate our user object. *hint: it works exactly the same*

```ruby
feature 'Edit Task' do
  let(:user) { create(:user) }
  let(:task) { create(:homework) }

  scenario 'User edits task' do
    sign_in(user)
    visit task_path(task)
    expect(page).to have_content('complete homework')

    click_link("Edit")

    fill_in 'Name', with: 'Master Rspec!!!!!'
    fill_in "Priority", with: 1
    fill_in "Due date", with: DateTime.now
    select(task.user.email, from: 'task_user_id')

    click_button('Update Task')

    expect(current_path).to eq(task_path(task.id))

    expect(page).to have_content('Master Rspec!!!!!')
  end
end
```

Yes! Now we have authentication working for all of our tasks. Next thing we can do is scope our tasks to only display the users specific tasks on the tasks index page. 



## Homework && Challenges

###**Due 12/5/16**

- clone down the TodoApp repo
- checkout a new branch
	- use the following naming convention: `user_tasks_YOUR_INTIALS_HERE` 	
- update your `index` spec to expect user's will only see their 
- write the passing code
	- push your new branch to GitHub

- Read [Stubs, Mocks, and Spies](https://about.futurelearn.com/blog/stubs-mocks-spies-rspec/)
	- follow along with the code examples (optional)
	- write your own explanation of mocks and stubs
	- turn in on slack

- Within your blog application
	- setup code coverage 
	- destroy your user model
	- Add a user model with devise
		- Add back in your User specs  
		- remember to 
			- install devise first
			- drop your database
			- recreate it	
	- swap in test doubles where applicable (using Factory Girl)
	- Add authentication tests

	
<!--## Challenge 1 Answer

##### `spec/controllers/tasks_controller_spec.rb`

```ruby
it 'returns all tasks for specific user' do
  sign_out(:user)

  user = create(:user_with_tasks)
  sign_in(user)

  task = create(:homework)

  get :index

  expect(Task.all.length).to eq(3)
  expect(assigns(:tasks).length).to eq(2)
end
```

##### `app/controllers/tasks_controller.rb`

```ruby
def index
  @tasks = Task.where(user: current_user)
end
```-->
