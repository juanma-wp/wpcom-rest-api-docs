# WordPress.com Connect

WordPress.com Connect is a streamlined authentication solution designed specifically for **"Login with WordPress.com"** functionality. It provides a secure and user-friendly way for millions of WordPress.com users to authenticate with your application using their existing WordPress.com credentials.

> **Note**: WordPress.com Connect is a specialized implementation of OAuth2 focused on user authentication and identity verification. For full API access to WordPress.com sites and content management, see the complete [OAuth2 Authentication](5-oauth2-authentication.md) documentation.

## What is WordPress.com Connect?

WordPress.com Connect allows WordPress.com users to quickly log in to your service without creating new accounts. When users connect, your application receives their basic profile information (name, email, avatar) while they maintain control over their WordPress.com data and privacy.

**Key characteristics of WordPress.com Connect**:
- **Identity-focused**: Designed for user authentication, not content management
- **Limited scope**: Access restricted to basic profile information via the `/me/` endpoint
- **Simplified flow**: Optimized for "Login with WordPress.com" buttons
- **User-friendly**: Familiar interface for millions of WordPress.com users

![Image showing the Connect with WordPress.com button.](https://s0.wp.com/i/wpcc-button.png)
  

### Benefits

**Millions Of Users** – WordPress.com consists of millions of users and is growing every day. By adding WordPress.com Connect you will become part of a large family that makes it easy for WordPress.com users to explore new services.

**Compatible with your existing sign-in system** – WordPress.com Connect can be used on its own or as a complimentary sign-in option to your existing registration system. Once a user connects, you will get access to their profile information which you can use in your own app.

**Trusted Relationship** – Allow users to sign-in with the same credentials they use every day on WordPress.com. This takes the pain out of having to remember and manage a new log-in for another service.

## OAuth2 Implementation Details

WordPress.com Connect uses the **OAuth2 Authentication Endpoint** (`/oauth2/authenticate`) rather than the standard authorization endpoint. This specialized endpoint is optimized for identity verification and automatically limits token scope to basic profile access.

**Technical Flow**:
1. **User Authorization**: Redirect to `/oauth2/authenticate` (not `/oauth2/authorize`)
2. **Code Exchange**: Exchange authorization code at `/oauth2/token` (same as full OAuth2)
3. **Limited Access**: Resulting token provides access only to `/me/` endpoint
4. **Profile Data**: Retrieve user identity from `/rest/v1.1/me`

> For detailed technical information about the `/oauth2/authenticate` endpoint, see the [Authentication Endpoint section](5-oauth2-authentication.md#authentication-endpoint) in the OAuth2 documentation.

## Prerequisites

Before implementing WordPress.com Connect, you need to register your application:

1. **Create a WordPress.com Application** at [developer.wordpress.com/apps](https://developer.wordpress.com/apps/)
2. **Configure your application**: Use the same title as your website (shown in login forms)
3. **Obtain credentials**: You'll receive a `CLIENT_ID` and `CLIENT_SECRET`
4. **Set redirect URI**: Configure where users return after authentication

Additional resources:
- [Code examples in multiple languages](https://github.com/Automattic/wpcom-connect-examples) (PHP, Python, Node.js)
- Complete technical documentation in [OAuth2 Authentication guide](5-oauth2-authentication.md)

## Implementation Example (PHP)

Here's a complete example demonstrating how to implement WordPress.com Connect for user authentication and profile retrieval.

### Configuration Setup

First, configure your application credentials. Replace these values with those from your [WordPress.com Application](https://developer.wordpress.com/apps/):

```php
<?php
// config.php - WordPress.com Connect Configuration
define('CLIENT_ID', 'your_client_id');
define('CLIENT_SECRET', 'your_client_secret');
define('REDIRECT_URI', 'https://yourapp.com/auth-callback');

// WordPress.com OAuth2 endpoints (no changes needed)
define('AUTHENTICATE_URL', 'https://public-api.wordpress.com/oauth2/authenticate');
define('TOKEN_URL', 'https://public-api.wordpress.com/oauth2/token');
define('USER_INFO_URL', 'https://public-api.wordpress.com/rest/v1.1/me');

session_start(); // Required for state parameter security
?>
```

> **Complete example**: See the [wpcom-connect-examples repository](https://github.com/Automattic/wpcom-connect-examples/blob/master/php/config.php) for additional language implementations.

### Step 1: Create Authorization URL

Generate the "Connect with WordPress.com" button that redirects users to WordPress.com for authentication. This uses the **authentication endpoint** (not the standard authorization endpoint).

**Security Note**: The `state` parameter prevents CSRF attacks and must be validated when users return.

```php
<?php
require_once 'config.php';

// Generate secure state parameter for CSRF protection
if (!isset($_SESSION['wpcc_state'])) {
    $_SESSION['wpcc_state'] = bin2hex(random_bytes(16)); // More secure than md5(mt_rand())
}

// Build authentication URL using /oauth2/authenticate endpoint
$auth_url = AUTHENTICATE_URL . '?' . http_build_query([
    'response_type' => 'code',
    'client_id'     => CLIENT_ID,
    'redirect_uri'  => REDIRECT_URI,
    'state'         => $_SESSION['wpcc_state'],
    'scope'         => 'auth' // Limited scope for profile access only
]);

// Display the Connect button
echo '<a href="' . htmlspecialchars($auth_url) . '">';
echo '<img src="https://s0.wp.com/i/wpcc-button.png" width="231" alt="Connect with WordPress.com" />';
echo '</a>';
?>
```

This generates the familiar WordPress.com Connect button:

![Image showing the Connect with WordPress.com button.](https://s0.wp.com/i/wpcc-button.png)

### Step 2: Handle Authorization Response

When users click the Connect button, they see a WordPress.com authorization screen:

![oauth-approve](https://wpdeveloperstaging.files.wordpress.com/2024/02/e124d-oauth-approve.png)

After approval, WordPress.com redirects users back to your `redirect_uri` with an authorization code. Your callback handler must validate the state parameter and exchange the code for an access token:

```php
<?php
// auth-callback.php - Handle the authorization response
require_once 'config.php';

// Validate authorization response
if (!isset($_GET['code'])) {
    die('Error: No authorization code received. User may have declined access.');
}

if (!isset($_GET['state']) || $_GET['state'] !== $_SESSION['wpcc_state']) {
    die('Error: State mismatch. Possible CSRF attack detected.');
}

// Exchange authorization code for access token
$curl = curl_init(TOKEN_URL);
curl_setopt_array($curl, [
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => [
        'client_id'     => CLIENT_ID,
        'client_secret' => CLIENT_SECRET,
        'code'          => $_GET['code'],
        'grant_type'    => 'authorization_code',
        'redirect_uri'  => REDIRECT_URI
    ],
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_SSL_VERIFYPEER => true
]);

$response = curl_exec($curl);
$http_code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
curl_close($curl);

if ($http_code !== 200) {
    die('Error: Failed to obtain access token.');
}

$token_data = json_decode($response, true);
$access_token = $token_data['access_token'];

// Clean up session state
unset($_SESSION['wpcc_state']);
?>
```

**Successful token response**:
```json
{
    "access_token": "your_access_token_here",
    "token_type": "bearer",
    "blog_id": 0,
    "blog_url": "https://public-api.wordpress.com",
    "scope": "auth"
}
```

Note the `scope: "auth"` - this confirms the token has limited access for identity verification only.

### Step 3: Retrieve User Profile

With the access token, retrieve the user's profile information from the [`/me/` endpoint](https://developer.wordpress.com/docs/api/1/get/me/):

```php
<?php
// Fetch user profile using the access token
function get_user_profile($access_token) {
    $curl = curl_init(USER_INFO_URL);
    curl_setopt_array($curl, [
        CURLOPT_HTTPHEADER => [
            'Authorization: Bearer ' . $access_token
        ],
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_SSL_VERIFYPEER => true // Always verify SSL in production
    ]);
    
    $response = curl_exec($curl);
    $http_code = curl_getinfo($curl, CURLINFO_HTTP_CODE);
    curl_close($curl);
    
    if ($http_code !== 200) {
        throw new Exception('Failed to fetch user profile');
    }
    
    return json_decode($response, true);
}

// Get the user's WordPress.com profile
try {
    $user_profile = get_user_profile($access_token);
    
    // Store or process user information
    $user_id = $user_profile['ID'];
    $display_name = $user_profile['display_name'];
    $email = $user_profile['email'];
    $avatar_url = $user_profile['avatar_URL'];
    $is_verified = $user_profile['verified'];
    
} catch (Exception $e) {
    die('Error: ' . $e->getMessage());
}
?>
```

**Profile response format**:
```json
{
  "ID": 12345,
  "display_name": "Bob Smith",
  "username": "bobsmith",
  "email": "bob@example.com",
  "primary_blog": 67890,
  "avatar_URL": "https://gravatar.com/avatar/abc123?s=96",
  "profile_URL": "https://en.gravatar.com/bobsmith",
  "verified": true
}
```

### Step 4: Complete User Authentication

Once you have the user profile, integrate them into your application:

```php
<?php
// Complete user authentication flow
if ($is_verified) {
    // User has verified their email - safe to trust profile data
    $existing_user = find_user_by_wpcom_id($user_id);
    
    if ($existing_user) {
        // Log in existing user
        login_user($existing_user);
        redirect_to_dashboard();
    } else {
        // Create new account with WordPress.com profile data
        $new_user = create_user([
            'wpcom_id' => $user_id,
            'username' => $user_profile['username'],
            'email' => $email,
            'display_name' => $display_name,
            'avatar_url' => $avatar_url
        ]);
        login_user($new_user);
        redirect_to_welcome();
    }
} else {
    // Unverified email - handle with caution
    redirect_to_verification_required();
}
?>
```

**Important**: Always check the `verified` flag before trusting profile information. Unverified accounts may contain unreliable data.

## WordPress.com Connect vs Full OAuth2

Understanding when to use each approach:

| Feature | WordPress.com Connect | [Full OAuth2](5-oauth2-authentication.md) |
|---------|----------------------|---------------------------|
| **Purpose** | User authentication & identity | Full API access & content management |
| **Endpoint** | `/oauth2/authenticate` | `/oauth2/authorize` |
| **Token Scope** | `auth` (limited to `/me/`) | Custom scopes (`posts`, `media`, etc.) |
| **Use Cases** | "Login with WordPress.com" | WordPress.com site management |
| **Data Access** | Basic profile only | Blog posts, media, comments, etc. |

> **Important**: Do not use the same WordPress.com application for both Connect authentication and full API access. Connect tokens are limited to the `/me/` endpoint and cannot access blog content or management features.