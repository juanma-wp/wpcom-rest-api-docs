# REST API URLs by Site Type

When interacting with the WordPress.com REST API, it's essential to understand that the base URL for your REST API endpoint will mainly vary based on whether your site uses a WordPress.com Subdomain or a custom domain. For sites with a custom domain, there are also some additional options for REST API endpoints to choose from.

This guide will walk you through the correct REST API URL structure for each type of site.

## Public Sites on a WordPress.com Subdomain

Most sites on the WordPress.com free plan use a `wordpress.com` subdomain, for example, `example.wordpress.com`. For these public sites, the REST API endpoint is structured similarly to a self-hosted WordPress site.

You can access the API by appending `/wp-json/` to the end of your site's URL.

The general URL format is: `https://<your-site-name>.wordpress.com/wp-json/`

**Example**

For a site at `https://example.wordpress.com`, the REST API root can be found at: `https://example.wordpress.com/wp-json/`

To retrieve a list of posts from this site, you would make a `GET` request to: `https://example.wordpress.com/wp-json/wp/v2/posts`

## Public Sites with a Custom Domain

WordPress.com sites using a custom domain, such as [`www.example.com`](http://www.example.com) use a different REST API endpoint.

For these sites, you must use the WordPress.com public API proxy, which centralizes API access. You'll need to include your site's domain in the API request URL.

The general URL format is: `https://public-api.wordpress.com/wp/v2/sites/<your-custom-domain.com>/`

**Example**

If your site's custom domain is `www.example.com`, its REST API root is: `https://public-api.wordpress.com/wp/v2/sites/www.example.com/`

A request to fetch posts from this site would be sent to: `https://public-api.wordpress.com/wp/v2/sites/www.example.com/posts`

It is crucial to use your custom domain in the API URL, not any `.wordpress.com` subdomain that might still be linked to your account.

For sites with advanced hosting features active, there’s also the option of `https://<your-site-name>/wp-json/wp/v2/posts` . 

## Comparison Table: REST API URLs by Site Type

| Site Type                  | Example Site URL                        | REST API Base URL                                                      |
|----------------------------|-----------------------------------------|------------------------------------------------------------------------|
| WordPress.com Subdomain    | `https://example.wordpress.com`         | `https://example.wordpress.com/wp-json/`                               |
| Custom Domain             | `https://www.example.com`               | `https://public-api.wordpress.com/wp/v2/sites/www.example.com/` or `https://www.example.com/wp-json/` |