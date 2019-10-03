---
title: OneDB API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: curl

toc_footers:

---

# Introduction

```shell
https://api.onedb.xyz
```

Welcome to OneDB!

Today, when you're building an app, you use one database for storing records, one for searching, and one for storing media files (images, attachments). As your app scales, you're always optimizing query performance or start considering switching to different database to support the scale. This leads to dealing with migrations, managing multiple databases for data consistency, security, backups, and syncing. All of this takes time away that could be spent building features and serving your customers.

This is why we're building OneDB, the modern database for the cloud. OneDB doesn't lock you in and promotes data portability through standard data format (JSON) and REST APIs to manage all of your data needs. Our REST APIs makes it easy to move your data from OneDB to serve your special needs, if you ever need to.

Easy to get started, scales seamlessly, and powerful with built-in search, validations, data security, and fully managed database.

This API reference provides information on available endpoints and how to interact with it.

The OneDB API is organized around [REST](http://en.wikipedia.org/wiki/Representational_State_Transfer). Our API has predictable, resource-oriented URLs, and uses HTTP response codes to indicate API errors. All requests should be over SSL. All request and response bodies, including errors are encoded in JSON.

# API Keys

OneDB authenticates your API requests using your account’s API keys. If you do not include your key when making an API request, or use one that is incorrect or outdated, OneDB returns an error.

The API key should be kept confidential and only stored on your own servers. Your account’s API key can perform any API request to OneDB without restriction.

Each account has a total of two keys: a key pair for test mode and live mode.

# Authentication

> Example Request

```shell
$ curl https://api.onedb.xyz/account \
  -H "Authorization: your_api_key"
```

Authentication is done via the API key which you can find in your accounts settings endpoint. Your API keys carry many privileges, so be sure to keep them secure! Do not share your secret API keys in publicly accessible areas such as GitHub, client-side code, and so forth.

Test mode secret keys have the prefix sk_test_ and live mode secret keys have the prefix sk_live_. Alternatively, you can use restricted API keys for granular permissions.

Authentication to the API is performed via [HTTP Basic Auth](http://en.wikipedia.org/wiki/Basic_access_authentication). Provide your API key as the basic auth username value. You do not need to provide a password.

If you need to authenticate via bearer auth (e.g., for a cross-origin request), use `-H "Authorization: Bearer base64(ai_test_app_id:sk_test_8fD39MqLyjWBarjtP1zdp7bc)"`

```shell
curl -X GET https://api.onedb.xyz/accounts/193954b0-0ae8-5db7-b640-8110afe56438 \
  -H "Authorization: Bearer base64(ai_test_app_id:sk_test_8fD39MqLyjWBarjtP1zdp7bc)"
```

All API requests must be made over HTTPS. Calls made over plain HTTP will fail. API requests without authentication will also fail.

# Errors

Our API returns standard HTTP success or error status codes. For errors, we will also include extra information about what went wrong encoded in the response as JSON. The various HTTP status codes we might return are listed below.

In general: Codes in the 2xx range indicate success. Codes in the 4xx range indicate an error that failed given the information provided (e.g., a required parameter was omitted, a charge failed, etc.). Codes in the 5xx range indicate an error with OneDB's servers (these are rare).

## HTTP Status codes

Code | Title | Description
---- | ----- | -----------
200 | OK | Everything worked as expected.
400 | Bad Request | The request was unacceptable, often due to missing a required parameter.
401 | Unauthorized | No valid API key provided.
402 | Request Failed | The parameters were valid but the request failed.
404 | Not Found | The requested resource doesn't exist.
409 | Conflict | The request conflicts with another request (perhaps due to using the same idempotent key).
429 | Too Many Request | Too many requests hit the API too quickly. We recommend an exponential backoff of your requests.
500,502,503,504 | Server Errors | Something went wrong on OneDB's end. (These are rare.)

## Error types

> Example error response.

```json
{
  "error": {
    "type": "invalid_request",
    "message": "name is required"
  }
}
```

Type | Description
---- | -----------
api_connection | Failure to connect to OneDB's API.
api_error | API errors cover any other type of problem (e.g., a temporary problem with our servers), and are extremely uncommon.
authentication_error | Failure to properly authenticate yourself in the request.
invalid_request | Invalid request errors arise when your request has invalid parameters.
rate_limit_exceeded | Too many requests hit the API too quickly.
validation_error | Unable to validate POST/PUT
not_found | Resource was not found.

# Rate limiting

> Example rate limit error response.

```json
{
  "error": [
    {
      "type": "rate_limit_exceeded",
      "message": "Too many requests"
    }
  ]
}
```

The OneDB API is rate limited to prevent abuse that would degrade our ability to maintain consistent API performance for all users. By default, each API key or app is rate limited at 10,000 requests per hour. If your requests are being rate limited, HTTP response code 429 will be returned with an rate_limit_exceeded error.

# Accounts

Account API endpoints for sign up and getting your account statistics.

## Sign up

> Example Request

```shell
curl -X POST https://api.onedb.xyz/accounts \
  -H "Content-Type: application/json" \
   -d $'{
      "first_name": "John",
      "last_name": "Smith",
      "company": "Sunrise, Inc",
      "email": "john@sunrise.com",
      "invite_code": "xVB20ljk"
    }'
```
> Example Response

```json
{
  "id": "193954b0-0ae8-5db7-b640-8110afe56438",
  "first_name": "John",
  "last_name": "Smith",
  "company": "Sunrise, Inc",
  "email": "john@sunrise.com",
  "status": "pending"
}
```

An account code is send to the associated account. Next step: verify the account.

### HTTP Request

`POST https://api.onedb.xyz/accounts`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`first_name` *required* | string | First Name
`last_name` *required* | string | Last Name
`company` *required* | string | Name of the company
`email` *required* | string | Email address associated with the account.
`invite_code` *required* | string | Required while we're in private beta

## Activate

```shell
curl -X PUT https://api.onedb.xyz/accounts/193954b0-0ae8-5db7-b640-8110afe56438/activate \
  -H "Content-Type: application/json" \
   -d $'{
      "code": "763549",
  }'
```
> Example Response

```json
{
  "id": "193954b0-0ae8-5db7-b640-8110afe56438",
  "first_name": "John",
  "last_name": "Smith",
  "company": "Sunrise, Inc",
  "email": "john@sunrise.com",
  "status": "activated",
  "test_app_id": "ai_test_*",
  "test_app_secret": "sk_test_*",
  "live_app_id": "ai_live_*",
  "live_app_secret": "sk_live_*"
}
```

### HTTP Request

`POST https://api.onedb.xyz/accounts/:account_id/activate`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`code` *required* | string | Code sent to the account email.

Once your account is activated, you can start using the APIs using your app credentials

# Collections

The collections API lets you group Items and define the schema or structure of the custom data stored in those Items. Collections can also be think of as a table in traditional databases.

> Example Request

```shell
curl -X POST https://api.onedb.xyz/collections \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
  -H "Content-Type: application/json" \
   -d $'{
      "name": "Posts",
      "fields": [
        {
          "required": true,
          "type": "Text",
          "name": "Title",
          "validations": {
            "max_length": 128
          }
        },
        {
          "required": false,
          "type": "LongText",
          "name": "Post Body"
        },
        ...
      ]
    }'
```
> Example Response

```json
{
  "id": "6fe54303-10ce-42b5-a4b0-38bc19cf5025",
  "name": "Posts",
  "slug": "posts",
  "updated_at": "2019-10-24T19:42:38.929Z",
  "created_at": "2016-10-24T19:41:48.349Z",
  "fields": [
    {
      "required": true,
      "type": "Text",
      "name": "Title",
      "slug": "title",
      "validations": {
        "max_length": 128
      },
      "searchable": true
    },
    {
      "required": false,
      "type": "LongText",
      "slug": "post_body",
      "name": "Post Body",
      "searchable": true
    },
    ...
  ]
}
```

### HTTP Request

`POST https://api.onedb.xyz/collections`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`name` *required* | string | Name given to the collection
`searchable` *optional* | boolean | If searching/filtering by this field is enabled. If enabled, search index is created, and all Items in this collection are indexed based on their Field type.
`fields` | array| See Full Fields Section

## List Collections

> Example Request

```shell
curl -X GET https://api.onedb.xyz/collections \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
```

> Example Response

```json
[
  {
    "id": "6fe54303-10ce-42b5-a4b0-38bc19cf5025",
    "updated_at": "2019-10-24T19:42:38.929Z",
    "created_at": "2016-10-24T19:41:48.349Z",
    "name": "Posts",
    "slug": "post",
  },
  {
    "id": "8bc54303-10ce-42b5-a4b0-38bc19cf5430",
    "updated_at": "2019-10-24T19:42:00.799Z",
    "created_at": "2019-10-24T19:42:00.777Z",
    "name": "Authors",
    "slug": "author",
  }
]
```

### HTTP Request

`GET https://api.onedb.xyz/collections`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`q` *optional* | string | search query string to filter collection by names

## Get Collection with Full Schema

> Example Request

```shell
curl -X GET https://api.onedb.xyz/collections/6fe54303-10ce-42b5-a4b0-38bc19cf5025 \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
```

> Example Response

```json
{
  "id": "6fe54303-10ce-42b5-a4b0-38bc19cf5025",
  "name": "Post",
  "slug": "post",
  "fields": [
    {
      "required": true,
      "type": "Text",
      "name": "Title",
      "slug": "title",
      "validations": {
        "max_length": 128
      }
    },
    {
      "required": false,
      "type": "LongText",
      "slug": "post_body",
      "name": "Post Body"
    },
    ...
  ]
}
```

### HTTP Request

`GET https://api.onedb.xyz/collections/:collection_id`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`collection_id` *required* | string | Name given to the collection


# Fields

Fields are what define the schema of a Collection - and ultimately all your dynamic data. Every field has a Field Type describing what kind of field it is. Each field also may have a validations object which defines all of the validations which have been set for the given field. The “TYPE” column defines the data type of the validations.

## Schema

Parameter | Type | Description
--------- | ---- | -----------
`name` *required* | string | Name given to the field
`type` *required* | string |	See Field Types below
`validations` *optional* | array | See Validations Section

## Field Types

Field Type | Data Type | Description
---------- | --------- | -----------
`Bool` | boolean | yes/no switch
`Date` | date | Date field used to display any combination of month, day, year, and time
`StringSet` | array | Field containing multiple unique strings
`Link` | string | URL field where the value can be used as a link destination
`Number` | number | Field for numbers (int or float)
`Text` | string | A single line of text
`LongText` | string | Multiple lines of text
`SingleSelect` | string | Selected option name. When creating or updating records, if the choice string does not exactly match an existing option, the request will fail with an INVALID_MULTIPLE_CHOICE_OPTIONS error.
`MultipleSelect` | array of strings | Array of selected option names. When creating or updating records, if a choice string does not exactly match an existing option, the request will fail with an INVALID_MULTIPLE_CHOICE_OPTIONS error
`Attachment` | object | Each attachment object may contain the following id, url, filename, size, type. See the uploading attachments section.

## Validations
Each field also may have a validations object which defines all of the validations which have been set for the given field. The “TYPE” column defines the data type of the validations.

VALIDATION NAME	| TYPE
--------------- | ----
`MaxLength` | number
`MinLength` | number
`Minimum` | number
`Maximum` | number
`MaxSize` | number
`Options` | number
`Format` | number

# Items

The fields that define the schema for a given Item are based on the Collection that Item belongs to. Beyond the user defined fields, there are a handful of additional fields that are automatically created for all items:

FIELD	| TYPE | DESCRIPTION
----- | ---- | -----------
`id`  | string | Unique identifier for the Item
`created_at` | date | Date Item was created
`updated_at` | date | Date Item was last updated

## Get All Items For a Collection

> Example Request

```shell
curl -X GET https://api.onedb.xyz/collections/6fe54303-10ce-42b5-a4b0-38bc19cf5025/items \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
```

> Example Response

```json
{
  "items": [
    {
      "id": "829b3c8d-1fad-5fbb-8b54-e1e6528ab777",
      "updated_at": "2019-10-24T19:42:38.929Z",
      "created_at": "2016-10-24T19:41:48.349Z",
      "fields": {
        "title": "Three Years of Building Investing Products — Lessons Learned",
        "body": "<p>The current and future of investing platforms.</p>",
        "tags": ["Fin-tech", "Startups", "API"],
        "attachments": {
          "url": "https://onedb-uploads/attachments/Investing%20platform%20marketplace.pdf",
          "id": "jguTNgbqhGZG",
          "size": 1414220,
          "type": "application/pdf",
          "file_name": "Investing platform marketplace.pdf"
        },
        "author_id": "38bc19cf-10ce-42b5-a4b0-54308bc54303"
      }
    },
    {
      "id": "48f990f0-fd66-592e-9f5d-094bc2862ebd",
      "updated_at": "2019-10-24T19:42:00.799Z",
      "created_at": "2019-10-24T19:42:00.777Z",
      "fields": {
        "title": "16 Business Models for Your Next Company",
        "body": "<p>The business model is an important decision that will have a significant impact on your profitability. It can differentiate you from competitors and give you an advantage over them. Your competitors can’t easily change their business model to match yours. Choose wisely.</p>",
        "tags": ["Entrepreneurship", "Business Models", "Sales"],
        "author_id": "38bc19cf-10ce-42b5-a4b0-54308bc54303"
      }
    },
    ...
  ],
  "limit": 25,
  "count": 25,
  "page_token": "eyJvcmdhbml6YXRpb25faWQiOnsiQiI6bnVsbCwiQk9PTCI6bnVsbCwiQlMiOm51bGwsIkwiOm51bGwsIk0iOm51bGwsIk4iOm51bGwsIk5TIjpudWxsLCJOVUxMIjpudWxsZGI0ZWIwMDctYmUWJkNjAtN2M5NzI0NGNkZWFhIiwiU1MiOm51bGx9LCJ1c2VyX2lkIjp7IkIiOm51bGwsIkJPT0wiOm51bGwsIkJTIjpudWxsLCJMIjpudWxsLCJNIjpudWxsLCJOIjpudWxsLCJOUyI6bnVsbCwiTlVMTCI6bnVsbCwiUyI6Ijc4OTU5YjMwLThjYTQtOS1iMzdmLTNjMTVjMmQzYTYwMiIsIlNTIjpudWxsfX0="
}
```

### HTTP Request

`GET https://api.onedb.xyz/collections/:collection_id/items`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`collection_id` *required* | string | Name given to the collection
`limit` *optional* | number | Maximum number of items to be returned. Default limit: 10, Max limit: 100
`fields` *optional* | string | comma delimited list of fields, e.g. fields=title,author_id
`page_token` *optional* | string | Page token used for pagination if collection has more than `limit` items.

**RESPONSE**

Parameter | Type | Description
--------- | ---- | -----------
`items` | array | List of Items within this collection
`count` | number | Number of Items returned
`limit` | number | The limit specified in the request (default: 10)
`page_token` | string | The page token specified for pagination. Page token is only returned if there are more than `limit` items.

## Search Items in a Collection

> Example Request

```shell
curl -X GET https://api.onedb.xyz/collections/6fe54303-10ce-42b5-a4b0-38bc19cf5025/items?q=title:business+model  \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
```

> Example Response

```json
{
  "items": [
    {
      "id": "48f990f0-fd66-592e-9f5d-094bc2862ebd",
      "updated_at": "2019-10-24T19:42:00.799Z",
      "created_at": "2019-10-24T19:42:00.777Z",
      "fields": {
        "title": "16 Business Models for Your Next Company",
        "body": "<p>The business model is an important decision that will have a significant impact on your profitability. It can differentiate you from competitors and give you an advantage over them. Your competitors can’t easily change their business model to match yours. Choose wisely.</p>",
        "tags": ["Entrepreneurship", "Business Models", "Sales"],
        "author_id": "38bc19cf-10ce-42b5-a4b0-54308bc54303"
      }
    },
    ...
  ],
  "count": 5,
  "page_token": "eyJvcmdhbml6YXRpb25faWQiOnsiQiI6bnVsbCwiQk9PTCI6bnVsbCwiQlMiOm51bGwsIkwiOm51bGwsIk0iOm51bGwsIk4iOm51bGwsIk5TIjpudWxsLCJOVUxMIjpudWxsZGI0ZWIwMDctYmUWJkNjAtN2M5NzI0NGNkZWFhIiwiU1MiOm51bGx9LCJ1c2VyX2lkIjp7IkIiOm51bGwsIkJPT0wiOm51bGwsIkJTIjpudWxsLCJMIjpudWxsLCJNIjpudWxsLCJOIjpudWxsLCJOUyI6bnVsbCwiTlVMTCI6bnVsbCwiUyI6Ijc4OTU5YjMwLThjYTQtOS1iMzdmLTNjMTVjMmQzYTYwMiIsIlNTIjpudWxsfX0="
}
```

### HTTP Request

`GET https://api.onedb.xyz/collections/:collection_id/items`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`collection_id` *required* | string | Name given to the collection
`q` *required* | string | Only returns items matching the specific query. e.g. `q=title:business+model author_id:38bc19cf-10ce-42b5-a4b0-54308bc54303`
`limit` *optional* | number | Maximum number of items to be returned. Default limit: 10, Max limit: 100
`fields` *optional* | string | comma delimited list of fields, e.g. fields=title,author_id
`page_token` *optional* | string | Page token used for pagination if collection has more than `limit` items.

**RESPONSE**

Parameter | Type | Description
--------- | ---- | -----------
`items` | array | List of Items within this collection
`count` | number | Number of Items returned
`limit` | number | The limit specified in the request (default: 10)
`page_token` | string | The page token specified for pagination. Page token is only returned if there are more than `limit` items.

## Get Single Item

> Example Request

```shell
curl -X GET https://api.onedb.xyz/collections/6fe54303-10ce-42b5-a4b0-38bc19cf5025/items/829b3c8d-1fad-5fbb-8b54-e1e6528ab777 \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
```

> Example Response

```json
{
  "id": "829b3c8d-1fad-5fbb-8b54-e1e6528ab777",
  "updated_at": "2019-10-24T19:42:38.929Z",
  "created_at": "2016-10-24T19:41:48.349Z",
  "fields": {
    "title": "Three Years of Building Investing Products — Lessons Learned",
    "body": "<p>The current and future of investing platforms.</p>",
    "tags": ["Fin-tech", "Startups", "API"],
    "attachments": [
      {
        "url": "https://onedb-uploads/attachments/jguTNgbqhGZG/Investing%20platform%20marketplace.pdf",
        "id": "jguTNgbqhGZG",
        "size": 1414220,
        "type": "application/pdf",
        "file_name": "Investing platform marketplace.pdf"
      }
    ],
    "author_id": "38bc19cf-10ce-42b5-a4b0-54308bc54303"
  }
}
```

### HTTP Request

`GET https://api.onedb.xyz/collections/:collection_id/items/:item_id`

**ARGUMENTS**

Parameter | Type | Description
--------- | ---- | -----------
`collection_id` *required* | string | Name given to the collection
`item_id` *required* | string | Unique identifier for the Item you are querying

<aside class="information">
In attachment objects included in the retrieved item (attachments), only id, url, and filename are always returned. Other attachment properties may not be included.
</aside>

## Create New Collection Item

> Example Request

```shell
curl -X POST https://api.onedb.xyz/collections/6fe54303-10ce-42b5-a4b0-38bc19cf5025/items \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
  -H "Content-Type: application/json" \
   -d $'{
      "fields": {
        "title": "Three Years of Building Investing Products — Lessons Learned",
        "body": "<p>The current and future of investing platforms.</p>",
        "tags": ["Fin-tech", "Startups", "API"],
        "attachments": {
          "url": "https://onedb-uploads/attachments/jguTNgbqhGZG/Investing%20platform%20marketplace.pdf",
          "id": "jguTNgbqhGZG",
          "size": 1414220,
          "type": "application/pdf",
          "file_name": "Investing platform marketplace.pdf"
        },
        "author_id": "38bc19cf-10ce-42b5-a4b0-54308bc54303"
      }
    }'
```

> Example Response

```json
{
  "id": "829b3c8d-1fad-5fbb-8b54-e1e6528ab777",
  "updated_at": "2019-10-24T19:42:38.929Z",
  "created_at": "2016-10-24T19:41:48.349Z",
  "fields": {
    "title": "Three Years of Building Investing Products — Lessons Learned",
    "body": "<p>The current and future of investing platforms.</p>",
    "tags": ["Fin-tech", "Startups", "API"],
    "attachments": [
      {
        "url": "https://onedb-uploads/attachments/jguTNgbqhGZG/Investing%20platform%20marketplace.pdf",
        "id": "jguTNgbqhGZG",
        "size": 1414220,
        "type": "application/pdf",
        "file_name": "Investing platform marketplace.pdf"
      }
    ],
    "author_id": "38bc19cf-10ce-42b5-a4b0-54308bc54303"
  }
}
```

### HTTP Request

`POST https://api.onedb.xyz/collections/:collection_id/items`

**ARGUMENTS**

Parameter | Description
--------- | -----------
`collection_id` | Name given to the collection
`fields` | The fields and data of the Item you are adding to the Collection

<aside class="information">
To add Attachments, use the Attachment API to upload the attachment and set the attachment object in the `attachments` field property.
</aside>

## Update Collection Item

> Example Request

```shell
curl -X PUT https://api.onedb.xyz/collections/6fe54303-10ce-42b5-a4b0-38bc19cf5025/items/829b3c8d-1fad-5fbb-8b54-e1e6528ab777 \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
  -H "Content-Type: application/json" \
   -d $'{
      "fields": {
        "title": "Three Years of Building Investing Products — Lessons Updated",
      }
    }'
```

> Example Response

```json
{
  "id": "829b3c8d-1fad-5fbb-8b54-e1e6528ab777",
  "updated_at": "2019-10-28T21:42:38.929Z",
  "created_at": "2016-10-24T19:41:48.349Z",
  "fields": {
    "title": "Three Years of Building Investing Products — Lessons Updated",
    "body": "<p>The current and future of investing platforms.</p>",
    "tags": ["Fin-tech", "Startups", "API"],
    "attachments": [
      {
        "url": "https://onedb-uploads/attachments/jguTNgbqhGZG/Investing%20platform%20marketplace.pdf",
        "id": "jguTNgbqhGZG",
        "size": 1414220,
        "type": "application/pdf",
        "file_name": "Investing platform marketplace.pdf"
      }
    ],
    "author_id": "38bc19cf-10ce-42b5-a4b0-54308bc54303"
  }
}
```

### HTTP Request

`PUT https://api.onedb.xyz/collections/:collection_id/items/:item_id`

**ARGUMENTS**

Parameter | Description
--------- | -----------
`collection_id` | Name given to the collection
`item_id` | Unique identifier for the Item you are querying
`fields` | The fields and data of the Item you are updating

## Remove Collection Item

> Example Request

```shell
curl -X PUT https://api.onedb.xyz/collections/6fe54303-10ce-42b5-a4b0-38bc19cf5025/items/829b3c8d-1fad-5fbb-8b54-e1e6528ab777 \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
```

> Example Response

```json
  {
    "deleted": true
  }
```

### HTTP Request

`DELETE https://api.onedb.xyz/collections/:collection_id/items/:item_id`

**ARGUMENTS**

Parameter | Description
--------- | -----------
`collection_id` | Name given to the collection
`item_id` | Unique identifier for the Item you are querying

# Attachments

## Upload Attachment

> Example Request

```shell
curl -X POST https://api.onedb.xyz/collections/6fe54303-10ce-42b5-a4b0-38bc19cf5025/attachments \
  -H "Authorization Bearer: sk_yourapikey" \
  -H "Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW" \
  -F "file=Investing platform marketplace.pdf"
```

> Example Response

```json
{
  "url": "https://onedb-uploads/attachments/jguTNgbqhGZG/Investing%20platform%20marketplace.pdf",
  "id": "jguTNgbqhGZG",
  "size": 1414220,
  "type": "application/pdf",
  "file_name": "Investing platform marketplace.pdf"
}
```

### HTTP Request

`POST https://api.onedb.xyz/:collection_id/attachments`

**ARGUMENTS**

Parameter | Description
--------- | -----------
`collection_id` | Name given to the collection

To upload a new attachment, set this attachment object to the corresponding item's field in a POST or PUT request. In response, only id, url, and filename are always returned. Other attachment properties may not be included.

# Batch Requests

The batch API makes it possible to perform many create, update, delete, get operations in a single API call. There is a limit of 50 items in a batch requests.

> Example Request

```shell
curl -X PUT https://api.onedb.xyz/collections/6fe54303-10ce-42b5-a4b0-38bc19cf5025/batch \
  -H "Authorization Bearer: base64(app_id:app_secret)" \
  -H "Content-Type: application/json" \
   -d $'{
     "items": [
       {
         "action": "update",
         "id": "829b3c8d-1fad-5fbb-8b54-e1e6528ab777",
         "fields": {
           "title": "Three Years of Building Investing Products — Lessons Updated"
         }
       },
       {
          "action": "get",
          "id": "48f990f0-fd66-592e-9f5d-094bc2862ebd",
       }
     ]
    }'
```

> Example Response

```json
{
  "items": [
    {
      "action": "update",
      "id": "829b3c8d-1fad-5fbb-8b54-e1e6528ab777",
      "result": "success",
      "updated_at": "2019-10-28T21:42:38.929Z",
      "created_at": "2016-10-24T19:41:48.349Z",
      "fields": {
        "title": "Three Years of Building Investing Products — Lessons Updated",
        "body": "<p>The current and future of investing platforms.</p>",
        "tags": ["Fin-tech", "Startups", "API"],
        "attachments": [
          {
            "url": "https://onedb-uploads/attachments/jguTNgbqhGZG/Investing%20platform%20marketplace.pdf",
            "id": "jguTNgbqhGZG",
            "size": 1414220,
            "type": "application/pdf",
            "file_name": "Investing platform marketplace.pdf"
          }
        ],
        "author_id": "38bc19cf-10ce-42b5-a4b0-54308bc54303"
      }
    },
    {
      "action": "get",
      "id": "48f990f0-fd66-592e-9f5d-094bc2862ebd",
      "result": "success",
      "updated_at": "2019-10-24T19:42:00.799Z",
      "created_at": "2019-10-24T19:42:00.777Z",
      "fields": {
        "title": "16 Business Models for Your Next Company",
        "body": "<p>The business model is an important decision that will have a significant impact on your profitability. It can differentiate you from competitors and give you an advantage over them. Your competitors can’t easily change their business model to match yours. Choose wisely.</p>",
        "tags": ["Entrepreneurship", "Business Models", "Sales"],
        "author_id": "38bc19cf-10ce-42b5-a4b0-54308bc54303"
      }
    }
  ]
}
```

### Batch Request

`POST https://api.onedb.xyz/collections/:collection_id/batch`

**ARGUMENTS**

Parameter | Description
--------- | -----------
`collection_id` | Name given to the collection
`action` *required* | Operation to perform for the Item in the batch request
`fields` *required (create, update)* | The fields and data of the Item for create or update operation

### Batch Response

Parameter | Description
--------- | -----------
`items` | Name given to the collection
`action` | Operation to perform for the Item in the batch request
`result` | Result of the operation `success` or `error_code`. See the errors section.
`fields` (not returned for delete action)| The fields and data of the Item

# Backups

OneDB creates and saves automated backups of your data. You can also back up your data manually, by setting the backup schedule. Each backup file contains the full data. Only the last 30 backup files are retained.

## Backup schedule

> Example Request

```shell
curl -X POST https://api.onedb.xyz/backups \
  -H "Authorization Bearer: sk_yourapikey" \
  -d $'{
    "schedule": "daily"
  }'
```

> Example Response

```json
{
  "schedule": "daily"
}
```

### HTTP Request

`POST https://api.onedb.xyz/backups`

**ARGUMENTS**

Parameter | Description
--------- | -----------
`schedule` | Backup schedule in increments of 5m. Minimum backup window is `every 5m`.

Automated backups occur daily during the preferred backup window. If you don't specify a preferred backup schedule, OneDB assigns a default daily backup window.

## List Backups

> Example Request

```shell
curl -X GET https://api.onedb.xyz/backups \
  -H "Authorization Bearer: sk_yourapikey" \
```

> Example Response

```json
{
  "backups": [
    {
      "id": "NEDW6VOJ45en",
      "url": "https://api.onedb.xyz/backups/NEDW6VOJ45en",
      "timestamp": "2019-10-28T21:42:38.929Z"
    },
    ...
  ],
}
```

### HTTP Request

`GET https://api.onedb.xyz/backups`

## Download a backup

> Example Request

```shell
curl -X GET https://api.onedb.xyz/backups/NEDW6VOJ45en \
  -H "Authorization Bearer: sk_yourapikey" \
```

### HTTP Request

`GET https://api.onedb.xyz/backups/NEDW6VOJ45en`

Downloads backup-NEDW6VOJ45en.json file.

# Security

Your customers data is in good hands.

* The data on rest is encrypted with 256-bit encryption
* All API requests should be over SSL
* All data access API endpoints are authenticated
* Our Servers are in a private VPC
* We implement Least access privilege for every employee
* Every access to our systems is monitored and logged
* All commands run on our servers are logged
* We enforce 2FA for all 3rd party services
