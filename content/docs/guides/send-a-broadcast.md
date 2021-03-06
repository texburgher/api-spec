---
title: "Broadcasts and the API"
---

# Publishing Broadcasts via the API

<div class="alert alert-success alert-block">
To start sending broadcasts directly from the API, you'll need an access token. If you're not familiar with how to get an access token, <a href='/docs/guides/create-an-app/'>check out our guide on creating an app.</a>
</div>

Broadcast Channels are designed to carry low-volume, high-value updates of interest to users. We call the actual updates themselves Broadcast Messages (or sometimes just "broadcasts"). Because broadcasts are built on top of the existing App.net [Channel](/docs/resources/channel/) and [Message](/docs/resources/message/) APIs, it's helpful to be familiar with them, but for simple tasks, there's not much you need to know.

Just a reminder: while you can do all of this using our API, you don't have to. We have [tools for publishers](https://broadcast.app.net/manage/) to help you quickly get started pulling content in from elsewhere on the web, and you can send them manually via the web or the App.net iOS and Android apps. To get started, **we recommend that you create and set up your broadcast channel with [our web publisher tools](https://broadcast.app.net/manage/)** and only use the API to send broadcasts via the channel you created.

* TOC
{:toc}

## Send a Broadcast

There are a number of parts to each broadcast message. To illustrate, here's a screenshot of a broadcast in the App.net iOS app:

![Anatomy of a broadcast](https://files.app.net/rw8fPgku.png)

The important parts are the **Headline**, **Photo**, **Text** and **Read More Link**. Everything but the headline is optional, but we recommend that your broadcasts contain a compelling image and some context through a summary text body and a link to content.

The easiest way to send a broadcast with the API is via the [ADNPy](http://adnpy.readthedocs.org) Python module, the [adn](https://github.com/adn-rb/adn) Ruby gem, the [appnet.js](https://github.com/duerig/appnet.js) library for Node.js, or the [AppDotNetPHP](https://github.com/jdolitsky/AppDotNetPHP) library. You can also use the HTTP API with your own client, if you'd like.

Starting with Python, here's an example of using the ADNPy `BroadcastMessageBuilder` recipe. If you're interested in the specific API calls the library is making behind the scenes, [look at the source](https://github.com/appdotnet/ADNpy/blob/master/adnpy/recipes/broadcast.py).

Once you've installed the ADNPy library, it's as simple as:

~~~ python
import adnpy

# Set the default access token for API calls.
adnpy.api.add_authorization_token('your access token here')

# Send a broadcast with the BroadcastMessageBuilder recipe.
builder = adnpy.recipes.BroadcastMessageBuilder(adnpy.api)
builder.channel_id = 24204  # Get this channel ID from the web publisher tools
builder.headline = 'Hello World!'
builder.text = 'Sending this from [ADNPy](https://github.com/appdotnet/ADNPy) was easy!'
builder.parse_markdown_links = True
builder.read_more_link = 'http://adnpy.readthedocs.org/'
builder.send()
~~~

You can easily attach a photo:

~~~ python
builder.photo = '/home/paul/celeryman.gif'
~~~

Or with the [adn](https://github.com/adn-rb/adn) Ruby gem:

~~~ ruby
require 'adn'

ADN.token = 'your access token here'

builder = ADN::Recipes::BroadcastMessageBuilder.new
builder.channel_id = 24204
builder.headline = 'sending from ruby'
builder.read_more_link = 'https://app.net'
builder.photo = '/home/mthurman/oyster-smiling.png'
builder.send
~~~

If you want to send the equivalent broadcast from the API without using a library, use this curl command:

~~~ sh
curl -X POST -H "Authorization: Bearer <YOUR ACCESS TOKEN>" \
    -H "X-adn-pretty-json: 1" -H "Content-Type: application/json" \
    --data-ascii '{
      "text": "Here is a [link](http://www.google.com) for my text body.",
      "annotations": [
        {
          "value": {
            "subject": "Hello world!"
          },
          "type": "net.app.core.broadcast.message.metadata"
        },
        {
          "value": {
            "canonical_url": "http://www.example.com"
          },
          "type": "net.app.core.crosspost"
        }
      ],
      "entities": {
        "parse_markdown_links": true,
        "parse_links": true
      }
    }' \
    \
    'https://alpha-api.app.net/stream/0/channels?include_annotations=1'
~~~

## Create a Broadcast Channel

You can create broadcast channels with the API as well.

Broadcast channels are created like any other channel, with a specific type value (`net.app.core.broadcast`) and a few special annotations. For information on creating channels, see the [Channel lifecycle documentation](/docs/resources/channel/lifecycle/). Broadcast channels fully support [ACLs](/docs/resources/channel/) for private applications.

Here is an example channel body, set to be public, with multiple editors, which you can POST to us:

~~~ js
{
    "type": "net.app.core.broadcast",
    "readers": {
        "public": true
    },
    "editors": {
        "user_ids": ["@samsharp", "@sfborscht"],
    },
    "annotations": [{
        "type": "net.app.core.broadcast.metadata",
        "value": {
            "title": "SF Borscht Truck Updates",
            "description": "Get a notification when the SF Borscht Food Truck will be on the streets!"
        }
    }]
}
~~~

Here's a sample curl command that'll create a channel:

~~~ sh
curl -X POST -H "Authorization: Bearer <YOUR ACCESS TOKEN>" \
    -H "X-adn-pretty-json: 1" -H "Content-Type: application/json" \
    --data-ascii '{
        "type": "net.app.core.broadcast",
        "readers": {
            "public": true
        },
        "editors": {
            "user_ids": ["@samsharp", "@sfborscht"]
        },
        "annotations": [{
            "type": "net.app.core.broadcast.metadata",
            "value": {
                "title": "SF Borscht Truck Updates",
                "description": "Get a notification when the SF Borscht Food Truck will be on the streets!"
            }
        }]
    }' \
    \
    'https://alpha-api.app.net/stream/0/channels?include_annotations=1'
~~~

This will return a fully-populated channel resource, including the `id` of the channel, a few auto-generated annotations, and the `fallback_url`, which is the short URL for the channel's detail page.

~~~ js
{
    "data": {
        "annotations": [
            {
                "type": "net.app.core.broadcast.metadata",
                "value": {
                    "description": "Get a notification when the SF Borscht Food Truck will be on the streets!",
                    "tags": [],
                    "title": "SF Borscht Truck Updates"
                }
            },
            {
                "type": "net.app.core.broadcast.freq",
                "value": {
                    "avg_freq": "less than 1 per week",
                    "intensity": 0.1
                }
            },
            {
                "type": "net.app.core.fallback_url",
                "value": {
                    "url": "https://app.net/c/2rds"
                }
            }
        ],
        "counts": {
            "messages": 0,
            "subscribers": 1
        },
        "editors": {
            "any_user": false,
            "immutable": false,
            "public": false,
            "user_ids": [
                "190151",
                "190195"
            ],
            "you": true
        },
        "has_unread": false,
        "id": "37332",
        "is_inactive": false,
        "owner": {
            ...
        },
        "readers": {
            "any_user": false,
            "immutable": false,
            "public": true,
            "user_ids": [],
            "you": true
        },
        "type": "net.app.core.broadcast",
        "writers": {
            "any_user": false,
            "immutable": false,
            "public": false,
            "user_ids": [],
            "you": true
        },
        "you_can_edit": true,
        "you_muted": false,
        "you_subscribed": true
    },
    "meta": {
        "code": 200
    }
}
~~~
