# WordPress.com Connect


"WordPress.com Connect" is a secure and easy way for millions of WordPress.com users to log in to your website or app. By integrating with "WordPress.com Connect", WordPress.com users can quickly log in and start using your service. Their profile information is shared with your app saving you the hassle of collecting name, email address, etc. You even get their Gravatar profile picture.

![Image showing the Connect with WordPress.com button.](https://s0.wp.com/i/wpcc-button.png)
  

## Benefits

**Millions Of Users** – WordPress.com consists of millions of users and is growing every day. By adding WordPress.com Connect you will become part of a large family that makes it easy for WordPress.com users to explore new services.

**Compatible with your existing sign-in system** – WordPress.com Connect can be used on its own or as a complimentary sign-in option to your existing registration system. Once a user connects, you will get access to their profile information which you can use in your own app.

**Trusted Relationship** – Allow users to sign-in with the same credentials they use every day on WordPress.com. This takes the pain out of having to remember and manage a new log-in for another service.

## Get Started

The first thing you need to do is create a new [WordPress.com Application](https://developer.wordpress.com/apps/), this will give you a chance to describe your application and how we should communicate with it. You should give your app the same title as your website as that information is used in the login form users see. Once configured you will receive your `CLIENT ID` and `CLIENT SECRET` to identify your app.

Integrate with your app using the below instructions or use the [code examples in this Github Project](https://github.com/Automattic/wpcom-connect-examples "WP.com Connect Code Examples") for implementing in other languages (PHP, Python, Node.js, maybe others in the future).

## PHP example integration

Here is a simple example of how you can connect a WordPress.com user to your app or website and then get back their profile information.

When you created your WordPress.com Application above you will have received a number of items back. Let's set those as constants to make them easy to reference. (Please replace the values of the `CLIENT_ID`, `CLIENT_SECRET` and `REDIRECT_URL` with those values from your WordPress.com Application)

You could [add these values on your `config.js` file](https://github.com/Automattic/wpcom-connect-examples/blob/master/php/config.php) 
```php
define('CLIENT_ID', 1);
define('CLIENT_SECRET', 'your-client-secret');
define('LOGIN_URL', 'http://localhost:8000');
define('REDIRECT_URL', 'http://localhost:8000/connected.php');

// You do not need to change these settings
define('REQUEST_TOKEN_URL', 'https://public-api.wordpress.com/oauth2/token');
define('AUTHENTICATE_URL', 'https://public-api.wordpress.com/oauth2/authenticate');

session_start(); // As we'll include this file in the rest of the files we can inititiate session here (we'll need the session to store the state value)
```

### Create a Connect button

In the following code snippets we will just use the constant `CLIENT_ID` etc. to denote the values that are unique to your WordPress.com Application. First up we need to generate the button that your users will click in order to connect to WordPress.com. An anti-forgery "state" token is required to protect the security of your users. Create the variable with a good random number generator and save it on your server. The token is sent back to your server after the user authenticates and you must verify that the tokens match. The button's link should navigate a visitor to the authentication URL and include query parameters describing your application.

```php
require_once dirname(__FILE__) . '/config.php';

if (!isset($_SESSION['wpcc_state'])) {
	$_SESSION['wpcc_state'] = md5(mt_rand());
}

$url_to = AUTHENTICATE_URL . '?' . http_build_query(array(
	'response_type' => 'code',
	'client_id'     => CLIENT_ID,
	'state'         => $_SESSION['wpcc_state'],
	'redirect_uri'  => REDIRECT_URL
));

echo '<a href="' . $url_to . '"><img src="//s0.wp.com/i/wpcc-button.png" width="231" /></a>';
```

This will output a button like so:

![Image showing the Connect with WordPress.com button.](http://s0.wp.com/i/wpcc-button.png)

### Request an access token

When the user clicks on the Connect button they will be brought to a form that asks them to log in and explains what information will be revealed to your app.

![oauth-approve](https://wpdeveloperstaging.files.wordpress.com/2024/02/e124d-oauth-approve.png)

Once they complete this they will be sent to the "redirect URL" you have set in your WordPress.com Application (which is the value of the `REDIRECT_URL` constant above). When this URL is loaded please check that the state variable returned matches the one stored locally and also look for the `code` parameter. Using that `code` variable your app then sends a request to WordPress.com looking for a permanent access\_token which you can use from then on to reference this user. You can store the access\_token in your app for future use.

```php
require_once dirname( __FILE__ ) . '/config.php';

if ( ! isset( $_GET[ 'code' ] ) ) {
	die( 'Warning! Visitor may have declined access or navigated to the page without being redirected.' );
}

if ( $_GET[ 'state' ] !== $_SESSION[ 'wpcc_state' ] ) {
	die( 'Warning! State mismatch. Authentication attempt may have been compromised.' );
}

$curl = curl_init( REQUEST_TOKEN_URL );
curl_setopt( $curl, CURLOPT_POST, true );
curl_setopt( $curl, CURLOPT_POSTFIELDS, array(
	'client_id'     =&amp;amp;amp;gt; CLIENT_ID,
	'redirect_uri'  =&amp;amp;amp;gt; REDIRECT_URL,
	'client_secret' =&amp;amp;amp;gt; CLIENT_SECRET,
	'code'          =&amp;amp;amp;gt; $_GET[ 'code' ],
	'grant_type'    =&amp;amp;amp;gt; 'authorization_code'
) );

curl_setopt( $curl, CURLOPT_RETURNTRANSFER, 1 );
$auth = curl_exec( $curl );
$secret = json_decode( $auth );
$token = $secret->access_token;
```

`$secret` is an object that contains the `access_token` that you will use to query the users profile information.

```
stdClass Object 
(
    [access_token] => XXXXXXXXXXXXXXXXXXXXXXXXXXXX
    [token_type] => bearer
    [blog_id] => 0
    [blog_url] => http://public-api.wordpress.com
    [scope] => auth
)
```

### Request user profile information

After you get an access\_token you must authenticate the user on your local site. You use the `access_token` to request their profile information through the [/me/](https://developer.wordpress.com/docs/api/1/get/me/) endpoint on WordPress.com.

```php
// Get user info from WordPress.com API
$apiUrl = 'https://public-api.wordpress.com/rest/v1.1/me';
$headers = [
    'Authorization: Bearer ' . $token
];

$ch = curl_init();
curl_setopt($ch, CURLOPT_URL, $apiUrl);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false); // For testing only
$response = curl_exec($ch);
curl_close($ch);

$user = json_decode($response, true);

```

A typical request for profile information would return JSON encoded data with the following profile information in an object:

```json
{
  "ID": 1,
  "display_name": "Bob",
  "username": "bobsmith",
  "email": "bob@example.com",
  "primary_blog": 1,
  "avatar_URL": "http://gravatar.com/avatar/xxxx?s=96",
  "profile_URL": "http://en.gravatar.com/xxxx",
  "verified": true
}
```

### Authenticate the user

The `verified` flag in the profile is true when the user has verified their email address on WordPress.com. You should take this into consideration before using the profile. If it is false be wary of trusting the information and using it in an automated way.  
With this information you should query your user database and log the user in if a record exists. Otherwise you may auto-generate a new account or redirect the user to a signup form. You should pre-populate as many fields as possible from the profile information above.

### Limits of login apps

If you already have [WordPress.com apps](https://developer.wordpress.com/apps/) do not use one app for both login and interacting with a WordPress.com blog. Access tokens generated through the authenticate endpoint used in login requests only have access to the /me/ endpoint. Your users will not be able to access their blogs if they attempt to login through the same app.