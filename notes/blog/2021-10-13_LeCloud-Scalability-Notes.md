---
date: 2021-10-13
tags:
    - blog
hubs:
    - "[[system-design]]"
urls:
    - https://www.lecloud.net/tagged/scalability
---

# LeCloud Scalability Notes

## Clones

- Horizontal scaling solution
- Load balancers evenly distribute user requests to application servers
- Every server needs to have exact clones
- Sessions & user data needs to be stored in centralized database or cache
    - Cache good choice for session data as will have better performance

## Database

- Optimize exacting database
    - Master-slave replication
        - Read from slaves, write to master
    - Add more RAM

- Denormalize
    - Do not join in read query - do these ahead of time and store in materialized view or handle in application code
    - Switch to NoSQL database and do joins in application code


## Cache

- Never do file-based caching
- DB queries can be cached with in-memory systems like Redis or Memcached
- Cached DB queries
    - Hash query as key and set result as value
    - Difficult to know when to invalidate
- Cached data objects
    - Strongly recommend this over cached DB queries
    - Allow application to assemble class from DB query result and cache that object
    - e.g.
        - User sessions
        - Blog articles
        - Activity streams
        - User-friend relationships

## Asynchronism

- Pre-compute content or data ahead of time
- When not possible to predict content or data, use message queues
    - e.g. user comes to site and starts computationally intensive task
        - Frontend of site sends job to job queue and signals to user that it has began
        - Workers monitor job queue and do work as needed, then signal back when done
        - Frontend monitors job done queue for complete signal and delivers result to user
    - Can be pub/sub, RabbitMQ, or even Redis list



