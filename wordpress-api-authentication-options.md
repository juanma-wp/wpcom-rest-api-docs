# WordPress.com API Authentication Options

When building applications that interact with WordPress.com sites, choosing the right authentication method is crucial for both security and functionality. WordPress.com offers two primary authentication approaches: **Application Passwords** and **OAuth 2.0**. Understanding when and how to use each method will help you build more secure and appropriate integrations.

## OAuth 2.0: Granular, User-Controlled Access

OAuth 2.0 represents a sophisticated and secure approach to API authentication, particularly designed for **third-party applications** that need to access user data across multiple accounts or services. Unlike [Application Passwords](#), which grant full administrative access, OAuth 2.0 provides a granular permissions system through scopes that allows applications to request only the specific access levels they need.

### A granular permission system through scopes

The key advantage of OAuth 2.0 lies in its **granular permission system**, implemented through scopes. When a user authorizes your application through OAuth, they authenticate using their **existing WordPress.com credentials** and they can see exactly what permissions your app is requesting and choose to grant or deny specific access levels. For instance, your application might request only read access to posts, or perhaps write access to media uploads, without needing full administrative privileges.

> Check [OAuth scopes](#) for a more detailed explanation of available scopes

### Implementing OAuth 2.0 Authentication

OAuth 2.0 requires a multi-step process to authenticate users and obtain access tokens. WordPress.com supports two main OAuth 2.0 workflows, each designed for different types of applications and security requirements:

- **Authorization Code Flow** – This is the regular OAuth workflow used by server-side (confidential) clients. It involves exchanging an authorization code for an access token via a secure server-to-server request, using your application's client secret to authenticate the exchange. The client secret must be kept secure and never exposed to the client side.

- **Implicit Flow** – This is the OAuth workflow designed for browser-based (public) clients, where the access token is returned directly in the URL fragment without an intermediate code exchange. It's now considered less secure and largely deprecated in favor of other alternatives like [PKCE](https://oauth.net/2/pkce/).

To implement these workflows, you first need to [create an application](https://developer.wordpress.com/apps/) to configure your app’s OAuth settings and to get your **Client ID** and **Client Secret** (needed for the OAuth workflows).

> Refer to the [OAuth workflows](#) section for a more detailed explanation on how to implement these workflows.

<!--

#### Step 1: Register Your Application

1. Go to [WordPress.com Developer Console](https://developer.wordpress.com/apps/)
2. Click **Create New Application**
3. Fill in your application details:
   - **Name**: Your application name
   - **Description**: What your app does
   - **Website URL**: Your application's homepage
   - **Redirect URLs**: Where users return after authorization
   - **JavaScript Origins**: Allowed domains for browser-based requests
4. Note your **Client ID** and **Client Secret**

#### Step 2: Authorization Flow

**Redirect users to WordPress.com for authorization:**

```javascript
// JavaScript example - redirect to authorization URL
const clientId = 'your-client-id';
const redirectUri = 'https://yourapp.com/callback';
const scopes = 'global,posts';
const state = 'random-string-for-security';

const authUrl = `https://public-api.wordpress.com/oauth2/authorize?` +
  `client_id=${clientId}&` +
  `redirect_uri=${encodeURIComponent(redirectUri)}&` +
  `response_type=code&` +
  `scope=${scopes}&` +
  `state=${state}`;

// Redirect user to authUrl
window.location.href = authUrl;
```

#### Step 3: Exchange Authorization Code for Access Token

When users authorize your app, they're redirected back with an authorization code:

```javascript
// JavaScript/Node.js example - exchange code for token
const exchangeCodeForToken = async (authCode) => {
  const response = await fetch('https://public-api.wordpress.com/oauth2/token', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
    },
    body: new URLSearchParams({
      client_id: 'your-client-id',
      client_secret: 'your-client-secret',
      code: authCode,
      grant_type: 'authorization_code',
      redirect_uri: 'https://yourapp.com/callback'
    })
  });

  const tokenData = await response.json();
  return tokenData.access_token;
};
```

```php
// PHP example - exchange code for token
function exchange_code_for_token( $auth_code ) {
    $response = wp_remote_post( 'https://public-api.wordpress.com/oauth2/token', array(
        'body' => array(
            'client_id'     => 'your-client-id',
            'client_secret' => 'your-client-secret',
            'code'          => $auth_code,
            'grant_type'    => 'authorization_code',
            'redirect_uri'  => 'https://yourapp.com/callback',
        ),
    ) );

    $body = wp_remote_retrieve_body( $response );
    $data = json_decode( $body, true );
    
    return $data['access_token'];
}
```

#### Step 4: Make Authenticated API Requests

Use the access token in your API requests:

```javascript
// JavaScript example - make authenticated requests
const makeAuthenticatedRequest = async (accessToken, endpoint) => {
  const response = await fetch(`https://public-api.wordpress.com/rest/v1.1/${endpoint}`, {
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${accessToken}`,
      'Content-Type': 'application/json'
    }
  });

  return response.json();
};

// Example usage
const posts = await makeAuthenticatedRequest(accessToken, 'sites/yoursite.wordpress.com/posts');
```

```php
// PHP example - make authenticated requests
function make_authenticated_request( $access_token, $endpoint ) {
    $response = wp_remote_get( "https://public-api.wordpress.com/rest/v1.1/{$endpoint}", array(
        'headers' => array(
            'Authorization' => "Bearer {$access_token}",
            'Content-Type'  => 'application/json',
        ),
    ) );

    return json_decode( wp_remote_retrieve_body( $response ), true );
}

// Example usage
$posts = make_authenticated_request( $access_token, 'sites/yoursite.wordpress.com/posts' );
```

```python
# Python example - make authenticated requests
import requests

def make_authenticated_request(access_token, endpoint):
    response = requests.get(
        f'https://public-api.wordpress.com/rest/v1.1/{endpoint}',
        headers={
            'Authorization': f'Bearer {access_token}',
            'Content-Type': 'application/json'
        }
    )
    return response.json()

# Example usage
posts = make_authenticated_request(access_token, 'sites/yoursite.wordpress.com/posts')
```
-->

#### OAuth Scopes

When requesting authorization, specify the minimum scopes your application needs:

- **`users`**: View user information
- **`sites`**: View general site information and options
- **`posts`**: View and manage posts
- **`comments`**: View and manage a post's comments
- **`taxonomy`**: View and manage a site's tags and categories
- **`follow`**: Follow and unfollow blogs
- **`sharing`**: Connect social media services
- **`freshly-pressed`**: View Freshly Pressed posts
- **`notifications`**: View and manage a user's notifications
- **`insights`**: View analytics for your application
- **`read`**: Manage and view a user's Reader subscriptions
- **`stats`**: View stats for a site
- **`media`**: Manage a site's media
- **`menus`**: View and manage a site's menus
- **`batch`**: Batch several GET requests
- **`videos`**: View video information


### Special Scopes

Some scopes serve specific authentication purposes:

- **`global`**: Grants comprehensive access to user data across WordPress.com services
- **`auth`**: Limited scope that only provides access to the `/me/` endpoint for basic authentication flows

Example with specific scopes:
```javascript
const scopes = 'global,posts,media'; // Request only necessary permissions
```

## Application Passwords: Full admin access

Application Passwords provide a straightforward authentication mechanism that grants **comprehensive access to WordPress REST API endpoints** on a specific site. Think of Application Passwords as creating a special username and password combination specifically designed for programmatic access, rather than human login.

### How Application Passwords Work

When you generate an Application Password, you're essentially creating credentials that **mirror admin-level access** to your WordPress site. This means your application can perform virtually any action that an administrator could perform through the WordPress interface, including creating posts, managing users, installing plugins, and accessing data from any installed plugins that expose REST API endpoints.

### Best Use Cases for Application Passwords

This broad access makes Application Passwords particularly valuable for **personal development scenarios**:

- **Custom backup scripts** for your own website
- **Personal dashboards** that aggregate data from multiple plugins on your site
- **Internal tools** that need comprehensive site access
- **Trusted developer scripts** running on sites you control

### Authenticating Requests with Application Passwords

> The [Application Passwords](https://wordpress.com/support/security/two-step-authentication/application-specific-passwords/) provides detailed instructions on how to obtain an Application Password for your account and for a specific site.


Use your WordPress.com username and the generated Application Password with Basic Auth:

```bash
# cURL example
curl -X GET \
  'https://public-api.wordpress.com/rest/v1.1/sites/yoursite.wordpress.com/posts' \
  -H 'Authorization: Basic eW91ci13b3JkcHJlc3MtdXNlcm5hbWU6YWJjZCBlZmdoIGlqa2wgbW5vcA==' \
  -H 'Content-Type: application/json'

# Or using cURL's built-in basic auth (more readable)
curl -X GET \
  'https://public-api.wordpress.com/rest/v1.1/sites/yoursite.wordpress.com/posts' \
  -u 'your-wordpress-username:abcd efgh ijkl mnop' \
  -H 'Content-Type: application/json'
```
### Security Considerations

However, this all-or-nothing access model presents important security implications. Since Application Passwords grant such broad permissions, they should only be distributed to highly trusted parties. **If these credentials were compromised, an attacker would have nearly complete control over your WordPress site.**

## Making the Right Choice

Your choice between Application Passwords and OAuth 2.0 should be guided by your application's specific needs and use case.

### Choose Application Passwords When:

- Building **personal scripts or tools** for your own WordPress sites
- Creating **internal applications** for trusted team members
- Developing **comprehensive site management tools** that need full access
- Working on **personal projects** where OAuth complexity isn't justified

### Choose OAuth 2.0 When:

- Building **third-party applications** for public use
- Creating **SaaS platforms** that integrate with WordPress.com
- Developing **mobile apps** or browser extensions
- Building services that need **specific, limited permissions**
- Working with **multiple user accounts** or sites

## Testing and Development Strategies

During development, OAuth applications require special consideration for testing workflows. You can use your own WordPress.com credentials for testing OAuth flows, which allows you to verify that your application correctly handles the authorization process and makes appropriate API calls.

### Handling Two-Factor Authentication

If you have two-factor authentication enabled on your WordPress.com account (which is highly recommended), you'll encounter a practical challenge during testing. OAuth flows typically redirect users through web browsers for authorization, but programmatic testing often requires direct credential validation. 

**Solution**: You can use Application Passwords for testing purposes, even if your production application will use OAuth 2.0. For example, you might develop automated tests that validate your application's API integration using Application Passwords, while implementing OAuth 2.0 for your production user interface.

**Important**: This testing approach should **never extend to production environments**. Application Passwords used for testing should be temporary, regularly rotated, and never embedded in production code.

## Security Implementation Guidelines

Understanding the security implications of each authentication method helps you make informed decisions and implement appropriate safeguards.

### Application Password Security

Application Passwords in WordPress.com function as **user-level security features**, integrating with the platform's two-factor authentication system. This differs from vanilla WordPress installations, where Application Passwords operate as blog-level features.

**Implementation best practices**:
- Store credentials securely using **environment variables** or secure credential management systems
- **Never commit** Application Passwords to version control systems
- Implement **credential rotation policies** for long-running applications
- Provide clear documentation about credential management

### OAuth 2.0 Security

OAuth 2.0 implementations should follow established security best practices:

- **Proper state parameter validation** to prevent CSRF attacks
- **Secure token storage** and appropriate token refresh handling
- **Minimum scope requests** - only ask for permissions you actually need
- **Clear communication** to users about what permissions are being requested and why


## Implementation Best Practices

Regardless of which authentication method you choose, always follow the **principle of least privilege**. Request only the access levels your application genuinely needs, and regularly audit your permission requirements as your application evolves.

### For OAuth Applications

- Implement proper **error handling** for authorization failures, token expiration, and scope changes
- Ensure users understand **when and why** authorization is required
- Handle scenarios where users **deny specific permissions** gracefully
- Provide clear **documentation** about required permissions

### For Application Password Applications

- Include **secure storage mechanisms** in your implementation
- Provide clear **documentation** about the credentials' purpose and scope
- If distributing tools, include guidance about **credential management** and security best practices
- Implement **comprehensive logging** for API access patterns

## Conclusion

The authentication method you choose sets the foundation for your WordPress.com integration's security and user experience. **Application Passwords excel for personal and internal tools** where comprehensive access is needed and trusted. **OAuth 2.0 is essential for third-party applications** where user trust and granular permissions matter most.

By understanding these strengths and appropriate use cases, you can build applications that are both secure and user-friendly, whether you're developing personal tools or building applications for the broader WordPress.com community.











Message agentdave









