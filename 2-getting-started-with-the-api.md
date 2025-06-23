# Getting Started with the API

## What is it?

Our REST API allows you to view, create or edit content on any WordPress.com site, as well as any self-hosted (WordPress.org) site connected via [Jetpack](https://jetpack.me/). This includes not only blog posts and pages but also comments, tags, categories, media, site stats, notifications, sharing settings, user profiles, and many other WordPress.com features.

Some requests (e.g. listing public posts) do not need to be authenticated, but any action that would require a user to be logged in (such as creating a post) requires an authentication token. 

To make authenticated requests, you’ll need to [set up an account on WordPress.com](https://wordpress.com/start) if you don’t have one already. You’ll also need to create a [WordPress.com client application](https://developer.wordpress.com/apps/).

## How To Use It

There are two ways to explore the features of the API: via the documentation or the development console. The documentation ([developer.wordpress.com/docs/api/](https://developer.wordpress.com/docs/api/)) lists all available endpoints and details the input and output of each one, along with example code in curl and PHP. The development console ([developer.wordpress.com/docs/api/console/](https://developer.wordpress.com/docs/api/console/)),  allows you to construct and try out real API requests.

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