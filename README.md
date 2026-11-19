<!-- Generated from the SocQ API catalog and docs. Do not edit by hand. -->
# Reddit API

[![SocQ Reddit API](https://img.shields.io/badge/API-SocQ%20Reddit%20API-F64C31)](https://socq.ai/apis/reddit?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)
[![API documentation](https://img.shields.io/badge/Docs-docs.socq.ai-111827)](https://docs.socq.ai/api-manual/reddit/overview)
[![License: MIT](https://img.shields.io/badge/License-MIT-2563EB)](LICENSE)
[![Check examples](https://github.com/SocQAPI/reddit-api/actions/workflows/check.yml/badge.svg)](https://github.com/SocQAPI/reddit-api/actions/workflows/check.yml)

13 endpoints for public Reddit data through one SocQ API key, with runnable cURL, Node.js and Python examples and a valid request body for every endpoint.

[Browse Reddit APIs](https://socq.ai/apis/reddit?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api) · [Get an API key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api) · [Documentation](https://docs.socq.ai/api-manual/reddit/overview) · [All SocQ examples](https://github.com/SocQAPI/socq-examples)

## Endpoints

Prices are in SocQ credits; 1 credit = $0.005.

| Endpoint | publicId | What it returns | Price | Launched |
| --- | --- | --- | --- | --- |
| [Reddit Posts API](#reddit-posts-api) | `reddit/posts` | Collect public Reddit posts from supported post URLs, returning normalized text, author and community context, media, timestamps, and visible engagement metrics. | 0.5 credits ($0.0025) per result | 2026-07-18 |
| [Reddit Comments API](#reddit-comments-api) | `reddit/comments` | Collect public Reddit comments from post URLs, returning text, author and thread context, timestamps, and visible engagement in normalized records. | 0.3 credits ($0.0015) per result | 2026-07-18 |
| [Reddit Subreddit Posts API](#reddit-subreddit-posts-api) | `reddit/subreddit-posts` | Discover public Reddit posts from subreddit URLs with supported sorting, a per-subreddit result limit, and normalized content, community context, timestamps, and visible engagement. | 0.5 credits ($0.0025) per result | 2026-07-18 |
| [Reddit Search API](#reddit-search-api) | `reddit/search` | Search public Reddit posts by query and publication window, returning normalized titles, text, authors, community context, timestamps, media, and visible engagement. | 0.6 credits ($0.003) per result | 2026-07-18 |
| [Reddit Subreddit Details API](#reddit-subreddit-details-api) | `reddit/subreddit-details` | Retrieve metadata for public subreddits by community URL, returning subscriber counts, weekly active users, rules, and community assets. | 0.5 credits ($0.0025) per result | 2026-09-11 |
| [Reddit Subreddit Search API](#reddit-subreddit-search-api) | `reddit/subreddit-search` | Search public discussions and posts within one specific subreddit by query, timeframe, and sort order. | 0.6 credits ($0.003) per result | 2026-09-11 |
| [Reddit User Profile API](#reddit-user-profile-api) | `reddit/user-profile` | Look up Reddit accounts by username and get karma, post and comment counts, account age and profile details. | 0.5 credits ($0.0025) per result | 2026-11-17 |
| [Reddit User Posts API](#reddit-user-posts-api) | `reddit/user-posts` | Collect the post history of a Reddit user, sorted by new, top or hot, with full text, score, comments and media. | 0.5 credits ($0.0025) per result | 2026-11-17 |
| [Reddit User Comments API](#reddit-user-comments-api) | `reddit/user-comments` | Collect the comment history of a Reddit user with score, time, comment link and the post each comment was left on. | 0.3 credits ($0.0015) per result | 2026-11-17 |
| [Reddit User Active Subreddits API](#reddit-user-active-subreddits-api) | `reddit/user-active-subreddits` | See the subreddits a Reddit user is active in, with member counts and community descriptions. | 0.5 credits ($0.0025) per call | 2026-11-17 |
| [Reddit Popular Feed API](#reddit-popular-feed-api) | `reddit/popular-feed` | Collect the posts trending on r/popular, sorted by hot, new, top or rising, with score, comments and media. | 0.5 credits ($0.0025) per result | 2026-11-17 |
| [Reddit Trending Searches API](#reddit-trending-searches-api) | `reddit/trending-searches` | Get the searches trending on Reddit today, ranked in the order Reddit shows them. | 0.5 credits ($0.0025) per call | 2026-11-17 |
| [Reddit Subreddit Rules API](#reddit-subreddit-rules-api) | `reddit/subreddit-rules` | Get the numbered posting rules of subreddits, each with its short title and full rule text. | 0.5 credits ($0.0025) per result | 2026-11-17 |

## Quick start

1. Create an API key at [socq.ai/dashboard/api-key](https://socq.ai/dashboard/api-key?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api).
2. Export it. The examples read only `SOCQ_API_KEY`.

```bash
git clone https://github.com/SocQAPI/reddit-api.git
cd reddit-api
export SOCQ_API_KEY="your-api-key"
```

3. Run any endpoint by its publicId. Without a second argument the script sends `payloads/<slug>.json`; pass a JSON file or an inline JSON object to change the input.

```bash
# Node.js 20+
node examples/node/socq.mjs reddit/posts

# Python 3.8+ (standard library only)
python3 examples/python/socq.py reddit/posts payloads/posts.json

# cURL + jq
bash examples/curl/socq.sh reddit/posts '{"urls":["https://www.reddit.com/r/technology/comments/1vbchyh/second_major_ai_company_says_its_systems_hacked/"]}'
```

Each script submits the task, polls it until it succeeds or fails, follows every result page, and writes the records to `output/<slug>-<task_id>.json`. Run `npm run check` to validate the payloads and scripts offline.

Never expose `SOCQ_API_KEY` in browser code, mobile apps, public repositories, screenshots, fixtures or logs.

## How the API works

Base URL `https://api.socq.ai`. Every request sends the API key as a bearer token:

```http
Authorization: Bearer <SOCQ_API_KEY>
Content-Type: application/json
```

All endpoints are asynchronous:

```text
POST /v1/<publicId>  ->  data.task_id  ->  GET /v1/tasks/{task_id}  ->  queued | running | succeeded | failed
```

1. **Submit** the JSON body to `POST https://api.socq.ai/v1/<publicId>`. The response returns `data.task_id` immediately; it means the task was created, not that data is ready.
2. **Poll** `GET https://api.socq.ai/v1/tasks/{task_id}` until `data.status` is `succeeded` or `failed` (a failed task carries `data.error_message`).
3. **Read results.** A succeeded task returns normalized records in `data.results.items`. Use `limit` (default 50, max 500) and pass `data.results.next_cursor` as `cursor` while `data.results.has_more` is true. `fields` selects up to 50 comma-separated fields or dot paths.
4. **Raw files (optional).** `GET https://api.socq.ai/v1/tasks/{task_id}/files` lists raw JSONL exports.

```bash
curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100" \
  -H "Authorization: Bearer $SOCQ_API_KEY"

curl "https://api.socq.ai/v1/tasks/$TASK_ID?limit=100&cursor=$NEXT_CURSOR" \
  -H "Authorization: Bearer $SOCQ_API_KEY"
```

`callback_url` (where an endpoint accepts it) makes SocQ POST to your URL when the task reaches a terminal state; keep polling as a fallback.

**Billing.** A submit may reserve credits from `results_limit` or the number of submitted inputs; when the task finishes SocQ settles the charge from the actual result count and returns unused reserved credits. `402` means insufficient credits.

**Errors.** `400` invalid body or unsupported field, `401` missing or invalid key, `402` insufficient credits, `403` IP allowlist or task owned by another account, `404` endpoint or task not found, `422` input validation failed, `429` rate or credit-window limit (respect `Retry-After`), `500` submission failed. A client-side timeout does not cancel a task: resume polling the same `task_id` instead of submitting again.

Reference: [Authentication](https://docs.socq.ai/api-manual/agent-api/authentication) · [Task status and results](https://docs.socq.ai/api-manual/agent-api/status) · [Task files](https://docs.socq.ai/api-manual/agent-api/files) · [Callbacks](https://docs.socq.ai/api-manual/agent-api/callbacks) · [Errors](https://docs.socq.ai/api-manual/agent-api/errors)

## Endpoint reference

### Reddit Posts API

Collect public Reddit posts from supported post URLs, returning normalized text, author and community context, media, timestamps, and visible engagement metrics.

- **Request:** `POST https://api.socq.ai/v1/reddit/posts`
- **publicId:** `reddit/posts`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-07-18
- **Example body:** [`payloads/posts.json`](payloads/posts.json)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/posts" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.reddit.com/r/technology/comments/1vbchyh/second_major_ai_company_says_its_systems_hacked/"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/posts](https://docs.socq.ai/api-manual/reddit/posts) · Landing page: [socq.ai/apis/reddit/posts](https://socq.ai/apis/reddit/posts?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit Comments API

Collect public Reddit comments from post URLs, returning text, author and thread context, timestamps, and visible engagement in normalized records.

- **Request:** `POST https://api.socq.ai/v1/reddit/comments`
- **publicId:** `reddit/comments`
- **Price:** 0.3 credits ($0.0015) per result
- **Launched:** 2026-07-18
- **Example body:** [`payloads/comments.json`](payloads/comments.json)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/comments" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.reddit.com/r/technology/comments/1vbchyh/second_major_ai_company_says_its_systems_hacked/"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/comments](https://docs.socq.ai/api-manual/reddit/comments) · Landing page: [socq.ai/apis/reddit/comments](https://socq.ai/apis/reddit/comments?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit Subreddit Posts API

Discover public Reddit posts from subreddit URLs with supported sorting, a per-subreddit result limit, and normalized content, community context, timestamps, and visible engagement.

- **Request:** `POST https://api.socq.ai/v1/reddit/subreddit-posts`
- **publicId:** `reddit/subreddit-posts`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-07-18
- **Example body:** [`payloads/subreddit-posts.json`](payloads/subreddit-posts.json)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `results_limit` | integer | No | (value 1 to 2000) |
| `sort_by` | string | No | - |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/subreddit-posts" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.reddit.com/r/learnpython/"
  ],
  "results_limit": 100,
  "sort_by": "new"
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/subreddit-posts](https://docs.socq.ai/api-manual/reddit/subreddit-posts) · Landing page: [socq.ai/apis/reddit/subreddit-posts](https://socq.ai/apis/reddit/subreddit-posts?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit Search API

Search public Reddit posts by query and publication window, returning normalized titles, text, authors, community context, timestamps, media, and visible engagement.

- **Request:** `POST https://api.socq.ai/v1/reddit/search`
- **publicId:** `reddit/search`
- **Price:** 0.6 credits ($0.003) per result
- **Launched:** 2026-07-18
- **Example body:** [`payloads/search.json`](payloads/search.json)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `published_within` | string | No | - |
| `query` | string | Yes | Search query. |
| `results_limit` | integer | No | (value 1 to 2000) |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "machine learning",
  "results_limit": 100,
  "published_within": "month"
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/search](https://docs.socq.ai/api-manual/reddit/search) · Landing page: [socq.ai/apis/reddit/search](https://socq.ai/apis/reddit/search?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit Subreddit Details API

Retrieve metadata for public subreddits by community URL, returning subscriber counts, weekly active users, rules, and community assets.

- **Request:** `POST https://api.socq.ai/v1/reddit/subreddit-details`
- **publicId:** `reddit/subreddit-details`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-09-11
- **Example body:** [`payloads/subreddit-details.json`](payloads/subreddit-details.json)

Retrieve metadata for public subreddits by community URL, returning subscriber counts, weekly active users, rules, and community assets.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/subreddit-details" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.reddit.com/r/technology/"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/subreddit-details](https://docs.socq.ai/api-manual/reddit/subreddit-details) · Landing page: [socq.ai/apis/reddit/subreddit-details](https://socq.ai/apis/reddit/subreddit-details?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit Subreddit Search API

Search public discussions and posts within one specific subreddit by query, timeframe, and sort order.

- **Request:** `POST https://api.socq.ai/v1/reddit/subreddit-search`
- **publicId:** `reddit/subreddit-search`
- **Price:** 0.6 credits ($0.003) per result
- **Launched:** 2026-09-11
- **Example body:** [`payloads/subreddit-search.json`](payloads/subreddit-search.json)

Search public discussions and posts within one specific subreddit by query, timeframe, and sort order.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `published_within` | string | No | (one of `all`, `day`, `hour`, `month`, `week`, `year`) |
| `query` | string | Yes | Search query. |
| `results_limit` | integer | No | (value 1 to 2000; default `100`) |
| `sort_by` | string | No | (one of `comments`, `hot`, `new`, `relevance`, `top`) |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/subreddit-search" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "query": "AI agents",
  "urls": [
    "https://www.reddit.com/r/technology/"
  ],
  "results_limit": 20,
  "sort_by": "relevance",
  "published_within": "all"
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/subreddit-search](https://docs.socq.ai/api-manual/reddit/subreddit-search) · Landing page: [socq.ai/apis/reddit/subreddit-search](https://socq.ai/apis/reddit/subreddit-search?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit User Profile API

Look up Reddit accounts by username and get karma, post and comment counts, account age and profile details.

- **Request:** `POST https://api.socq.ai/v1/reddit/user-profile`
- **publicId:** `reddit/user-profile`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/user-profile.json`](payloads/user-profile.json)

Look up Reddit accounts by username and get one row per account: total karma split into post and comment karma, how many posts and comments the account has made, when it was created, its profile description and avatar, trophy count, and flags for verified accounts and Reddit employees. Use it to vet a creator before outreach, judge how established the authors in a thread are, or keep a watchlist of accounts that matter in your niche.

Provide up to 100 Reddit usernames in usernames, with or without the u/ prefix. Each account returns one row with its karma, activity counts and profile details, and costs 0.5 credits.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `usernames` | array of string | Yes | Reddit usernames, with or without the u/ prefix. (items 1 to 100) |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/user-profile" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "usernames": [
    "spez",
    "GovSchwarzenegger"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/user-profile](https://docs.socq.ai/api-manual/reddit/user-profile) · Landing page: [socq.ai/apis/reddit/user-profile](https://socq.ai/apis/reddit/user-profile?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit User Posts API

Collect the post history of a Reddit user, sorted by new, top or hot, with full text, score, comments and media.

- **Request:** `POST https://api.socq.ai/v1/reddit/user-posts`
- **publicId:** `reddit/user-posts`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/user-posts.json`](payloads/user-posts.json)

Pull everything a Reddit user has posted, with the full text of self posts, the post link, score, comment count, publish time and any images or videos. Sort by newest to follow what someone is sharing right now, or by top to see which of their posts landed best. Handy for creator research, brand monitoring of a founder's account, or building a dataset from a domain expert's write-ups.

Provide one Reddit username in username. sort_by orders the posts by new (default), top or hot. results_limit defaults to 25 and accepts integers from 1 to 1000. Each post costs 0.5 credits; likes_count holds the post's score.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `results_limit` | integer | No | (value 1 to 1000; default `25`) |
| `sort_by` | string | No | Order of the posts: new, top or hot. (one of `hot`, `new`, `top`; default `"new"`) |
| `username` | string | Yes | Reddit username, with or without the u/ prefix. |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/user-posts" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "username": "spez",
  "sort_by": "new",
  "results_limit": 25
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/user-posts](https://docs.socq.ai/api-manual/reddit/user-posts) · Landing page: [socq.ai/apis/reddit/user-posts](https://socq.ai/apis/reddit/user-posts?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit User Comments API

Collect the comment history of a Reddit user with score, time, comment link and the post each comment was left on.

- **Request:** `POST https://api.socq.ai/v1/reddit/user-comments`
- **publicId:** `reddit/user-comments`
- **Price:** 0.3 credits ($0.0015) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/user-comments.json`](payloads/user-comments.json)

See what a Reddit user says in other people's threads. Each comment comes with its score, time, a link to the comment and the ID of the post it was left on. Comment history shows what someone actually knows and cares about better than their own posts do: use it to qualify experts and potential customers, track how an account talks about your product, or study the voice of a community through its most active members.

Provide one Reddit username in username. sort_by orders the comments by new (default), top or hot. results_limit defaults to 100 and accepts integers from 1 to 1000. text holds a preview of up to 300 characters, url opens the full comment and post_id identifies the post it was left on. Each comment costs 0.3 credits.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `results_limit` | integer | No | (value 1 to 1000; default `100`) |
| `sort_by` | string | No | Order of the comments: new, top or hot. (one of `hot`, `new`, `top`; default `"new"`) |
| `username` | string | Yes | Reddit username, with or without the u/ prefix. |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/user-comments" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "username": "spez",
  "sort_by": "top",
  "results_limit": 100
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/user-comments](https://docs.socq.ai/api-manual/reddit/user-comments) · Landing page: [socq.ai/apis/reddit/user-comments](https://socq.ai/apis/reddit/user-comments?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit User Active Subreddits API

See the subreddits a Reddit user is active in, with member counts and community descriptions.

- **Request:** `POST https://api.socq.ai/v1/reddit/user-active-subreddits`
- **publicId:** `reddit/user-active-subreddits`
- **Price:** 0.5 credits ($0.0025) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/user-active-subreddits.json`](payloads/user-active-subreddits.json)

Find out which communities a Reddit user spends time in. Each row is a subreddit the account has recently posted or commented in, with its member count and description. It is a quick way to map someone's interests, find the niche communities your audience hangs out in, or pick where to listen and engage next.

Provide one Reddit username in username. Each request costs 0.5 credits and returns every community on the account's public active list.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `username` | string | Yes | Reddit username, with or without the u/ prefix. |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/user-active-subreddits" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "username": "GovSchwarzenegger"
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/user-active-subreddits](https://docs.socq.ai/api-manual/reddit/user-active-subreddits) · Landing page: [socq.ai/apis/reddit/user-active-subreddits](https://socq.ai/apis/reddit/user-active-subreddits?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit Popular Feed API

Collect the posts trending on r/popular, sorted by hot, new, top or rising, with score, comments and media.

- **Request:** `POST https://api.socq.ai/v1/reddit/popular-feed`
- **publicId:** `reddit/popular-feed`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/popular-feed.json`](payloads/popular-feed.json)

Read r/popular the way Reddit shows it: the posts rising across all communities right now, with title and body, post link, author, score, comment count and media. Switch sort_by to top and pick a window from an hour to all time to see what broke through over a day or a week. Media teams use it to spot stories early, marketers to catch formats that are working, and researchers to sample what the platform is talking about.

All fields are optional. sort_by picks how r/popular is ordered: best, hot (default), new, top or rising; published_within sets the window for top, from hour to all. results_limit defaults to 25 and accepts integers from 1 to 500. Each post costs 0.5 credits.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `published_within` | string | No | Time window used when sort_by is top: hour, day (default), week, month, year or all. (one of `all`, `day`, `hour`, `month`, `week`, `year`; default `"day"`) |
| `results_limit` | integer | No | (value 1 to 500; default `25`) |
| `sort_by` | string | No | Order of the feed: best, hot (default), new, top or rising. (one of `best`, `hot`, `new`, `rising`, `top`; default `"hot"`) |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/popular-feed" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "sort_by": "hot",
  "results_limit": 25
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/popular-feed](https://docs.socq.ai/api-manual/reddit/popular-feed) · Landing page: [socq.ai/apis/reddit/popular-feed](https://socq.ai/apis/reddit/popular-feed?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit Trending Searches API

Get the searches trending on Reddit today, ranked in the order Reddit shows them.

- **Request:** `POST https://api.socq.ai/v1/reddit/trending-searches`
- **publicId:** `reddit/trending-searches`
- **Price:** 0.5 credits ($0.0025) per call
- **Launched:** 2026-11-17
- **Example body:** [`payloads/trending-searches.json`](payloads/trending-searches.json)

Get today's trending searches on Reddit, in the order Reddit lists them. These are the topics people are actively looking up right now, often news events, product launches and cultural moments while they are unfolding. Poll it a few times a day to feed a trend dashboard, plan timely content, or decide which threads to dig into with Reddit search.

No input is needed. Each call costs 0.5 credits and returns the current list, usually six searches, ranked from 1.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/trending-searches" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/trending-searches](https://docs.socq.ai/api-manual/reddit/trending-searches) · Landing page: [socq.ai/apis/reddit/trending-searches](https://socq.ai/apis/reddit/trending-searches?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

### Reddit Subreddit Rules API

Get the numbered posting rules of subreddits, each with its short title and full rule text.

- **Request:** `POST https://api.socq.ai/v1/reddit/subreddit-rules`
- **publicId:** `reddit/subreddit-rules`
- **Price:** 0.5 credits ($0.0025) per result
- **Launched:** 2026-11-17
- **Example body:** [`payloads/subreddit-rules.json`](payloads/subreddit-rules.json)

Read a subreddit's rules before you post there. Each subreddit comes back as one row with its numbered rules in order, each with the short title moderators show in the report menu and the full rule text. Brands and community managers use it to check what is allowed before a launch post, and teams running many accounts use it to keep a rulebook for every community they take part in.

Provide subreddit URLs in urls, such as https://www.reddit.com/r/AskReddit/. Each subreddit returns one row with its numbered rules and costs 0.5 credits.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `callback_url` | string (uri) | No | - |
| `urls` | array of string | Yes | Public profile or content URLs. (items at least 1) |

```bash
curl -X POST "https://api.socq.ai/v1/reddit/subreddit-rules" \
  -H "Authorization: Bearer $SOCQ_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
  "urls": [
    "https://www.reddit.com/r/AskReddit/"
  ]
}'
```

Full docs: [https://docs.socq.ai/api-manual/reddit/subreddit-rules](https://docs.socq.ai/api-manual/reddit/subreddit-rules) · Landing page: [socq.ai/apis/reddit/subreddit-rules](https://socq.ai/apis/reddit/subreddit-rules?utm_source=github&utm_medium=readme&utm_campaign=socq-reddit-api)

## Repository contents

| Path | Purpose |
| --- | --- |
| [`examples/curl/socq.sh`](examples/curl/socq.sh) | Submit, poll and paginate with cURL and jq |
| [`examples/node/socq.mjs`](examples/node/socq.mjs) | Same workflow in Node.js 20+ (no dependencies) |
| [`examples/python/socq.py`](examples/python/socq.py) | Same workflow in Python 3.8+ (standard library) |
| [`payloads/`](payloads) | One documented request body per endpoint |
| [`endpoints.json`](endpoints.json) | Endpoint list, prices and request schemas used by the scripts and checks |
| [`fixtures/`](fixtures) | Synthetic submit and task response shapes |
| [`docs/production-notes.md`](docs/production-notes.md) | Retries, timeouts, pagination and data handling |
| [`scripts/check.mjs`](scripts/check.mjs) | Offline check: payloads, syntax and secrets |

## Responsible use and platform scope

- Use only publicly accessible Reddit data and the fields supported by each endpoint.
- Do not use the examples to access restricted content, login-only surfaces, authentication controls or non-public personal information.
- SocQ is not an official API of the represented platform and is not affiliated with or endorsed by it. Platform names and trademarks belong to their respective owners.
- Before production use, assess the laws, platform terms, privacy obligations and retention requirements that apply to your organization and use case.
- Collect only the fields needed for a defined purpose, restrict access, set retention periods, and support correction or deletion workflows where required.

This section describes the public-data boundary; it is not legal advice.

## License

[MIT](LICENSE)
