# Tutorial: API Documentation First

https://apiblueprint.org/#get-started

Make a new `apiary.apib` file with the most basic API Blueprint syntax possible:

~~~ md
FORMAT: 1A
# My API

## GET /places

+ Response 200 (text/plain)

  ~~~
  Hello World!
  ~~~  
~~~

hub init
hub create

git add apiary.apib
git push origin master

Now this is API Blueprint is GitHub, so we can set up Apiary.


Make a new API

![image of create](go find it)

click settings and scroll down to connect GitHub

![image of link github](go find it)

This will automatically set up a Service Hook on Github, so you don't have to.

find the new repo we just made and link it up

![image of clicking the right repo]()

"Load blueprint from GitHub"

![image of clicking load blueprint from GitHub]()

Now whenever we push a change to our `apiary.apib` file, it'll update Apiary. If we update Apiary, it'll save the change to apiary.apib via a new commit.

## Formatted Responses

Having a Hello World is a nice start, but we will also need to learn how to work with data formats like JSON. The easiest way to output JSON in the response documentation is to change the response mime-type, and paste some example JSON into the code fence:

~~~ md
FORMAT: 1A
# My API

## GET /places

+ Response 200 (application/json)

  ~~~
  {
    "places" : [{
      "id" : "fRge5",
      "name" : "Battery Harris"
    }]
  }
  ~~~
~~~

Take a look at that in API Blueprint. Go ahead and use their editor with the live preview as we go along to save you committing stuff. You should see it in their output:

![show json example of places array]

Next, lets add another endpoint. We have `GET /places`, so lets add `GET /places/{id}` to get just a single place.
 
~~~ md
FORMAT: 1A
# My API

## GET /places

+ Response 200 (application/json)

  ~~~
  {
    "places" : [{
      "id" : "fRge5",
      "name" : "Battery Harris"
    }]
  }
  ~~~

## GET /places/{id}

+ Parameters
  + id: `fRge5` - The unique ID of the place.

+ Response 200 (application/json)

  ~~~
  {
    "places" : {
      "id" : "fRge5",
      "name" : "Battery Harris"
    }
  }
  ~~~
~~~

At a very simple level, this does work, but it is not very DRY. We are repeating the data in our examples over and over again, even though they share a really similar structure.

Let's leverage [MSON ("Markdown Syntax for Object Notation")](https://github.com/apiaryio/mson) to make our lives easier:

~~~ md
FORMAT: 1A
# My API

## GET /places

+ Response 200 (application/json)
    + Attributes
        - places (array[Place])
    
## GET /places/{id}

+ Parameters
  + id: `fRge5` - The unique ID of the place.

+ Response 200 (application/json)
    + Attributes
        - places (Place)

# Data Structures

## Place (object)

- id: `fRge5` (string)
- name: `Battery Harris` (string)
~~~

This has got a little large, so let's take a look at this bit by bit.

~~~ md
## GET /places

+ Response 200 (application/json)
    + Attributes
        - places (array[Place])
~~~

Here we have our `GET /places` request. It's got a JSON response, with attributes. Those attributes contain a top-level array of places, which is an array of `Place` data structures.

Yep, we've defined data structures! Where? This bit:

~~~ md
# Data Structures

## Place (object)

- id: `fRge5` (string)
- name: `Battery Harris` (string)
~~~

Here we have defined the name of two fields, given them default values (an optional but wise move) then given them the type. 

Because both `GET /places` and `GET /places/{id}` share this data structure (even though one is an array and the other is an object), we can expand on the fields and really improve the place documentation. 

~~~ md
# Data Structures

## Place (object)

- id: `fRge5` (string, required) - The unique ID of the place.
- name: `Battery Harris` (string, required) - Name of the place.
- lat: `40.712017` (number, required) - Latitude as a decimal.
- lon: `-73.950995` (number, required) - Longitude as a decimal.
- status (enum[string])
  - pending - They haven't finished their public profile or whatever
  - active - Good as gold
  - closed - This place doesn't exist
- created_at: `2015-01-07T14:03:43Z` (string, required) - ISO8601 date and time of when the rider was created.
~~~

Now we've got descriptions on our fields, and added a bunch more. Most of them should be self-explanitory, and a status field set as a bunch of strings with all the options explained.

You can learn more about the "Attributes" keyword in the [API Blueprint Spec](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md#def-attributes-section), and by reading more about the [MSON syntax](https://github.com/apiaryio/mson/blob/master/MSON%20Specification.md) specifically.

## Requests
