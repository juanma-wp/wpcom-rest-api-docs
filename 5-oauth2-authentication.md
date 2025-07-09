# OAuth2 Authentication

OAuth2 is a protocol that allows applications to interact with blogs on WordPress.com and self-hosted WordPress sites running Jetpack. The primary goal of OAuth is to allow developers to interact with WordPress.com and Jetpack sites without requiring them to store sensitive credentials. Our implementation also allows users to manage their own connections.

If you are new to the world of OAuth, you can read more at [https://oauth.net/](https://oauth.net/). If you are already familiar with OAuth, then all you really need to know about are the two authentication endpoints:

*   the _authorization endpoint_ which is `https://public-api.wordpress.com/oauth2/authorize`
*   the _token request endpoint_ which is `https://public-api.wordpress.com/oauth2/token`

The same endpoints are used for WordPress.com blogs and Jetpack sites. Before you begin to develop an application, you will need a client id, redirect URI, and a client secret key. These details will be used to authenticate your application and verify that the API calls being made are valid. You can create an application or view details for your existing applications with our [applications manager](https://developer.wordpress.com/apps/).

## [Receiving an Access Token](#0-receiving-an-access-token)

To act on a user’s behalf and make calls from our API you will need an access token. To get an access token you need to go through the access token flow and prompt the user to authorize your application to act on their behalf.

Access tokens can be requested per blog per user or as a global token per user. In addition to the global tokens, there are certain endpoints (e.g. likes and follows) where you can use a user’s token on any blog to act on their behalf.

To begin, you will need to send the user to the authorization endpoint.

### [Required parameters:](#1-required-parameters-)

*   `client_id` should be set to your application’s client ID as found in the applications manager.
*   `redirect_uri` should be set to the URL that the user will be redirected back to after the request is authorized. The `redirect_uri` must match the one in the applications manager.
*   `response_type` can be “code” or “token”. “Code” should be used for server side applications where you can guarantee that secrets will be stored securely. These tokens do not expire. “Token” should be used for client-side applications. This is called “Implicit OAuth”.  Tokens currently last two weeks and users will need to authenticate with your app once the token expires. Tokens are returned via the hash/fragment of the URL.

### [Optional parameters:](#2-optional-parameters-)

*   `blog`  You may pass along a blog parameter (`&blog=`) with the URL or blog ID for a WordPress.com blog or Jetpack site. If you do not pass along a blog, or if the user does not have administrative access to manage the blog you passed along, then the user will be prompted to select the blog they are granting you access to.
*   `scope`: see below.

#### **Token Scope**

The scope of the token defines the type of access your application will have to the user’s data. By default (if the scope is omitted), the token will grant the application **full access** to a **single blog**.

For example, if your application needs access to just the posts and comments endpoints, set the scope to `&scope=posts%20comments`

Additionally, the following special values are supported, and should not be used in combination of any other value:

*   `auth`: The auth scope (`&scope=auth`) will grant your application access to the `/me` endpoints only, and is primarily used for [WordPress.com Connect](https://developer.wordpress.com/docs/wpcc/).
*   `global`: The global scope (`&scope=global`)  will grant your application full access to **all** the blogs that the user has on WordPress.com, including any Jetpack blogs they have connected to their WordPress.com account. If you are specifying the scope as global then you should omit the `blog` parameter.

### [Server / Code Authentication](#3-server-code-authentication)

The redirect to your application will include a code which you will need in the next step. If the user has denied access to your app, the redirect will include ?error=access\_denied. Once the user has authorized the request, they will be redirected to the redirect\_url. The request will look like the following: `https://developer.wordpress.com/?code=cw9hk1xG9k`

This is a time-limited code that your application can exchange for a full authorization token. To do this you will need to pass the code to the token endpoint by making a **POST** request to `https://public-api.wordpress.com/oauth2/token`.

```php
$curl = curl_init( 'https://public-api.wordpress.com/oauth2/token' );
curl_setopt( $curl, CURLOPT_POST, true );
curl_setopt( $curl, CURLOPT_POSTFIELDS, array(
	'client_id' => your_client_id,
	'redirect_uri' => your_redirect_url,
	'client_secret' => your_client_secret_key,
	'code' => $_GET['code'], // The code from the previous request
	'grant_type' => 'authorization_code'
) );
curl_setopt( $curl, CURLOPT_RETURNTRANSFER, 1);
$auth = curl_exec( $curl );
$secret = json_decode($auth);
$access_key = $secret->access_token;
```

$curl = curl\_init( 'https://public-api.wordpress.com/oauth2/token' ); curl\_setopt( $curl, CURLOPT\_POST, true ); curl\_setopt( $curl, CURLOPT\_POSTFIELDS, array( 'client\_id' => your\_client\_id, 'redirect\_uri' => your\_redirect\_url, 'client\_secret' => your\_client\_secret\_key, 'code' => $\_GET\['code'\], // The code from the previous request 'grant\_type' => 'authorization\_code' ) ); curl\_setopt( $curl, CURLOPT\_RETURNTRANSFER, 1); $auth = curl\_exec( $curl ); $secret = json\_decode($auth); $access\_key = $secret->access\_token;CopyCopied

You are required to pass client\_id, client\_secret, and redirect\_uri for web applications. These parameters have to match the details for your application, and the redirect\_uri **must** match the redirect\_uri used during the Authorize step (above). grant\_type has to be set to “authorization\_code”. code must match the code you received in the redirect. If everything works correctly and the user grants authorization, you will get back a JSON-encoded string containing the token and some basic information about the blog:

```plain
{
    "access_token": "YOUR_API_TOKEN",
    "blog_id": "blog ID",
    "blog_url": "blog url",
    "token_type": "bearer"
}
```

{ "access\_token": "YOUR\_API\_TOKEN", "blog\_id": "blog ID", "blog\_url": "blog url", "token\_type": "bearer" }CopyCopied

You now have an access token which should be stored securely with the blog ID and blog URL. This access token allows your application to act on the behalf of the user on this specific blog. For an alternative example, check out our [Node implementation](https://github.com/Automattic/node-wpcom-oauth).

## [Testing an application as the client owner](#4-testing-an-application-as-the-client-owner)

As the client owner, you can authenticate with the `password` `grant_type`, allowing you to skip the authorization step of authenticating, instead logging in with your WordPress.com username and password. Note that if you are using [2-step authentication](https://wordpress.com/support/security/two-step-authentication/) (highly recommended), you will need to create an application password to be able to use the `password` `grant_type`.

```php
$curl = curl_init( 'https://public-api.wordpress.com/oauth2/token' );
curl_setopt( $curl, CURLOPT_POST, true );
curl_setopt( $curl, CURLOPT_POSTFIELDS, array(
    'client_id' => your_client_id,
    'client_secret' => your_client_secret_key,
    'grant_type' => 'password',
    'username' => your_wpcom_username,
    'password' => your_wpcom_password,
) );
curl_setopt( $curl, CURLOPT_RETURNTRANSFER, 1);
$auth = curl_exec( $curl );
$auth = json_decode($auth);
$access_key = $auth->access_token;
```

$curl = curl\_init( 'https://public-api.wordpress.com/oauth2/token' ); curl\_setopt( $curl, CURLOPT\_POST, true ); curl\_setopt( $curl, CURLOPT\_POSTFIELDS, array( 'client\_id' => your\_client\_id, 'client\_secret' => your\_client\_secret\_key, 'grant\_type' => 'password', 'username' => your\_wpcom\_username, 'password' => your\_wpcom\_password, ) ); curl\_setopt( $curl, CURLOPT\_RETURNTRANSFER, 1); $auth = curl\_exec( $curl ); $auth = json\_decode($auth); $access\_key = $auth->access\_token;CopyCopied

As noted above, this is only available to you as the owner of the application, and not to any other user. This is meant for **testing purposes only**.

## [Client/Implicit Oauth](#5-client-implicit-oauth)

Once the user authenticates their blog, they will be redirected back to your application. The token and user information will be included in the URL fragment.

[https://developer.wordpress.com/#access\_token=YOUR\_API\_TOKEN&expires\_in=64800&token\_type=bearer&site\_id=blog\_id](https://developer.wordpress.com/#access_token=YOUR_API_TOKEN&expires_in=64800&token_type=bearer&site_id=blog_id)

This token will allow you to make authenticated client-side calls using CORS/AJAX requests. The token currently only lasts two weeks. Use the expires\_in fragment to detect when you should prompt for a refresh.

## [Validating Tokens](#6-validating-tokens)

It can be helpful to be able to validate the authenticity of a token, specifically that it belongs to your application and the user you are authenticating. This is especially necessary when sending a token over the wire (e.g. mobile application sending token as login credentials to an API). To verify a token, use the \`/oauth/token-info\` endpoint, passing in the \`token\` and your \`client\_id\`:

[https://public-api.wordpress.com/oauth2/token-info?client\_id=your\_client\_id&token=your\_token](https://public-api.wordpress.com/oauth2/token-info?client_id=your_client_id&token=your_token)

If the token provided was not authorized for your application, the endpoint will return an error. If the token is valid, you will get a JSON-encoded string with the user ID and scope of the token:

```plain
{
    "client_id": "your client ID",
    "user_id": "user ID",
    "blog_id": "blog ID",
    "scope": "scope of the token"
}
```

{ "client\_id": "your client ID", "user\_id": "user ID", "blog\_id": "blog ID", "scope": "scope of the token" }CopyCopied

## [Making an API call](#7-making-an-api-call)

Our API is JSON-based. You can view all of the available endpoints at our [API documentation](https://developer.wordpress.com/docs/api/). You can also make API calls with our legacy XML-RPC API. In order to make an authenticated call to our APIs, you need to include your access token with the call. OAuth2 uses a BEARER token that is provided in an Authorization header.

```php
$access_key = 'YOUR_API_TOKEN';
$curl = curl_init( 'https://public-api.wordpress.com/rest/v1/me/' );
curl_setopt( $curl, CURLOPT_HTTPHEADER, array( 'Authorization: Bearer ' . $access_key ) );
curl_exec( $curl );
```

$access\_key = 'YOUR\_API\_TOKEN'; $curl = curl\_init( 'https://public-api.wordpress.com/rest/v1/me/' ); curl\_setopt( $curl, CURLOPT\_HTTPHEADER, array( 'Authorization: Bearer ' . $access\_key ) ); curl\_exec( $curl );CopyCopied

The above example would return information about the authenticated user.

```js
jQuery.ajax( {
    url: 'https://public-api.wordpress.com/rest/v1/sites/' + site_id + '/posts/new',
    type: 'POST',
    data: { content: 'testing test' },
    beforeSend : function( xhr ) {
        xhr.setRequestHeader( 'Authorization', 'BEARER ' + access_token );
    },
    success: function( response ) {
        // response
    }
} );
```

jQuery.ajax( { url: 'https://public-api.wordpress.com/rest/v1/sites/' + site\_id + '/posts/new', type: 'POST', data: { content: 'testing test' }, beforeSend : function( xhr ) { xhr.setRequestHeader( 'Authorization', 'BEARER ' + access\_token ); }, success: function( response ) { // response } } );CopyCopied

The above example would create a new post. You can make similar calls to the [other available endpoints](https://developer.wordpress.com/docs/api/).
