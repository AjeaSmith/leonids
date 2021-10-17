---
layout: blog
title: Custom Errors in Node/Express
date: 2021-10-17T03:55:33.444Z
description: "How I created custom errors in my Story Master (Full-Stack) app. "
categories: List [ "Errors", "Node/Express" ]
---
At first, I had trouble figuring out how to do this. Along with handling errors for different cases. What I decided to do was to create helper functions that will take care of this, then I would call these functions when appropriate. 

It would look something like this

```javascript
function UserExistException(message) {
	const error = new Error(message);
	error.code = 409;
	return error;
}
function ProfileNotFoundException(message) {
	const error = new Error(message);
	error.code = 404;
	return error;
}
```

I read prior that it is a good idea to make use of the **Error** object in node when creating errors so you can get helpful error info from the stack trace. Also, with the help of the **Error** object, I could also set the status code to whatever is appropriate. In the end, I would throw these custom errors when needed. (also called "exceptions").

How it would look then thrown

```javascript
// When a profile is not found, I throw my custom error(exception) message
const getProfile = async (email) => {
	const profile = await UserDataAccess.me(email);
	if (!profile) {
		throw new ProfileNotFoundException('Profile does not exist');
	}
	return profile;
};
```

The next step would be to catch these exceptions within a **try..catch** block or **.then/.catch** block and return it back to the client to display error message. (see below: code uses the **.then/.catch** method)

```javascript
router.get('/:email', async (req, res) => {
  UserService.getProfile(req.params.email)
	.then((profile) => {
		res.status(200).json(profile);
	})
	.catch((error) => {
		return res.status(error.code).send({ error: error.message });
	});
});
```