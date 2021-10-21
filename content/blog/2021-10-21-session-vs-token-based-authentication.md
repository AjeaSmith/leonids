---
layout: blog
title: Session vs token-based authentication
date: 2021-10-21T06:23:09.650Z
description: "Understanding session and token-based authentication. "
categories: authentication
---
**Session authentication (club analogy):** 

A user provides credentials to the club bouncer. The bouncer checks if the user is valid. If yes, the user can enter the club. If not, the user cant enter. Once the user is in the club, a **session** is created. This session is assigned to the user. This is to let the club know the user is active in their club. Along with a **cookie** being attached to the session as well. The cookie has the user's ID to identify them. (Cookie has an expiration date). For a user to enter the V.I.P areas of the club, behind the scenes, they need to give the V.I.P guard the **cookie** so they can check to see if you're in the system(DB). If yes, enter. If not, a user can't enter. Once the cookie expires, the user is kicked out (signed out). Then, the user needs to give credentials to the bouncer again. 

**Token (JWT/ refresh) club analogy:** 

Very similar to the session auth. Instead of using sessions and cookies to authenticate, a user is given a **token(JWT)** upon entering the club. The token is given directly to the VIP guards, the guards again check the token to see if it's valid. If valid, they can enter the VIP areas. At some point, the tokens will expire, that's when the refresh tokens come into play.

**Refresh token Job**: Within the club, there is a **refresh token** stand whose job is to give the user a new **token(JWT)** to use within VIP areas when theirs expire. However, the refresh token can expire too. Once it has, you can no longer get new tokens. Which will cause the need for the user to sign back in to get those back (token/refresh token).