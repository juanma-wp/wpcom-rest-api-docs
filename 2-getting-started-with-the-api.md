# Getting Started with the API

## What is it?

Our REST API allows you to view, create or edit content on any WordPress.com site, as well as any self-hosted (WordPress.org) site connected via [Jetpack](https://jetpack.me/). This includes not only blog posts and pages but also comments, tags, categories, media, site stats, notifications, sharing settings, user profiles, and many other WordPress.com features.

Some requests (e.g. listing public posts) do not need to be authenticated, but any action that would require a user to be logged in (such as creating a post) requires an authentication token. 

To make authenticated requests, you’ll first need to [set up an account on WordPress.com](https://wordpress.com/start) if you don’t have one already. ==Go to [XXXXXX](#) for more info about how to make requests (non-authenticated and authenticated ones)==

## How To Use It

There are two ways to explore the endpoints available for WordPress.com REST API:
-  The [REST API Reference docs page](developer.wordpress.com/docs/api/reference) lists available endpoints and details the input and output of each one, along with example code in curl and PHP. 
-  The [REST API development console](developer.wordpress.com/docs/api/console/) allows you to construct and try out real API requests.

Making unauthenticated requests is simple. Since there are no special headers required, you can even open this one in your browser to see what it will return: [https://public-api.wordpress.com/rest/v1.1/sites/en.blog.wordpress.com/posts/?number=2\&pretty=true](https://public-api.wordpress.com/rest/v1.1/sites/en.blog.wordpress.com/posts/?number=2&pretty=true)

Making authenticated requests requires a few more steps. (If you’re already familiar with OAuth2, now might be a good time to skip directly to the [technical documentation](https://developer.wordpress.com/docs/oauth2/).)

We use the OAuth2 protocol, which is simply a way for your application to *act on behalf of a user.* This is why you need to create a client application, even for a single-user app; the client mediates between any WordPress.com user and our API. For a single-user application, this can be confusing, because in this case you are both the client and the user who is logging in. However, one client can acquire tokens for many users, similar to a professional dog-walker who’s been entrusted with many leashes, or a hotel valet with the keys to many cars.

One of the features of OAuth is the use of *tokens*, strings which act as a sort of key. They represent the user’s consent to act on their behalf. This has several advantages: client apps do not need to request or store user’s passwords, and tokens can expire or be revoked on a per-client basis. Unlike a username \+ password combination, which can be used all over the place, tokens can only be used by the client that requested them.

If you’ve ever logged into a website using your Facebook or Twitter login, you’ve seen OAuth in action. The interesting part is the way it sends you over to Facebook, asking Facebook to log you in and allow access. You can think of it as a three-way conversation:

**User:** “I’d like to make a post via this API client.”  
**Client:** “Okay. Hey, WordPress.com, I’d like to do something on behalf of this user. Can you ask them if it’s okay?”  
**WordPress.com:** “Sure. Hey, user, is it okay if Client acts on your behalf?”  
**User:** “Yes, that is okay. I trust this client to take actions for me in the future.”  
**WordPress.com:** “Okay, Client, here is a token that will allow you to take actions for this user. Keep it secret. Keep it safe.”  
*(later)*  
**Client:** “Hi, WordPress.com. I’m making a post on behalf of a particular user. Here are all the details about the post, and to go with it, here’s the token that shows I’m allowed to post as this user.”

So, how does this work from the client’s perspective? The first thing you need to do is send the user over to WordPress.com to request access. You’ll link them to a specific URL that includes, most importantly, **your client ID** and a **redirect URL**. The client ID tells us which client is making the request; the redirect URL is where we’ll send the user back to. As a security measure, we ask for the redirect URL to be defined in your application’s settings as well; if the one you pass in doesn’t match the one in your settings, the request cannot be completed.

When the user authorizes the request, we’ll send them back to your redirect URL, but with a bonus: the token will be tacked on to the end. Your application should be prepared to handle that URL, extracting incoming tokens which it can then save for later use.

Once you have your token, you can authenticate any API request by including it in the request headers. You do this by including a header called “`Authorization"` with the value “`BEARER your_token_here".`

Now that you understand the basics of the authentication process, refer to the [OAuth2 docs](https://developer.wordpress.com/docs/oauth2/) for a more technical walkthrough, including code examples.

## Base URL Structure

The WordPress.com REST API provides a standardized base URL structure that ensures consistent access across all site types, hosting configurations, and API namespaces. All available endpoints are organized and grouped under different namespaces (such as `wp`, `rest`, and `wpcom`) and their respective versions (like `v1`, `v1.4`, `v2`, `v4`), providing logical separation between different API functionalities and allowing for independent versioning strategies. This unified approach simplifies API integration and eliminates the need to determine different URL formats based on site characteristics or API versions.

For detailed information about available namespaces, their versions, and what endpoints each namespace provides, see the [Namespaces & Versions](3-namespaces.md) documentation.


### General URL Structure

All WordPress.com REST API endpoints follow this standardized pattern:

```
https://public-api.wordpress.com/{namespace}/{version}/{endpoint}
```

**Placeholders:**
- `{namespace}`: The API namespace (e.g., 'rest', 'wp', 'wpcom')
- `{version}`: The API version (e.g., 'v1', 'v1.4', 'v2', 'v4')
- `{endpoint}`: The specific API endpoint you want to access

**Examples:**
```
https://public-api.wordpress.com/rest/v1.4/me
https://public-api.wordpress.com/wpcom/v4/notifications
https://public-api.wordpress.com/wp/v2/posts
```

### Site-Specific URL Structure

When accessing endpoints that operate on specific WordPress.com sites, the URL structure includes a site identifier:

```
https://public-api.wordpress.com/{namespace}/{version}/sites/{site_id}/{endpoint}
```

**Parameters:**
- `{namespace}`: The API namespace (e.g., 'rest', 'wp', 'wpcom')
- `{version}`: The API version (e.g., 'v1', 'v1.4', 'v2', 'v4')
- `{site_id}`: Your WordPress.com site's unique numeric identifier
- `{endpoint}`: The specific site-related endpoint (e.g., 'posts', 'pages', 'media', 'users')

**Examples:**
```
https://public-api.wordpress.com/wp/v2/sites/241031857/posts
https://public-api.wordpress.com/rest/v1.4/sites/241031857/stats
https://public-api.wordpress.com/wpcom/v2/sites/241031857/follows
```

#### Getting Your Site ID

To use site-specific endpoints, you'll need to obtain your site's unique numeric identifier. You can get this info by doing a request to `/rest/v1.1/me/sites` endpoint from the API Console:

1. Visit the [WordPress.com API Console](https://developer.wordpress.com/docs/api/console/)
2. Navigate to the `/rest/v1.1/me/sites` endpoint
3. Execute the request to retrieve all sites associated with your account
4. Locate the `ID` field in the response for your desired site

**Understanding the Response**

The `/rest/v1.1/me/sites` endpoint returns comprehensive details about all sites associated with your WordPress.com account, including:

- `ID`: The unique numeric site identifier (what you need for API calls)
- `name`: The site's display name
- `URL`: The site's public URL
- `jetpack`: Whether the site is a Jetpack-connected site
- `is_private`: Whether the site is private
- `capabilities`: What actions you can perform on the site

**Example Response:**
```json
{
  "sites": [
    {
      "ID": 241031857,
      "name": "My Blog",
      "URL": "https://myblog.wordpress.com",
      "jetpack": false,
      "is_private": false,
      "capabilities": {
        "edit_posts": true,
        "publish_posts": true
      }
    }
  ]
}
```

### Alternative URL Formats (Not Recommended)

While these formats may work in some cases, they are not recommended due to inconsistency across different site configurations:

```
https://yoursite.com/wp-json/wp/v2/posts
https://public-api.wordpress.com/wp/v2/sites/yoursite.com/posts
```

## Authentication Requirements

The WordPress.com REST API supports both authenticated and unauthenticated requests, depending on the endpoint and the data you're trying to access. Understanding when and how to authenticate is crucial for successful API integration.

### When Authentication Is Required

**Unauthenticated requests** work for:
- Public site information (e.g., site details, public posts)
- Reading public content from WordPress.com sites
- Accessing publicly available stats and data

**Authentication is required** for:
- Creating, editing, or deleting content (posts, pages, comments)
- Accessing private sites or private content
- Managing site settings and configuration
- Accessing user-specific data (notifications, followed sites, personal stats)
- Any operation that would require a user to be logged in when using WordPress.com directly

### Authentication Methods

The WordPress.com REST API supports two primary authentication methods:

#### Personal Access Token

**Best for:** Personal projects, scripts, testing, and applications accessing your own sites.

**How to get a Personal Access Token:**
1. Log in to your WordPress.com account
2. Go to [Account Settings → Security](https://wordpress.com/me/security)
3. Scroll down to "Application Passwords" section
4. Click "Create New Application Password"
5. Enter a descriptive name for your application
6. Copy the generated token (you won't be able to see it again)

**How to use in requests:**
Include the token in the `Authorization` header using the `Bearer` token format:

```bash
# Using curl
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     https://public-api.wordpress.com/rest/v1.4/me

# Using curl with POST request
curl -X POST \
     -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"title":"My New Post","content":"Post content here"}' \
     https://public-api.wordpress.com/wp/v2/sites/YOUR_SITE_ID/posts
```

```javascript
// Using JavaScript fetch
fetch('https://public-api.wordpress.com/rest/v1.4/me', {
  headers: {
    'Authorization': 'Bearer YOUR_ACCESS_TOKEN',
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(data => console.log(data));
```

```php
// Using PHP with cURL
$access_token = 'YOUR_ACCESS_TOKEN';
$curl = curl_init('https://public-api.wordpress.com/rest/v1.4/me');
curl_setopt($curl, CURLOPT_HTTPHEADER, array(
    'Authorization: Bearer ' . $access_token,
    'Content-Type: application/json'
));
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($curl);
$data = json_decode($response);
```

#### OAuth2 Authentication

**Best for:** Third-party applications that need to access other users' sites and data.

OAuth2 provides secure, user-controlled access delegation, allowing your application to act on behalf of users without storing their passwords. The user explicitly grants permission to your application through WordPress.com's authorization interface.

**OAuth2 Flow Summary:**
1. **Register your application** at [WordPress.com Apps](https://developer.wordpress.com/apps/)
2. **Redirect users** to WordPress.com's authorization URL with your client ID
3. **User grants permission** and is redirected back to your app with an authorization code
4. **Exchange the code** for an access token using your client secret
5. **Use the access token** in API requests

For detailed OAuth2 implementation guidance, including code examples and security considerations, refer to the [OAuth2 Authentication](5-oauth2-authentication.md) documentation.

### Request Examples

#### Unauthenticated Request Example

```bash
# Get public information about a site
curl https://public-api.wordpress.com/rest/v1.4/sites/en.blog.wordpress.com/

# Get public posts from a site
curl https://public-api.wordpress.com/wp/v2/sites/discover.wordpress.com/posts?per_page=5
```

```javascript
// JavaScript example for public data
fetch('https://public-api.wordpress.com/rest/v1.4/sites/en.blog.wordpress.com/')
  .then(response => response.json())
  .then(data => {
    console.log('Site info:', data);
  })
  .catch(error => {
    console.error('Error:', error);
  });
```

#### Authenticated Request Examples

```bash
# Get your user profile (requires authentication)
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     https://public-api.wordpress.com/rest/v1.4/me

# Create a new post (requires authentication)
curl -X POST \
     -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     -H "Content-Type: application/json" \
     -d '{"title":"My New Post","content":"This is the post content","status":"publish"}' \
     https://public-api.wordpress.com/wp/v2/sites/YOUR_SITE_ID/posts

# Get your site's stats (requires authentication)
curl -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
     https://public-api.wordpress.com/rest/v1.4/sites/YOUR_SITE_ID/stats
```

```javascript
// JavaScript example for authenticated requests
const accessToken = 'YOUR_ACCESS_TOKEN';
const siteId = 'YOUR_SITE_ID';

// Get user profile
fetch('https://public-api.wordpress.com/rest/v1.4/me', {
  headers: {
    'Authorization': `Bearer ${accessToken}`
  }
})
.then(response => response.json())
.then(data => console.log('User profile:', data));

// Create a new post
fetch(`https://public-api.wordpress.com/wp/v2/sites/${siteId}/posts`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${accessToken}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    title: 'My New Post',
    content: 'This is the post content',
    status: 'publish'
  })
})
.then(response => response.json())
.then(data => console.log('New post created:', data));
```

```php
// PHP example for authenticated requests
$access_token = 'YOUR_ACCESS_TOKEN';
$site_id = 'YOUR_SITE_ID';

// Get user profile
$curl = curl_init('https://public-api.wordpress.com/rest/v1.4/me');
curl_setopt($curl, CURLOPT_HTTPHEADER, array(
    'Authorization: Bearer ' . $access_token
));
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
$user_data = json_decode(curl_exec($curl));

// Create a new post
$post_data = array(
    'title' => 'My New Post',
    'content' => 'This is the post content',
    'status' => 'publish'
);

$curl = curl_init('https://public-api.wordpress.com/wp/v2/sites/' . $site_id . '/posts');
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, json_encode($post_data));
curl_setopt($curl, CURLOPT_HTTPHEADER, array(
    'Authorization: Bearer ' . $access_token,
    'Content-Type: application/json'
));
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
$post_response = json_decode(curl_exec($curl));
```

### Authentication Troubleshooting

#### Common Authentication Errors

**401 Unauthorized**
- **Cause:** Invalid or missing access token
- **Solution:** Verify your token is correct and included in the `Authorization` header

**403 Forbidden**
- **Cause:** Valid token but insufficient permissions for the requested action
- **Solution:** Check that your user account has the necessary capabilities for the site

**Invalid Token Format**
- **Cause:** Incorrect header format
- **Solution:** Ensure you're using `Authorization: Bearer YOUR_TOKEN` (note the space after "Bearer")

#### Security Best Practices

1. **Never expose tokens in client-side code** - Personal access tokens should only be used in server-side applications
2. **Use environment variables** - Store tokens in environment variables, not in your source code
3. **Rotate tokens regularly** - Generate new tokens periodically and revoke old ones
4. **Use OAuth2 for user-facing apps** - Don't use personal access tokens for applications that other users will authenticate with
5. **Limit token scope** - Only request the minimum permissions necessary for your application

### Browser-Based Applications

If you're building a browser-based application, you'll need to:

1. **Use OAuth2 implicit flow** - Personal access tokens should not be used in client-side code
2. **Whitelist your domains** - Configure allowed origins in your [WordPress.com app settings](https://developer.wordpress.com/apps/)
3. **Handle CORS properly** - The API will send appropriate CORS headers for whitelisted domains

For detailed guidance on browser-based implementations, see [Using the REST API from JS and the Browser](7-using-the-rest-api-from-js-and-the-browser-cors.md).




## [Next Steps](https://developer.wordpress.com/docs/api/getting-started/#2-next-steps)

The [full documentation](https://developer.wordpress.com/docs/api/) is a great way to explore all the features of the API. You might also want to check out [our example apps](https://developer.wordpress.com/docs/api/rest-api-example-apps/), or familiarize yourself with the [development console](https://developer.wordpress.com/docs/api/console/).

Not sure where to start? Here are a few interesting endpoints:

- [GET /sites/$site/posts](https://developer.wordpress.com/docs/api/1.1/get/sites/%24site/posts/): Get a list of posts on a site. For example, the latest posts featured on [discover.wordpress.com](https://discover.wordpress.com/):  
  `https://public-api.wordpress.com/rest/v1.1/sites/discover.wordpress.com/posts/?number=2&pretty=true`  
- [GET /read/tags/$tag/posts](https://developer.wordpress.com/docs/api/1.1/get/read/tags/cats/posts/): Get a list of posts with a given tag. For example, recent posts about cats:  
  `https://public-api.wordpress.com/rest/v1.1/read/tags/cats/posts?number=2&pretty=true`  
- [GET /sites/$site/stats/top-posts](https://developer.wordpress.com/docs/api/1.1/get/sites/%24site/stats/top-posts/): View the top posts on your site (requires authentication).  
- [GET /read/recommendations/mine](https://developer.wordpress.com/docs/api/1.1/get/read/recommendations/mine/): Get a list of blog recommendations based on blogs you’re already following (requires authentication).  
- [POST /sites/$site/posts/new](https://developer.wordpress.com/docs/api/1.1/post/sites/%24site/posts/new/): Create a new post (requires authentication).

Questions? Comments? Want to tell us about the cool stuff you’ve built with our API? [Get in touch](https://developer.wordpress.com/contact/).  