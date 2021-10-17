---
layout: blog
title: Understanding Passport Authenticate with Nodejs
date: 2021-10-17T05:39:17.439Z
description: How I used passport local within my project
categories: passport authentication, nodejs
---
There are a few required pieces to get passport authentication working. **Pick a strategy**, **configure it**, **serialize/deserialize**, **initialize it**, and use it as a **middleware**. 

**Define passport:** Authentication package for nodejs to authenticate users in various ways (username/password, JWT token, or using 0Auth), which handles all the hard parts for you. 

**Strategy:** Pick a way to authenticate a user. Ex: I'm using the local strategy, which uses a username and password to log in a user. There are other strategies like using the 0Auth way. 

**Configure it:** During configuration, I needed to specify which strategy I'm using(local) along with a verify callback function that takes in **username** and **password** and another **done(cb)**. Within the verify callback, I check if the user exists already in DB, if there is an error, and if passwords don't match. If yes then authentication will fail with the *done(cb)* return unauthorized. If not, the authentication will succeed and the *done(cb)* will pass user info to the **deserialize/serialize** passport functions. Which helps log a user in behind the scenes. (code below)

```javascript
const User = require('../src/auth/User');
const LocalStrategy = require('passport-local').Strategy;
const bcrypt = require('bcryptjs');

module.exports = function (passport) {
	passport.use(
		new LocalStrategy((username, password, done) => {
			User.findOne({ username: username }, (err, user) => {
				if (err) throw err;
				if (!user) return done(null, false);
				bcrypt.compare(password, user.password, (err, result) => {
					if (err) throw err;
					if (result === true) {
						return done(null, user);
					} else {
						return done(null, false);
					}
				});
			});
		})
	);

	passport.serializeUser((user, cb) => {
		// code shown later
	});
	passport.deserializeUser((id, cb) => {
		// code shown later
	});
};
```

**Serialize User:** If the user logs in successfully a session is created, to keep track of that logged-in user. Passport stores the user's id within a cookie(the expiry time is set within the express session portion)

```javascript
passport.serializeUser((user, cb) => {
	cb(null, user.id);
});
```

**Deserialize User:** finds a user by the id from the cookie, if the user is found passport stores user info on req object so you can have access to the user's info within authorized routes when needed. 

```javascript
passport.deserializeUser((id, cb) => {
		User.findOne({ _id: id }, (err, user) => {
			const userInformation = {
				username: user.username,
			};
			cb(err, userInformation);
		});
	});
```

**Initialize it:** Of course, none of this will matter if you don't initialize it. 

***Important***: I'm using sessions, therefore I need to use sessions from passport, and express sessions(need to be installed). Then set up express sessions and use it as middleware before the passport sessions. 

```javascript
// ------------  Middleware (Session configuration) ---------------

// this is express session
app.use(
	session({
		secret: 'something secure',
		resave: false,
		saveUninitialized: false,
		cookie: {
			maxAge: 1000 * 60 * 60 * 24, // Equals 1 day (1 day * 24 hr/1 day * 60 min/1 hr * 60 sec/1 min * 1000 ms / 1 sec)
		},
	})
);
app.use(cookieParser('something secure'));

// ----------- Passport Authentication -------------------

app.use(passport.initialize());
app.use(passport.session());

// brings in our passpoer configuration from our config/passport.js file
require('./config/passport')(passport);
```

**Middleware**: You can simply use passport as middleware within the route. In my case, the login route. This is were all the magic happens :) if everything goes well, i'll be sending back a success message. 

```javascript
router.post('/login', passport.authenticate('local'), (req, res, next) => {
	res.send({ msg: 'Successfully logged in' });
});
```