## Lesson 1 - Intro to Rspec

###**Due 11/16/16**

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

- - - -

## Lesson 2 - Intro to Testing in Rails

###**Due 11/21/16**

- Read and Follow along with [How to Test Rails Models](https://semaphoreci.com/community/tutorials/how-to-test-rails-models-with-rspec)
	- Once finished push your copy of code with the following naming convention: `Auction_YOUR_INITIALS_HERE`
	- Make a comment in the read me giving credit to the author or website
- Read [Factory Girl 101](https://code.tutsplus.com/articles/factory-girl-101--cms-25087) 
	- Write a sentence about your 3 biggest takeaways from the article 
	- Turn them in through DM on slack
- Write the User and Post models *and specs* for a Blog application
	- Configure your application to use Rspec  
	-Your user model will have the following 2 attributes: 
		- username
		- email
	- The Post model will have the following attributes 
		- title
		- article 
	- Write the following tests: 
		- Validation of all fields 
		- Assocations tests
	- Use the Todo App for inspiration
	- Write your tests first!

- - - - 

## Lesson 3 - Testing Rails Controllers and Features

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
	
- - - - 

## Lesson 4 - rspec-mocks, coverage, & Devise


###**Due 12/5/16**

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

- - - - 

## Lesson 5 - Moving Towards API's - REST, HTTP, cURL, Postman

###**Due 12/7/16**

- Use [JSONPlaceholder](https://jsonplaceholder.typicode.com/) to create a Postman collection that: 
	- Returns all posts
	- Returns all posts from a specific user
	- Returns all todos
	- Returns a specific todo
	- Creates a new User 
	- Creates a new Todo
	- Submit collection through Slack

###**Due 12/12/16**

- Read [The Joy of cURL](http://www.computerworld.com/article/2992017/operating-systems/the-joy-of-curl.html)
	- write about 2 takeaways 
	- turn in on slack 
- Read [What's CSRF and What Exactly Are We Are Protecting From Forgery?](http://annaershova.github.io/blog/2015/10/25/whats-csrf-and-what-exactly-are-we-are-protecting-from-forgery-in-controllers/)
	- write about 1 takeaway
	- turn in on slack 
