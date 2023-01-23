---
toc: true
layout: post
description: Implementing JWT Tokens into spring boot project
categories: [markdown]
comments: true
title: JWT Tokens
---

## The Code
A majority of the code was taken from https://www.javainuse.com/spring/boot-jwt. The new dependencies we need is in this [Commit](https://github.com/aidanywu/spring_port/commit/a5447a6269bd2bae123c415606ac5d0f97db2d25). This site provided the basic code needed to generate a jwt token and how we would configure the backend to require authorization when accessing a page. ([Commit1](https://github.com/aidanywu/spring_port/commit/6aad61a5902917e225f3b3dbaf7bd1451b986123) & [Commit2](https://github.com/aidanywu/spring_port/commit/08f3cc8c03b44b41ee7c79c3ce2b30ef6165386e)). JwtTokenUtil.java, just like its name, contains utilities/functions that is needed to generate JWT tokens and get information like the email from the JWT tokens which is needed for making sure the JWT token is valid. JWTUserDetailsService.java implements the Spring Security UserDetailsService interface and overrides the loadUserByUsername so we can later get user details from the database using the username. The name JwtAuthenticationController.java on the article was changed to JwtApiController.java to fit the already existing API controllers and this creates an API endpoint of /authenticate which validates that the given email and password in the body is validate and then generates a JWT token for the credentials if it was valid. We did not need the JwtRequest.java because it is used to store each user as an object and that is the Person class we've already coded. We also do not need JwtResponse.java. The JwtRequestFilter.java extends the Spring Web Filter OncePerRequestFilter class and overrides the doFilterInternal function so each request sent to the server is processed through the function. The function checks if the JWT token is valid and sets the Authentication in the context to specify that the current user is authenticated. JwtAuthenticationEntryPoint.java implements AuthenticationEntryPoint and overrides the commence function to specify what to do when a user was not authenticated, which is to return an unauthorized error. WebSecurityConfig.java is our existing SecurityConfig.java. It extends WebSecurityConfigurerAdapter and overrides configure to allow /authenticate to not need the request be authenticated because /authenticate is where you generate the JWT token is get authenticated, add the jwtRequestFilter.java filter to validate the token with every request, and configure other features of web security like a stateless session. This can also be used to allow specific roles when that is configured (as seen in last year's csa project).

JWTUserDetailsService.java was changed from the article because we already have a database storing users while the article hardcoded a user and password because it did not.

In the article, it assumed that our passwords stored inside the database was already encrypted using bcrypt, therefore, we have to configure /post to encrypt the passwords when adding users ([Commit](https://github.com/aidanywu/spring_port/commit/5a869fd7fd37883628880a55699aba8394a1cf68)).

Since the article coded for /authenticate to return the JWT token inside its body, the frontend would have to take the JWT token and configure it into the browser. This would be susceptible to Cross Site Scripting Attacks. We can avoid forcing the frontend's javascript to set the cookie by coding for /authenticate api's response to automatically set the browser's cookies with the JWT token using the Set-Cookie header. For this part, I followed along with https://reflectoring.io/spring-boot-cookies/ and made some changes to JwtApiController.java. Now the request sent by the browser would contain the JWT token using a HttpOnly cookie, so we have to code JwtRequestFilter.java so it gets the JWT token from the request's Cookie header instead of its Authorization header ([Commit](https://github.com/aidanywu/spring_port/commit/fcecd4e650a894912b5abddc2005f94d46fd8f65)).

## Testing
Using the existing account's credentials test2@gmail.com and test2 to generate JWT Token through /authenticate.
![generating jwt token](https://user-images.githubusercontent.com/56620132/213969351-4f9bedc3-7780-4e24-b908-f63dae90e47d.png)

Trying to access /api/person/ without jwt token stored in Cookies
![unauthorized](https://user-images.githubusercontent.com/56620132/213969664-91397851-1484-4169-8187-e3a0ea8fda0d.png)

Trying to access /api/person/ with generated jwt token stored in Cookies
![authorized](https://user-images.githubusercontent.com/56620132/213969894-d5a83af9-614e-45ec-afe9-123d8d422713.png)


Roles can be be implemented next based on the old csa project and I am looking at that next.