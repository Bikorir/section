---
layout: engineering-education
status: publish
published: true
url: /connect-flash-express-sessions-and-boostrap/
title: Handling Authentication using Flash, Express Sessions, and Bootstrap
description: This article explores how to handle authentication using the Passport authentication module, display flash messages to the user with proper bootstrap styling, and store messages in sessions to display them correctly in web page navigation.
author: simon-salva
date: 2021-09-21T00:00:00-08:20
topics: [Languages]
excerpt_separator: <!--more-->
images:

  - url: /engineering-education/connect-flash-express-sessions-and-boostrap/hero.png
    alt: Connect flash, express sessions and bootstrap image example
---
This article explores how to handle authentication using the Passport authentication module, display flash messages to the user with proper bootstrap styling, and store messages in sessions to display them correctly in web page navigation.
<!--more-->
We will build an authentication system using `express-session`, `connect-flash`, `passport`, and `bootstrap` modules in a single application.

### Prerequisites
To follow along, the reader should have.
- A good understanding of [Node.js](https://nodejs.org/).
- Node.js installed.
- A suitable code editor, preferably [VS Code](https://code.visualstudio.com/download).
- Basics of MongoDB database.

### Installing dependencies
Execute the command below in the terminal:

```bash
npm install express passport passport-local bcryptjs ejs express-ejs-layouts mongoose connect-flash express-sessions 

npm i -D nodemon
```

### Setting app, the entry point
We begin by setting up the entry point as `main.js` and importing the dependencies as demonstrated below:

```js
const express  = require('express')
const app = express()
const passport = require('passport');
const expressLayouts = require('express-ejs-layouts')
const flash = require('connect-flash');
const session = require('express-session');
```

### Project setup
We will have a separate folder for our routes, so create a `routes` folder in the application's root directory. 

In the folder, create two files named `index.js` and `users.js`. The authentication requests will go to the `user.js`  while other requests will go to the `index.js` route.

Create another folder called `views`. This folder will contain our view files which will be rendered to the user on the screen. 

In the `routes` directory, add the following files; `login.ejs` `register.ejs` `layout.ejs` and `dashboard.ejs`. 

When a user navigates to the index route of the project, the system directs him/her to the welcome page, which renders the `welcome.ejs` file. 

From there, he chooses to log in or register a new account. Once the user is successfully registered, he can log in, he is then redirected to the dashboard page.

In the `login.ejs`, we have a form that submits `user email` and `password` for authentication, as illustrated below:

```html
 <form action="/users/login" method="POST">
    <div class="form-group">
        <input  type="email"  id="email" name="email" class="form-control" placeholder="Enter Email" />
    </div>
    <div class="form-group">
        <input type="password" id="password"  name="password" class="form-control" placeholder="Enter Password" />
    </div>
    <button type="submit" class="btn btn-success btn-block">Login</button>
</form>
```

For the `registration` page, we will have a form that submits user data to the database generated by the snippets below:

```html
<form action="/users/register" method="POST">
    <div class="form-group">
        <input type="name" id="name" name="name" class="form-control" placeholder="Enter Name"/>
    </div>
    <div class="form-group">
        <input  type="email" id="email"  name="email" class="form-control"  placeholder="Enter Email" />
    </div>
    <div class="form-group">
        <input type="password" id="password" name="password" class="form-control" placeholder="Create Password"/>
    </div>
    <div class="form-group">
        <input type="password" id="password2" name="password2" class="form-control" placeholder="Confirm Password" />
    </div>
    <button type="submit" class="btn btn-success btn-block">
        Register
    </button>
</form>
```

### Connecting to database
Since the application will use Mongo Atlas, we will create a connection string and store it into a file. 

First, create a folder named `config` in the root directory of the application. In this folder, add two files named `connection.js` and `config.env`.

The `config` file contains environmental variables that are uniform throughout the application.

This file stores our `connection string` and the `port` in which our application will run.

```js
PORT = 5000
MONGO_URI = 'YOUR CONNECTION STRING'
```

The `connection.js` file will contain the `connection function`. The function is responsible for creating a connection between the application and the remote database using the database URL in the config file. 

We export this function to the application's entry point such that the database is connected as the server is initialized.

```js
const mongoose = require('mongoose')

//connnect the system to the mongo atlas remote db
const connectToRemoteDatabase = async () => {
    try {
        const conn = await mongoose.connect(process.env.MONGO_URI, {
            useNewUrlParser: true,
            useUnifiedTopology: true,
            useFindAndModify: false,
        })
        console.log(`Database connection successful`)
    } catch (error) {
        console.error(error)
        process.exit(1)
    }
}

module.exports = connectDatabase
```

### Creating the user model
Models define how database records look like. For our case, users will keep a record of their emails, usernames, and passwords. 

These fields should appear in the user model and the database. 

In the root folder of the application, create a new directory named `models`. Then create a new file called `user.js` in the generated folder and add the snippet below:

```js
const mongoose = require('mongoose');

//user schema
const UserSchema = new mongoose.Schema({
    username: {type: String, required: true },
    useremail: {type: String, required: true },
    password: {type: String, required: true },
    date: {type: Date, default: Date.now }
});

const User = mongoose.model('User', UserSchema);

//export user model
module.exports = User;
```

### User registration
In this phase, we will register users by collecting the data from the form outline below. 

Since we are collecting data from a form, we need to use body-parser middleware:

```js
//Body parser
app.use(express.urlencoded({ extended: false}))
```

Next, we set up a `register-handler` that collects the form data from the request made in the front end. 

In the first step, the registration route extracts the form data from the request body then checks if the form data are entered correctly.

```js
//registration handler
router.post('/register', (request, response) =>{

    //extract the data from request body
    const name= request.body.name;
    const email= request.body.email;
    const password,  = request.password;
    const passwordConfirm  = request.passwordConfirm;
    let errors = []

    //validation
    if (!name || !email || !password || !password2) {
        errors.push({ message: 'All the fields must be filled to proceed' });
    }

    if (password != password2) {
        errors.push({ message: 'The two passwords must match to proceed' });
    }

    if (password.length < 5) {
        errors.push({ message: 'Sorry the password must be at least 5 characters long' });
    }

    if (errors.length > 0) {
        response.render('register', { errors, name, email,  password, password2  });
    }else{
        //Check if the user exists
    }
})
```

After the form validation is passed, we have to confirm if the email submitted by the user already exists in the database or not. 

If the email exists, we push the error to the `errors` array, then render the `registration` page and display the `errors`.

```js
User.findOne({email: email}).then(user =>{
    if(user){
        errors.push({messageg: 'Email already in the database'})
        res.render('register', {  errors,  name, email,  password,   password2 })
    }else{

        //hash the password and register the user
    }
})
```

If the supplied email is unique, `bcryptjs` hashes the password. 

Saving a plain text password is a security risk, so we hash the password to avoid system breaches. 

After hashing, the user instance is saved to the database, and then the system redirects the user to the login page.

```js
const newSystemUser = new User({username, useremail, password});

bcrypt.genSalt(10, (error, saltpass) =>{
    bcrypt.hash(password, saltpass, (error, passwordhash) => {
        if(error){
            throw error;
        }else{
            newSystemUser.password = hash
            newSystemUser.save().then(user =>{
                request.flash('success_msg', 'Successfully registered. Login')
                response.redirect('/users/login')
            }).catch(err =>{
                console.log(err)
            })
        }
    })
})
```

![Registration validation](/engineering-education/connect-flash-express-sessions-and-boostrap/connect-flash-errors.png)

### Implementing connect-flash module
At the moment, we are passing the errors to a view that will render on the registration page. 

However, we want to store the messages in a session to display them after a redirect. This operation requires the `connect-flash` middleware.

```js
const flash = require('connect-flash');
const session = require('express-session');

// Express sessions
app.use(session({ secret: 'yoursecret', resave: true,  saveUninitialized: true }));

// Connect flash
app.use(flash());
```

To make each error appear in a different color, we create global variables and set up `colors` for every error in the application's entry point:

```js
// Global variables
app.use(function(request, response, next) {
    response.locals.success_alert_message = request.flash('success_alert_message');
    response.locals.error_message = request.flash('error_message');
    response.locals.error = request.flash('error');
    next();
});
```

In the `messages.js` file, we check whether a message is a `success` or an `error` message then render the respective alert. 

```html
<% if(success_alert_message != ''){ %>
    <div class="alert alert-success alert-dismissible fade show" role="alert">
      <%= success_alert_message %>
      <button type="button" class="close" data-dismiss="alert" aria-label="Close">
        <span aria-hidden="true">&times;</span>
      </button>
    </div>
<% } %>

<% if(error_message != ''){ %>
    <div class="alert alert-danger alert-dismissible fade show" role="alert">
      <%= error_message %>
      <button type="button" class="close" data-dismiss="alert" aria-label="Close">
        <span aria-hidden="true">&times;</span>
      </button>
    </div>
<% } %>
```

![Login page redirect](/engineering-education/connect-flash-express-sessions-and-boostrap/login-page-redirect.png)

### Passport authentication setup
Create a new file called `passport.js` in the' config' folder, then add the snippet below:

```js
const LocalStrategy = require('passport-local').Strategy;
const bcrypt = require('bcryptjs');

// Loading the mongoose model from the models folder
const User = require('../models/User');
```

First, we need to bring in the `local strategy` and `mongoose` to find users in the database. 

In this case, we are using `bcrypt` to compare the password entered by the user during registration to the one entered during login. 

Passport needs to check email and password, then find a user with the same email. If a user with the same email exists, then the supplied password is compared against the user's password to see if there is a match; if the password is similar to the entered password, the passport authenticates the user. 

However, if the password is not similar, an error is displayed to the user telling him to enter the correct password.

```js
function passport() {
    passport.use(
        new LocalStrategy({ usernameField: 'useremail' }, (useremail, userpassword, done) => {
            // find user with supplied email
            User.findOne({
            useremail: useremail
            }).then(user => {

                //the fetched user  is not found
                if (!user) {
                    return done(null, false, { message: 'The user email entered is not with our records' });
                }
        
                //use bcrypt to compare the passwords and validate
                bcrypt.compare(password, user.password, (error, isMatch) => {
                    if (err) throw error;
                    if (isMatch) {
                        return done(null, user);
                    } else {
                        return done(null, false, { message: 'Password entered is incorrect.' });
                    }
                });
            });
        })
    );
  
    passport.serializeUser( (user, done) => {
      done(null, user.id);
    });
  
    passport.deserializeUser( (id, done) => {
      User.findById(id, (error, user) => {
        done(error, user);
      });
    });
};

module.exports = passport
```
[Login validation](/engineering-education/connect-flash-express-sessions-and-boostrap/login-form-validation.png)

### Building the login module handler.
The login handler uses the Passport middleware to authenticate users. An authenticated user is redirected to the `home` route to see his account details. 

However, if the user is not authenticated, the system redirects him to the login page to correct their details and try again.

```js
//handling sign in route
router.post('/login', (request, response, next) => {
    passport.authenticate('local', {
        successRedirect: '/home',
        failureRedirect: '/users/login',
        failureFlash: true
    })(request, response, next);
});
```

### Securing routes
We secure a route to make it inaccessible to unauthenticated users. For our case, the only route that can be secured is the `home` route. 

A user needs to be authenticated to access the resources on this page. So first, we need to create a new file called `autheticate.js` in the `config` folder to secure the route, then add the snippet below:

```js
module.exports = {
    ensureUserIsAuthenticated: function(request, response, next) {
        if (request.isAuthenticated()) {
            return next();
        }
        request.flash('error_message', 'Please log in to access the requested page');
        response.redirect('/users/login');
    },

    forwardAuthenticatedUser: function(request, response, next) {
        if (!request.isAuthenticated()) {
            return next();
        }
        response.redirect('/home');      
    }
};
```

In the `index.js` file in the `routes` folder, add the code below to import the authentication and secure the `home` route:

```js
router.get('/home', ensureUserIsAuthenticated, (request, response) => {
    response.render('dashboard')
})
```

[User's homepage](/engineering-education/connect-flash-express-sessions-and-boostrap/homepage.png)

### Setting up the logout handler
The logout handler is responsible for signing out a user and destroying the session of the logged-in user. 

When users are logged in, their sessions are stored in a cookie that the logout handler destroys.

```js
//logout handler
router.get('/logout', (request, response) => {
    request.logout();
    request.flash('success_alert_message', 'You are succesfully logged out');
    response.redirect('/users/login');
});
```
![Log out](/engineering-education/connect-flash-express-sessions-and-boostrap/logout.png)

### Conclusion
This article has showed you how to use connect-flash to display error messages in a system, store the messages in express sessions, and style error messages using bootstrap. 

We implemented these concepts by building a complete authentication system based on Passport. 

This project should provide a head start for actively working on the authentication module of any node.js project.

You can find the source code for this application in [this.](https://replit.com/@salvador02/authetication)

---
Peer Review Contributions by: [Jerim Kaura](/engineering-education/authors/jerim-kaura/)
