## Namespaces & Versions

WordPress.com REST API 


### `/rest/` namespace

All `/rest/` namespace versions are available under the **WP.COM API** option in the top selector of the API Console.

#### **v1**

  - URL: `https://public-api.wordpress.com/rest/v1/`
  - The original REST API for WordPress.com sites.
  - Covered a broad set of endpoints for sites, posts, users, stats, etc.

#### **v1.1 — v1.4**
  - URLs:  
    - `https://public-api.wordpress.com/rest/v1.1/`  
    - `https://public-api.wordpress.com/rest/v1.2/`  
    - `https://public-api.wordpress.com/rest/v1.3/`  
    - `https://public-api.wordpress.com/rest/v1.4/`
  - Incremental updates and improvements: each version added new endpoints or modified existing ones while preserving backward compatibility.
  - Sometimes, duplicate endpoints exist across versions (e.g., `v1/sites/${site}/users` and `v1.1/sites/${site}/users`). Usually, the newer version offers extra fields, improved performance, or bug fixes.

### `/wp/` namespace

The `/wp/v2` namespace is available under the **WP REST API** option in the top selector of the API Console.

#### **v2**

  - URL: `https://public-api.wordpress.com/wp/v2/`
  - Mirrors the official WordPress (self-hosted) core REST API.
  - It adds extra features specific to WordPress.com-hosted or Jetpack-connected sites.
  - Provides endpoints for posts, pages, taxonomies, users, etc.
  - Adheres to the standards defined by the WordPress core team, making it the main and most "official" namespace for WordPress REST endpoints.

### `/wpcom/` namespace

All `/wpcom/` namespace versions are available under the **WP REST API** option in the top selector of the API Console.

#### **v2, v3**

  - URLs:  
    - `https://public-api.wordpress.com/wpcom/v2/`  
    - `https://public-api.wordpress.com/wpcom/v3/`  
  - These expose WordPress.com-specific features and data not available in the open-source WordPress core.
  - Typically used for endpoints that only make sense in a WordPress.com context (e.g., Reader, Stats, Notifications).
  - Newer versions (v3, v4) add additional endpoints, improvements, or security changes.

---

## Frequently Asked Questions

### Why are there so many versions?

- **Backward Compatibility**: New versions are created when breaking changes or significant improvements are needed without disrupting existing integrations.
- **Incremental Features**: Each version may introduce new endpoints, fields, or behaviors.
- **Legacy Support**: Older apps or integrations may depend on specific behaviors that can’t be changed without a version bump.

### Why are there `/rest/*`, `/wp/*`, and `/wpcom/*` namespaces?

- **/rest/**: The original WordPress.com REST API, providing general access to WordPress.com data and management.
- **/wp/**: Implements the WordPress core REST API specification, ensuring parity with self-hosted WordPress sites and plugins.
- **/wpcom/**: Focused on uniquely WordPress.com features, not present in the open-source WordPress core.

### Which endpoints are grouped in each namespace?

- **/rest/**: General WordPress.com data — sites, posts, users, stats, etc.
- **/wp/**: Core WordPress data — posts, pages, comments, taxonomies, users, etc., following the structure of the WordPress core REST API.
- **/wpcom/**: WordPress.com-specific features — Reader, Stats, Notifications, and other services unique to WordPress.com.

### Which version is the most stable or recommended?

- **/wp/v2/** is the most standardized and stable for WordPress core functionality, aligning with the open-source WordPress REST API.
- **/wpcom/v4/** (or the highest available) is best for WordPress.com-specific features, offering the latest functionality and improvements.
- For **/rest/**, use the highest available version (v1.4) unless your integration depends on older behaviors.

### Why do some endpoints exist in multiple versions? Are there differences?

- When a new version is released, it often carries forward all existing endpoints but may add new fields, change response formats, or deprecate old behaviors.
- The endpoints may appear identical at first glance but can differ in supported parameters, returned fields, or performance.
- Check the documentation or test responses for subtle differences.
- Unless you have a reason to target an older version, you should prefer the latest available version to benefit from improvements and bug fixes.

---

## Resources for Changelog & Version Differences

- **Official Documentation**:  
  - [WordPress.com REST API Reference](https://developer.wordpress.com/docs/api/)
  - [API Console](https://developer.wordpress.com/docs/api/console/)
- **Changelogs**:  
  - There is no single, comprehensive changelog for all versions, but individual endpoint pages sometimes note changes.
  - [WordPress.com Developer Blog](https://developer.wordpress.com/) occasionally posts about major API changes.
- **Community & Support**:  
  - [WordPress.com Developer Forums](https://wordpress.com/forums/developers)
  - [GitHub Issues](https://github.com/Automattic/wp-calypso/issues?q=label%3AAPI)

---

## Suggested Doc Section

> ### API Namespaces & Versions
> WordPress.com offers several REST API namespaces and versions to balance new features with backward compatibility. Use the latest available version for each namespace to benefit from improvements and fixes.  
> - **/wp/v2/**: Main, official REST API, aligns with WordPress core.  
> - **/wpcom/v4/**: Latest WordPress.com-specific endpoints.  
> - **/rest/v1.4/**: Legacy endpoints; use only if needed for backward compatibility.  
>  
> **Why so many versions?** To allow for safe improvements and support legacy applications.
>  
> **Why different namespaces?** Each serves a different purpose: general REST, WordPress core, and WordPress.com-specific features.
>  
> **Version differences?** Most new versions add features or fix issues. Use the highest version unless you need legacy support.

---

If you’d like, I can help you draft a ready-to-use markdown section to add to your documentation! Just let me know your preferences or if you want more details on any part.

# What's the difference between `/rest/v1-v1.4` endpoints AND `/wpcom` endpoints?

Here’s a concise explanation of the differences between the /rest/v1–v1.4 endpoints and the /wpcom endpoints in the WordPress.com API:

---

## /rest/v1–v1.4 Endpoints

- **Legacy WordPress.com REST API.**
- **Purpose:** The original set of REST endpoints that provide access to core WordPress.com features, such as sites, posts, users, stats, and management actions.
- **Versions:** Versions 1 through 1.4 (v1, v1.1, v1.2, v1.3, v1.4) exist for backward compatibility, with newer versions adding features or fixing issues while retaining older behaviors.
- **Structure:** Custom to WordPress.com (not standardized across the wider WordPress ecosystem).
- **Scope:** Broad access to WordPress.com site data and management, but not always consistent with modern REST API conventions.
- **Examples:**  
  - `/rest/v1/sites/{site}/posts`  
  - `/rest/v1.3/sites/{site}/stats`  

---

## /wpcom (v2, v3, v4) Endpoints

- **Modern WordPress.com-specific REST API.**
- **Purpose:** Newer endpoints focused on features and services unique to WordPress.com that are not part of the WordPress core (self-hosted) API.
- **Versions:** v2, v3, and v4, with each new version introducing new features, improved security, or changes in the data structure.
- **Structure:** Designed to be more modular, modern, and maintainable, often reflecting improvements in API design and platform evolution.
- **Scope:** Targets advanced or platform-specific features—such as Reader, Stats, Notifications, and other WordPress.com services not available in self-hosted WordPress.
- **Examples:**  
  - `/wpcom/v2/read/tags`  
  - `/wpcom/v3/notifications`  
  - `/wpcom/v4/me/follows`  

---

## Summary Table

| Feature            | /rest/v1–v1.4                  | /wpcom/v2–v4                 |
|--------------------|--------------------------------|------------------------------|
| API Age            | Legacy (older)                 | Modern (newer)               |
| Focus              | Core WP.com site mgmt/data     | WP.com-specific platform features |
| Versioning         | v1, v1.1, v1.2, v1.3, v1.4     | v2, v3, v4                    |
| Standardization    | Custom to WP.com               | More modular and modern       |
| Self-hosted parity | No                             | No                            |
| Example Endpoints  | `/rest/v1/sites/{site}/posts`  | `/wpcom/v3/notifications`     |

---

**In short:**  
- Use **/rest/v1–v1.4** for legacy WordPress.com site and content management endpoints.
- Use **/wpcom/v2–v4** for modern, WordPress.com-only features and services, especially newer platform functionalities.

If you need a markdown-ready section or want further details on a specific endpoint set, just let me know!

---

# What's the difference between `/wp/v2/` endpoints and `/rest/v1-v1.4` endpoints?


The main differences between the /wp/v2/ and /rest/v1-v1.4 endpoints in the WordPress.com API are:

### /wp/v2/ Endpoints

- **Purpose:** Implements the official WordPress REST API (WP REST API), designed for both WordPress.com and self-hosted WordPress (core).
- **Standardization:** Follows the standard defined by the WordPress core team, ensuring maximum compatibility with plugins, themes, and third-party tools that use the WP REST API.
- **Structure:** Unified and consistent endpoint structure for posts, pages, users, comments, taxonomies, media, etc.
- **Extensibility:** Easily extended by plugins and custom code on self-hosted WordPress sites.
- **Best for:** New projects, integrations that need to work with both WordPress.com and self-hosted WordPress, and when you want to follow the official WP API structure.

---

### /rest/v1–v1.4 Endpoints

- **Purpose:** The original, legacy WordPress.com REST API, predating the WP REST API.
- **WordPress.com-Specific:** Designed specifically for WordPress.com, including features and data unique to the platform, such as Reader, Stats, and special management endpoints.
- **Structure:** Custom structure, not standardized with core WordPress. Some endpoints and responses differ from the WP REST API, and there may be additional fields or behaviors tailored to WordPress.com.
- **Versions:** Multiple versions (v1, v1.1, v1.2, v1.3, v1.4) were introduced to add features and maintain backward compatibility. Newer versions have more features and improvements.
- **Best for:** Accessing WordPress.com-specific features not available in core WordPress, or when maintaining legacy integrations that already use these endpoints.

---

### Summary Table

| Namespace   | Purpose                | Standardization | WordPress.com-Specific | Best Use                  |
|-------------|------------------------|-----------------|------------------------|---------------------------|
| /wp/v2/     | Official WP REST API   | Yes             | No                     | New/core-compatible apps  |
| /rest/v1*   | Legacy WP.com API      | No              | Yes                    | WP.com-specific features, legacy support |

---

**In short:**
- Use **/wp/v2/** for standard WordPress data and compatibility.
- Use **/rest/v1-v1.4** for WordPress.com-specific features or when legacy support is required.

---

# WordPress.com REST API: Namespaces & Versions

The WordPress.com REST API provides a powerful way to programmatically interact with WordPress.com sites and services. Over time, the API has evolved to support new features, maintain compatibility, and expose both standard WordPress and WordPress.com-specific functionality. This guide introduces the available namespaces and versions, explains their differences, and helps you choose the right endpoints for your integration.

---

## Overview of API Namespaces

The API is organized into several namespaces, each with their own versions and focus areas:

| Namespace      | Typical Versions                | Focus                                                         |
|----------------|--------------------------------|---------------------------------------------------------------|
| `/rest/`       | v1, v1.1, v1.2, v1.3, v1.4     | Legacy WordPress.com management and data (original API)        |
| `/wp/`         | v2                             | Official WordPress core resources (compatible with self-hosted)|
| `/wpcom/`      | v2, v3, v4                     | Modern WordPress.com-specific features and services            |

---

## Namespace Details

### 1. `/rest/` (v1–v1.4) — Legacy WordPress.com REST API

- **Purpose:** Original API for managing WordPress.com sites, users, posts, stats, and more.
- **Versions:** v1 (original), v1.1–v1.4 (incremental improvements and new features).
- **Focus:** Covers a broad range of site and user management, including WordPress.com-specific features such as Stats, Reader, Domains, Jetpack, and Notifications.
- **Best for:** Accessing legacy endpoints or features not available elsewhere, or maintaining compatibility with earlier integrations.
- **Examples:**  
  - `/rest/v1/sites/{site}/posts`  
  - `/rest/v1/sites/{site}/stats`  
  - `/rest/v1/sites/{site}/options`

> **Note:** Many endpoints in this namespace are not available in `/wp/v2/` or `/wpcom/v2/`, especially those related to stats, Reader, and advanced site management.

---

### 2. `/wp/` (v2) — Official WordPress Core REST API

- **Purpose:** Implements the standardized REST API introduced in WordPress core, designed for both WordPress.com and self-hosted WordPress.
- **Version:** v2 (the official and only version for this namespace).
- **Focus:** Standard WordPress resources such as posts, pages, comments, taxonomies, users, and media.
- **Best for:** New integrations, or when you want maximum compatibility with self-hosted WordPress plugins and tools.
- **Examples:**  
  - `/wp/v2/posts`  
  - `/wp/v2/pages`  
  - `/wp/v2/users`

> **Tip:** Use `/wp/v2/` if you need to build tools or apps that work across WordPress.com and self-hosted sites.

---

### 3. `/wpcom/` (v2–v4) — Modern WordPress.com Platform API

- **Purpose:** Provides access to new and evolving WordPress.com-specific features, often reflecting improvements in API design and platform evolution.
- **Versions:** v2, v3, v4 (newer versions add features, improvements, or security changes).
- **Focus:** Features unique to WordPress.com such as Reader, Notifications, enhanced Stats, and follow management.
- **Best for:** Integrations needing the latest WordPress.com-specific services not available in WordPress core.
- **Examples:**  
  - `/wpcom/v3/notifications`  
  - `/wpcom/v2/read/tags`  
  - `/wpcom/v4/me/follows`

> **Note:** Not all legacy features from `/rest/v1–v1.4` are available in `/wpcom/` endpoints.

---

## Why So Many Versions and Namespaces?

- **Backward Compatibility:** New versions allow improvements without breaking existing integrations.
- **Standardization:** The `/wp/` namespace aligns with the official WordPress REST API specification.
- **Platform Evolution:** WordPress.com-specific features are released in `/rest/` (legacy) or `/wpcom/` (modern) namespaces, as needed.

---

## How to Choose the Right Namespace & Version

| Use Case                                 | Recommended Namespace & Version     |
|-------------------------------------------|-------------------------------------|
| Standard WordPress data (posts, pages, etc.) | `/wp/v2/`                          |
| WordPress.com-specific features (Reader, Stats, Notifications) | `/wpcom/v4/` (or latest available) |
| Legacy features or advanced site management | `/rest/v1.4/` (or required version)|
| Broadest compatibility (self-hosted + .com) | `/wp/v2/`                          |

- **For new projects:** Prefer `/wp/v2/` for core resources, `/wpcom/v4/` for modern platform features.
- **For legacy apps:** Continue using `/rest/v1–v1.4` as needed, but migrate to newer namespaces when possible.
- **For features found only in `/rest/`:** Use `/rest/v1–v1.4` if no equivalent exists in `/wp/` or `/wpcom/`.

---

## Frequently Asked Questions

### Why are some endpoints duplicated across versions?

Some endpoints exist in multiple versions (e.g., `/rest/v1/sites/{site}/users` and `/rest/v1.1/sites/{site}/users`). Newer versions typically add features, improve performance, or fix issues while maintaining backward compatibility. Always prefer the latest version unless you need older behavior.

### Are endpoints in `/wp/v2/` and `/wpcom/` the same?

No. `/wp/v2/` focuses on standard WordPress core resources, while `/wpcom/` is for features unique to WordPress.com.

### Are there features only available in `/rest/v1–v1.4/`?

Yes. Especially for advanced stats, Reader, site management, and some notifications, `/rest/v1–v1.4/` has endpoints not present in the newer namespaces.

---

## Resources

- [API Console](https://developer.wordpress.com/docs/api/console/)
- [API Reference](https://developer.wordpress.com/docs/api/)
- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/)
- [Developer Blog](https://developer.wordpress.com/)

---

## Summary Table

| Namespace      | Version(s)         | Use for...                                   | Self-hosted Compatible? |
|----------------|--------------------|----------------------------------------------|------------------------|
| `/rest/`       | v1–v1.4            | Legacy WP.com management, stats, Reader      | No                     |
| `/wp/`         | v2                 | Core posts, pages, media, users, etc.        | Yes                    |
| `/wpcom/`      | v2–v4              | Modern WP.com-specific features              | No                     |

---

Still unsure which namespace or version to use? [Check the API Console](https://developer.wordpress.com/docs/api/console/) or search this documentation for your specific use case.
