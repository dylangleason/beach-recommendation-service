# Table of Contents

1.  [User Stories](#user-stories)
2.  [Technical Requirements](#technical-requirements)
    1.  [Domain Model](#domain-model)
    2.  [Server Architecture](#server-architecture)
    3.  [REST API](#rest-api)
        1.  [View a List of Beaches](#view-a-list-of-beaches)
        2.  [View a Beach](#view-a-beach)
        3.  [Leave a Beach Rating](#leave-a-beach-rating)
3.  [Possible Improvements](#possible-improvements)

# User Stories

As a user, I would like to:

-   See a scrollable list of beaches, sorted by rating.
-   Click on each beach to view more details, where I should be able to see a picture, aggregate rating and comments about the beach.
-   Rate (using a 5-star system) and comment on a beach.

# Technical Requirements

Implement a web service that comprises the beach recommendation system and exposes a REST API for viewing information on beaches, their recommendations and comments, as well as allowing users to create recommendations and comments on those beaches.

## Domain Model

Our domain model deals with beaches and recommendations associated with those beaches.

-   **Beach:** A beach represents a beach listing in our system that we can view, rate and comment on.
-   **Rating:** A rating represents a rating and comment on a particular beach, associated with a particular User.

In our domain model, the beach and ratings represent a single bounded context. The User exists in their own bounded context, perhaps implemented as part of a User Authentication service. So, we will need a way to globally identify the user and associate it with a rating in the Beach Recommendation system.

## Server Architecture

Our web service will be implemented in Python and uses a basic layered architecture. The four components of our layered architecture are:

1.  _Transport layer_: modules for handling web API requests, per the data contract laid out in our API specification. For our project we will use the Flask micro-framework in Python to handle the machinery of servicing web requests.
2.  _Business layer_: modules that are responsible for executing business logic based on the type of API request we're handling, e.g. for viewing and sorting listings, or creating new recommendations associated with a a particular beach. For our project, the business logic will be written in Python.
3.  _Persistence layer_: modules that wrap access to and interaction with the database layer. For our project, we'll use a standard ORM, such as SQLAlchemy.
4.  _Database layer_: Where the data is stored to and retrieved from. For this project we will use a relational database PostgreSQL.

We are opting to use Python for this project because it has great third-party library support, the tooling is mature and the language itself is easy to learn and highly expressive. Dynamic typing will help us get the project off the ground quickly, but type annotations and static analysis tools like `mypy` can be integrated as the codebase matures.

For the database, we chose to go with PostgreSQL due to familiarity and backend programmer's strengths in relational database design versus a non-relational storage model. However, there is a tradeoff: non-relational storage models like a document-based system might provide us with more flexibility for an immature project like this, especially if the domain model is subject to change. More upfront design work will be required to ensure extensibility, performance and maintainability with a relational data store.

## REST API

Based on the [User Stories](#user-stories) described earlier, we need to support the following basic functionality:

-   Viewing a list of beaches, sorted by rating
-   Viewing a single beach with details, including their individual ratings & comments
-   Creating a new recommendation for a particular beach.

For our REST API, I have chosen to use the [jsonapi](https://jsonapi.org/) specification since it does a lot of the upfront API design work for us and is very flexible.

### View a List of Beaches

-   **URL:** `/beaches`
-   **Verb:** `GET`

####  Request Parameters

The following query parameters are supported for our API:
    
-   **`sort`:** specify sorting criteria, specifically by rating
-   **`page`:** support for pagination, to support infinite scroll

#### Response Codes

-   **200:** The resource was found
-   **400:** The request syntax was malformed

#### Example Request

`/beaches?sort=rating&page[number]=1&page[size]=50`

#### Example Response

```json
{
  "data": [
    {
      "id": "f5182ab3-69ad-4292-afc8-162f775043a4",
      "type": "beaches",
      "attributes": {
        "name": "Cannon Beach",
        "description": "Located in the city of Cannon Beach, OR, this beaach is home to the famous Haystack Rocks and spans 4 miles.",
        "location": {
          "latitude": 45.889167,
          "longitude": -123.960833
        },
        "average_rating": 4.5,
        "total_ratings": 100,
        "image_url": "https://some.image.host/some_image_id"
      },
      "links": {
        "self": "/beaches/f5182ab3-69ad-4292-afc8-162f775043a4"
      }
    },
    {
      "id": "0942ed0e-b395-40ad-8fc8-aaa6fbd10d27",
      "type": "beaches",
      "attributes": {
        "name": "Rockaway Beach",
        "description": "Rockaway Beach is a popular beach destination. Tillamook State Forest is a popular attraction here.",
        "location": {
          "latitude": 45.613333, 
          "longitude": -123.942778
        },
        "average_rating": 4.0,
        "total_ratings": 125,
        "image_url": "https://some.image.host/another_image_id"
      },
      "links": {
        "self": "/beaches/0942ed0e-b395-40ad-8fc8-aaa6fbd10d27"
      }
    },
    ...
  ],
  "meta": {
    "page": 1,
    "size": 50
  }
}
```

### View a Beach

-   **URL:** `/beaches/:beach_id`
-   **Verb:** `GET`

#### Request Parameters

The following query parameters are supported for querying including the related ratings resources for the requested beach resource:
    
-   **`included`:** Request the included relationships.

#### Response Codes

-   **200:** The resource was found
-   **404:** The resource was not found
-   **400:** The request syntax is malformed

#### Example Request

`/beaches/0942ed0e-b395-40ad-8fc8-aaa6fbd10d27?included=ratings`

#### Example Response

```json
{
  "data": {
    "id": "0942ed0e-b395-40ad-8fc8-aaa6fbd10d27",
    "type": "beaches",
    "attributes": {
      "name": "Rockaway Beach",
      "description": "Rockaway Beach is a popular beach destination. Tillamook State Forest is a popular attraction here.",
      "location": {
        "latitude": 45.613333, 
        "longitude": -123.942778
      },
      "average_rating": 4.0,
      "total_ratings": 125,
      "image_url": "https://some.image.host/another_image_id"
    },
    "links": {
      "self": "/beaches/0942ed0e-b395-40ad-8fc8-aaa6fbd10d27"
    },
    "relationships": {
      "ratings": {
        "links": {
          "related": "/beaches/0942ed0e-b395-40ad-8fc8-aaa6fbd10d27/ratings"
        }
      }
    }
  },
  "included": [
    {
      "id": "20d4cc0d-ccbb-43fb-9a2a-4f40df78c64d",
      "type": "ratings",
      "attributes": {
        "user_id": "003561e4-7cf8-4711-b571-4b18b250e9ae",
        "rating": 4.0,
        "comment": "I really liked this beach a lot. It was a bit crowded, though. Get there early in the day, if possible."
      }
    },
    ...
  ]
}
```
    
_Note_: if requesting the related `ratings` resource, the response payload would be the same as the `included` resources above but would appear under the `data` property.

### Leave a Beach Rating

-   **URL:** `/rating`
-   **Verb:** `POST`

#### Response Codes

-   **201:** Created the rating resource
-   **400:** The request syntax was malformed
-   **409:** The request could not be completed, likely because a beach relationship was not specified
-   **422:** The request cannot be processed, for example if the rating goes beyond our 1-5 star rating system.

#### Example Request Body

```json
{
  "data": {
    "type": "ratings",
    "attributes": {
      "user_id": "003561e4-7cf8-4711-b571-4b18b250e9ae",
      "rating": 4.5,
      "comment": "I really liked this beach a lot"
    },
    "relationships": {
      "beach": {
        "data": {
          "type": "beaches",
          "id": "3554e8bb-fcca-480c-a676-e7835b704338"
        }
      }
    }
  }
}
```

#### Example Response

```json
{
  "data": {
    "id": "eefda97c-a2c8-40be-ad5e-8bc8328ba607",
    "type": "ratings",
    "attributes": {
      "user_id": "003561e4-7cf8-4711-b571-4b18b250e9ae",
      "rating": 4.5,
      "comment": "I really liked this beach a lot"
    },
    "relationships": {
      "beach": {
        "data": {
          "type": "beaches",
          "id": "3554e8bb-fcca-480c-a676-e7835b704338"
        }
      }
    }
  }
}
```

# Possible Improvements

-   For the API, it would be nice to support different sorting criteria, as well as things like filtering and sparse fields (i.e., allowing the API client to define only the attributes they want to include for a resource).
-   For the underlying Data Model, it might be better to use a generalized "Listing" abstraction that would allow us to capture information on any geographic-based attraction to which users can rate.
-   For the Persistence and Database layers, a Document based object model and non-relational data store, respectively, might simplify and make for a more scalable architecture, especially when aggregating data. I chose PostgreSQL due to my familiarity with the technology.

