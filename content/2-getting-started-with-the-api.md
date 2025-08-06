# Getting Started with the API

## What's the WordPress.com REST API

WordPress.com REST API allows you to view, create or edit content on any WordPress.com site, as well as any self-hosted (WordPress.org) site connected via [Jetpack](https://jetpack.me/). This includes not only blog posts and pages but also comments, tags, categories, media, site stats, notifications, sharing settings, user profiles, and many other WordPress.com features.

Some requests (e.g. listing public posts) do not need to be authenticated, but any action that would require a user to be logged in (such as creating a post) requires an authentication token. 

To make authenticated requests, you’ll first need to [set up an account on WordPress.com](https://wordpress.com/start) if you don’t have one already.

> Looking for code examples? Check out the [WordPress.com REST API Examples repository](https://github.com/Automattic/wpcom-rest-api-examples) which contains sample projects demonstrating OAuth authentication and API usage in various programming languages and frameworks. The repository includes examples of both OAuth-based authentication for user-authorized operations and Application Password authentication for direct API endpoint access.

## How To Use It

There are two ways to explore the endpoints available for WordPress.com REST API:
-  The [REST API Reference docs page](developer.wordpress.com/docs/api/reference) lists available endpoints and details the input and output of each one, along with example code in curl and PHP. 
-  The [REST API development console](developer.wordpress.com/docs/api/console/) allows you to construct and try out real API requests.

Making unauthenticated requests is simple. Since there are no special headers required, you can even open this one in your browser to see what it will return: [https://public-api.wordpress.com/rest/v1.1/sites/en.blog.wordpress.com/posts/?number=2\&pretty=true](https://public-api.wordpress.com/rest/v1.1/sites/en.blog.wordpress.com/posts/?number=2&pretty=true)

Making authenticated requests requires a few more steps. You can authenticate your requests in two ways:

1. **Application Passwords** - Simple and quick for personal projects and testing
2. **OAuth2 authentication** - The most secure and granular way, required for third-party applications

Application Passwords use Basic authentication (`Authorization: Basic` with username:password), while OAuth2 provides Bearer tokens (`Authorization: Bearer YOUR_ACCESS_TOKEN`). Check the [Authentication methods](#) section for detailed instructions on both approaches.

We recommend OAuth2 authentication as the most secure and granular way to access the WordPress.com REST API. If you're already familiar with OAuth2, you can skip directly to the [technical documentation](https://developer.wordpress.com/docs/oauth2/).

OAuth2 lets your application act on behalf of a user without ever seeing their password. Here's how it works: When someone wants to use your app with their WordPress.com account, your app sends them to WordPress.com to log in. WordPress.com shows them exactly what your app wants to do (like read their posts or create new ones) and asks if that's okay. If they say yes, WordPress.com gives your app a special access token. This token is like a temporary key that lets your app do only the things the user agreed to.

You can think of it as a three-way conversation:

- **User:** “I’d like to make a post via this API client.”  
- _**Client (App):**_ “Okay. Hey, WordPress.com, I’d like to do something on behalf of this 
user. Can you ask them if it’s okay?”  
- **WordPress.com:** “Sure. Hey, user, is it okay if Client acts on your behalf?”  
- **User:** “Yes, that is okay. I trust this client to take actions for me in the 
future.”  
- **WordPress.com:** “Okay, Client, here is a token that will allow you to take 
actions for this user. Keep it secret. Keep it safe.”  

Once the Client (App) has obtained the token, it can make authenticated requests to WordPress.com. Here's how a typical interaction works:

- _**Client (App):**_ "Hello WordPress.com, I'd like to create a new post. Here's my access token proving I'm authorized to act on behalf of the user, along with the post title, content, and other details."
- **WordPress.com:** "I've validated your token and confirmed you have permission to create posts. The post has been successfully created and published. Here's the response with the new post ID, URL, and other metadata."

This OAuth2 token-based authentication workflow provides secure, granular access control - the Client can only perform actions that the user explicitly authorized during the OAuth flow. The token can be revoked at any time if needed, and WordPress.com validates the token's permissions on every request.



The beauty of this system is that users stay in control. They can see exactly what your app is asking for, and they can revoke access anytime. Your app never stores passwords, and if a token gets compromised, it only affects that one app's access—not the user's entire account.

You've probably seen this before when logging into websites using your Google or Facebook account. The process works the same way: you click "Log in with Facebook," get sent to Facebook to confirm, and then get redirected back to the original site.

From your app's perspective, the process involves a few steps:
- First, you register your application on WordPress.com to get a client ID. 
- Then you direct users to WordPress.com with a special link that includes your client ID and tells WordPress.com where to send the user back to. 
- When users authorize your app, WordPress.com redirects them back to your app with an authorization code. You exchange this code for an access token using your client secret, and then you can use that token to make API requests on behalf of the user.

Once you have an access token, making authenticated requests is straightforward. You include the token in the Authorization header of your requests like this: `Authorization: Bearer your_token_here`.

For complete implementation details, code examples, and security best practices, check out the [OAuth2 authentication guide](https://developer.wordpress.com/docs/oauth2/).

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

### Alternative URL Formats

You may encounter some alternative URL formats:

- **Direct site access**, like `https://yoursite.com/wp-json/wp/v2/posts` - This format works only for self-hosted sites with Jetpack. May fail due to security settings, firewall rules, or authentication issues.

- **Domain-based WordPress.com access**, like `https://public-api.wordpress.com/wp/v2/sites/yoursite.com/posts` - This format is unreliable for sites with custom domains, DNS configurations, or when domains change.


To avoid any issues the recommended approach is to use the format with numeric site IDs which is more reliable, faster, it works consistently across all site types, and supports full WordPress.com features and authentication methods.

## Authentication Requirements

The WordPress.com REST API supports both authenticated and unauthenticated requests, depending on the endpoint and the data you're trying to access. Understanding when and how to authenticate is crucial for successful API integration.

**Unauthenticated requests** work for:
- Public site information (e.g., site details, public posts)
- Reading public content from WordPress.com sites
- Accessing publicly available stats and data

_Examples of unauthenticated requests:_

```bash
# Get public information about a site
curl https://public-api.wordpress.com/rest/v1.4/sites/en.blog.wordpress.com/

# Get public posts from a site
curl https://public-api.wordpress.com/wp/v2/sites/discover.wordpress.com/posts?per_page=5
```

**Authentication is required** for:
- Creating, editing, or deleting content (posts, pages, comments)
- Accessing private sites or private content
- Managing site settings and configuration
- Accessing user-specific data (notifications, followed sites, personal stats)
- Any operation that would require a user to be logged in when using WordPress.com directly

_Examples of authenticated requests (require a token):_

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

### Authentication Methods

The WordPress.com REST API supports two primary authentication methods:
- Application Passwords
- OAuth2 Authentication

#### Application Passwords

Application Passwords provide a simple and direct way to authenticate API requests without implementing the full OAuth2 flow. They are ideal for developers working on personal projects, command-line tools, scripts, and applications that only need to access their own WordPress.com sites. Since these passwords have the same permissions as your WordPress.com account, they should be kept secure and only used in trusted applications that you control.

**How to get an Application Password:**
1. Log in to your WordPress.com account
2. Go to [Account Settings → Security](https://wordpress.com/me/security)
3. Scroll down to "Application Passwords" section
4. Click "Create New Application Password"
5. Enter a descriptive name for your application
6. Copy the generated password (you won't be able to see it again)

**How to use Application Passwords in requests:**
Use your WordPress.com username and the generated Application Password with HTTP Basic Authentication:

```bash
# Using curl with Basic Authentication
curl -X GET \
  'https://public-api.wordpress.com/rest/v1.1/sites/YOUR_SITE_ID/posts' \
  -u 'your-wordpress-username:your-application-password' \
  -H 'Content-Type: application/json'

# Using curl with POST request
curl -X POST \
  'https://public-api.wordpress.com/wp/v2/sites/YOUR_SITE_ID/posts' \
  -u 'your-wordpress-username:your-application-password' \
  -H 'Content-Type: application/json' \
  -d '{"title":"My New Post","content":"Post content here"}'

# Or using explicit Authorization header (base64 encoded)
curl -X GET \
  'https://public-api.wordpress.com/rest/v1.1/sites/YOUR_SITE_ID/posts' \
  -H 'Authorization: Basic base64(username:password)' \
  -H 'Content-Type: application/json'
```

_Example in JavaScript_
```javascript
// Using JavaScript fetch with Basic Authentication
const username = 'your-wordpress-username';
const password = 'your-application-password';
const credentials = btoa(`${username}:${password}`);

fetch('https://public-api.wordpress.com/rest/v1.4/me', {
  headers: {
    'Authorization': `Basic ${credentials}`,
    'Content-Type': 'application/json'
  }
})
.then(response => response.json())
.then(data => console.log(data));
```

_Example in PHP_
```php
// Using PHP with cURL and Basic Authentication
$username = 'your-wordpress-username';
$password = 'your-application-password';
$curl = curl_init('https://public-api.wordpress.com/rest/v1.4/me');
curl_setopt($curl, CURLOPT_HTTPHEADER, array(
    'Authorization: Basic ' . base64_encode($username . ':' . $password),
    'Content-Type: application/json'
));
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
$response = curl_exec($curl);
$data = json_decode($response);
```

<!--
**How to use in requests:**
Include the token in the `Authorization` header using the `Bearer` token format:

_Example with [CURL](https://curl.se/)_
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

_Example in Javasript_
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

_Example in PHP_
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

-->

#### OAuth2 Authentication

OAuth2 authentication is a more secure approach and the best option for third-party applications that need to access other users' sites and data.

OAuth2 provides secure, user-controlled access delegation, allowing your application to act on behalf of users without storing their passwords. The user explicitly grants permission to your application through WordPress.com's authorization interface.

**OAuth2 Flow Summary:**
1. **Register your application** at [WordPress.com Apps](https://developer.wordpress.com/apps/)
2. **Redirect users** to WordPress.com's authorization URL with your client ID
3. **User grants permission** and is redirected back to your app with an authorization code
4. **Exchange the authorization code** for an access token using your client secret
5. **Use the access token** in API requests

For detailed OAuth2 implementation guidance, including code examples and security considerations, refer to the [OAuth2 Authentication](5-oauth2-authentication.md) documentation.

### Authentication Troubleshooting and Best Practices

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

1. **Never expose credentials in client-side code** - Application passwords should only be used in server-side applications
2. **Use environment variables** - Store usernames and passwords in environment variables, not in your source code
3. **Rotate passwords regularly** - Generate new application passwords periodically and revoke old ones
4. **Use OAuth2 for user-facing apps** - Don't use application passwords for applications that other users will authenticate with
5. **Limit access scope** - Only use application passwords for applications that genuinely need full access to your account


<!-- TO-DO: Review this section later to complement info with OAuth2 Authentication -->
### Browser-Based Applications

If you're building a browser-based application, you'll need to:

1. **Use OAuth2 implicit flow** - Application passwords should not be used in client-side code
2. **Whitelist your domains** - Configure allowed origins in your [WordPress.com app settings](https://developer.wordpress.com/apps/)
3. **Handle CORS properly** - The API will send appropriate CORS headers for whitelisted domains

For detailed guidance on browser-based implementations, see [Using the REST API from JS and the Browser](7-using-the-rest-api-from-js-and-the-browser-cors.md).


