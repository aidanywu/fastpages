---
toc: true
layout: post
description: Implementing logout into spring boot project
categories: [markdown]
comments: true
title: JWT Logout
---

## Logout
For logout to work properly, there are two main jobs the logout request has to do: (1) remove the existing JWT cookie on the user's browser, and (2) stop the user from using the same JWT to access server resources.

Removing the existing JWT cookie is easy, we simply have to return a new cookie with the key "jwt" and a Max-Age of 0. This will replace the JWT cookie on the user's browser and expire instantly.

To stop the user from using the same JWT, we have to store expired JWTs somewhere on the server. Two ways to do this are to use cache or another table inside the existing database.

### Database Approach
First, we need to create a table inside the database that stores the logged out JWTs. An API Controller that allows you to get all the blacklisted JWTs with /blacklist/ and to logout by saving the JWT into the blacklist and is created as well.
The above is done in [commit](https://github.com/aidanywu/spring_port/commit/187159326cd48d0e77dfc6fe6efa7ac832f79aca)

Next, we need to check if every request's JWT token is inside the blacklist. If so, the request should return unauthorized because they shouldn't be able to use a JWT they have already logged out with to be authenticated.
This is done in [commit](https://github.com/aidanywu/spring_port/commit/d6d3a60e10ad0d8e9c2390fdc6e0773cb38f5ca2).
(Note: I also made each JWT to expire in 1 hour instead of 5 hours when the user authenticates to match with the cookie Max-Age which is 1 hour.)

Finally, if we never go through the database to clear out blacklisted tokens that have already expired (so it is meaningless to keep the token in there because the user can't be authenticated with an expired token anyways), it will stack up. We can use the scheduler annotation in spring boot to run the method that clears expired token out of the blacklist periodically. This is done in [commit](https://github.com/aidanywu/spring_port/commit/c282a2bf405588910eef0018b7cce522afcea629).
(Note: All these durations can be configured/changed at your convenience)

#### Postman Testing
Note: I made the JWT token to expire in 2 minute and the scheduler to run at every 30 seconds for testing convenience.

First, we authenticate to get a JWT token:
![authenticating](https://user-images.githubusercontent.com/56620132/222934688-644a7814-ea0b-4f4a-aec7-b2f6925474c8.png)


Then, we verify that the JWT token is working by /api/person/ which is restricted to only authenticated users:
![JWT works](https://user-images.githubusercontent.com/56620132/222934816-2083d794-f7f5-4ee8-8e97-97a38fdaff9f.png)


The blacklist is empty as of now because we haven't logged out:
![blacklist pre-logout](https://user-images.githubusercontent.com/56620132/222934731-b456508a-06df-48c8-b45f-4db844ea53dc.png)


Now, we logout using /blacklist/logout with the JWT token stored in the Cookie header. The request returns a Set-Cookie header with an empty JWT cookie of Max-Age 0 to delete the existing client-side JWT cookie:
![logging out](https://user-images.githubusercontent.com/56620132/222934706-d25092bb-2f12-45c1-badc-9f4ba2c39243.png)


The logout request has taken the JWT token in our cookie header and stored it into the blacklist:
![blacklist post-logout](https://user-images.githubusercontent.com/56620132/222934833-9a2982b1-679b-46d9-8167-7089cc731ca9.png)


We can no longer access /api/person/ with the same JWT token because the server has checked that the JWT token is inside the blacklist and we should not be authenticated:
![JWT does not work](https://user-images.githubusercontent.com/56620132/222934752-17775720-04cc-4b32-b4b6-b07f109e3963.png)


Finally, to test the cleaning feature, we wait for 2 minutes or so until the scheduled task runs to delete the JWT token inside the blacklist which has now expired:
![blacklist cleaned](https://user-images.githubusercontent.com/56620132/222934843-c2f74df5-82c7-44f3-a75b-9a30ea1aa36e.png)



### Cache Approach
Not finished implementing yet
