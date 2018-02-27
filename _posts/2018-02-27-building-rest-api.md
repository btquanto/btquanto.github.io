---
title: What you need to know before building your REST API
category: Programming Techniques
tags: ["programming", "REST"]
descriptions: What you need to know before building your REST API
keywords: REST API, HTTP, JSON API, RPC API, JSON
---

# Terminologies

REST API centers around resources that are grouped into collections

+ **Services** expose a set of collections and resources in order to provide a correlated functionalities.

    Services are customarily named with single nouns.

    Example: `/auth` service provides functionalities related to authentication, e.g. login, logout, refresh token.

+ **Resources** are pieces of any kind of information, e.g. documents, audio/video files, xml/csv/json data, web pages…

    A resource is identified by a `{resource_id}`.

    Example: `/users/1` is a user resource belonging to the collection users, identified by resource id 1.

+ **Collections** typically are plural nouns. Collections give an identity to all the resources they contain.

    A collection can be a part of a resource.

    Example: `/auth/tokens` is a collection of tokens, belonging to the authentication service.

# REST API URL structure

```xml
# Standalone collection
/<collection>
# Collection managed by a service
/<service>/<collection>
# Nested collection and resource
/<parent_collection>/<parent_id>/<child_collection>
# Accessing a resource
/<collection>/<resource_id>
/<service>/<collection>/<resource_id>
/<parent_collection>/<parent_id>/<child_collection>/<child_id>
<service>/<parent_collection>/<parent_id>/<child_collection>/<child_id>
```

Since every resource is identified with a `{resource_id}`, `/<collection>/<resource_id>` (**#1**) and `/<parent_collection>/<parent_id>/<child_collection>/<child_id>` (**#2**) are basically referring to the same resource. The notable difference is that  **#1** refers purely to the resource, while  (**#2**) stresses more on the relationship between the resources.

# Resource and collection relationship: composition vs. aggregation

**Composition** is a type of containment relationship, in which a child collection belongs to a parent resource, but not any others. To rephrase, the child resource is managed by the parent resource; it cannot exist independently of the parent resource, and thus, the destruction of the parent resource will lead to its own destruction.

For example:

+ A house is composed of several rooms. The rooms cannot exist on their own.The demolishment of the house will cause the rooms cease to exist.

+ A time table has many time entries. The time entries don’t have any meaning without belonging in a time table. If the time table is removed, the time entries must as well be deleted.

**Aggregation** is a type of containment relationship in which the the child resource can exist independently of a parent resource.

For example:

+ A course has many students. However, the students are free to join multiple courses. If a course is removed from the school’s curriculum, the students are simply disenrolled from that course.

+ A chatroom channel may have many users. A user can join multiple chatroom channels. If the channel admin suddenly decides to delete the channel, other channel users would simply be forced to leave the original channel.

+ A user can has many friends, who are just other users. When a user delete his accounts, only his relationships with the other users are removed, not the users themselves.

# Interacting with collections and resources

REST APIs use HTTP methods to interact with collections and resources. There are more HTTP methods, but the following five are the most used, and are often adequate for any REST API implementations.

## GET

The HTTP GET method is used for retrieving a resource or collection from the REST API service.

When applying GET method onto a collection, it is expected get a list of the collection members.

GET is safe and idempotent. It means that GET requests do not have any side effects to the API server. The consequence is that calling the same request multiple times would produce the same result (unless the server’s state has been changed by something else)

When implementing GET APIs, it is recommended to provide filtering, sorting, fields selection and pagination for collections.

Examples:

+ Filtering:
```
# Get user’s device whose type is electronic
GET /users/1/devices?type=electronic
# Get user’s female friends
GET /users/1/friends?gender=female
# Show friend requests dated after Dec 31st, 2015
GET /users/1/friend_requests?after=12-31-2015
# List user posts between Jan 1st, 2016 and Dec 31st, 2017
GET /users/1/posts?after=1-1-2016&before=12-31-2017
```

+ Sorting:
```
# Sort posts by ascending title and descending created_date
GET /users/1/posts?sort=+title,-created_date
Fields selection:
# Get user friends, but only retrieve their id and username
GET /users/1/friends?fields=id,username
```

+ Pagination:
```
# Get news articles from the first page, 10 articles per page
GET /news?page=0&size=10
```

## POST

POST method only performs on a collection. It creates a new resource in the collection.

POST is non-idempotent. Calling the same request twice, if succeeds, will create two resources in that same collection, each with a different resource id (unless the server returns some errors due to logical constraints, e.g. disallowing duplications in certain fields).

POST can as well be used to perform actions in the API server, though this is considered hacky and should be avoided if possible. However, in case you really need it, create a new action resource in the collection actions.

Example:

```
# Resend activation code to user
POST /auth/actions { action: “resend_activation_code”, “user_id”: 1 }
# Tell a car to stop
POST /car_manager/cars/1/actions { action: “stop” }
# Tell all cars to stop
POST /car_manager/actions { action: “stop all” }
```

## PUT

PUT methods only performs on a resource. PUT request replaces the resource with new data.

It is recommended to include all fields when using PUT, else non-included fields would replace the resource corresponding fields with empty data. To update a resource, use PATCH.

PUT is not safe, since it changes the state of the API server by updating the resource with new data. It is idempotent, though; calling the same request multiple times would create the same result.


## PATCH

PATCH methods only performs on a resource. PATCH is similar to put, except that it updates instead of replacing the resource. Non-included fields will not update the resource’s corresponding fields.

## DELETE

DELETE performs on both resources and collections:

+ If performs on a resource, it will delete that resource.
+ If performs on a collection, it will empty that collection.

DELETE is non-idempotent. Calling the same request the second time should returns a 404 error if the first call was successful.

**Notes:**

When designing DELETE APIs, we must pay attention to what we really want to delete.

+ If we want to delete a resource, use direct access URL structure
```
# A user is a standalone entity, thus we can delete it directly
DELETE /users/1
```

+ If we want to dissociate the resource with its parent (especially in aggregation relationship), use the nested collection URL structure.
```
# A friend is not a user. It is a relationship between a user and another user. Delete a friend doesn’t really make any sense, but rather dissociate the two users. Therefore, we use the nested collection and resource URL structure.
DELETE /users/1/friends/1
```