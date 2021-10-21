---
layout: blog
title: Handling navigation within Redux
date: 2021-10-21T07:11:50.519Z
description: "**The problem I was having:** I was trying to redirect a user on a
  successful register using `history.push` from `react-router-dom`. However,
  even when the form was submitted with invalid credentials it still redirects.
  **\\*Ex below\\***"
categories: react-router-dom, redux
---
**The problem I was having:** I was trying to redirect a user on a successful register using `history.push` from `react-router-dom`. However, even when the form was submitted with invalid credentials it still redirects. **\*Ex below\***

```javascript
// register.js - before implementation
const Register = ({ history }) => {

	const submit = (data) => {
		dispatch(UserActionCreators.register(data.username, data.email, data.password));
        history.push('/')
	};
    
    return(
      <div>A form goes here....</div>
    )
 }
```

**My Solution:** I figured out in order to handle this correctly, I needed to add the redirect logic within the action creators. So, when register form is submitted (triggered by `dispatch`) I passed in the `history` object alongside the credentials, therefore only calling `history.push` on a successful dispatch.

```javascript
// register.js - after implementation
const Register = ({ history }) => {

	const submit = (data) => {
		dispatch(UserActionCreators.register(data.username, data.email, data.password, history));
	};
    
    return(
      <div>A form goes here....</div>
    )
 }
```

What my register action creator looks like

```javascript
export const register = (email, username, password, history) => async (
	dispatch
) => {
	dispatch({ type: 'REGISTER_PENDING' });

	return axios
		.post(
			'http://localhost:8080/api/user/register',
			{
				email,
				username,
				password,
			},
			{
				headers: { 'Content-Type': 'application/json' },
				withCredentials: true,
			}
		)
		.then(({ data }) => {
			dispatch({
				type: 'REGISTER_SUCCESS',
				payload: { message: data.msg, error: '' },
			});
            // * Important - only redirects when register was an success *
			setTimeout(() => {
				history.push('/login');
				dispatch({ type: 'RESET' });
			}, 2000);
		})
		.catch((err) => {
			dispatch({
				type: 'REGISTER_FAIL',
				payload: {
					message: '',
					error: err.response.data.error,
				},
			});
			setTimeout(() => {
				dispatch({ type: 'RESET' });
			}, 3000);
		});
};
```