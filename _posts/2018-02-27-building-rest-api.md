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
    ```

+ Fields selection:
    ```
    # Get user friends, but only retrieve their id and username
    GET /users/1/friends?fields=id,username
    ```

+ Pagination:
    ```
    # Get news articles from the first page, 10 articles per page
    GET /news?page_number=0&page_size=10
    ```

## POST

POST method only performs on a collection. It creates a new resource in the collection.

POST is non-idempotent. Calling the same request twice, if succeeds, will create two resources in that same collection, each with a different resource id (unless the server returns some errors due to logical constraints, e.g. disallowing duplications in certain fields).

POST can as well be used to perform actions in the API server, though this is considered hacky and should be avoided if possible. However, in case you really need it, create a new action resource in the collection actions.

Example:

```
# Resend activation code to user
POST /auth/actions { action: "resend_activation_code", "user_id": 1 }

# Tell a car to stop
POST /car_manager/cars/1/actions { action: "stop" }

# Tell all cars to stop
POST /car_manager/actions { action: "stop all" }
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

**Notes:** When designing DELETE APIs, we must pay attention to what we really want to delete.

+ If we want to delete a resource, use direct access URL structure
    ```
    # A user is a standalone entity, thus we can delete it directly
    DELETE /users/1
    ```

+ If we want to dissociate the resource with its parent (especially in aggregation relationship), use the nested collection URL structure.
    ```
    # A friend is not a user. 
    # It is a relationship between a user and another user. 
    # Delete a friend doesn’t really make any sense, 
    # but rather dissociate the two users. 
    # Therefore, we apply the nested collection and resource URL structure.
    DELETE /users/1/friends/1
    ```

# Receiving responses

## HTTP elements

The JSON (JavaScript Object Notation) format is a popular way to represent resources in order to communicate between the API server and the client. There are two types of JSON object:
+ JSON Object
+ JSON Array

However, though it's possible, not all information should be included in the json object. REST API makes use of HTTP message elements to attach additional metadata:
+ REST API uses HTTP status codes to  indicate if a request is successful or errorneous; and give additional information about the result if applicable.
+ Sometimes, HTTP message header fields are utilized to communicate metadata not directly related to the information being exchanged, e.g: authentication information, response format, expected content language, content language...

The main HTTP status codes used are generally of class 2xx and 4xx. The following are the most frequently used ones.

### 2xx: success

+ **200 OK**: The standard response code for a successful request. A response body is expected.
+ **201 Created**: The request was successful, and a resource was created
+ **202 Accepted**: The request was accepted and being processed. However the result of the process would be unknown. This status code is used when we do not immediately care about the result of the process

    For example: 
```
# Asking the server to shutdown
POST /system/actions { action: shutdown }
```
+ **204 No Content**: The server successfully processed the request, but is not returning any content. This status code is used when we only care if the request is successful or not.

_Examples of **2xx** responses:_
```
POST /auth/tokens
Host: https://api.theitfox.com
Accept-Language: en-US
Accept: application/json
Content-Type: application/x-www-form-urlencoded

grant_type=credentials&username=<username>&password=<password>
--------------------------------------------------------------
HTTP/1.1 200 OK
Date: Fri, 2 Feb 2018 17:26:00 PST
Server: Nginx
Content-Language: en-US
Content-Type: application/json; charset=utf-8

{
    "expiration": <unix timestamp>,
    "access_token": <access token>,
    "refresh_token": <refresh token>
}
```
```
POST /auth/tokens
Host: https://api.theitfox.com
Accept-Language: en-US
Accept: application/json
Content-Type: application/x-www-form-urlencoded

grant_type=refresh&token=<token>
--------------------------------------------------------------
HTTP/1.1 200 OK
Date: Fri, 2 Feb 2018 17:26:00 PST
Server: Nginx
Content-Language: en-US
Content-Type: application/json; charset=utf-8

{
    "expiration": <unix timestamp>,
    "access_token": <access token>,
    "refresh_token": <refresh token>
}
```
```
POST /users
Host: https://api.theitfox.com
Accept-Language: en-US
Accept: application/json
Authorization: Bearer <token>
Content-Type: application/x-www-form-urlencoded

username=<username>&password=<password>&email=<email>
--------------------------------------------------------------
HTTP/1.1 201 Created
Date: Fri, 2 Feb 2018 17:26:00 PST
Server: Nginx
Content-Language: en-US
Content-Type: application/json; charset=utf-8

{
    "id": <user id>,
    "username": <username>,
    "email": <email>,
}
```

### 4xx: client errors

+ **400 Bad request**: The server cannot process the request due to a client error (i.e malformed request syntax, size too large, general logical error...).
+ **401 Unauthorized**: Authentication failed, expired, or not yet provided. This is used when the server cannot identify the user.
+ **403 Forbidden**: The server is refusing to serve the user. This is generally used when the server knows who the user is, but the user does not have the required permission to access the resource, or due to some other logical constraints.
+ **404 Not found**: The resource does not exist
+ **405 Method not allowed**: The HTTP method used in the request was not supported for this resource. For example, you can only create (POST), access (GET) or delete (DELETE) a friend relationship, not replacing (PUT) or updating (PATCH) it.

_Examples of **4xx** responses:_
```
GET /users/@me
Host: https://api.theitfox.com
Accept-Language: en-US
Accept: application/json
Authorization: Bearer <token>
Content-Type: application/x-www-form-urlencoded
--------------------------------------------------------------
HTTP/1.1 401 Unauthorized
Date: Fri, 2 Feb 2018 17:26:00 PST
Server: Nginx
Content-Language: en-US
Content-Type: application/json; charset=utf-8

{
    "code": "UNAUTHENTICATED",
    "message": "Session expired"
}
```
```
GET /cars/2018
Host: https://api.theitfox.com
Accept-Language: en-US
Accept: application/json
Authorization: Bearer <token>
Content-Type: application/x-www-form-urlencoded
--------------------------------------------------------------
HTTP/1.1 404 Not found
Date: Fri, 2 Feb 2018 17:26:00 PST
Server: Nginx
Content-Language: en-US
Content-Type: application/json; charset=utf-8

{
    "code": "NOT FOUND",
    "message": "Resource not found"
}
```

## JSON response format convention

There are several different JSON formats to represent the response data

### The naive style

In the naive style, JSON object and JSON array are utilized directly.

+ **Requesting a single resource**
    ```
    GET /tracks/1
    Host: https://api.theitfox.com
    Accept-Language: en-US
    Accept: application/json
    Content-Type: application/x-www-form-urlencoded
    --------------------------------------------------------------
    HTTP/1.1 200 OK
    Date: Fri, 2 Feb 2018 17:26:00 PST
    Server: Nginx
    Content-Language: en-US
    Content-Type: application/json; charset=utf-8

    {
        "id": 1,
        "title": "Happy new year",
        "artist": "ABBA"
    }
    ```
+ **Requesting a collection**
    ```
    GET /tracks
    Host: https://api.theitfox.com
    Accept-Language: en-US
    Accept: application/json
    Content-Type: application/x-www-form-urlencoded
    --------------------------------------------------------------
    HTTP/1.1 200 OK
    Date: Fri, 2 Feb 2018 17:26:00 PST
    Server: Nginx
    Content-Language: en-US
    Content-Type: application/json; charset=utf-8

    [
        {
            "id": 1,
            "title": "Dancing Queen",
            "artist": "ABBA"
        },
        {
            "id": 2,
            "title": "SOS",
            "artist": "ABBA"
        },
        {
            "id": 3,
            "title": "Mamma Mia",
            "artist": "ABBA"
        }
    ]
    ```

The advantages of this approach are:
+ The JSON is small, reducing the bandwidth
+ The approach is straight forward, and easy to understand

The disadvantage of this approach is that this doesn't contain metadata, including pagination for collection resources. To overcome this problems, some developers include these metadata as response header fields, for example:
```
GET /tracks
Host: https://api.theitfox.com
Accept-Language: en-US
Accept: application/json
Content-Type: application/x-www-form-urlencoded

page_number=1&page_size=20
--------------------------------------------------------------
HTTP/1.1 200 OK
Date: Fri, 2 Feb 2018 17:26:00 PST
Server: Nginx
Content-Language: en-US
Content-Type: application/json; charset=utf-8
X-Page-Number: 1
X-Page-Size: 10
X-Total-Size: 3

[
    {
        "id": 1,
        "title": "Dancing Queen",
        "artist": "ABBA"
    },
    {
        "id": 2,
        "title": "SOS",
        "artist": "ABBA"
    },
    {
        "id": 3,
        "title": "Mamma Mia",
        "artist": "ABBA"
    }
]
```

However, since [RFC 6648](https://tools.ietf.org/html/rfc6648), custom header fields are officially deprecated.

### The enveloped style

The enveloped style solves the pagination problem with collection resources by wrapping the collection inside a named element.
```
GET /tracks
Host: https://api.theitfox.com
Accept-Language: en-US
Accept: application/json
Content-Type: application/x-www-form-urlencoded

page_number=1&page_size=20
--------------------------------------------------------------
HTTP/1.1 200 OK
Date: Fri, 2 Feb 2018 17:26:00 PST
Server: Nginx
Content-Language: en-US
Content-Type: application/json; charset=utf-8

{
    "records": [
        {
            "id": 1,
            "title": "Dancing Queen",
            "artist": "ABBA"
        },
        {
            "id": 2,
            "title": "SOS",
            "artist": "ABBA"
        },
        {
            "id": 3,
            "title": "Mamma Mia",
            "artist": "ABBA"
        }
    ],
    "_metadata": {
        "page_number": 1,
        "page_size": 20,
        "total_size": 3
    }
}

```

This approach is generally good for most REST applications.

### The JSON API style

The JSON API uses the enveloped style for both single resources and collection resources format, since metadata can in some cases be useful to single resources as well. It is a specification created to standardize data communication with JSON. The full specification can be read at [jsonapi.org](http://jsonapi.org/format/).

Here's an example of a JSON API response.
```
{  
    "links":{  
        "self":"http://api.theitfox.com/articles",
        "next":"http://api.theitfox.com/articles?page[offset]=2",
        "last":"http://api.theitfox.com/articles?page[offset]=10"
    },
    "data":[  
        {  
            "type":"articles",
            "id":"1",
            "attributes":{  
                "title":"JSON API paints my bikeshed!"
            },
            "relationships":{  
                "author":{  
                    "links":{  
                        "self":"http://api.theitfox.com/articles/1/relationships/author",
                        "related":"http://api.theitfox.com/articles/1/author"
                    },
                    "data":{  
                        "type":"people",
                        "id":"9"
                    }
                },
                "comments":{  
                    "links":{  
                        "self":"http://api.theitfox.com/articles/1/relationships/comments",
                        "related":"http://api.theitfox.com/articles/1/comments"
                    },
                    "data":[  
                        {  
                            "type":"comments",
                            "id":"5"
                        },
                        {  
                            "type":"comments",
                            "id":"12"
                        }
                    ]
                }
            },
            "links":{  
                "self":"http://api.theitfox.com/articles/1"
            }
        }
    ],
    "included":[  
        {  
            "type":"people",
            "id":"9",
            "attributes":{  
                "first-name":"Dan",
                "last-name":"Gebhardt",
                "twitter":"dgeb"
            },
            "links":{  
                "self":"http://api.theitfox.com/people/9"
            }
        },
        {  
            "type":"comments",
            "id":"5",
            "attributes":{  
                "body":"First!"
            },
            "relationships":{  
                "author":{  
                    "data":{  
                        "type":"people",
                        "id":"2"
                    }
                }
            },
            "links":{  
                "self":"http://api.theitfox.com/comments/5"
            }
        },
        {  
            "type":"comments",
            "id":"12",
            "attributes":{  
                "body":"I like XML better"
            },
            "relationships":{  
                "author":{  
                    "data":{  
                        "type":"people",
                        "id":"9"
                    }
                }
            },
            "links":{  
                "self":"http://api.theitfox.com/comments/12"
            }
        }
    ]
}
```