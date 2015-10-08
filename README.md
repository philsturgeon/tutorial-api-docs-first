# HTTP Documentation with API Blueprint

When planning my talk and book on REST/HTTP API development, I ended up mentioning APIs towards the end, and flippantly said "Oh an API Blueprint is pretty good probably just use that." This is something I'd love to fix with a time machine, as these days I spec out an API in [API Blueprint](https://apiblueprint.org) before I get anywhere near the code. 

Documentation first gives you and the team a chance to play around with what fields are coming and going, and you can even dump this documentation file into a collaborative editing tool like Dropbox Notes or have a Google Hangouts with a tmate session. When the API team and the clients (web, iOS, Android, etc.) have reached a good level of agreement, the contracts can be "locked-down" and put into Git, meaning changes are tracked and blame can be used for anyone who f**ks up the contract.

If your API documentation is then [tested with Dredd](https://philsturgeon.uk/api/2015/01/28/dredd-api-testing-documentation/), you know your contracts are holding up as expected over time.

## Getting Started 
 
[Getting Started with API Blueprint](https://apiblueprint.org/#get-started) is fairly vague and open-ended. They teach you about Drafter which early on is totally irrelevant other than for syntax checking, which the Apiary live editor does a better job of doing. Getting Started is a lot different to what they have documented, so here is how I go about it.

We'll assume for now that you want to use their hosted [Apiary](https://apiary.io/) platform, and I'll follow up with how to use alternatives in later articles.

## 1.) Setting up the API with Apiary

Make a new directory to work in, and in that shove a new `apiary.apib` file. We'll put in the most basic API Blueprint syntax possible:

~~~ md
FORMAT: 1A
# My API

## GET /places

+ Response 200 (text/plain)

        Hello World!
~~~

_**Note:** Normally I would use `~~~` to wrap the response body and only one level of indentation, but Markdown parsers [freak out](http://bit.ly/1OmPR9X) if you try to put fenced code blocks inside fenced code blocks. You can use 8 spaces, 2 tabs, or wrap it in `~~~` yourself._

Now, to get this file to Apiary, we have to make a GitHub repo. If you're cool and use the [hub CLI tool](https://github.com/github/hub) then just do this:

~~~ sh
$ hub init
$ hub create
$ git add apiary.apib
$ git commit -m "Added basic API Blueprint"
$ git push origin master
~~~

Now this API Blueprint is in a GitHub repo, we can set up Apiary.

Firstly, click the dropdown next to where it has the default "Notes API" and select "Create New API".

![image of create](article_images/2015-10-08-http-documentation-with-api-blueprint/create-new-api.png)

click settings and scroll down to connect GitHub

![image of link github](article_images/2015-10-08-http-documentation-with-api-blueprint/link-github-repo.png)

This will automatically set up a Service Hook on Github, so you don't have to.

find the new repo we just made and link it up

![image of clicking the right repo](article_images/2015-10-08-http-documentation-with-api-blueprint/select-repo.png)

"Load blueprint from GitHub"

![image of clicking load blueprint from GitHub](article_images/2015-10-08-http-documentation-with-api-blueprint/load-blueprint-from-github.png)

Now whenever we push a change to our `apiary.apib` file, it'll update Apiary. If we update Apiary, it'll save the change to apiary.apib via a new commit.

## Formatted Responses

Having a Hello World is a nice start, but we will also need to learn how to work with data formats like JSON. The easiest way to output JSON in the response documentation is to change the response mime-type, and paste some example JSON in.

~~~ md
FORMAT: 1A
# My API

## GET /places

+ Response 200 (application/json)

        {
            "places" : [{
                "id" : "fRge5",
                "name" : "Battery Harris"
            }]
        }
~~~

Take a look at that in API Blueprint. Go ahead and use their editor with the live preview as we go along to save you committing stuff. You should see it in their output:

![show json example of places array](article_images/2015-10-08-http-documentation-with-api-blueprint/basic-json-output.png)

Next, lets add another endpoint. We have `GET /places`, so lets add `GET /places/{id}` to get just a single place.
 
~~~ md
FORMAT: 1A
# My API

## GET /places

+ Response 200 (application/json)

        {
            "places" : [{
                "id" : "fRge5",
                "name" : "Battery Harris"
            }]
        }


## GET /places/{id}

+ Parameters
  + id: `fRge5` - The unique ID of the place.

+ Response 200 (application/json)

        {
            "places" : {
                "id" : "fRge5",
                "name" : "Battery Harris"
            }
        }

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

![show mson output](article_images/2015-10-08-http-documentation-with-api-blueprint/mson-json-output.png)

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

Now we've got descriptions on our fields, and added a bunch more fields. The coolest is the status field, setting a bunch of string values in an enum, with all the options explained.

![show advanced mson](article_images/2015-10-08-http-documentation-with-api-blueprint/mson-advanced-output.png)

You can learn more about the "Attributes" keyword in the [API Blueprint Spec](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md#def-attributes-section), and by reading more about the [MSON syntax](https://github.com/apiaryio/mson/blob/master/MSON%20Specification.md) specifically.

## Requests

APIs are more than just getting data, so time to look at a request.

I've made an updated full example, which uses Resource Groups to define a URL and then has h3's with the HTTP method to define what happens in the actions.

~~~ md
FORMAT: 1A
# My API

Bit of a description or intro, and an introduction to how to OAuth, etc.

# Group Places

## Places Collection [/places]

### GET

+ Response 200 (application/json)
    + Attributes
        - places (array[Place])

## Place Resource [/places/{id}]

+ Parameters
  + id: `fRge5` - The unique ID of the place.

### GET

+ Response 200 (application/json)
    + Attributes
        - places (Place)

### PUT

+ Request (application/json)
    + Attributes
        - places (Place)

+ Response 200 (application/json)
    + Attributes
        - places (Place)

### DELETE

+ Response 204


# Data Structures

## Place (object)

- id: `fRge5` (string, required) - The unique ID of the place.
- name: `Battery Harris` (string, required) - Name of the place.
- lat: `40.712017` (number, required) - Latitude as a decimal.
- lon: `-73.950995` (number, required) - Longitude as a decimal.
- status (enum[string])
  - pending - They haven't finished their public profile or whatever.
  - active - Good as gold.
  - closed - This place doesn't exist.
- created_at: `2015-01-07T14:03:43Z` (string, required) - ISO8601 date and time of when the rider was created.
~~~

Looking good huh? We've got `PUT` and `DELETE` in there too now, showing off a `204` no less. 

Well, one thing left: Using `POST` to create a resource on the collection.

The tough part here is that most `POST` requests involve sending partial representations, and letting the server fill in the gaps. To do that, our `Place` data structure falls over a little bit. 

So, we can split them up a bit. Take a look at this full example to see what I changed:

~~~ md
FORMAT: 1A
# My API

Bit of a description or intro, and an introduction to how to OAuth, etc.

# Group Places

## Places Collection [/places]

### GET

+ Response 200 (application/json)
    + Attributes
        - places (array[Place Full])

### POST

+ Request (application/json)
    + Attributes
        - places (Place Create)

+ Response 200 (application/json)
    + Attributes
        - places (Place Full)

## Place Resource [/places/{id}]

+ Parameters
  + id: `fRge5` - The unique ID of the place.

### GET

+ Response 200 (application/json)
    + Attributes
        - places (Place Full)

### PUT

+ Request (application/json)
    + Attributes
        - places (Place Full)

+ Response 200 (application/json)
    + Attributes
        - places (Place Full)

### DELETE

+ Response 204


# Data Structures

## Place Create (object)

- name: `Battery Harris` (string, required) - Name of the place.
- lat: `40.712017` (number, required) - Latitude as a decimal.
- lon: `-73.950995` (number, required) - Longitude as a decimal.
- status (enum[string])
  - pending - They haven't finished their public profile or whatever.
  - active - Good as gold.
  - closed - This place doesn't exist.

## Place Full (object)

- id: `fRge5` (string, required) - The unique ID of the place.
- Include Place Create
- created_at: `2015-01-07T14:03:43Z` (string, required) - ISO8601 date and time of when the rider was created.
~~~

We now have a `Place Create` object which has the fields required for creating, and the `Place Full` object, which includes `Place Create` for that full effect along with two extra fields that'll only show up afterwards.

## Summary

There's a lot more to Apiary and MSON than this, and I want to show you how to use this for mock servers next, but having this stuff built out is a great way to get a team to agree on the contracts for requests and responses before people start messing around actually building stuff.
