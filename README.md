# Imgur API Endpoint Reference

This README is a practical reference for Imgur-related HTTP endpoints observed in working integrations.

It focuses on what each endpoint does, the HTTP method it uses, and the general authentication style required. It intentionally avoids application-specific logic, usernames, account IDs, API keys, tokens, and other credentials.

> Some endpoints listed here are part of Imgur's public v3 API, while others are internal or web-client-facing endpoints and may change without notice.

---

## Base URLs

### Imgur v3 API

```text
https://api.imgur.com/3
```

### Imgur OAuth

```text
https://api.imgur.com/oauth2
```

### Imgur post API

```text
https://api.imgur.com/post/v1
```

### Imgur messaging token API

```text
https://api.imgur.com
```

### Imgur Stream Chat backend

```text
https://chat-us-east-imgur.stream-io-api.com
```

---

# Authentication

Imgur endpoints generally use one of the following authentication methods.

## Client ID

Typically used for public/read-only requests.

```http
Authorization: Client-ID <IMGUR_CLIENT_ID>
```

## OAuth Bearer Token

Used for account-specific or authenticated operations.

```http
Authorization: Bearer <IMGUR_ACCESS_TOKEN>
```

## OAuth Refresh Token

Used to obtain a new access token.

Typical required values:

```text
client_id
client_secret
refresh_token
grant_type=refresh_token
```

## Stream Chat Authentication

Imgur messaging uses a Stream Chat backend.

Typical requests use:

```http
Authorization: <STREAM_JWT>
stream-auth-type: jwt
Content-Type: application/json
```

Additional query parameters commonly include:

```text
api_key
user_id
connection_id
```

---

# Account Endpoints

## Get account information

```http
GET /3/account/{username}
```

Returns information about an Imgur account.

Typical uses include retrieving:

- Account ID
- Username
- Account creation date
- Profile metadata
- Account existence/status

---

## Get the authenticated account

```http
GET /3/account/me
```

Returns information about the Imgur account associated with the current OAuth access token.

Requires authenticated access.

---

## Resolve account information by account ID

```http
GET /3/account/me?account_id={accountId}
```

Can be used to resolve account information using a numeric Imgur account ID.

This is useful when a stable numeric account ID is known but the username may have changed.

---

## Get account submissions

```http
GET /3/account/{username}/submissions
```

Returns submissions made by an account.

---

## Get account submissions page 0

```http
GET /3/account/{username}/submissions/0
```

Returns the first page of an account's submissions.

This is commonly useful when only the newest submissions are needed.

---

## Get account comments

```http
GET /3/account/{username}/comments
```

Returns comments made by an account.

---

# Gallery Endpoints

## Get a gallery page

```http
GET /3/gallery/{section}/{sort}/{window}/{page}
```

Returns a page of gallery posts.

Common path values include:

### `section`

```text
user
hot
top
```

### `sort`

```text
viral
top
time
rising
```

### `window`

```text
day
week
month
year
all
```

### `page`

A zero-based or numeric gallery page value.

---

## Get a gallery post

```http
GET /3/gallery/{postId}
```

Returns metadata for a specific gallery post.

Typical information includes:

- Title
- Author/account
- Score
- Creation time
- Image/album information
- Media URLs

---

## Get gallery comments

```http
GET /3/gallery/{postId}/comments
```

Returns comments associated with a gallery post.

---

## Get gallery comments sorted newest first

```http
GET /3/gallery/{postId}/comments?sort=new
```

Returns comments for a gallery post with the newest comments first.

---

## Upvote a gallery post

```http
POST /3/gallery/{galleryHash}/vote/up
```

Upvotes a gallery post using the authenticated account.

Requires OAuth authentication.

---

# Album Endpoints

## Get album comments

```http
GET /3/album/{postId}/comments
```

Returns comments associated with an album.

---

## Get album comments sorted newest first

```http
GET /3/album/{postId}/comments?sort=new
```

Returns album comments with the newest comments first.

---

## Create an album

```http
POST /3/album
```

Creates a new Imgur album.

Requires OAuth authentication.

The returned album ID can then be used when uploading images or publishing the album.

---

# Image Endpoints

## Get image comments

```http
GET /3/image/{postId}/comments
```

Returns comments associated with an individual image.

---

## Get image comments sorted newest first

```http
GET /3/image/{postId}/comments?sort=new
```

Returns image comments with the newest comments first.

---

## Update image metadata

```http
POST /3/image/{imageId}
```

Updates metadata associated with an uploaded image.

For example:

```json
{
  "description": "Example description"
}
```

Depending on the workflow, additional values such as a related post or album ID may also be included.

Requires OAuth authentication.

---

## Upload an image

```http
POST /3/upload
```

Uploads an image to Imgur.

Usually sent as multipart form data.

Example fields:

```text
image=<file data>
type=file
name=<filename>
album=<albumId>
```

The `album` field is optional when uploading a standalone image.

Requires OAuth authentication for account-owned uploads.

---

# Comment Endpoints

## Create a comment

```http
POST /3/comment
```

Creates a new comment.

Typical JSON body:

```json
{
  "image_id": "<postId>",
  "comment": "Comment text"
}
```

Requires OAuth authentication.

---

## Reply to a comment

The same endpoint is used for replies:

```http
POST /3/comment
```

Include `parent_id` to create a threaded reply.

```json
{
  "image_id": "<postId>",
  "comment": "Reply text",
  "parent_id": 123456789
}
```

---

## Delete a comment

```http
DELETE /3/comment/{commentId}
```

Deletes a comment owned by the authenticated account.

Requires OAuth authentication.

---

## Get replies to a comment

```http
GET /3/comment/{commentId}/replies
```

Returns replies beneath a specific comment.

---

## Upvote a comment

```http
POST /3/comment/{commentId}/vote/up
```

Upvotes a comment.

Requires OAuth authentication.

---

## Downvote a comment

```http
POST /3/comment/{commentId}/vote/down
```

Downvotes a comment.

Requires OAuth authentication.

---

# Notification and History Endpoints

## Get account history

```http
GET /3/larynx/history
```

Returns authenticated account history used by Imgur's web client.

This can include notification-style entries such as:

```text
commentmention
commentreplygroup
```

Possible data includes:

- Notification ID
- Notification type
- Read/unread state
- Comment metadata
- Author
- Comment text
- Related post/image
- Update timestamp

This appears to be an internal Imgur web-client endpoint rather than a normal documented public v3 endpoint.

Because it is internal, its behavior or response format may change without notice.

Requires OAuth authentication.

---

## Get reply notifications

```http
GET /3/account/{username}/notifications/replies?new=false
```

Returns reply-related notifications for an account.

The `new` parameter controls whether only new notifications are requested.

Example:

```text
new=false
```

returns both previously seen and unseen reply notifications.

This endpoint may not expose every notification type available through Imgur's internal history feed.

Requires OAuth authentication.

---

## Mark a notification as viewed

```http
POST /3/notification/{notificationId}
```

Marks an Imgur notification/history entry as viewed.

Requires OAuth authentication.

---

# OAuth Endpoints

## Refresh an access token

```http
POST /oauth2/token
```

Obtains a new OAuth access token using a refresh token.

Typical form data:

```text
refresh_token=<IMGUR_REFRESH_TOKEN>
client_id=<IMGUR_CLIENT_ID>
client_secret=<IMGUR_CLIENT_SECRET>
grant_type=refresh_token
```

Typical response data includes:

```text
access_token
refresh_token
account_username
```

Applications should securely store the newly returned refresh token when Imgur rotates it.

---

# Gallery Publishing

## Share a post or album to the gallery

```http
POST /post/v1/posts/{albumId}/share
```

Publishes an uploaded post or album to Imgur's public gallery.

Typical JSON body:

```json
{
  "title": "Example title",
  "terms": true,
  "mature": false,
  "tags": [],
  "is_album": true,
  "subtype": "desktop"
}
```

This endpoint belongs to Imgur's newer `post/v1` API rather than the traditional `/3` API.

It may require OAuth authentication and, in some web-client-style workflows, additional headers or session information.

---

# Imgur Messaging

Imgur messaging is backed by Stream Chat rather than the standard Imgur v3 API.

The general flow is:

```text
Imgur authentication
        |
        v
Request messaging token
        |
        v
Receive Stream JWT
        |
        v
Call Stream Chat backend
```

---

## Get an Imgur messaging token

```http
POST /account/v1/accounts/{accountName}/messaging/token
```

Returns the token required to access Imgur's Stream-backed messaging service.

Observed authentication includes:

```http
Authorization: Client-ID <IMGUR_CLIENT_ID>
Cookie: accesstoken=<IMGUR_ACCESS_TOKEN>
```

The response contains a JWT that can then be used with the Stream Chat backend.

This is not part of the standard Imgur v3 API.

---

# Stream Chat Endpoints

Base URL:

```text
https://chat-us-east-imgur.stream-io-api.com
```

Typical query parameters:

```text
api_key=<STREAM_API_KEY>
user_id=<IMGUR_USER_ID>
connection_id=<STREAM_CONNECTION_ID>
```

Typical headers:

```http
Authorization: <STREAM_JWT>
Content-Type: application/json
stream-auth-type: jwt
x-stream-client: <client identifier>
```

---

## List messaging channels

```http
POST /channels
```

Returns messaging channels that match a filter.

Example request body:

```json
{
  "filter_conditions": {
    "type": "messaging",
    "members": {
      "$in": ["<userId>"]
    }
  },
  "sort": [
    {
      "field": "last_message_at",
      "direction": -1
    }
  ],
  "limit": 30,
  "offset": 0,
  "state": true,
  "watch": false,
  "presence": false
}
```

---

## Query a messaging channel

```http
POST /channels/messaging/{channelId}/query
```

Returns the current state of a specific Imgur direct-message channel.

A browser-originated request commonly looks like:

```text
https://chat-us-east-imgur.stream-io-api.com/channels/messaging/{CHANNEL_ID}/query
    ?user_id={USER_ID}
    &api_key={API_KEY}
    &connection_id={CONNECTION_ID}
```

### Parameters

- `CHANNEL_ID` — identifies the Stream messaging channel / Imgur DM conversation.
- `USER_ID` — the authenticated Imgur/Stream user ID.
- `API_KEY` — the Stream API key used by Imgur's messaging frontend.
- `CONNECTION_ID` — identifies the active Stream client connection/session.
- `Authorization` — contains the Stream JWT obtained through Imgur's messaging-token flow.

Typical headers:

```http
Authorization: <STREAM_JWT>
Content-Type: application/json
stream-auth-type: jwt
x-stream-client: stream-chat-javascript-client-browser-1.2.2
```

A typical request body is:

```json
{
  "data": {},
  "state": true,
  "watch": true,
  "presence": false
}
```

`state: true` requests the current channel state.

`watch: true` tells Stream to watch the channel for updates associated with the active connection.

`presence: false` means presence information is not requested.

The exact body can vary by client. For example, an integration that only wants to query state without subscribing to channel updates may use `watch: false`.

### cURL example

```bash
curl -X POST \
  'https://chat-us-east-imgur.stream-io-api.com/channels/messaging/{CHANNEL_ID}/query?user_id={USER_ID}&api_key={API_KEY}&connection_id={CONNECTION_ID}' \
  -H 'Authorization: {STREAM_JWT}' \
  -H 'Content-Type: application/json;charset=UTF-8' \
  -H 'stream-auth-type: jwt' \
  -H 'x-stream-client: stream-chat-javascript-client-browser-1.2.2' \
  -H 'Referer: https://imgur.com/' \
  --data-raw '{
    "data": {},
    "state": true,
    "watch": true,
    "presence": false
  }'
```

Browser DevTools may include many additional headers such as `Accept-Language`, `DNT`, `Sec-Fetch-*`, `sec-ch-ua-*`, and `User-Agent`. Those are normally browser metadata rather than core requirements of the API call.

### Node.js `fetch()` example

Node.js 18 and newer include `fetch()` globally.

```js
const channelId = process.env.IMGUR_STREAM_CHANNEL_ID;
const userId = process.env.IMGUR_STREAM_USER_ID;
const apiKey = process.env.IMGUR_STREAM_API_KEY;
const connectionId = process.env.IMGUR_STREAM_CONNECTION_ID;
const streamJwt = process.env.IMGUR_STREAM_JWT;

const url =
  `https://chat-us-east-imgur.stream-io-api.com/channels/messaging/` +
  `${encodeURIComponent(channelId)}/query?` +
  new URLSearchParams({
    user_id: userId,
    api_key: apiKey,
    connection_id: connectionId,
  });

const response = await fetch(url, {
  method: "POST",
  headers: {
    Accept: "application/json, text/plain, */*",
    Authorization: streamJwt,
    "Content-Type": "application/json;charset=UTF-8",
    "stream-auth-type": "jwt",
    "x-stream-client": "stream-chat-javascript-client-browser-1.2.2",
    Referer: "https://imgur.com/",
  },
  body: JSON.stringify({
    data: {},
    state: true,
    watch: true,
    presence: false,
  }),
});

if (!response.ok) {
  throw new Error(
    `Stream query failed: ${response.status} ${response.statusText}`
  );
}

const data = await response.json();
console.log(data);
```

Keeping the IDs, API key, connection ID, and JWT in environment variables avoids accidentally committing live credentials or session values to source control.

---

## Send a message

```http
POST /channels/messaging/{channelId}/message
```

Sends a message to an Imgur messaging channel.

Example body:

```json
{
  "message": {
    "text": "Hello"
  }
}
```

---

## Mark a channel as read

```http
POST /channels/messaging/{channelId}/read
```

Marks the specified messaging channel as read.

---

# Direct Imgur Media URLs

Imgur API responses often contain direct media URLs such as:

```text
https://i.imgur.com/<imageId>.jpg
https://i.imgur.com/<imageId>.png
https://i.imgur.com/<imageId>.gif
https://i.imgur.com/<imageId>.mp4
```

These URLs can be requested directly.

---

## Check whether media exists

```http
GET <Imgur media URL>
```

A lightweight streamed request can be used to determine whether media still exists.

Deleted Imgur media may:

- Return HTTP `404`
- Redirect to:

```text
https://i.imgur.com/removed.png
```

Applications that archive, hash, or compare media should detect the removed-image placeholder so unrelated deleted images are not accidentally treated as identical content.

---

## Download media

```http
GET <Imgur media URL>
```

Downloads the media bytes directly.

For animated posts, API responses may expose an MP4 URL. Using the direct MP4 URL avoids downloading an HTML GIFV wrapper.

---

# Endpoint Summary

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/3/account/{username}` | Get account information |
| `GET` | `/3/account/me` | Get authenticated account |
| `GET` | `/3/account/me?account_id={accountId}` | Resolve account information from account ID |
| `GET` | `/3/account/{username}/submissions` | Get account submissions |
| `GET` | `/3/account/{username}/submissions/0` | Get first submissions page |
| `GET` | `/3/account/{username}/comments` | Get account comments |
| `GET` | `/3/gallery/{section}/{sort}/{window}/{page}` | Browse gallery |
| `GET` | `/3/gallery/{postId}` | Get gallery post |
| `GET` | `/3/gallery/{postId}/comments` | Get gallery comments |
| `GET` | `/3/gallery/{postId}/comments?sort=new` | Get newest gallery comments |
| `POST` | `/3/gallery/{galleryHash}/vote/up` | Upvote gallery post |
| `GET` | `/3/album/{postId}/comments` | Get album comments |
| `GET` | `/3/album/{postId}/comments?sort=new` | Get newest album comments |
| `POST` | `/3/album` | Create album |
| `GET` | `/3/image/{postId}/comments` | Get image comments |
| `GET` | `/3/image/{postId}/comments?sort=new` | Get newest image comments |
| `POST` | `/3/image/{imageId}` | Update image metadata |
| `POST` | `/3/upload` | Upload image |
| `POST` | `/3/comment` | Create comment or reply |
| `DELETE` | `/3/comment/{commentId}` | Delete comment |
| `GET` | `/3/comment/{commentId}/replies` | Get comment replies |
| `POST` | `/3/comment/{commentId}/vote/up` | Upvote comment |
| `POST` | `/3/comment/{commentId}/vote/down` | Downvote comment |
| `GET` | `/3/larynx/history` | Get authenticated account history |
| `GET` | `/3/account/{username}/notifications/replies?new=false` | Get reply notifications |
| `POST` | `/3/notification/{notificationId}` | Mark notification viewed |
| `POST` | `/oauth2/token` | Refresh OAuth token |
| `POST` | `/post/v1/posts/{albumId}/share` | Publish post/album to gallery |
| `POST` | `/account/v1/accounts/{accountName}/messaging/token` | Get messaging token |
| `POST` | `/channels` | List messaging channels |
| `POST` | `/channels/messaging/{channelId}/query` | Query messaging channel |
| `POST` | `/channels/messaging/{channelId}/message` | Send message |
| `POST` | `/channels/messaging/{channelId}/read` | Mark channel read |
| `GET` | `<Imgur media URL>` | Check/download media |

---

# Stability Notes

The following endpoints appear to be internal or web-client-facing rather than normal public Imgur v3 endpoints:

```text
/3/larynx/history
/account/v1/accounts/{accountName}/messaging/token
/post/v1/posts/{albumId}/share
```

The Stream Chat endpoints are provided by the messaging backend used by Imgur rather than by the normal Imgur API.

Internal endpoints may change without notice, so integrations that depend on them should include:

- Error handling
- Response-shape validation
- Authentication failure handling
- Rate-limit handling
- Logging
- Retry limits

---

# Security Notes

Never commit the following values to a public repository:

```text
IMGUR_CLIENT_SECRET
IMGUR_ACCESS_TOKEN
IMGUR_REFRESH_TOKEN
Stream JWTs
Session cookies
Private API credentials
```

Store secrets in environment variables, a secrets manager, or another configuration source excluded from version control.
