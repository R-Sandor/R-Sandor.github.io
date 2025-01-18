---
layout: post
title: Integration Testing a Developer's Story
date: 2023-12-22
categories: ["Spring Boot", "Intgeration Test", "Testing"]
---

I have been developing a side app for the past 6 months. Recently decided now that everything is
working together to switch to TDD from here out. But the application is still small, there is not a lot
of internal logic for Java to handle, making unit testing pointless. As if often the case with full stack
applications in the infancy 70% of the code is just getting all the pieces working together.

### How to test it?

Since it largely just a database and some security configuration, mocking it doesn't
prove much. We'll have some issues for you to comb through too coming up :)

### Answer: Integration Testing

The best option was to test what a request going to the rest endpoint from the database looks like, I can hear what one is thinking, hold on that should be E2E? Hold on and see the value of Integration below.

The key takeaways to be presented:

1. Integration tests are crucial
2. Don't assume end-to-end will find all the problems.
3. TestContainers are easy.
4. Defense in depth.

I set up the TestContainer, seeded it with some data and simulated a rest request. But something strange happened, I didn't get the response I was expecting, I got an empty array. Well that's good I thought, an unauthenticated user is not getting any data, especially not everyone else's.
First Inclination is to change the test return an empty.
![testcontainers](/assets/photos/testcontainers.png)

Looking at the logs, with SQL logging enabled, one can see that SQL is being run? Should we panic?
![sql](/assets/photos/sql.png)

Not yet, while this isn't ideal, that last where clause is saying that only return those that belong to a tenantID. The only way to get a tenantID on this query is through a valid JWT.

# Fix 1

Remembering for testing it allowed `/api/*` to be available for all. Well that is not good, sure from the web browser it seemed to work in development, the SecurityFilterChainAlways worked. There was previously some issues with pattern matching with with API, it was disabled it and never fixed...

![Removeignore](/assets/photos/removeIgnore.png)

Now, run the test again, and it should fail?
![failure1](/assets/photos/failure 1.png)
Running it from the IDE, I see that there is a failure not sure if that is correct one but its failing. Will investigate this next.
Changing the type to Bookmark.class seems to have resolved it, not sure why...
![failure2](/assets/photos/unauthworks 2.png)
