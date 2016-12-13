# Lesson 7 - Building API's in Rails Part 2

## Recap and Agenda

Last class we discussed the basics of building an API in rails. This included: 

- Discussing what an API is
- Rails 5 `--api` flag
- Serialization with Active Model Serializers
- Associations
 
Tonight we'll be moving on to: 

- Enabling our API's for Cross Origin Requests
- Authentication
- JSON Web Tokens
 
- - - -  
 
 
## rack-cors

Due to the fact that our API is expected to be consumed by clients on different domains, we will need to provide support for [CORS](http://enable-cors.org/) (Cross Origin Resource Sharing)

So what is CORS? 

By default browsers implement a security protocol named Same Origin Policy. Which denies requests/responses if the request domain does not match the API/server domain. While this policy is good for security, it can be a headache for developers. 

CORS looks to solve this problem by adding specific Headers that instruct the browser on how to handle Cross Origin requests/responses. 

CORS configuration within Rails is pretty simple with the `rack-cors` gem. Let's take a look at basic setup. 

Since the Gem is already in your gemfile, all you need to do is *uncomment* and `bundle install`. Afterwards, navigate to `config/initializers/cors.rb` and *uncomment* the following code: 

```ruby
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'

    resource '*',
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

The only modification, is that we changed `origins 'example.com'` to `origins '*'`. This will allow requests from all domains. Though, I'd only recommend this if you intend for your API to be public. Otherwise, You can add the specific domains you're willing to take requests from. 

Postman has a built in mechanism to deal with CORS, so let's send a cURL request to check our headers using the following request: 

```shell
curl -X GET -I http://localhost:3000/v1/tasks -H 'Origin: *'
```

*note: for ease, temporarily comment out the `before_filter` checking for authentication in your Tasks controller.*

Before CORS: 

```shell
HTTP/1.1 200 OK
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: application/json; charset=utf-8
ETag: W/"4f53cda18c2baa0c0354bb5f9a3ecbe5"
Cache-Control: max-age=0, private, must-revalidate
X-Request-Id: adf559b8-6f46-4dda-809a-37b6e05a5891
X-Runtime: 0.175815
Transfer-Encoding: chunked
```

After CORS: 

```shell
HTTP/1.1 200 OK
X-Frame-Options: SAMEORIGIN
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
Content-Type: application/json; charset=utf-8
ETag: W/"4f53cda18c2baa0c0354bb5f9a3ecbe5"
Cache-Control: max-age=0, private, must-revalidate
X-Request-Id: a2ac18d5-7788-4697-a01a-dc590da73035
X-Runtime: 0.227190
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD
Access-Control-Max-Age: 1728000
Access-Control-Allow-Credentials: true
Vary: Origin
Transfer-Encoding: chunked
```

CORS is yet another topic that warrants a bit of independent research. Thankfully, like many things, Rails takes care of the initial configuration, so that we can focus on building. That being said, please take some time and read up on CORS 


### Challenge 1 

- Set your application to use CORS
	- enable the rack-cors gem
	- enable the appropriate code
- Make curl requests with and without the CORS enabled to verify the headers

  
 
## Authentication
 
Before we dig into the how-to of authentication, we need to discuss sessions. 

### Sessions

The Rails Guide's [Session documentation](http://guides.rubyonrails.org/security.html#what-are-sessions-questionmark) states that:

> Most applications need to keep track of certain state of a particular user. This could be the contents of a shopping basket or the user id of the currently logged in user. **Without the idea of sessions, the user would have to identify, and probably authenticate, on every request**. Rails will create a new session automatically if a new user accesses the application. It will load an existing session if the user has already used the application.

Meaning that, when building a standard MVC Rails app with erb or haml views, you have session data that get's passed along with your requests. Normally, (in rails) session information is sent through a *secure session cookie*. This helps your rails back-end understand who is making the request and allows your browser to create a session. 

The documentation goes on the say: 

> A session usually consists of a hash of values and a session id... Every cookie sent to the client's browser includes the session id. And the other way round: the browser will send it to the server on every request from the client.

Rails takes care of this for you, unless need to do something a little more complex. In that case, you'll probably want to create a session controller for handling your sessions.<br>*note: see [how rails sessions work](http://www.justinweiss.com/articles/how-rails-sessions-work/) for more info*

So, what does this look like? 

Let's look at logging in to an application. 

Generally, the flow of logging in, is that a client makes a request with an email and password. The response will include the user resource, along with a session cookie. The browser will store that session cookie and send it along with each request. 

This is great for our standard rails applications. However, when building an API, a browser may not be involved at all. 

Furthermore, **API's are stateless** - meaning that your API takes a request and gives a response... nothing more. No knowledge of previous or future state should be required for the API to work. 

### Tokens

One of the more popular ways of handling Authentication in API's is to use Token based authentication. 

<!--Authorization tokens are similar to key-cards. Let's say you move into a new apartment with key-card entrance to the parking garage. Each time you use the key-card, it is checked for validity, and you are granted access. Access will be granted to any person using the key-card.--> 

Token based authentication is stateless. 

The process is actually pretty straight-forward

- User submits login credentials through a client
- The API verifies the credentials
- The API then responds with a *signed* authentication token
- The client stores that token 
- The client will issue the token with each subsequent request (in the header)
- The API will verify the authenticity of the token
- If the token is valid, the API will respond with the appropriate data

The major difference here is that the API remains stateless and does not persist (keep track of) the session information. 

This puts the work of managing authentication state on the client, which is more REST oriented. 

So this presents a question (at least to me), if the server is still authenticating and checking signatures, is it really stateless? 

The answer is *yes*

The server has a unique signature (that only the server knows), on each request the server compares the signature (included in the request) to it's own signature, and only approves the request if the signature's are identical. 

This token authentication scheme also allows us to easily integrate our API with other 3rd party applications, such as Twitter, Facebook, Github, etc. 

### JWT

The most popular form of token based authentication is [JSON Web Tokens](https://jwt.io/) (aka: JWT). 

JWT adheres to the [RFC 7519](https://tools.ietf.org/html/rfc7519) standard, and is easy to implement in any application. This is because it's simply JSON! 

In fact, JWT works similar to session cookies... just, stateless. 
<!--
JWT is great for a number of reasons, 

- Scalable and lightweight
- Can be used with multiple programming languages (because it's just JSON)-->

The [JWT introduction](https://jwt.io/introduction/) has all you need to know to get started with JSON Web Tokens. Though you should read it (and you will for homework). Here are the high-notes:

Two of the major benefits of JSON Web Tokens are: 

> - Compact: Because of its smaller size, JWTs can be sent through an URL, POST parameter, or inside an HTTP header. Additionally, the smaller size means transmission is fast.

> - Self-contained: The payload contains all the required information about the user, avoiding the need to query the database more than once.

JSON Web Tokens consist of 3 parts: 

- header
- payload
- signature

The end result looks like something like this: 

![](./images/jwt_example.png)


In this example, the token is: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ`

Each part is seperated by a `.`

#### Header

In the above case, the **header** is: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`. According to JWT: 

> The header typically consists of two parts: the type of the token, which is JWT, and the hashing algorithm being used...

#### Payload 

The **payload** would then be: `eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9`. JWT states that: 

> ...the payload... contains the claims. Claims are statements about an entity (typically, the user) and additional metadata. There are three types of claims: reserved, public, and private claims.

Let's look at those claims in more detail... 

**Reserved Claims** are:

> a set of predefined claims which are not mandatory but recommended, to provide a set of useful, interoperable claims. Some of them are: **iss** (issuer), **exp** (expiration time), **sub** (subject), **aud** (audience), and others.

A full list of claims can be found [here](https://www.iana.org/assignments/jwt/jwt.xhtml#claims)

**Public Claims** are the claims that we create ourselves

JWT states that public claims:

> can be defined at will by those using JWTs. But to avoid collisions they should be defined in the IANA JSON Web Token Registry or be defined as a URI that contains a collision resistant namespace. 

**Private claims** are: 

> the custom claims created to share information between parties that agree on using them.

from the screenshot above, the private claims are: 

- sub
- name
- admin

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```


*Both the header and payload are [Base64URL encoded](https://en.wikipedia.org/wiki/Base64)*  

#### Signature

According to JWT: 

> To create the signature part you have to take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that.


> The signature is used to verify that the sender of the JWT is who it says it is and to ensure that the message wasn't changed along the way.


The signature is also then Base64 encoded, and all 3 parts are then concatenated and separated by a `.`


### Bearer

Once the token is created, it's stored in the client's state and then sent in the request header using the `Bearer` schema.

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ
```


## Knock

While the above is an intro to JWT, the real question is, how do we use this in our own API's. Thankfully, there's a gem for that!

Let me introduce you to [knock](https://github.com/nsarno/knock). 

The description of knock pretty much sums up what it is, and what it does

> Knock is an authentication solution for Rails API-only application based on JSON Web Tokens.

Knock's documentation is fairly short and straight-forward. We will start by adding knock to our Gemfile and `bundle install`ing. 

```ruby
gem 'knock', github: 'psantos10/knock', branch: 'master'
```

*note: At the time of this writing, there is a bug in the knock source code that does not properly recognize verisioned API's. Normally, you would simply add the version of knock hosted on ruby gems `gem 'knock'` see [issue 120](https://github.com/nsarno/knock/issues/120) for more info*


Next, we need to run through the config steps. 

Start by activating knock

```shell
$ rails generate knock:install
```

Then configure Knock to work with your User resource by creating a controller and custom route for JSON Web Token generation

```shell
$ rails generate knock:token_controller user
```

Now, we can move this newly created `user_token_controller` to V1 of our API

```shell
mv ./app/controllers/user_token_controller.rb ./app/controllers/api/v1/
```

And wrap it in our module

```ruby
module Api::V1
  class UserTokenController < Knock::AuthTokenController
  end
end
```


This controller inherits from the Knock authentication that can be found at [`knock/app/controllers/knock/auth_token_contoller.rb`](https://github.com/nsarno/knock/blob/master/app/controllers/knock/auth_token_controller.rb) Which contains the logic for creating our JSON web token. 


Don't forget to also modify your routes!

```ruby
Rails.application.routes.draw do

  scope module: 'api' do
    namespace :v1 do
      resources :tasks
      resources :users
      post 'user_token' => 'user_token#create'
    end
  end
  
end
```

From here, we just need to decide what resources will require authentication. I would say that we require for all tasks outside of user profile creation

Start by including Knock's Authentication module in our ApiController

```ruby
module Api::V1
  class ApiController < ApplicationController
    include Knock::Authenticable

  end
end
```

Then add a before filter in each of your user and task controllers 

##### `app/controllers/api/v1/tasks_controller.rb`

```ruby
module Api::V1
  class TasksController < ApiController

    before_action :authenticate_user
    
    # ... other code
  end
end
```

##### `app/controllers/api/v1/users_controller.rb`

```ruby
module Api::V1
  class UsersController < ApiController
    before_action :authenticate_user, except: [:create]
    
    
    # ... other code
  end
end
```

The only difference here is that we don't won't to require a token when a user is signing up. 

Now that we have all of pieces in place, let's test it out!

Start by sending a `POST` request to `http://localhost:3000/v1/users` with user information. 
 
![](./images/create_user_postman.png)


**Now the fun part!**

We need to request a token, so let's  send a `post` request to `http://localhost:3000/v1/user_token` 

![](./images/jwt_postman.png)


Sweet! It worked! 

Here we sent a request with a pretty simple JSON structure 

```json
{ 
	"auth" : {
		"email" : "shane@techtalentsouth.com",
		"password": "password"
	}
}
```

Our Knock Auth Contoller expects the `email` and `password` data to be wrapped in `"auth"`. 

Check out that nifty response btw! 

```json
{
  "jwt": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0ODE1NzA4NzgsInN1YiI6NH0.xfWxPgp8uIlHo7SSX0Ut4tzdmDWNGbSL2xCcOEw_1fE"
}
```

There's our JSON Web Token complete with `header`, `payload` and `signature`!

Now, we can issue a request to view tasks by simply including the JSON Web Token in our header like so: 

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE0ODE1NzA4NzgsInN1YiI6NH0.xfWxPgp8uIlHo7SSX0Ut4tzdmDWNGbSL2xCcOEw_1fE
```

![](./images/authorization_postman.png)


For fun, you can try removing the token and making a request. Which should return a `401` status code. 

Now, each request will be authenticated, and the responsibility of providing all necessary information has been placed on the client. 

Congrats! You just authenticated your application! 

There are several other token authenication schemes out there that you should review. Personally, I really like Knock, but it won't necessarily meet all of your authentication needs. So make sure to brush up on alternative solutions. And don't be afraid to try and roll your own solution!


## Challenge 2 

- Use Knock to add Authentication to your Restaurant API 
	- Create a User Resource
	- Associate with Orders 
	- Add Knock
	- Configure
		- A User must be authenticated to:
		- view a menu and items
		- place an order
	- Once you've got things setup, make some requests and Have Fun!


## Homework 

###**Due 12/14/16**

- Complete the Restaurant API
- Read [jwt-auth-in-rails](http://www.thegreatcodeadventure.com/jwt-auth-in-rails-from-scratch/)
	- follow along with code examples (optional) 
	- Write about 1 takeaway 
	- Turn in on slack
- Read this description of [CORS](https://www.maxcdn.com/one/visual-glossary/cors/)
	- Write a 1-2 sentence description of CORS in your own words
	- Turn in on slack
- Write up at least 2 items for feedback on this entire class
	- Topics can include: 
		- What did you like? 
		- Is there anything you would change
		- Something that should have been covered more in depth
		- Something that should have been left out
		- Something you wish we would have covered
		- or anything else you can think of   
