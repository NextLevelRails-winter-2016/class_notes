# Lesson 1 - Intro to Next Level Rails


## Welcome & Class Overview
* Intro / Ice Breaker
	1. What’s your name?
	2. Where are you from?
	3. What background in programming do you have, if any? (HTML/CSS?, JS?, Ruby?, Python? etc)
	4. What do you want to do after you complete the class?
	5. Tell us about a time you met a famous person. If you didn’t, who would you like to meet?

* This is an ACCELERATED course

## Administrative

* **OS** - macOS or Linux is preferred
* **Text Editor** - Use [Atom](https://atom.io/) or Vim
* [**Rspec**](http://rspec.info/documentation/3.5/rspec-rails/)
* **Google Chrome**
* [**Postman**](https://www.getpostman.com/)
* [**Github Account**](https://github.com/NextLevelRails-winter-2016)
* **Slack**
* **Homework** - Do it... or else... 
* **Lectures** - Computers down for lecture. 
* **Pairing** - All classroom assignments will be completed with a partner. 


## Syllabus
1. Why we test & Rspec basics
2. Application setup, unit tests, controller specs, and factories
3. Acceptance/Integration Tests
4. Authentication, advanced testing, and other tools
5. HTTP, API Basics and Postman
6. Rails API pt 1
7. Rails API pt 2
8. Testing, CI and deployment 


## Why Testing? 

Test Driven Development, also known as Behavior Driven Development is the process of writing automated tests before you write application code. 

While this can sound counter-intuitive, the process of writing tests before your application code can offer several benefits including: 

- Increased Code Quality 
- Faster Development Cycles
- Fewer regression bugs 

Let's look at the above points 

##### Increased Code Quality

The process of evaluating the problem and talking through a solution is of great value. You can think of tests as a virtual todo list for your application. 

Writing tests allows you to think of your problem domain and solve accordingly. 

Furthermore, your code _serves as documentation_ for other developers. Because of the conversational style of Rspec, fellow developers can easily understand the problem and begin working quickly to provide a solution. 

##### Faster Development Cycles

Tests allow you to quickly iterate on a particular problem, solve it, and move on. By thinking through a solution, you spend much less time guessing and writing code that simply does not work.  

Personally, I've found that I write applications about 30% faster when following TDD best-practice. 

##### Fewer Regression Bugs

We've all been there. You make an update to a feature in your application, only to find out hours (or days) later, that another seemingly unrelated feature is now broken. You quickly check the application logs (because this usually happens in production) only to be completely lost as to where the problem is coming from. After several minutes of panic, you find the error and make an adjustment. 

Often, this is due to one something minor in your new code that unexpectedly interferes with exisiting code. This is known as a regression bug. Having adequate code coverage often solves this problem almost entirely. 

Before you ever make a major commit or push to production, you can simply run your test suite to ensure that all of your code is working as expected. 

#### Not all agree

There are many philosophies on testing. Debates on whether or not you should always test first, and how much test coverage you should have. 

However, most developers agree that I well constructed test-suite is a must. 

We will explore the test-first methodology, as well as writing some tests after implementation of code. So that you can make an educated decision about how to test your applications.  

### Red Green Refactor

Test Driven methodology follows a pattern known as Red Green Refactor. Meaning:

- Write a test and watch it fail
- Write code to pass the test
- Refactor small bits of code as you go. 

This is a very helpful process as it allows you to move quickly through your code, while keeping it nice and clean. 

The main takaway is: a test should always fail before passing. 


## Why Rspec?

The Ruby community has embraced and adopted testing more than almost any other group. This means there are quite a few testing frameworks that you can utilize. 

The two biggest players are 
- Rspec 
- MiniTest

In this class, we will be using Rspec for several specific reasons. 

- Rspec is written in a converstaional manner
	- allowing for developers and non-technical users to quickly understand the application. 
- Rspec is widely used
	- Lots of resources are available 
	- Many existing projects already use Rspec 
- Rspec is (in my opinion) the ideal testing framework developers that are new to testing 
	- simple, clean, approachable DSL 
	- principles can be applied to almost any other testing framework  


## Basic Ruby App

### Setup

Let's start with a basic example in Ruby... *because Ruby is awesome!*

We'll start by creating a new directory and creating the file structure

```shell
$ mkdir record_store
$ cd record_store
$ touch record.rb
```

Next we're going to get setup for rspec and create our specs 

```shell
$ gem install rspec
```

this is a one time thing, we're installing rspec globally 

Now, let's create our spec directory and add our spec file 

```shell
$ mkdir spec
$ touch spec/record_spec.rb
```

*note: rspec convention tells us to name our spec file after the ruby file we are testing appending `_spec`*


### Test First

It's time to start writing our specs. We'll begin with requiring the `record` file 

Add the following to `spec/record_spec.rb`

```ruby
require_relative '../record'
```

You are probably familiar with the `require` statement in Ruby. Consider `require_relative` a cousin to `require`. The major difference is that `require_relative` locates a file *relative* to our current file. For more on this, you can read [Including Other Files in Ruby](http://rubylearning.com/satishtalim/including_other_files_in_ruby.html)

Okay! Let's write our first test!

```ruby 
require_relative '../record'

describe "Record" do
  it 'instantiates an object with name and artist' do
    record = Record.new('Seven Swans', 'Sufjan Stevens')

    expect(record).to be_an_instance_of(Record)
  end
end
```

Let's break this down line-by-line 


#### `describe` and `it`

According to the [Rspec documentation](http://rspec.info/documentation/3.5/rspec-core/#Basic_Structure)
> RSpec uses the words "describe" and "it" so we can express concepts like a conversation:

This is nice, because it truly does allow us to read our test like: 

"Record instantiates an object with name and artist"

This makes enough sense, but lets break it down a bit more. 

- Within our Record class
- Instantiate a new record object
- **expect** the newly instantiated object to be an _instance_ of the Record class

#### `expect` and matchers

The other new piece of syntax that you're seeing is `expect`.<br>
According to the [Rspec::Expectations Documentation](http://www.rubydoc.info/github/rspec/rspec-expectations/RSpec/Expectations) 
> RSpec::Expectations provides a simple, readable API to express the expected outcomes in a code example. 

> To express an expected outcome, wrap an object or block in `expect`, call `to` or `to_not` (aliased as `not_to`) and pass it a matcher object:


[Rspec's matchers](http://www.rubydoc.info/github/rspec/rspec-expectations/RSpec/Matchers) are sets of expressions used to compare actual vs. expected values.

```ruby
foo = []
expect(foo).to be_empty
```

Another way of looking at it: 

`expect(ACTUAL_VALUE).to eq(EXPECTED_VALUE)`

So, in our example code, we are **expecting** record **to** _be an instance of_ the Record class

*note: Rspec has a good number of built in matchers and we'll cover many of them. As you may expect, you can also create your own custom matchers (which we'll cover later). To learn more about Rspec's built in matchers check out [this handy list](http://cheatrags.com/rspec-matchers)*

#### Running our tests

Okay, now that we have a foundation for the Rspec syntax, let's run the test by calling:

```shell
$ rspec spec/record_spec.rb
```


Sweet! Our first failure!


```shell
F

Failures:

  1) Record instantiates an object with name and artist
     Failure/Error: record = Record.new('Seven Swans', 'Sufjan Stevens')

     NameError:
       uninitialized constant Record
     # ./spec/record_spec.rb:5:in `block (2 levels) in <top (required)>'

Finished in 0.00059 seconds (files took 0.17468 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/record_spec.rb:4 # Record instantiates an object with name and artist
```

Looking at the test, this should make sense, but let's talk about it. 

We have an unitialized constant Record... why? 

The Record class has not been created... Now, the test is *driving our development* and telling us that we need to create a Record class. 

Navigate over to `record.rb` and add your Record class

```ruby
class Record
end
```

*note: you may be tempted to do more, but don't. let the test tell you what's next!*

**Let's run our test again**, this time passing the `--color` flag to our rspec command

```shell
$ rspec spec/record_spec.rb --color
```

This will add some color to our tests and make them more readable. 

Alright! Another failure! We're moving in the right direction. 

```shell
F

Failures:

  1) Record instantiates an object with name and artist
     Failure/Error: record = Record.new('Seven Swans', 'Sufjan Stevens')

     ArgumentError:
       wrong number of arguments (given 2, expected 0)
     # ./spec/record_spec.rb:5:in `initialize'
     # ./spec/record_spec.rb:5:in `new'
     # ./spec/record_spec.rb:5:in `block (2 levels) in <top (required)>'

Finished in 0.00061 seconds (files took 0.16946 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/record_spec.rb:4 # Record instantiates an object with name and artist
```

We are getting an `ArgumentError`. Obviously, our next move will be to create an `initialize` method with 2 arguments. 

Let's do that now. 

```ruby
class Record
	def initialize(name, artist)
    	@name = name
    	@artist = artist
 	end	
end
```

Let's run our test again

```shell
$ rspec spec/record_spec.rb --color
```

![](./images/ch_success.gif)

That's right! Our first passing spec!

```shell
.

Finished in 0.00079 seconds (files took 0.10415 seconds to load)
1 example, 0 failures
```

#### More specs

This is great, now let's write a few more specs 

In `record_spec.rb` add: 

```ruby
  it 'returns a string with the record name' do
    record = Record.new('Helplessness Blues', 'Fleet Foxes')

    expect(record.name).to eq('Helplessness Blues')
  end

  xit 'returns a string with artist ' do
    record = Record.new('Kind of Blue', 'Miles Davis')

    expect(record.artist).to eq('Miles Davis')
  end
```

Here we are adding two new specs, 

- ensure that the Record's name is accessible
- ensure that the Record's artist is accessible 

you'll notice that the second spec has `xit`, this syntax marks a test as pending. Allowing us to focus on our current feature implementation. As soon as we write code to pass this spec, we can covert `xit` to `it` and start working on the next feature implementation. 

running our spec command, we get the following output: 

```shell
.F*

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) Record returns a string with artist
     # Temporarily skipped with xit
     # ./spec/record_spec.rb:16


Failures:

  1) Record returns a string with the record name
     Failure/Error: expect(record.name).to eq('Helplessness Blues')

     NoMethodError:
       undefined method `name' for #<Record:0x007fc53d0ff8e8>
     # ./spec/record_spec.rb:13:in `block (2 levels) in <top (required)>'

Finished in 0.0018 seconds (files took 0.16693 seconds to load)
3 examples, 1 failure, 1 pending

Failed examples:

rspec ./spec/record_spec.rb:10 # Record returns a string with the record name
```

Once you get into writing tests first, you can essentially create a todo list for yourself. Making your development workflow much easier. 

Now, focusing on our failure, let's see what the error is telling us. 

We need to add a _getter_ method to our Record class. 

in `record.rb` add: 

```ruby 
class Record
	# ... def initialize...
	
	def name
	end
end
```

**Run your tests again** 

the output should look something like this: 

```shell
Failures:

  1) Record returns a string with the record name
     Failure/Error: expect(record.name).to eq('Helplessness Blues')

       expected: "Helplessness Blues"
            got: nil

       (compared using ==)
     # ./spec/record_spec.rb:13:in `block (2 levels) in <top (required)>'

Finished in 0.07921 seconds (files took 0.41469 seconds to load)
3 examples, 1 failure, 1 pending

Failed examples:

rspec ./spec/record_spec.rb:10 # Record returns a string with the record name
```

Alright! this is good, we're seeing that our *actual value* is `nil` while our *expected value* is "Helplessness Blues" (which is a wonderful album btw). 

To fix this, we'll `return` the album name

```ruby
class Record
	# ... def initialize...
	
	def name
		return "#{@name}"
	end
end
```

Run your test again and you'll get 

```shell
..*

Pending: (Failures listed here are expected and do not affect your suite's status)

  1) Record returns a string with artist
     # Temporarily skipped with xit
     # ./spec/record_spec.rb:16


Finished in 0.00191 seconds (files took 0.15521 seconds to load)
3 examples, 0 failures, 1 pending
```

Alright! change `xit` to `it` on your next spec, run your test, then write the code to make it pass.

```ruby
 class Record
	# ... def initialize...
	
	def name
		return "#{@name}"
	end
	
	def artist
		return "#{@artist}"
	end
end
```

Okay, at this point, we can do a small bit of refactoring. Instead of using *getter* methods, let's implement Ruby's `attr_reader` method. 

```ruby
class Record

  attr_reader :name, :artist
  def initialize(name, artist)
    @name = name
    @artist = artist
  end
end
```

Run your test, and you'll notice the specs still pass. Now we have cleaner code that still works. If for some reason, our change failed to accomplish the desired results, our test would fail and notify us exactly where to look. 



### Your Turn

Now that you've got a basic idea, it's time for a challenge. 

#### Challenge 1

- Clone [the Record Collection Repo](https://github.com/NextLevelRails-winter-2016/record_collection) 
- checkout the branch `01_purchase`
- run the specs
- write the appropriate code to make your test pass


#### Challenge 1 Answer

The implementation of this method could look something like this: 

```ruby
  def purchase
    collection = []
    collection.push(self)
  end
```


Now, while this works, it's not going to work long term. Let's think about how we could make this better. 

We can create a RecordCollection class to store our collection. 

```shell
$ touch record_collection.rb
$ touch spec/record_collection_spec.rb
```

This is also a great opportunity to introduce a `spec_helper`

```shell
$ touch spec/spec_helper.rb
```

A spec helper will allow us to contain all of our spec dependencies in one place. 

in `spec/spec_helper.rb` add:

```ruby
require_relative '../record'
require_relative '../record_collection'
```

In `spec/record_spec.rb` and `spec/record_collection_spec.rb` add: 

```ruby
require 'spec_helper
```

Now run your specs by typing:

```shell
$ rspec --color
```

All of your tests should still pass. 


### Record Collection 

We can start by writing our first test for Record Collection 

```ruby
describe "Record collection" do
  it 'creates an array when instantiated' do
    rc = RecordCollection.new
    expect(rc.collection).to eq([])
  end
end
```

Here we are expecting Record Collection to contain an empty array 

Our passing code will be: 

```ruby
class RecordCollection

  attr_accessor :collection
  def initialize
    @collection = []
  end
end
```

Next, we'll write a couple of specs

```ruby
  it 'adds record object to collection' do

    record = Record.new("Heartbreaker", "Ryan Adams")
    other_record = Record.new("Lonesome Dreams", "Lord Huron")

    rc = RecordCollection.new
    rc.add_to_collection(record)

    expect(rc.collection).to include(record)
  end

  it 'adds multiple records' do
    record = Record.new("Heartbreaker", "Ryan Adams")
    other_record = Record.new("Lonesome Dreams", "Lord Huron")
    rc = RecordCollection.new
    rc.add_to_collection(record)
    rc.add_to_collection(other_record)

    expect(rc.collection.length).to eq(2)
  end
``` 

These two specs are ensuring that a single record, and multiple records can be added to the collection 

#### before

There is some code duplication (which is not necessarily a bad thing in specs), however, we can implement a new bit of syntax to clean up our code. 

Refactor your spec to this: 

```ruby
describe "Record collection" do

  before :each do
    @record = Record.new("Heartbreaker", "Ryan Adams")
    @other_record = Record.new("Lonesome Dreams", "Lord Huron")
    @rc = RecordCollection.new
  end

  it 'creates an array when instantiated' do
    expect(@rc.collection).to eq([])
  end

  it 'adds record object to collection' do
    @rc.add_to_collection(@record)

    expect(@rc.collection).to include(@record)
  end

  it 'adds multiple records' do
    @rc.add_to_collection(@record)
    @rc.add_to_collection(@other_record)

    expect(@rc.collection.length).to eq(2)
  end
end
```

The major change here is the `before :each do` block. Which, as you can guess, performs the actions before each spec is run. 

This makes our test a bit cleaner and enhances readability.<br>*note: only DRY out your tests when it enhances readibility, if shortening your tests makes it harder to follow, opt to keep the longer syntax*

Let's run the tests and implement the passing code

```ruby
def add_to_collection(record)
    @collection.push(record)
end
```

### Challenge 2

- checkout the branch `02_collection`
- write working code for `browse_collection` spec


### Challenge 2 Answer

```ruby
  def browse_collection
    @collection.each do |r|
      puts "#{r.artist}: #{r.name}"
    end
  end
```


Now that we've implemented working code for both classes, you remove the purchase method and purchase spec for our Record class


This wraps up our intro to testing, but don't worry! We'll revisit the Record App in a later class! 
 
## Homework

- Join our slack channel
- Send Shane a DM with your GitHub username

- Follow the tutorial found at the [Rspec website](http://rspec.info/)
	- watch each video and follow along in your text editor
	- Push the finished code to GitHub using the naming convention: `rspec_intro_YOUR_INITIALS_HERE` 	
- Build a basic calculator with (testing first)
	- Your calculator app should include the following methods (taking 2 arguments and returning the appropriate value): 
		- Add
		- Subtract
		- Multiply
		- Divide 
	- Once complete, push the working code to our GitHub org with the naming convention: `calculator_app_YOUR_INITIALS_HERE`
